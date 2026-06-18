# Telegram Notification Service Tasks

- [x] 1. Request and securely store the Telegram Bot Token from the user.
- [x] 2. Scaffold the new microservice (`elitepass-noti-telegram`) using Node.js.
- [x] 3. Design and create the database schema/table to map `phone_number` to Telegram `chat_id`.
- [x] 4. Implement the Telegram Bot logic (handle `/start` and contact sharing).
- [x] 5. Implement the internal REST API endpoint (`POST /api/v1/notifications/send`).
- [x] 6. Secure the internal API endpoint with an internal API key.
- [x] 7. Update `elitepass-reservas` to interact with this new service.
- [x] 8. Update `elitepass-pos` to interact with this new service.
- [x] 9. Configure PM2/Nginx/Podman deployment for the new microservice.
