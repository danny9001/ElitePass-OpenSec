# Tasks: Auditoría de Contraseñas y Políticas

- [x] Escanear y revisar archivos `.env` en cada proyecto en busca de secretos y contraseñas débiles
- [x] Analizar política de complejidad y hashing de contraseñas en el código de `elitepass-identity`
- [x] Analizar flujos de verificación de sesión y sincronización entre Identity y Reservas/POS/Payments
- [x] Crear informe final detallando los hallazgos y las recomendaciones de hardening
- [x] Rotar claves de JWT por defecto en `elitepass-pos` (`JWT_SECRET` y `JWT_REFRESH_SECRET`)
- [x] Rotar clave interna de API en `elitepass-noti-telegram` e inyectar `TELEGRAM_SERVICE_API_KEY` en `elitepass-reservas` y `elitepass-pos`
- [x] Cambiar contraseña de superadmin (`dlandivar@genial-it.net`) en base de datos Identity a una contraseña compleja de 14 caracteres (`K8#sNx9$wP2!v6`)
- [x] Actualizar contraseñas por defecto de superadmin en las configuraciones locales del POS a 14 caracteres (`J5$xWv9#kP2!r8`)
- [x] Reiniciar los servicios modificados en PM2 aplicando `--update-env` para cargar las nuevas variables
- [x] Actualizar el estado de estas tareas a completado
