# Telegram Notification Service Design

## Technical Decisions

### Architecture
1. **Standalone Microservice**: We will create a new microservice (`elitepass-noti-telegram`) to isolate the Telegram bot logic from the core systems (`elitepass-reservas` and `elitepass-pos`). This ensures that rate limits or issues with Telegram do not block the main backend threads.
2. **Framework**: Node.js with Express or Fastify (matching the existing backend tech stack).
3. **Database**: We need a small table or database schema to map cell phone numbers to Telegram Chat IDs (`chat_id`). If the main PostgreSQL database is shared, we can create a `telegram_subscriptions` table. If it's a separate database, we will use a separate PostgreSQL schema.
4. **Communication**: The service will expose a REST API `POST /api/v1/notifications/send` to send messages.

### Telegram Interaction
- **Subscription Flow**: 
  - User starts the bot (`/start`).
  - Bot asks for the user's phone number (using Telegram's `request_contact` button for verification).
  - The service receives the contact, verifies the cell phone number, and stores the `chat_id` mapped to the `phone_number`.
- **Sending Messages**:
  - The `elitepass-reservas` or `pos` makes an HTTP request to the notification service with the `phone_number` and the `message`.
  - The notification service looks up the `chat_id` and sends the message using the Telegram Bot API.

### Security
- The API endpoints exposed by this microservice must be protected (e.g., via an internal API key or JWT) so that only our internal services can trigger messages.
- Input validation to prevent abuse.
