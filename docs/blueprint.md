# Restaurant Reservation Bot — Bot specification

**Archetype:** booking

**Voice:** professional and warm — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that manages restaurant reservations with real-time availability checks, allowing guests to book, reschedule, or cancel via inline buttons. Issues reference codes for bookings, sends pre-visit reminders, and provides the owner with a dashboard to view upcoming reservations, track daily capacity, and mark no-shows directly in Telegram.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- restaurant guests
- restaurant owner

## Success criteria

- Guests can book only genuinely available time slots
- Owner receives instant notifications of booking changes
- No-show tracking works via inline owner actions

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu with party size and date selection
- **Book now** (button, actor: user, callback: booking:start) — Initiate booking flow with party size and date prompts
  - inputs: party_size, booking_date
  - outputs: available_time_slots
- **/today** (command, actor: owner, command: /today) — Show owner dashboard for today's bookings and capacity
  - inputs: none
  - outputs: booking_list, capacity_summary
- **/upcoming** (command, actor: owner, command: /upcoming) — Show owner dashboard for all future bookings
  - inputs: none
  - outputs: booking_list, no_show_actions

## Flows

### booking_flow
_Trigger:_ booking:start

1. Select party size
2. Select date
3. Show available time slots
4. Select time slot
5. Request optional guest info
6. Confirm booking with reference code

_Data touched:_ booking, venue_configuration, table_inventory

### reminder_flow
_Trigger:_ scheduled_reminder

1. Send pre-visit reminder with reference code
2. Offer reschedule/cancel options

_Data touched:_ booking

### owner_dashboard
_Trigger:_ /today or /upcoming

1. List bookings with status
2. Show remaining capacity by time window
3. Offer no-show marking

_Data touched:_ booking, venue_configuration

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **venue_configuration** _(retention: persistent)_ — Restaurant operating parameters
  - fields: weekday_hours, sitting_duration, table_inventory
- **booking** _(retention: persistent)_ — Reservation records with status tracking
  - fields: guest_name, contact_info, party_size, datetime, tables, reference_code, status
- **reminder_settings** _(retention: persistent)_ — Reminder timing configuration
  - fields: pre_visit_hours

## Integrations

- **Telegram** (required) — Bot API messaging and owner notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure weekday-specific opening hours
- Adjust sitting duration
- Modify table inventory (seat counts)
- Change reminder timing

## Notifications

- Guests receive reminders with reschedule/cancel options
- Owner gets instant alerts for new/cancelled/rescheduled bookings
- Owner can mark no-shows via inline actions

## Permissions & privacy

- Guest contact info stored only for reservation purposes
- Owner has full access to all booking data
- No third-party data sharing

## Edge cases

- No available slots for requested date
- Guest tries to book during closed hours
- Owner attempts to mark a non-existent booking as no-show

## Required tests

- End-to-end booking from slot selection to confirmation
- Owner dashboard shows accurate capacity after multiple bookings
- Reminder message triggers with correct timing

## Assumptions

- Default opening hours set to 11:00-22:00
- Default table inventory includes 2/4/6-seat tables
- Owner uses provided Telegram account for admin functions
