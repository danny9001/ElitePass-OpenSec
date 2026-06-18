# Design: Auditoría de Contraseñas y Sincronización SSO

## Metodología de Auditoría
El análisis técnico se dividirá en tres áreas:

### 1. Detección de Contraseñas y Secretos Débiles
- Se escanearán los archivos `.env` activos de todos los proyectos (`elitepass-reservas`, `elitepass-pos`, `elitepass-payments`, `elitepass-identity`, `elitepass-noti-telegram`, `elitepass-monitor`) buscando credenciales de PostgreSQL, Redis, contraseñas de API y claves maestras de cifrado.
- Se clasificará como **Débil** cualquier credencial que:
  - Sea idéntica al usuario (ej: `user:user`).
  - Utilice palabras genéricas (`password`, `admin`, `secret`, `elitepass`).
  - Tenga una longitud inferior a 12 caracteres para secretos criptográficos o contraseñas de base de datos.
  - Tenga valores por defecto idénticos a los archivos `.env.example`.

### 2. Políticas de Contraseñas en el Código
- Se revisará el código de registro y autenticación en `elitepass-identity` (el IdP del ecosistema) para determinar:
  - Validación de complejidad de contraseñas (longitud mínima, mezcla de caracteres).
  - Algoritmo de hashing y factor de coste (debe ser `bcrypt` con coste >= 12, o `argon2id`).
  - Implementación de Passkeys (WebAuthn) como mecanismo que desafía las debilidades de contraseñas tradicionales.

### 3. Mecanismo de Sincronización y Validación SSO
- Se analizará la validez y expiración del JWT emitido por `elitepass-identity`.
- Se verificará la rotación de secretos y la seguridad de las cookies de sesión BetterAuth en las apps satélite.
