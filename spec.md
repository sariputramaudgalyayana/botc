# Blood on the Clocktower — Player Board Spec

## Overview

Single-file HTML mobile app for tracking player roles and death state during a Blood on the Clocktower game. Player-side deduction tool only — no storyteller features. All role assignments are guesses.

## Architecture

- **Single file**: `index.html` — all HTML, CSS, and JS inline
- **No build step, no server, no dependencies** (except Google Fonts CDN)
- **State**: single JS object, rendered to DOM on mutation
- **Persistence**: `localStorage` key `botc-state`, JSON serialized
- **Offline**: works without network after first font load (system fallback fonts defined)

## Data Model

```js
state = {
  phase: "setup" | "playing",
  playerCount: 5..20,           // default 7
  distribution: {
    townsfolk: Number,
    outsiders: Number,
    minions: Number,
    demons: Number,
    travelers: Number
  },
  players: [
    {
      seat: Number,             // 1-indexed, immutable after setup
      roles: [String],          // role guess names, may be empty
      alignment: String | null, // "townsfolk" | "outsider" | "minion" | "demon" | "traveler" | null
      dead: Boolean,
      hasVote: Boolean          // dead player's one remaining vote
    }
  ],
  customRoles: {
    townsfolk: [String],
    outsider: [String],
    minion: [String],
    demon: [String],
    traveler: [String]
  }
}
```

### Alignment derivation

- If `roles` is empty → `alignment = null` (unassigned, default purple/gold styling)
- If all role guesses belong to the same category → alignment = that category
- If role guesses span multiple categories → alignment = category of the **first** role in the array (the primary guess drives token color)

### Default distribution

When player count changes, auto-suggest the official Trouble Brewing distribution:

| Players | Townsfolk | Outsiders | Minions | Demons |
|---------|-----------|-----------|---------|--------|
| 5       | 3         | 0         | 1       | 1      |
| 6       | 3         | 1         | 1       | 1      |
| 7       | 5         | 0         | 1       | 1      |
| 8       | 5         | 1         | 1       | 1      |
| 9       | 5         | 2         | 1       | 1      |
| 10      | 7         | 0         | 2       | 1      |
| 11      | 7         | 1         | 2       | 1      |
| 12      | 7         | 2         | 2       | 1      |
| 13      | 9         | 0         | 3       | 1      |
| 14      | 9         | 1         | 3       | 1      |
| 15      | 9         | 2         | 3       | 1      |

Travelers default to 0. User can override any count freely — no enforcement.

## App Flow

### 1. First launch

- `localStorage` empty → show Setup sheet automatically
- Board is empty (no tokens rendered yet)

### 2. Setup phase

User configures:
1. Player count (stepper, 5–20)
2. Distribution counts per category (individual steppers)
3. Tap "Begin" → `phase` transitions to `"playing"`, tokens rendered on board

### 3. Playing phase

- Board shows all player tokens in a circle
- Tally bar shows current distribution counts
- Footer buttons: "Setup" (re-open setup) and "Roles" (no-op until token tapped, see below)

### 4. Re-entering setup

- Tapping "Setup" during play opens setup sheet as an overlay
- Changing player count resets all player data (with confirmation dialog)
- Changing distribution counts does NOT reset player data

### 5. Resuming

- On page load, if `localStorage` has saved state with `phase: "playing"`, restore directly to the board

## UI Components

### Header

- Fixed at top
- Title: "Clocktower" in Uncial Antiqua
- Decorative pointed arch below (CSS triangle)

### Tally Bar

- Row of 5 compact items below header: Town, Out, Min, Dem, Trav
- Each shows the count from `state.distribution`
- Color-coded per alignment

### Board (main area)

- Fills remaining vertical space between tally and footer
- Contains the circular token layout
- Decorative center cross (CSS, purely visual)
- Decorative stained-glass ring behind tokens (CSS)

#### Token positioning

Tokens arranged in a circle using trigonometric placement:

