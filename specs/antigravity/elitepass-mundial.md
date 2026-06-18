# Functional Specification: apuestas-mundial-2026

## Contexto
Plataforma PWA de pronósticos y quinielas del Mundial de Fútbol 2026.
Multi-empresa, rankings privados por empresa, marcadores en tiempo real.

## Scenario: Login con email/password
**Given** un usuario registrado en la empresa
**When** envía credenciales válidas a /api/auth/login
**Then** recibe JWT en cookie HttpOnly
**And** es redirigido al dashboard de pronósticos de su empresa

## Scenario: Login con Passkey (WebAuthn)
**Given** un usuario con passkey registrada
**When** completa el challenge FIDO2 en el browser
**Then** recibe JWT firmado sin haber ingresado contraseña
**And** la passkey se verifica contra el registro en PostgreSQL

## Scenario: Pronóstico de partido
**Given** un usuario autenticado con modo de apuesta activo
**When** ingresa su pronóstico (goles local / goles visitante)
**Then** el sistema guarda el pronóstico con timestamp
**And** bloquea modificaciones si el partido ya comenzó (timestamp >= kickoff - 1h)

## Scenario: Marcadores en tiempo real
**Given** un partido en curso
**When** el scheduler actualiza via API FIFA (cada hora)
**Then** los marcadores se actualizan en la DB
**And** el ranking se recalcula automáticamente

## Scenario: Notificación push de partido próximo
**Given** un usuario con push subscription activa
**When** el scheduler detecta un partido en las próximas 2 horas
**Then** envía Web Push VAPID al dispositivo del usuario
**And** el mensaje incluye equipos, hora y recordatorio de pronóstico pendiente

## Scenario: Ranking semanal
**Given** cada lunes a las 8 AM
**When** el scheduler ejecuta el cálculo de ranking semanal
**Then** genera el ranking por empresa con puntos acumulados
**And** envía notificación push con la posición de cada usuario

## Scenario: Modo TV
**Given** una pantalla en modo kiosk accediendo a /tv
**When** se carga la vista
**Then** muestra split-flap animado con partidos del día y marcadores
**And** se auto-refresca sin interacción de usuario

## Scenario: Vista Planilla (ingreso masivo)
**Given** un usuario con permisos de editor
**When** accede a la vista planilla
**Then** puede ingresar pronósticos de múltiples partidos en una tabla estilo Excel
**And** el sistema valida y guarda todos en una sola operación

## Scenario: Historial y CRUD de Mensajes (Admin & SuperAdmin)
**Given** un usuario autenticado como Admin o SuperAdmin
**When** accede a la pestaña de Mensajes en el panel de administración
**Then** el sistema presenta un formulario de envío y un listado/historial completo de notificaciones
**And** los administradores pueden editar y eliminar los mensajes creados por ellos mismos
**And** los superadministradores pueden ver, editar y eliminar cualquier notificación del sistema

## Scenario: Notificaciones Automáticas y Flujo Telegram
**Given** un evento de partido (inicio, gol, finalización) o actualización del ranking
**When** el programador detecta el cambio o se alcanzan las horas límite
**Then** envía alertas en tiempo real tanto por Web Push como por Telegram
**And** envía un recordatorio de pronósticos "Registra tu pronóstico" 1 hora y 30 minutos antes del límite de apuestas (2 horas y 30 minutos antes del partido, considerando que las apuestas cierran 1 hora antes)

## Scenario: Instalación como Aplicación PWA y Onboarding de Notificaciones
Given un dispositivo móvil o navegador compatible con PWA
When el usuario ingresa a la aplicación y no ha cerrado el onboarding
Then si la aplicación no está en modo standalone, ofrece "Instalar Aplicación" mediante un banner interactivo
And si la aplicación ya está en modo standalone pero las notificaciones no están activadas, ofrece "Activar Notificaciones" mediante el mismo banner
And al hacer clic en instalar se ejecuta el prompt nativo y se guarda en localStorage, y al hacer clic en activar se habilitan las notificaciones push

## Scenario: Consola de Logs y Estado de Correos (Super Admin)
Given un usuario autenticado con rol de superadmin
When accede a la sección de Logs en el panel de administración
Then puede consultar logs filtrados por tipo: Correos Enviados/Fallados, Logs de Eventos de la aplicación y Auditoría de Superadmin
And los logs históricos se auto-limpian al cumplir 90 días de antigüedad
And los logs de correos indican detalladamente el destinatario, asunto, estado (exitoso/fallido) y los mensajes de error devueltos por la API

## Scenario: Envío en Lotes de Correos (Microsoft Graph Throttling Protection)
Given múltiples correos generados por el sistema de quinielas
When se invoca el envío de correos
Then los correos se registran inicialmente en la cola `mail_queue` como pendientes
And un programador recurrente (scheduler de fondo) procesa la cola de forma segura cada 60 segundos
And despacha los correos en rondas de máximo 5 correos por minuto para mitigar el bloqueo por throttling de Microsoft Graph

## Scenario: Edición de Pagos con Soporte de Vouchers
Given un administrador del sistema gestionando el control de pagos
When edita un registro de pago existente en el historial del participante
Then puede modificar el monto, la fecha de registro y las observaciones
And puede adjuntar o actualizar el archivo del voucher de pago (PDF o Imagen)
And el sistema procesa el archivo para guardarlo en Azure Blob Storage y actualiza la base de datos

## Scenario: Reporte y Consola de PWA de Dispositivos (Administración)
Given un administrador del sistema o superadmin
When los usuarios acceden a la aplicación y reportan en cada sesión su estado de instalación
Then el sistema guarda el indicador `pwa_installed` y la fecha `pwa_updated_at` en la base de datos
And el administrador puede ver una pestaña "PWA" en el panel de administración
And dicha pestaña muestra estadísticas (Total participantes, PWA instalada, Navegador, Tasa de adopción %) y un buscador de usuarios
And presenta una tabla con el listado detallado de cada usuario, su empresa, estado de PWA (badge verde si está instalado, zinc/gris si usa navegador) y la fecha del último reporte

