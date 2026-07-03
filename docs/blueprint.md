# MenuBot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that allows chat owners to create, manage, and present interactive menus with buttons that perform actions like sending messages, forwarding preset replies, or opening URLs. Menus can be set per chat or managed privately by admins.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Small business owners
- Community managers
- Telegram group admins

## Success criteria

- Admins can create and manage menus with buttons in Telegram chats
- Users can interact with menus to perform actions like sending messages or opening URLs
- Admins receive confirmation messages when menus are created or updated

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **/menu** (command, actor: user, command: /menu) — Show the active menu for the chat
- **/createmenu** (command, actor: admin, command: /createmenu) — Start creating a new menu with an interactive setup
- **/editmenu** (command, actor: admin, command: /editmenu) — Open the menu editor to modify existing menus
- **/setmenu** (command, actor: admin, command: /setmenu) — Select which menu is shown by /menu and for publishing
- **/publish** (command, actor: admin, command: /publish) — Post the active menu into the chat with inline buttons
- **/addadmin** (command, actor: admin, command: /addadmin) — Add another admin to manage menus for the chat

## Flows

### Onboarding
_Trigger:_ /start

1. Bot explains basic commands
2. Auto-assigns the Telegram chat creator as admin

_Data touched:_ Chat, Admin user

### View Menu
_Trigger:_ /menu

1. Display active menu as inline buttons
2. Perform action when button is tapped

_Data touched:_ Menu, Menu item

### Create Menu
_Trigger:_ /createmenu

1. Prompt for menu name
2. Collect title and description
3. Add items step-by-step

_Data touched:_ Menu, Menu item

### Edit Menu
_Trigger:_ /editmenu

1. Open menu editor
2. Allow adding/editing/reordering items
3. Save changes and send confirmation

_Data touched:_ Menu, Menu item

### Set Active Menu
_Trigger:_ /setmenu

1. List available menus
2. Select one to activate
3. Update chat settings

_Data touched:_ Chat, Menu

### Publish Menu
_Trigger:_ /publish

1. Post active menu in chat
2. Option to pin the message
3. Send confirmation to admin

_Data touched:_ Chat, Menu

### Add Admin
_Trigger:_ /addadmin

1. Request admin verification
2. Add new admin to chat settings
3. Send confirmation to existing admin

_Data touched:_ Chat, Admin user

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Chat** _(retention: persistent)_ — Telegram chat or channel where a menu is attached
  - fields: chat_id, active_menu, admins
- **Menu** _(retention: persistent)_ — Named collection of menu items
  - fields: name, title, description, items
- **Menu Item** _(retention: persistent)_ — Button with label and action
  - fields: label, action_type, action_value, order, visible
- **Admin User** _(retention: persistent)_ — User who can edit menus for a chat
  - fields: user_id, chat_id

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Create and manage menus
- Add or remove admins
- Set active menu
- Publish menu in chat

## Notifications

- Confirmation message when menu is created or updated
- Notification when admin is added

## Permissions & privacy

- Only admins can create/edit menus
- Menu data is stored persistently but not shared externally
- Private chat management requires verification in the target chat

## Edge cases

- User tries to edit a menu without admin permissions
- User tries to access a menu that doesn't exist
- Multiple admins attempt to edit the same menu simultaneously

## Required tests

- Verify that only admins can create/edit menus
- Test that menu items perform the correct actions when tapped
- Ensure confirmation messages are sent after menu changes

## Assumptions

- Menus default to dynamic (editable via the bot)
- One active menu per chat is enforced
- Admins are by default the chat creator
- Item actions are limited to three types
- Bot operates in Russian by default with English fallbacks