```
angle = (2 * PI * seatIndex / playerCount) - PI/2   // start at 12 o'clock
x = 50% + radius * cos(angle)
y = 50% + radius * sin(angle)
```

- `radius`: 42% of board size
- Board size: `min(82vw, 82vh - 200px)`, max 480px
- Token size: 68px, centered on calculated point via negative margins

#### Token rendering

Each token is a circular element displaying:

**No role guesses (unassigned)**:
- Seat number only, centered
- Default dark purple styling

**Single role guess**:
- Seat number centered
- Role name below in small uppercase text (7px)

**Two role guesses (pie chart)**:
- Circle divided into top/bottom halves
- Conic gradient with alternating subtle shades
- Horizontal gold divider line through center
- Seat number centered (17px)
- Role names positioned at midpoint of each half segment
- First role at top, second at bottom

**Three role guesses (pie chart)**:
- Circle divided into three 120-degree wedges
- Three gold divider lines radiating from center
- Seat number centered (15px)
- Role names at midpoint of each wedge (5.5px font)
- Positions: upper-right, bottom-center, upper-left

**Alignment colors** (applied to token border + glow):

| Alignment  | Border     | Glow base        | Seat number |
|------------|------------|------------------|-------------|
| Townsfolk  | `#4a6aaa`  | `rgb(70,100,170)` | `#7aaae0`   |
| Outsider   | `#3a7a5a`  | `rgb(58,122,90)`  | `#5aba8a`   |
| Minion     | `#aa6040`  | `rgb(170,96,64)`  | `#da8a60`   |
| Demon      | `#8a2a2a`  | `rgb(138,42,42)`  | `#da5050`   |
| Traveler   | `#6a5a8a`  | `rgb(120,90,170)` | `#b090d0`   |
| Unassigned | `#3a2a5c`  | none              | `#c9a84c`   |

**Dead state**:
- `filter: saturate(0.2) brightness(0.5)` on token-inner
- Red cross symbol (✝) positioned top-right of token
- Dead token remains in circle, same position

### Footer

- Two buttons: "Setup" and "Roles"
- Gothic styled with gold text, dark gradient background, gold accent line
- Min tap target: 44px

### Overlay

- Semi-transparent dark backdrop (`rgba(6,4,10,0.75)`)
- Tapping overlay closes any open sheet
- Fade transition 0.3s

### Setup Sheet (bottom sheet)

Slides up from bottom, max-height 85vh.

**Sections**:
1. **Players** — large stepper (-, value, +), range 5–20
2. **Role Distribution** — one row per category with color pip, label, and mini stepper (-, value, +)

**Controls**:
- "Begin" button at bottom confirms and transitions to playing phase
- Drag handle at top — tapping closes the sheet
- If re-opened during play, "Begin" becomes "Apply"

### Roles Sheet (bottom sheet)

Opens when a player token is tapped during the playing phase.

**Header area**:
- Player seat indicator (small token circle showing seat number)
- "Select role guesses" label

**Role groups** (5 sections, one per alignment):
Each group has:
1. Color pip + colored category label + decorative line
2. Grid of role chips (flex-wrap)
3. Custom role text input + "+" button

**Role chips**:
- Tappable, min-height 36px
- Default: dark background, subtle border
- Selected: alignment-colored border and background tint, colored text
- Tapping a chip toggles its selected state for that player
- Multiple chips can be selected (multiple guesses)

**Built-in roles** (Trouble Brewing script):

- Townsfolk: Washerwoman, Librarian, Investigator, Chef, Empath, Fortune Teller, Undertaker, Monk, Ravenkeeper, Virgin, Slayer, Soldier, Mayor
- Outsiders: Butler, Drunk, Recluse, Saint
- Minions: Poisoner, Spy, Scarlet Woman, Baron
- Demons: Imp
- Travelers: Scapegoat, Gunslinger, Beggar, Bureaucrat, Thief

**Custom roles**:
- Text input + "+" button per category
- Typing a name and tapping "+" adds it as a new chip in that category
- Custom roles persist in `state.customRoles` and appear in all future role sheets
- Custom roles can be removed (long-press or X button — TBD, keep simple for v1: no removal)

