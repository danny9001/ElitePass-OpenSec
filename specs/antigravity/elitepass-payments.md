# elitepass-payments: Microservicio de Pagos y Facturación SIAT

Microservicio descentralizado que unifica el procesamiento de pagos QR (APIs BCP y BNB) y la facturación electrónica boliviana (SIAT) para los tenants del ecosistema **Antigravity**. Opera en modo ELITE (credenciales compartidas del operador) o CUSTOM (credenciales propias del comercio).

---

## 1. Core Técnico y Arquitectura

- **Tecnología:** Node.js + Express 5 + TypeScript
- **ORM:** Prisma 6 + PostgreSQL 16
- **Validaciones:** Zod — esquemas estrictos en todos los endpoints
- **Cifrado:** AES-256-GCM para credenciales bancarias almacenadas
- **Compresión HTTP:** middleware `compression` nivel 6 en todas las respuestas
- **PM2:** modo **Fork** — heap limit **256 MB**

### Modos de Operación

| Modo | Descripción | Credenciales |
|---|---|---|
| **ELITE** | Todas las organizaciones usan las credenciales del operador principal | Configuradas en `.env` del servicio |
| **CUSTOM** | La organización tiene sus propias credenciales bancarias | Almacenadas cifradas AES-256-GCM en DB, campo `encryptedCredentials` |

El modo se resuelve por organización en tiempo de request, consultando `Merchant.provider` y la presencia de `encryptedCredentials`.

### Diagrama de Integración Completo

```mermaid
sequenceDiagram
    autonumber
    participant App as reservas / pos
    participant Gateway as elitepass-payments :3100
    participant Banco as API BCP / BNB
    participant Cliente as App Banco

    App->>Gateway: POST /api/v1/payments/qr {monto, referencia, organizationId} + X-App-Key
    Gateway->>Gateway: Valida X-App-Key (APP_KEY_RESERVAS o APP_KEY_POS)
    Gateway->>Gateway: Resuelve modo ELITE o CUSTOM
    Gateway->>Gateway: Descifra credenciales AES-256-GCM con ENCRYPTION_KEY
    Gateway->>Banco: POST /qr/generate {monto, referencia, comercio}
    Banco-->>Gateway: {qrBase64, transaccionId}
    Gateway->>Gateway: INSERT Payment {status: PENDING, externalRef: transaccionId}
    Gateway-->>App: {qr: base64, transaccionId, paymentId}

    Note over Cliente: Escanea QR y paga
    Banco->>Gateway: Webhook POST /api/v1/webhooks/bcp|bnb (firma bancaria)
    Gateway->>Gateway: Verifica firma del webhook
    Gateway->>Gateway: UPDATE Payment {status: PAID, confirmedAt: NOW()}

    opt Event.requiresInvoice === true
        Gateway->>Gateway: Obtiene CUFD vigente del SIN
        Gateway->>Gateway: Genera factura SIAT (CUF, XML, PDF)
        Gateway->>Gateway: INSERT Invoice {cufd, cuf, pdfUrl}
    end

    Gateway->>App: POST callback_url {paymentId, status: PAID} + X-HMAC-Signature
    App->>App: Verifica HMAC-SHA256 con CALLBACK_HMAC_SECRET
    App->>App: Confirma reserva / venta
```

---

## 2. Capa de Datos y Persistencia

Opera sobre `elitepass_payments` vía PgBouncer `:6432` con `?pgbouncer=true&connection_limit=5`.

### Esquema de Entidades (ERD)

```mermaid
erDiagram
    MERCHANT ||--o{ PAYMENT : processes
    PAYMENT ||--o| INVOICE : may_generate

    MERCHANT {
        string id PK
        string organizationId
        string organizationSlug
        string provider
        string encryptedCredentials
        boolean isActive
        datetime createdAt
    }
    PAYMENT {
        string id PK
        string merchantId FK
        string externalRef
        float amount
        string currency
        string status
        string callbackUrl
        string appKey
        datetime createdAt
        datetime confirmedAt
    }
    INVOICE {
        string id PK
        string paymentId FK
        string nit
        string razonSocial
        string cufd
        string cuf
        string cafc
        string pdfUrl
        datetime issuedAt
    }
```

