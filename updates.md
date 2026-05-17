I have a sports venue booking wireframe project in this folder. There are two single-file React 18 prototypes (no build step — React/Babel via CDN, all code inline):

- `index (3).html` — customer mobile app (phone frame, 375px wide)
- `dashboard (1).html` — admin dashboard (desktop sidebar layout)

**YOUR TASK: Merge both files into a single new file called `bounce.html` and implement 5 changes. Do NOT modify the original files.**

---

## ARCHITECTURE — Single File, Side-by-Side Layout

`bounce.html` renders both panels side by side in one browser window:


One single `ReactDOM.createRoot` call. One top-level `<SharedApp>` component owns all cross-cutting state and passes it down to both `<CustomerApp>` and `<AdminShell>` as props.

CSS: merge both `<style>` blocks. Prefix all admin-specific class names with `adm-` where they clash with customer app class names (e.g. `.adm-shell`, `.adm-sidebar`, `.adm-nav-item`, `.adm-topbar`, `.adm-page-content`). Customer app class names stay as-is.

The outer layout CSS:
```css
body { margin: 0; background: #1a1a1a; font-family: system-ui, sans-serif; }
.root-layout { display: flex; height: 100vh; overflow: hidden; }
.left-panel { width: 430px; flex-shrink: 0; display: flex; flex-direction: column; align-items: center; justify-content: flex-start; padding: 20px 16px; overflow-y: auto; background: #1a1a1a; }
.right-panel { flex: 1; display: flex; flex-direction: column; overflow: hidden; background: #fff; border-left: 2px solid #444; }
```

---

## SHARED STATE (lifted to SharedApp, passed as props to both sides)

```js
const [sharedBookings, setSharedBookings]       // customer confirms → admin sees new rows
const [sharedBids, setSharedBids]               // customer bids → admin bid board; admin accepts → customer sees status
const [hotspotConfig, setHotspotConfig]         // admin toggles → customer slot grid reads
const [equipmentConfig, setEquipmentConfig]     // admin edits prices/names → customer form reads
const [packages, setPackages]                   // admin toggles active/inactive → customer packages screen reads
const [sharedRecovery, setSharedRecovery]       // customer adds recovery → admin recovery table
```

Initialise `sharedBookings` from the existing `BOOKINGS_INIT` array (18 mock bookings). Initialise `sharedBids` from `MOCK_BIDS`. Initialise `sharedRecovery` from `MOCK_RECOVERY`.

For `hotspotConfig`, initialise as:
```js
[
  { court:'Court A', date:'2026-04-20', hour:18, floor:3000, isHotspot:true },
  { court:'Court A', date:'2026-04-20', hour:19, floor:3000, isHotspot:true },
  { court:'Court A', date:'2026-04-20', hour:20, floor:3000, isHotspot:true },
]
```

For `equipmentConfig`, initialise as:
```js
{
  pickleball: [
    { id:'paddle', name:'Paddle Set (×2)', price:300 },
    { id:'ball',   name:'Ball Pack',        price:200 },
  ],
  badminton: [
    { id:'racket',  name:'Racket Pair',      price:300 },
    { id:'shuttle', name:'Shuttlecock Pack', price:200 },
  ],
  cricket: [
    { id:'bat',    name:'Bat',              price:400 },
    { id:'ball',   name:'Ball',             price:200 },
    { id:'pads',   name:'Batting Pads',     price:300 },
    { id:'gloves', name:'Gloves',           price:200 },
  ],
  futsal: [
    { id:'ball', name:'Football',           price:300 },
    { id:'bibs', name:'Bibs / Vests',       price:200 },
  ],
}
```

---

## CHANGE 1 — New User Onboarding Screen

Add a new screen `onboarding` to the customer app.

**Flow:** After `profilesetup` saves, navigate to `onboarding` instead of `home`. The onboarding screen navigates to `home` when finished.

**Screen design:** A 3-step card walkthrough inside the phone frame.

- Progress dots at the top (3 dots, filled = current step)
- Step 1 — "Welcome to Bounce 👋": "Your all-in-one sports booking platform. Book courts, find players, and manage everything in one place."
- Step 2 — "Book a Court 🏓": "Pick your sport → choose a time slot → add it to your cart. Supports Pickleball, Badminton, Cricket, Futsal, and Kids Zone."
- Step 3 — "Combo Booking 🛒": "Add Kids Zone, Recovery Area, or Equipment directly during court checkout — all in one transaction."
- "Next" button advances steps. On step 3, button reads "Get Started" and navigates to `home`.
- "Skip" link in top-right navigates to `home` immediately.

**Profile screen:** Add a new menu row `{ icon:'📖', label:'How to Use the App', action: () => navigate('onboarding') }` above the Logout row.

Add `onboarding` to the `SCREENS` array and `SCREEN_FEATURES` object.

---

## CHANGE 2 — Cricket & Futsal: No Player Count Limit

In `BookingSummaryScreen`, the player count stepper currently clamps all sports to min 2, max 4.

Change it to be sport-aware:
- `pickleball` / `badminton`: min 2, max 4 (unchanged)
- `cricket`: min 1, no upper cap (practical max 22)
- `futsal`: min 1, no upper cap (practical max 10)

