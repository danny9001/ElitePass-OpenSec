# Telegram Notification Service Proposal

## Business Intent
The goal is to create a new, centralized Telegram-based notification service for the ElitePass/Genial-it ecosystem. This microservice will expose an API that can be consumed by other systems, specifically the `elitepass-reservas` and `pos` modules. 

The primary business requirement is to allow users of these systems to subscribe to receive notifications via Telegram using their cell phone number as an identifier.

## Target Bot
Bot Username: `@ElitePassBO_bot`
(Note: Token has not been provided yet and must be requested).

## Scope
- Create a standalone API service to handle Telegram Webhooks/Polling and message sending.
- Implement subscription flows where users can link their cell phone number to their Telegram account.
- Expose an endpoint for `elitepass-reservas` and `elitepass-pos` to trigger notifications.