### Estados de Pago

```
PENDING → PAID      (webhook bancario exitoso)
PENDING → FAILED    (webhook bancario con error o timeout)
PENDING → EXPIRED   (sin confirmación en 30 min)
```

---

## 3. Mecanismos de Seguridad e Hardening

### Autenticación de Apps Cliente

El header `X-App-Key` es obligatorio en todos los endpoints de creación de pagos:
- `APP_KEY_RESERVAS` — para peticiones de elitepass-reservas
- `APP_KEY_POS` — para peticiones de elitepass-pos

Requests sin key válida → HTTP 401 inmediato. Las keys se rotan manualmente y se actualizan en `.env` de los tres servicios simultáneamente.

### Cifrado de Credenciales Bancarias (AES-256-GCM)

```
ENCRYPTION_KEY (64 hex chars = 32 bytes)
    ↓
AES-256-GCM encrypt(credenciales_json)
    ↓
{iv: hex, tag: hex, data: hex}  ← almacenado en Merchant.encryptedCredentials
```

La clave maestra `ENCRYPTION_KEY` nunca toca la base de datos. Solo existe en la variable de entorno del proceso.

### Verificación de Webhooks Bancarios

Cada banco firma sus webhooks de forma diferente:
- **BCP:** HMAC-SHA256 del body con clave compartida en el contrato comercial
- **BNB:** Firma verificada vía certificado o token específico del contrato

El gateway rechaza cualquier webhook cuya firma no coincida → HTTP 400.

### Callbacks Firmados hacia Apps

El gateway firma su callback hacia las apps con `CALLBACK_HMAC_SECRET`:
```
X-HMAC-Signature: HMAC-SHA256(body_json, CALLBACK_HMAC_SECRET)
```
La app destino debe verificar esta firma antes de confirmar la transacción.

### Facturación SIAT (Bolivia)

- Integración con el Sistema de Impuestos Nacionales (SIN) de Bolivia
- CUFD (Código Único de Factura Diaria): renovado diariamente al primer pago
- CUF (Código Único de Facturación): generado por transacción
- El PDF de factura se sube a Azure Blob Storage y la URL se retorna al caller
- **Pendiente:** configurar credenciales SIN del tenant (NIT, clave SIAT) en tabla Merchant

---

## 4. Despliegue e Infraestructura

- **Puerto:** `3100` — enrutado bajo `/pagos-api/` en Nginx (interno, no dominio público directo)
- **Dashboard Admin:** SPA estática servida por Nginx en `/pagos-admin/`
- **Proceso PM2:** `elitepass-payments` — modo **Fork** — heap 256 MB
- **ID PM2:** 6

### Deploy

```bash
cd /home/soporte/elitepass-payments
# No requiere build — tsx corre fuente TypeScript directamente
pm2 restart elitepass-payments
```

### Variables de Entorno

```env
DATABASE_URL="postgresql://user:password@127.0.0.1:6432/elitepass_payments?pgbouncer=true&connection_limit=5"
ENCRYPTION_KEY="64_caracteres_hexadecimales_clave_maestra_aes"
CALLBACK_HMAC_SECRET="clave_firmado_callbacks_hacia_apps"
APP_KEY_POS="app_key_pos_seguro"
APP_KEY_RESERVAS="app_key_reservas_seguro"
BCP_WEBHOOK_SECRET="clave_webhook_banco_bcp"
BNB_WEBHOOK_SECRET="clave_webhook_banco_bnb"
AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;..."
SIAT_NIT="nit_del_comercio_principal"
SIAT_API_URL="https://piloto.factura.impuestos.gob.bo"
PORT=3100
```
