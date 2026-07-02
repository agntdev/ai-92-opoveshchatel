# AI-92 Fuel Finder — Bot specification

**Archetype:** custom

**Voice:** helpful and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot that notifies users when nearby gas stations (within 5 km) are selling AI-92 fuel. Users share their location and receive alerts with station details including distance, address, and map link. Manual queries and alert settings are supported.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual drivers using Telegram

## Success criteria

- User receives real-time AI-92 availability alerts within 5 km radius
- Users can manually query nearby stations at any time
- Alerts include actionable map links and cooldown controls

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu and explain the bot's purpose
- **/subscribe** (command, command: /subscribe) — Toggle subscription to AI-92 alerts within 5 km radius
- **/nearby** (command, command: /nearby) — Request immediate check for nearby AI-92 stations
- **/stop** (command, command: /stop) — Unsubscribe and stop the bot
- **Share location** (button, callback: location:request) — Request user to share current location
- **Mute alerts** (button, callback: alert:cooldown) — Activate cooldown period to mute further alerts

## Flows

### onboarding
_Trigger:_ /start

1. Display welcome message
2. Request location sharing
3. Confirm subscription to alerts

_Data touched:_ User

### alert_flow
_Trigger:_ station availability update

1. Check user location
2. Find nearby stations with AI-92
3. Send alert with station details
4. Apply cooldown if enabled

_Data touched:_ User, Station, Alert

### manual_query
_Trigger:_ /nearby

1. Get current location
2. Search for AI-92 stations within 5 km
3. Display sorted list with details

_Data touched:_ User, Station

### settings_flow
_Trigger:_ /subscribe or /stop

1. Update subscription status
2. Confirm changes

_Data touched:_ User

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user profile with subscription status and location data
  - fields: telegram_id, last_known_location, subscription_status, radius_km, cooldown_minutes
- **Station** _(retention: persistent)_ — Gas station record with fuel availability and location
  - fields: name, location, fuels_available, last_updated
- **Alert** _(retention: persistent)_ — Record of sent notifications to users
  - fields: user_id, station_id, timestamp, cooldown_until

## Integrations

- **Telegram** (required) — Bot API messaging and location sharing
- **Fuel Availability API** (required) — External data source for station fuel availability
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure fuel data source/API
- Set default radius (5 km)
- Set default cooldown period (30 minutes)
- View alert logs

## Notifications

- Real-time AI-92 availability alerts with station details
- Manual query results for nearby stations
- Subscription status confirmation messages
- Cooldown activation confirmation

## Permissions & privacy

- Only location data necessary for fuel availability alerts is collected
- Telegram ID is stored for message delivery
- No personal data beyond location and subscription status is collected
- Location data is stored securely and used only for alert purposes

## Edge cases

- User shares invalid location
- No AI-92 stations found within 5 km
- Multiple alerts triggered simultaneously
- User disables alerts but later re-enables them
- Fuel data source returns stale or inconsistent information

## Required tests

- End-to-end alert flow from station update to user notification
- Manual query returns accurate sorted results
- Cooldown period prevents duplicate alerts
- Location sharing and storage works correctly
- Subscription toggle updates user status properly

## Assumptions

- Fuel data source will provide up-to-date and accurate information
- Users will share location data via Telegram's built-in location sharing
- Default 5 km radius is sufficient for user needs
- Default 30 minute cooldown period is acceptable for most users
