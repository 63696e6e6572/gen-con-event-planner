# CLAUDE.md

# Project

Gen Con Event Scheduler & Smart Convention Planner

Version: v1.0
Status: Initial Build Specification
Target Platform: Offline Single-File Browser Application
Primary Runtime: Modern Chromium-based browsers
Architecture Style: Offline-first SPA (Single Page Application)

---

# Environment & Commands

## Runtime Environment

This project has no build step and no package manager. The final deliverable is a single `index.html` file opened directly in a browser via `file://`.

- No Node.js, npm, or bundler involved
- No local dev server required
- Development target: Chromium-based browsers (Chrome, Edge)
- All dependencies loaded from CDN at runtime - internet connection required during development, not during use

## Development Workflow

1. Edit `index.html` (or the modular source files during development)
2. Open `index.html` directly in Chrome/Edge (`File > Open`, or drag into browser)
3. Use browser DevTools (F12) for debugging
4. Hard-refresh (`Ctrl+Shift+R`) to clear cached state between edits
5. Test localStorage persistence using DevTools > Application > Local Storage

## Resetting State During Development

To clear all persisted data during testing:

```js
// Paste into browser console
localStorage.removeItem("gencon_scheduler_v1");
location.reload();
```

## Final Deliverable

When development is complete, all modular JS and CSS files must be inlined into a single `index.html`. No external file references should remain in the final artifact.

---

# Project Goal

Build a fully offline-capable browser application that imports Gen Con event spreadsheets and allows users to:

- Browse and filter convention events
- Build optimized schedules
- Prevent event overlaps automatically
- Apply travel buffer logic between venues
- Recommend compatible alternative sessions
- Recommend best-fit events during open schedule windows
- Persist schedules locally
- Export schedules in multiple formats

The application must function entirely without a backend or internet connection after initial load.

The output should be a SINGLE self-contained HTML file with embedded JavaScript and CSS.

---

# Core Requirements

## Functional Requirements

### Event Importing

The application must:

- Import Gen Con XLSX exports
- Support future spreadsheet updates gracefully
- Normalize imported data
- Detect malformed rows safely
- Allow repeated imports without duplicating existing events

Supported import methods:

- Drag/drop XLSX
- File picker upload

---

### Event Browsing

Users must be able to:

- Search events by keyword
- Filter by:
  - game system
  - category/type
  - day
  - start time
  - duration
  - cost
  - venue/building
  - ticket availability
- Sort by:
  - start time
  - duration
  - cost
  - popularity
  - alphabetical

---

### Scheduling Engine

The scheduler must:

- Prevent overlapping events
- Apply configurable travel buffers
- Detect venue-aware transitions
- Support overnight events
- Support multi-day conventions

---

### Travel Buffer Logic

Default behavior:

- Same venue/building: 0-5 minute buffer
- Different venues: configurable 15-30 minute buffer

User-configurable settings:

- 15 minutes
- 30 minutes
- custom value

Conflict detection must account for:

- event duration
- travel time
- overnight events

---

### Overnight Event Handling

Overnight events must use HYBRID rendering:

- Logical model: single continuous event
- Visual model: split across calendar days for readability

Example:

- Event from 11:00 PM to 3:00 AM
- Renders across both days visually
- Remains one event internally

---

### Alternative Session Suggestions

When conflicts occur, the application must automatically suggest:

1. Same event title
2. Same GM
3. Same system/category
4. Closest compatible timeslot
5. Best-fit available slot

Suggestions should prioritize:

- minimal schedule disruption
- same event identity
- shortest travel requirements

---

### Recommendation Engine

The recommendation engine should analyze:

- user favorites
- preferred systems
- open schedule windows
- event availability
- current schedule density

Recommend:

- best-fit events during free time
- nearby compatible events
- events matching favorite systems/categories

Avoid:

- impossible travel
- overpacked schedules
- severe fatigue chains

---

### Favorites/Wishlist

Users must be able to:

- favorite events
- maintain a wishlist
- distinguish scheduled vs. favorited

Favorites persist locally.

---