Implementation:
```js
const isFieldSport = sport === 'cricket' || sport === 'futsal';
const maxPlayers = sport === 'cricket' ? 22 : sport === 'futsal' ? 10 : 4;
const minPlayers = isFieldSport ? 1 : 2;

// stepper buttons:
onClick={() => setPlayerCount(Math.max(minPlayers, playerCount - 1))}
onClick={() => setPlayerCount(Math.min(maxPlayers, playerCount + 1))}
```

Below the stepper, add a small label:
- For cricket/futsal: `"No fixed limit — enter your team size"`
- For pickleball/badminton: `"2–4 players"`

---

## CHANGE 3 — Sport-Specific Equipment + Admin Editor

### Customer app — `BookingSummaryScreen`

Replace the single "Equipment Needed" toggle with a per-item checklist driven by `equipmentConfig[sport]`.

Each item gets its own toggle row showing item name and individual price. Selected items are tracked in `selectedEquipment` as an array of item IDs. Equipment total = sum of selected item prices.

In the Price Breakdown, list each selected equipment item on its own line with its price.

If the current sport has no equipment config entry (e.g. `kids`), show nothing for the equipment section.

### Admin dashboard — Equipment Config tab in Packages page

In `PackagesPage` (admin), add a second tab bar at the top of the page:
- Tab 1: "Packages" (existing packages grid)
- Tab 2: "Equipment Config"

**Equipment Config tab:**

Shows a table per sport (Pickleball, Badminton, Cricket, Futsal). Each sport section has a heading and a table:

| Item Name | Price (LKR) | Actions |
|---|---|---|
| Paddle Set (×2) | 300 | Edit |
| Ball Pack | 200 | Edit |

"Edit" opens an inline edit row (or small modal) with a text input for name and number input for price, and a Save button. Changes update `equipmentConfig` shared state — which the customer app reads live.

---

## CHANGE 4 — Remove Court A/B Selection for Pickleball & Badminton

In `HomeScreen`, the routing currently sends pickleball/badminton to `CourtSelectionScreen`. Change it so all 4 sports skip court selection and go straight to `slots`:

```js
const handleSport = s => {
  setSport(s.id);
  if (s.id === 'cricket' || s.id === 'futsal') { setCourt('Field'); navigate('slots'); return; }
  if (s.id === 'pickleball' || s.id === 'badminton') { setCourt('Auto-Assigned'); navigate('slots'); return; }
};
```

In `BookingSummaryScreen`, display `"Auto-Assigned"` for the Court row when sport is pickleball or badminton.

In `SlotSelectionScreen`, when `court === 'Auto-Assigned'`, use `COURTA_PB_SLOTS` as the slot data (default to Court A slots for display purposes).

The `CourtSelectionScreen` component stays in the file but is no longer routed to.

Update `SCREEN_FEATURES['court']` to note it is no longer in the standard flow.

---

## CHANGE 5 — Merge & Live Connection

Both apps share state as described above. Key live connections to wire:

**Customer → Admin:**
- When `ConfirmationScreen` fires `setBookings`, also call `setSharedBookings(prev => [...prev, ...newItems])`. The admin `BookingsPage`, `DashboardHome` timeline, and `CourtManagementPage` all read from `sharedBookings` instead of the static `BOOKINGS_INIT`.
- When customer places a bid in `BidFlowScreen`, also push to `sharedBids`. Admin `HotspotBidsPage` reads from `sharedBids`.
- When customer adds recovery (recoveryAdded=true on AddToCart), push a new entry to `sharedRecovery`. Admin `RecoveryAreaPage` reads `sharedRecovery`.

**Admin → Customer:**
- `hotspotConfig` (from admin `HotspotBidsPage` toggle) is read by `SlotSelectionScreen` to determine which slots show the 🔥 hotspot state. Merge hotspot config with base slot data at render time.
- `equipmentConfig` (from admin Equipment Config editor) is read by `BookingSummaryScreen`.
- `packages` (admin PackagesPage toggle active/inactive) is read by customer `PackagesScreen` — only show active packages.

**Admin bid acceptance → Customer:**
- When admin clicks "Accept" on a bid, update `sharedBids`. Customer `MyBidsScreen` reads `sharedBids` and shows the updated status.

---

## ADDITIONAL REQUIREMENTS

- Keep the Journey Tracker (left sidebar in dark panel) and Features Panel (below phone) from the original customer app.
- Admin panel must still have its Login page (`admin@bounce.com` / `bounce2024`). The admin login shows in the right panel only. The customer phone frame is always visible regardless of admin login state.
- All mock data constants go at the top of the script, deduplicated. Both apps share `BOOKINGS_INIT`, `MOCK_BIDS`, `MOCK_RECOVERY`, `MOCK_PLAYERS`, `MATCH_REQUESTS_INIT`, `PACKAGES_INIT`, `ADMINS_INIT`, `EQUIPMENT_CONFIG_INIT`, `HOTSPOT_CONFIG_INIT`.
- No npm, no build, no external files. Everything stays in one HTML file.
- Greyscale/monospace design language throughout. Hotspot amber = `#b8860b` border, `#fffbf0` bg — only exception to greyscale.
- Currency LKR, phone prefix +94, all names Sri Lankan.
- Save the output as `bounce.html` in the same folder as the two source files.