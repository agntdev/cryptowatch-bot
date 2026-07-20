# Crypto Price Alert Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that lets users track cryptocurrency prices with customizable threshold and percent-change alerts, manage private watchlists via buttons, and receive morning summaries. Owner gains access to aggregate usage metrics and a leaderboard of most-triggered alerts. Features include quiet hours, alert cooldowns, and error handling for invalid tickers.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram users interested in cryptocurrency monitoring
- Bot owner/admin for usage analytics

## Success criteria

- Users receive accurate alerts when price thresholds/changes are met
- Owner can view real-time metrics dashboard via /admin_stats
- All alert types are configurable and cancellable per user

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Begin onboarding or reopen main menu
- **Watchlist** (button, actor: user, callback: watchlist:main) — Manage cryptocurrency watchlist items
- **Add Coin** (button, actor: user, callback: watchlist:add) — Add predefined or custom cryptocurrency ticker
- **/price** (command, actor: user, command: /price) — Request current price for specific ticker or view watchlist prices
- **/admin_stats** (command, actor: owner, command: /admin_stats) — View global usage metrics and alert leaderboard

## Flows

### onboarding_flow
_Trigger:_ /start

1. Request timezone selection
2. Offer quiet hours configuration
3. Offer morning summary setup

_Data touched:_ User profile

### add_coin_flow
_Trigger:_ watchlist:add

1. Show predefined coin buttons
2. Handle custom ticker input
3. Normalize ticker if possible
4. Add to watchlist with alert options

_Data touched:_ Watchlist item, Alert rule

### threshold_alert_flow
_Trigger:_ alert:threshold

1. Request price threshold
2. Confirm direction (above/below)
3. Set cooldown period

_Data touched:_ Alert rule, Sent alert record

### percent_alert_flow
_Trigger:_ alert:percent

1. Request percent change value
2. Select time window (1h/24h)
3. Confirm direction (up/down/both)

_Data touched:_ Alert rule, Sent alert record, Price sample

### morning_summary_flow
_Trigger:_ scheduled:summary

1. Compile watchlist prices
2. Identify notable movers
3. Check for queued alerts

_Data touched:_ User profile, Sent alert record, Price sample

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User profile** _(retention: persistent)_ — User-specific settings and preferences
  - fields: telegram_user_id, display_name, timezone, quiet_hours_start, quiet_hours_end, morning_summary_time, alert_cooldowns
- **Watchlist item** _(retention: persistent)_ — Tracked cryptocurrency ticker with metadata
  - fields: ticker, normalized_name, user_label, alert_rules
- **Alert rule** _(retention: persistent)_ — Price alert configuration per watchlist item
  - fields: rule_type, direction, threshold_price, percent_change, time_window, is_active
- **Price sample** _(retention: session)_ — Historical price data for alert calculations
  - fields: ticker, timestamp, price
- **Sent alert record** _(retention: persistent)_ — Record of delivered alerts for cooldown tracking and analytics
  - fields: user_id, watchlist_item_id, rule_id, trigger_time, price_before, price_after, percent_change

## Integrations

- **Telegram** (required) — Bot API messaging and scheduled notifications
- **Market data API** (required) — Fetch live cryptocurrency prices
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /admin_stats command to view metrics
- Leaderboard generation logic for most-triggered alerts

## Notifications

- Telegram direct messages for price alerts
- Morning summary delivery at configured local time
- Quiet hours alert queue processing at end of period

## Permissions & privacy

- All user data is private and not shared
- Owner can only view aggregate metrics, not individual user data
- Price samples are purged after 24h to minimize storage

## Edge cases

- Invalid/unrecognized tickers during manual entry
- Market data API outages during alert checks
- Overlapping quiet hours and alert windows
- Price oscillations triggering repeated alerts

## Required tests

- Verify threshold alerts trigger at exact price points
- Test percent-change alerts across 1h/24h windows
- Validate quiet hours suppression and queue delivery
- Confirm /admin_stats shows correct metrics after multiple alerts

## Assumptions

- Default alert cooldown is 6 hours
- Percent-change alerts default to 1h window
- Predefined coins include Bitcoin, Ethereum, Toncoin
- Queued alerts during quiet hours are delivered once at end