### Persistence

Use browser localStorage.

Persist:

- imported datasets
- filters
- favorites
- schedules
- user preferences

No backend storage allowed.

---

### Multi-User Foundation

v1 does NOT require full multi-user scheduling. However, architecture MUST support:

- multiple user profiles (future)
- shared schedules (future)
- merged group schedules (future)

Design data models accordingly.

---

### Exporting

The application must support:

#### CSV Export

Export schedule as spreadsheet-compatible CSV.

#### Google Sheets-Compatible Export

Generate CSV optimized for Google Sheets import.

#### iCal Export

Generate standards-compliant `.ics` files compatible with:

- Google Calendar
- Apple Calendar
- Outlook

#### Printable Schedule

Generate a clean printable itinerary with:

- day grouping
- readable typography
- page-break-safe layout

---

# UI/UX Requirements

## Layout

Three-column desktop layout:

### Left Sidebar

Filters/search:

- keyword search
- filters
- favorites toggle

### Center Panel

Primary calendar UI:

- Google Calendar-inspired
- day columns
- draggable event blocks
- overlap visualization
- free-time visualization

### Right Sidebar

Contextual tools:

- event details
- recommendations
- conflict warnings
- alternate sessions
- export controls

---

# Visual Design

Style goals:

- clean
- readable
- information dense
- convention-friendly
- desktop-first

Do NOT:

- use dark mode
- use excessive animations
- use skeuomorphic styling

Preferred:

- subtle shadows
- soft borders
- responsive spacing
- modern flat UI

---

# Technical Requirements

## Architecture

Single-file application:

- one HTML file
- embedded CSS
- embedded JavaScript

No build process required for the final artifact.

---

## Allowed Dependencies

Use CDN-delivered browser libraries only.

Preferred:

- SheetJS (xlsx parsing)
- FullCalendar OR custom lightweight calendar
- Luxon or Day.js for datetime handling

Avoid:

- React
- Vue
- Angular
- heavy frameworks

Goal: portable standalone artifact.

## Dependency Constraints & Known Gotchas

### SheetJS (xlsx)

- Use the Community Edition (free). Load from CDN: `https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js`
- Read files with `XLSX.read(data, { type: 'array' })` after using `FileReader` to get an `ArrayBuffer`
- Extract the first sheet: `wb.Sheets[wb.SheetNames[0]]`
- Convert to JSON with `XLSX.utils.sheet_to_json(sheet, { defval: '' })` - the `defval` option ensures missing cells return empty string rather than `undefined`
- Column headers in the JSON output will match the XLSX header row exactly (whitespace and special characters included) - use a mapping object to normalize to internal field names rather than referencing raw column strings throughout the codebase

### FullCalendar

- FullCalendar v6+ requires a build step or ES module setup. For a no-build single-file app, use **FullCalendar v5** via CDN, which ships as a UMD bundle compatible with plain `<script>` tags
- CDN: `https://cdn.jsdelivr.net/npm/fullcalendar@5.11.3/main.min.js` + matching CSS
- Drag/drop repositioning requires the `interaction` plugin: `fullcalendar/interaction`
- If FullCalendar proves too heavy for 20,000+ events, fall back to a custom lightweight canvas- or DOM-based calendar renderer. Do not attempt to render all events simultaneously - use windowing

### Luxon / Day.js

- Prefer **Luxon** for its strong timezone and duration support, which matters for overnight events spanning midnight
- Load via CDN: `https://cdn.jsdelivr.net/npm/luxon@3/build/global/luxon.min.js`
- Gen Con runs in Indianapolis (America/Indiana/Indianapolis, effectively UTC-4 during convention). All event times in the XLSX are local - do not assume UTC
- If Luxon feels heavy, Day.js with the `duration` and `timezone` plugins is an acceptable alternative

### localStorage

- `localStorage` has a ~5MB limit per origin. A full Gen Con dataset of 20,000-30,000 events serialized to JSON may approach or exceed this limit
- Store the raw imported dataset compressed if possible, or store only user-generated data (schedule, favorites, preferences) in localStorage and require re-import of the XLSX on each session
- Gracefully handle `QuotaExceededError` on write and inform the user rather than silently failing

