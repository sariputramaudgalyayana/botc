# Seed

## Initial prompt

We are going to develop the following app:

1. Single file HTML
2. Mobile first
3. Theme based on Blood on the Clock Tower
   1. see balance/experiments/BoCT/5721_tiAx2eaqc_6vrehg_1cssy7Zrpz.jpeg fore reference
4. User see all the players circle of players of all the players
5. Each player is represented as a smaller circle
   1. See balance/experiments/BoCT/sects-and-violets-grimoire.jpg for reference
6. At any point during the game user can enter the the number of players
7. At any point during the game user can says how many players are townsfolk, outsiders, minions & demons
8. At any point during the game player can guess which player is what precise character
9. At any point during the game the User can indicate that a player is dead

I would like to start with a functionless design, then a spec, then implementation. However, before even that, what in my instructions is missing or unclear.

## Project Requirements — Blood on the Clocktower Player Board (Mobile) #ChatGPT

### Purpose

- Player-side deduction board.
- No storyteller functionality.
- All roles are guesses only.

### Platform

- Single HTML file.
- Mobile-first design.
- Fully local.
- Persistent via local storage.
- No server.
- No sharing.
- Offline capable.

### Player Configuration

- Supports 5–20 players.
- Player count fixed after setup.
- Seats arranged in a full circle.
- Circle scales dynamically to player count.
- Dead players remain in position.

### Role Configuration

- Supports any script.
- Roles grouped by:
  - Townsfolk
  - Outsiders
  - Minions
  - Demons
  - Travelers
- User manually sets category counts.
- No enforcement of official distribution rules.

### Player Token Display

Each player token shows:

- Seat number
- Alignment (derived from most recent role guess)
- Multiple role guesses
- Death marker

No:

- Player names
- Notes
- Confidence levels

### Role Guessing

- Tap a player token to open role selection.
- Roles displayed grouped by alignment.
- Tap role to add guess.
- Tap again to remove guess.
- Multiple guesses allowed per player.
- Alignment color automatically reflects role guesses.
- Mixed alignment guesses display mixed visual state.
- No search.
- No typing required for role selection.

### Death State

- User can toggle a player as dead.
- Dead players remain in the circle.
- Death indicated visually via:
  - Desaturated token
  - Red ring
  - Skull indicator

### Visual Design

- Always circular layout.
- Responsive scaling.
- Mobile tap targets minimum 44px.
- Alignment color-coded.
- Theme inspired by Blood on the Clocktower aesthetic.
- Gothic styling with dark purple and gold palette.

### Constraints

- No center tokens.
- No seat collapsing.
- No mid-game player count changes.
- No drag-and-drop.
- No long-press interactions.
- Tap-based interaction model only.

## Design refinement #Claude

Unsaved chat back and forth
