# Gen Con Event Planner

A fully offline, single-file browser application for importing Gen Con event spreadsheets, building a personal schedule, and exporting it in multiple formats.

No installation. No server. No account. Open `index.html` in Chrome or Edge and go.

---

## Features

### Import & Browse
- Drag-and-drop or file-picker import of the official Gen Con XLSX export
- Keyword search across titles, descriptions, GMs, and game systems
- Filter by event type, game system, day, cost, venue, duration, and ticket availability
- Game System filter shows live event counts that reflect all other active filters
- Sort by start time, duration, cost, or title
- Multi-session indicator — events with multiple time slots are linked together

### Scheduling
- Three calendar views: **Convention** (all days), **Day** (single day), and **Agenda** (paginated list)
- Click any event to add it to your schedule; click again to remove
- Drag scheduled events to a different time slot to reschedule
- Conflict detection prevents overlapping events
- Venue-aware travel buffers (configurable: 15 min, 30 min, or custom)
- Overnight events render across two calendar days and remain one logical event
- Up to 20-level undo (Ctrl+Z or toolbar button)

### Recommendations
- Analyzes your scheduled events and favorites to infer preferred categories and game systems
- Suggests unscheduled events that fit open time windows in your schedule
- Deprioritizes events that would cause travel conflicts or back-to-back fatigue

### Favorites & Wishlist
- Star any event to add it to your favorites/wishlist
- Favorites persist across sessions and inform recommendations

### Export
- **CSV** — spreadsheet-compatible export of your schedule
- **Google Sheets CSV** — cleaned CSV optimized for direct Google Sheets import
- **iCal (.ics)** — standards-compliant calendar file for Google Calendar, Apple Calendar, and Outlook
- **Printable itinerary** — day-grouped, print-ready layout with page-break-safe formatting

### Persistence
- All data (imported events, schedule, favorites, settings) stored in `localStorage`
- LZ-String compression used to maximize storage within the ~5 MB browser limit
- Graceful fallback if quota is exceeded — user data is always preserved

---

## Usage

1. Download the Gen Con event spreadsheet (XLSX) from the Gen Con event registration site
2. Open `index.html` in Chrome or Edge
3. Drag the XLSX file onto the import area, or use the file picker
4. Browse and filter the ~24,000 events using the left sidebar
5. Click events to add them to your schedule; the calendar updates in real time
6. Use the right panel to view event details, see conflicts, and get recommendations
7. Export your final schedule from the right panel or toolbar

To reset all data during testing:
```js
localStorage.removeItem("gencon_scheduler_v1");
location.reload();
```

---

## Technical Details

### Architecture

Single-file application — all CSS, HTML, and JavaScript are embedded in `index.html`. No build step, no package manager, no server.

**Internal modules:**

| Module | Responsibility |
|--------|---------------|
| `Parser` | XLSX import, column normalization, deduplication, venue name normalization |
| `Storage` | localStorage read/write with LZ-String compression and migration hooks |
| `Scheduler` | Overlap detection, travel buffer math, conflict resolution, undo stack |
| `Recs` | Free-time analysis, preference weighting, recommendation scoring |
| `Exports` | CSV, iCal, and printable schedule generation |
| `CalRender` | Custom DOM-based calendar renderer (day columns, event blocks, drag/drop) |
| `App` | Top-level state, event wiring, filter pipeline, view coordination |

### Dependencies (CDN)

All dependencies load from CDN at startup. The app works offline after the first load if the browser has cached them.

| Library | Version | Purpose |
|---------|---------|---------|
| [SheetJS](https://sheetjs.com) | 0.20.3 | XLSX parsing |
| [Luxon](https://moment.github.io/luxon/) | 3.4.4 | Timezone-aware datetime handling |
| [LZ-String](https://pieroxy.net/blog/pages/lz-string/index.html) | 1.5.0 | localStorage compression |

### XLSX Schema

The app expects the official Gen Con event export format (32 columns, A–AF):

| Column | Field | Notes |
|--------|-------|-------|
| A | Game ID | Unique dedup key |
| B | Group | Event series |
| C | Title | Display name |
| D | Short Description | |
| E | Long Description | |
| F | Event Type | Category (BGM, RPG, SEM, etc.) |
| G | Game System | |
| H | Rules Edition | |
| I | Minimum Players | |
| J | Maximum Players | |
| K | Age Required | |
| L | Experience Required | |
| M | Materials Required | Yes/No |
| N | Materials Required Details | |
| O | Start Date & Time | `MM/DD/YYYY HH:MM AM/PM` |
| P | Duration | Numeric hours — converted to minutes |
| Q | End Date & Time | `MM/DD/YYYY HH:MM AM/PM` |
| R | GM Names | |
| S | Website | |
| T | Email | Not stored (privacy) |
| U | Tournament? | Yes/No |
| V | Round Number | |
| W | Total Rounds | |
| X | Minimum Play Time | Numeric hours |
| Y | Attendee Registration? | |
| Z | Cost $ | Numeric |
| AA | Location (venue) | Building name |
| AB | Room Name | |
| AC | Table Number | |
| AD | Special Category | |
| AE | Tickets Available | 0 = sold out |
| AF | Last Modified | Excel serial date |

All times are parsed as `America/Indiana/Indianapolis` (UTC−4 during convention).

### Known Constraints

- **Desktop-first** — layout assumes a wide screen; not optimized for mobile
- **Browse cap** — the calendar browse view renders at most 150 events per day column to maintain performance; the full dataset is always available in Agenda and filter views
- **localStorage limit** — approximately 5 MB per origin; very large datasets may require re-importing the XLSX each session if compression is insufficient
- **Internet required on first load** — CDN dependencies must be fetched once; subsequent offline use works if the browser cached them

---

## Convention Context

Built for **Gen Con**, the largest tabletop gaming convention in North America, held annually in Indianapolis, Indiana. The 2026 convention runs **July 29 – August 2** at the Indiana Convention Center and surrounding hotels.

The official event catalog contains approximately 24,000 events across 19 event types (board games, RPGs, seminars, workshops, tournaments, LARPs, and more) spread across multiple venues within walking distance of each other.

---

## Browser Support

**Primary:** Chrome, Edge (Chromium-based)  
**Secondary:** Firefox  
**Not tested:** Safari

---

## License

This project is provided as-is for personal convention planning use.