### file:// Protocol Limitations

- Running via `file://` disables certain browser APIs (e.g. some Service Worker features, `fetch()` to local paths). Do not use `fetch()` for anything in this app
- All file reading must go through the `FileReader` API or drag/drop `DataTransfer`
- CDN script tags load normally over `file://` as long as the machine has internet access during development

---

## Performance Requirements

Dataset size: 20,000-30,000 events

Application must:

- remain responsive
- support fast filtering
- avoid full re-rendering
- virtualize large lists where appropriate

---

## Source XLSX Schema

The Gen Con event export contains the following columns, in this order:

| XLSX Column | Normalized Field | Notes |
|---|---|---|
| Game ID | `id` | Unique event identifier - use as dedup key on re-import |
| Group | `group` | Event group/series identifier |
| Title | `title` | Display name |
| Short Description | `shortDescription` | Brief summary for list views |
| Long Description | `description` | Full detail for event detail panel |
| Event Type | `category` | Primary filter dimension (e.g. RPG, BGM, SEM) |
| Game System | `system` | Secondary filter dimension |
| Rules Edition | `rulesEdition` | Supplemental to system |
| Minimum Players | `minPlayers` | Integer |
| Maximum Players | `maxPlayers` | Integer |
| Age Required | `ageRequirement` | String (e.g. "Everyone (6+)", "Teen (13+)", "Adult (18+)") |
| Experience Required | `experienceRequired` | String (e.g. "None (Never played)", "Some (1-2 plays)") |
| Materials Required | `materialsRequired` | Boolean-like string ("Yes"/"No") |
| Materials Required Details | `materialsDetails` | Free text if materials required |
| Start Date & Time | `start` | Parse to JS Date; format varies by export year |
| Duration | `durationMinutes` | Stored in hours in XLSX (e.g. "2.0") - convert to minutes |
| End Date & Time | `end` | Parse to JS Date; derive or use directly |
| GM Names | `gm` | Comma-separated string of one or more names |
| Website | `website` | Optional URL |
| Email | `email` | Optional contact email - do not display publicly |
| Tournament? | `isTournament` | "Yes"/"No" string - normalize to boolean |
| Round Number | `roundNumber` | Integer; null if not a tournament |
| Total Rounds | `totalRounds` | Integer; null if not a tournament |
| Minimum Play Time | `minPlayTime` | In hours; may differ from Duration |
| Attendee Registration? | `requiresRegistration` | "Yes"/"No" - normalize to boolean |
| Cost $ | `cost` | Numeric string (e.g. "4.00") - parse to float |
| Location | `venue` | Building name - primary travel buffer key |
| Room Name | `room` | Room within venue |
| Table Number | `tableNumber` | Table within room; may be blank |
| Special Category | `specialCategory` | Optional tag (e.g. "BGM.MTG" for Magic events) |
| Tickets Available | `ticketsAvailable` | Integer; 0 means sold out |
| Last Modified | `lastModified` | ISO timestamp - use to detect stale data on re-import |

## Normalized Event Object

```js
{
  id,                  // string - Game ID, used as dedup key
  group,               // string
  title,               // string
  shortDescription,    // string
  description,         // string
  category,            // string - Event Type
  system,              // string - Game System
  rulesEdition,        // string
  minPlayers,          // number
  maxPlayers,          // number
  ageRequirement,      // string
  experienceRequired,  // string
  materialsRequired,   // boolean
  materialsDetails,    // string
  start,               // Date
  end,                 // Date
  durationMinutes,     // number - converted from hours
  minPlayTime,         // number - in minutes, converted from hours
  gm,                  // string
  website,             // string
  isTournament,        // boolean
  roundNumber,         // number | null
  totalRounds,         // number | null
  requiresRegistration,// boolean
  cost,                // number
  venue,               // string - Location (building)
  room,                // string
  tableNumber,         // string
  specialCategory,     // string
  ticketsAvailable,    // number
  lastModified,        // Date
  originalRow          // object - raw parsed row for debugging
}
```

