# Telegram Notification Service - Functional Specification

## Scenarios

### Scenario: User subscribes to notifications
**Given** a user interacts with `@ElitePassBO_bot` on Telegram
**When** the user sends the `/start` command
**Then** the bot should reply asking the user to share their contact (phone number)
**When** the user shares their contact
**Then** the system should register the `chat_id` against the `phone_number` and confirm the subscription.

### Scenario: Internal system sends a notification
**Given** the user is subscribed (their `phone_number` is linked to a `chat_id`)
**When** an internal system (`elitepass-reservas` or `pos`) sends a request to the notification service with the `phone_number` and `message`
**Then** the service should look up the `chat_id`
**And** send the `message` to the user via Telegram.

### Scenario: User not found
**Given** an internal system attempts to send a notification to a `phone_number`
**When** the `phone_number` is not registered in the system
**Then** the notification service should return a `404 Not Found` or `400 Bad Request` indicating the user has not subscribed.