**Closing the roles sheet**:
- Tap overlay, tap handle, or tap "Done" (if added)
- Saves selected roles to `state.players[seat].roles`
- Token on board immediately updates to reflect new guesses

### Death toggle

- Tap-and-hold on a token (300ms) toggles dead/alive state
- Visual feedback: brief pulse animation on toggle
- Dead players keep their role guesses and seat position
- `hasVote` defaults to `true` when a player dies (ghost vote)

Wait — seed.md says "No long-press interactions. Tap-based interaction model only."

**Revised death toggle**: Tapping a dead/alive indicator within the roles sheet, or a dedicated icon on the token. Two options:

**Option chosen**: When the roles sheet is open for a player, include a "Mark Dead" / "Mark Alive" toggle button at the top of the sheet, next to the seat indicator. Single tap toggles death state.

## Interactions Summary

| Action | Trigger | Result |
|--------|---------|--------|
| Open setup | Tap "Setup" button | Setup sheet slides up |
| Close any sheet | Tap overlay or drag handle | Sheet slides down, overlay fades |
| Change player count | Tap -/+ in setup stepper | Updates count, recalculates default distribution |
| Begin game | Tap "Begin" in setup sheet | Creates player array, renders board, closes sheet |
| Open role selection | Tap a player token on board | Roles sheet opens for that player's seat |
| Toggle role guess | Tap a role chip in roles sheet | Adds/removes role from player's guesses |
| Add custom role | Type name + tap "+" | Adds chip to category, persists |
| Toggle death | Tap death toggle in roles sheet | Flips player's dead state |
| Toggle ghost vote | Tap vote icon (only visible when dead) | Flips hasVote |

## Visual Design Tokens

### Colors

```
Background:       #0e0b14
Surface:          #1c1528
Surface dark:     #120e1c / #140e22
Gold:             #c9a84c
Text primary:     #c8bda0
Text muted:       #9a8a7a
Border default:   #3a2a5c
Overlay:          rgba(6,4,10,0.75)
Divider:          rgba(201,168,76,0.12)
```

### Typography

```
Display:    'Uncial Antiqua', serif     — title, seat numbers, section headings
Body:       'Cormorant Garamond', serif — buttons, labels, role text
Fallbacks:  Georgia, 'Times New Roman', serif
```

### Sizing

```
Token:          68px diameter
Min tap target: 44px
Board radius:   42% of board container
Board max:      480px
Sheet max-h:    85vh
Sheet radius:   16px (top corners)
```

## State Persistence

### Save triggers

State is saved to `localStorage` on every mutation:
- Player count change
- Distribution change
- Role guess toggle
- Death toggle
- Vote toggle
- Custom role added
- Game begin

### Storage key

`botc-state`

### Migration

If stored state schema doesn't match expected version, reset to defaults. Include a `version: 1` field in state for future migrations.

### Reset

"Setup" sheet includes a "Reset Game" button (destructive, requires confirmation tap) that clears all state and returns to first-launch experience.

## Rendering Strategy

No framework. Vanilla JS with a simple render loop:

1. State mutations go through a `setState(patch)` function
2. `setState` merges patch into state, saves to localStorage, calls `render()`
3. `render()` updates the DOM:
   - Tally bar counts
   - Board tokens (position, classes, inner content)
   - Sheet content when open
4. Token creation uses `document.createElement` with class-based styling
5. Sheets are persistent DOM elements toggled via `.open` class

### Performance considerations

- Max 20 tokens — no virtualization needed
- Re-render only changed tokens when possible (compare previous roles/dead state)
- CSS transitions handle animations (no JS animation loops)

## Scope Exclusions (from seed.md constraints)

- No player names
- No notes
- No confidence levels
- No search in role list
- No center tokens on board
- No seat collapsing
- No mid-game player count changes (requires re-setup)
- No drag-and-drop
- No long-press interactions
- No storyteller features
- No sharing or networking
- No role enforcement (any count of any category is allowed)