## Parser Notes

- `Duration` in the XLSX is expressed in hours as a decimal string (e.g. `"2.0"`, `"0.5"`). Multiply by 60 to get minutes.
- `Start Date & Time` and `End Date & Time` format may shift between Gen Con export years. The parser should attempt multiple known formats and log a warning on failure rather than hard-crashing.
- `Game ID` is the canonical dedup key. On re-import, if a row's `Game ID` already exists in the dataset, update the record only if `Last Modified` is newer.
- `GM Names` may contain multiple names separated by commas. Store as a raw string; split only at display time.
- `Cost $` may be `"0"` or `"0.00"` for free events. Normalize to a float; treat 0 as free.
- `Tickets Available` of `0` means sold out. This is distinct from events with no registration requirement.

---

## Conflict Detection Specification

### Conflict Rules

Two events conflict if:

- times overlap, OR
- travel buffer overlap exists

Example:

- Event A ends at 2:00 PM at Venue A
- Event B starts at 2:10 PM at Venue B
- With a 15-minute travel buffer: **CONFLICT**

---

## Calendar Requirements

### Views

Required:

- multi-day convention view
- day view
- agenda/list view

Optional (future):

- heatmap mode
- compact mobile mode

### Calendar Features

Must support:

- click-to-add
- remove from schedule
- drag/drop repositioning
- hover tooltips
- color coding by category/system

---

## Storage Schema

Use versioned storage keys. Example:

```
localStorage["gencon_scheduler_v1"]
```

Include migration support hooks.

---

## Error Handling

Application must gracefully handle:

- malformed XLSX files
- missing columns
- duplicate imports
- invalid dates
- corrupt localStorage

No hard crashes allowed.

---

## Accessibility

Minimum requirements:

- keyboard navigation
- readable contrast
- semantic HTML
- ARIA labels where appropriate

---

## Browser Support

Primary:

- Chrome
- Edge

Secondary:

- Firefox
- Safari (optional)

---

## Security Constraints

Application must:

- never upload user data
- never require accounts
- never transmit schedule information externally

Offline-first is mandatory.

---

# Future Expansion Hooks

Architect for future support of:

- shared schedules
- cloud sync
- badge integration
- ticket purchasing workflows
- recommendation AI
- route optimization
- mobile PWA packaging

Do NOT tightly couple systems in ways that block future expansion.

---

# Development Priorities

Build order:

1. XLSX import
2. Data normalization
3. Calendar rendering
4. Conflict engine
5. Scheduling interactions
6. Persistence
7. Recommendations
8. Exporting
9. UI polish

---

# Suggested File Structure (During Development)

```
/project
  index.html
  /js
    app.js
    parser.js
    scheduler.js
    recommendations.js
    exports.js
    storage.js
    ui.js
  /css
    styles.css
```

Final deliverable: merged into a single-file HTML.

---

# Code Quality Requirements

Code must:

- be modular
- avoid global pollution
- use clear naming
- include comments for core systems
- avoid unnecessary abstractions

Favor:

- maintainability
- readability
- deterministic behavior

---

# Recommended Internal Modules

### Parser Module

Responsible for:

- XLSX import
- normalization
- schema validation

### Scheduler Module

Responsible for:

- overlap detection
- buffer calculations
- conflict resolution
- alternative suggestions

### Recommendation Module

Responsible for:

- free-time analysis
- best-fit suggestions
- category matching

### Export Module

Responsible for:

- CSV
- iCal
- printable output

### Storage Module

Responsible for:

- local persistence
- migrations
- backups

---

# Recommendation Scoring

Potential scoring factors:

- favorite category match
- available seats
- schedule fit quality
- travel efficiency
- duration fit
- conflict risk
- fatigue score

Keep scoring modular and tunable.

---

# Final Product Goal

The finished application should feel like:

> "Google Calendar + Gen Con planner + intelligent scheduling assistant"

while remaining lightweight, offline, portable, fast, and maintainable.
