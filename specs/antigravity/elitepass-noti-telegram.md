# elitepass-noti-telegram: Microservicio de Notificaciones Telegram

Microservicio asíncrono que permite a usuarios finales (cajeros, administradores, clientes) vincular su número de celular con su cuenta de Telegram para recibir alertas, comprobantes, solicitudes de traspasos y notificaciones en tiempo real.

---

## 1. Core Técnico y Arquitectura

Consta de dos componentes que operan simultáneamente:

1. **Bot de Telegram (Polling):** Escucha interacciones directas de usuarios en Telegram para vincular su celular.
2. **API REST (Express):** Endpoint interno para que otras apps del ecosistema soliciten el envío de mensajes.

- **Tecnología:** Node.js + Express + TypeScript — ejecutado con `tsx` (sin build)
- **PM2:** modo **Cluster** — reinicio automático
- **ID PM2:** 2
- **Base de datos:** `jet_club_db` schema `telegram` — conexión **directa al puerto 5432** (NO PgBouncer) porque gestiona su propio pool mínimo y usa `?schema=telegram`

### Diagrama de Flujo Completo

```mermaid
graph TD
    subgraph Bot de Suscripción
        UserTG[Usuario Telegram] -->|Envía /start| Bot[Bot Telegram]
        Bot -->|Solicita compartir contacto| UserTG
        UserTG -->|Comparte número de celular| Bot
        Bot -->|Normaliza número a 8 dígitos| Bot
        Bot -->|INSERT telegram_user| DB[(jet_club_db.telegram)]
        Bot -->|Mensaje de confirmación| UserTG
    end

    subgraph API REST de Envío
        App[elitepass-reservas / POS] -->|POST /api/v1/notifications/send + x-api-key| Express[Express :3200]
        Express -->|Normaliza teléfono| Express
        Express -->|SELECT chatId WHERE phone_number = ?| DB
        Express -->|sendMessage HTML| BotAPI[API Telegram Bot]
        BotAPI -->|Entrega directa| Chat[Chat del Usuario]
    end
```

---

## 2. Capa de Datos y Persistencia

Opera en el schema `telegram` dentro de `jet_club_db`. Usa conexión directa a PostgreSQL en puerto `5432` (no PgBouncer) para evitar problemas con el modo transacción y el schema personalizado.

### Tabla de Suscripción (ERD)

```mermaid
erDiagram
    TELEGRAM_USER {
        string id PK
        string phoneNumber
        string chatId
        string firstName
        string username
        datetime createdAt
        datetime updatedAt
    }
```

### Normalización de Números Telefónicos Bolivianos

El microservicio limpia los números de teléfono antes de insertar o buscar, convirtiendo múltiples formatos al estándar nacional de 8 dígitos:

| Formato de entrada | Resultado |
|---|---|
| `+59170123456` | `70123456` |
| `59170123456` | `70123456` |
| `70123456` | `70123456` (sin cambio) |
| `591-70123456` | `70123456` |

La búsqueda siempre normaliza ambos lados (número buscado y número almacenado).

---

## 3. API y Mecanismos de Seguridad

### Autenticación

El endpoint `POST /api/v1/notifications/send` requiere el header `x-api-key` con el valor de `INTERNAL_API_KEY`. Requests sin key → HTTP 401.

### Payload de Envío

```json
{
  "phoneNumber": "70123456",
  "message": "🚨 <b>Alerta:</b> El stock de <i>Fernet Branca</i> bajó del límite mínimo."
}
```

### Formato HTML Soportado

Telegram acepta un subconjunto de HTML:
- `<b>` negrita, `<i>` cursiva, `<code>` monoespaciado, `<a href="">` enlace
- NO soporta: `<div>`, `<span>`, clases CSS, estilos inline

### Flujo de Integración desde Reservas (BullMQ)

```mermaid
sequenceDiagram
    participant Reservas
    participant BullMQ as Redis BullMQ :6379/0
    participant Noti as noti-telegram :3200

    Reservas->>BullMQ: job.add('send-notification', {userId, phone, message})
    BullMQ->>Noti: POST /api/v1/notifications/send (x-api-key)
    Noti->>Noti: Normaliza phone → 8 dígitos
    Noti->>Noti: SELECT chatId FROM telegram.telegram_user
    alt chatId encontrado
        Noti->>Telegram: sendMessage(chatId, HTML)
        Telegram-->>Noti: {ok: true}
        Noti-->>BullMQ: HTTP 200 — job completed
    else chatId no encontrado
        Noti-->>BullMQ: HTTP 404 — job failed (retry automático)
    end
```

---

## 4. Despliegue e Infraestructura

- **Puerto:** `3200` — interno únicamente, **NO expuesto por Nginx a internet**
- **Proceso PM2:** `elitepass-noti-telegram` — modo **Cluster** — reinicio automático
- **ID PM2:** 2
- **Directorio:** `/home/soporte/elitepass-noti-telegram/`

### Deploy

```bash
cd /home/soporte/elitepass-noti-telegram
# No requiere build — tsx corre directamente
pm2 restart elitepass-noti-telegram
```

### Variables de Entorno

```env
TELEGRAM_BOT_TOKEN="token_provisto_por_botfather"
DATABASE_URL="postgresql://user:password@127.0.0.1:5432/jet_club_db?schema=telegram"
INTERNAL_API_KEY="api_key_secreta_de_uso_interno_ecosistema"
PORT=3200
```

> **Nota:** `DATABASE_URL` apunta directamente al puerto `5432` de PostgreSQL, **no** al `6432` de PgBouncer. Esto es intencional — el schema `telegram` y el pool mínimo de este servicio no son compatibles con el modo transacción de PgBouncer.
