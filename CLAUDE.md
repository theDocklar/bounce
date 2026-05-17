# BOUNCE — Wireframe Upgrade Guide

## Project Context

Two self-contained React 18 wireframe prototypes (no build step — React/Babel loaded via CDN):

| File | Role |
|---|---|
| `index (3).html` | Customer-facing mobile app (375 × 812 phone frame) |
| `dashboard (1).html` | Admin dashboard (desktop shell) |

Both use inline `<style>` blocks and a single `<script type="text/babel">` block.
All mock data is defined as `const` arrays at the top of the script.

**Do not introduce a build system, npm, or external files.** Every upgrade stays inside the existing single-file HTML pattern.

---

## Court & Sport Model (Updated)

### Previous assumption (wrong)
- Pickleball and Badminton share courts with a 30-min buffer between them.

### Correct model
```
Court A  ──► Pickleball  OR  Badminton   (no buffer — clean swap, tracked in admin)
Court B  ──► Pickleball  OR  Badminton   (no buffer — clean swap, tracked in admin)
Field    ──► Cricket     OR  Futsal      (no buffer — clean swap, tracked in admin)
Kids Zone ──► Kids Play Area             (dedicated, no overlap)
Recovery  ──► Recovery Area             (dedicated, unlocked by court booking)
```

**Tidy-up logic (not a buffer — a UI-only signal):**
- If the outgoing sport on a court ≠ the incoming sport, flag that slot header in admin as "⚑ Sport switch: [Old] → [New] — check equipment".
- No slots are blocked or greyed for this. It is purely an admin visual alert.
- In the customer app, no changeover info is shown. Slots are simply available or booked.

**Remove all buffer-zone slot states** (`buffer` slot type) from both files.

---

## Feature Upgrades — Customer App (`index (3).html`)

### 1 · Player Profiles

**Post-registration screen: `profilesetup`**
- Fields: Display Name, Skill Level (Beginner / Intermediate / Advanced), Preferred Sports (checkbox: Pickleball, Badminton, Cricket, Futsal), Availability (multi-select day chips: Mon–Sun).
- Save → navigate to `home`.
- Profile data stored in top-level App state: `playerProfile = { name, skillLevel, sports[], availability[] }`.

**Profile screen updates:**
- Show skill level badge and preferred sports under the avatar.
- Add "Edit Profile" nav item that opens `profilesetup` pre-populated.

---

### 2 · Player Matching

**New screens:**

**`matchhub`** — entry point from bottom nav (replace Packages tab or add 5th tab icon 🤝)
- Two cards: "Find a Player" and "My Match Requests".
- If user has an upcoming Play Date, show a banner: "Your play date is [date] — Kids Zone auto-added ✓".

**`playerfind`** — browse/filter player cards
- Filter chips: Sport, Skill Level.
- Player card shows: avatar initials, name, skill badge, preferred sports icons, availability days.
- "Request Match" button on each card → sets pending request in `matchRequests` state.
- Mock player list: 6 players with varied profiles (use Sri Lankan names consistent with existing mock data).

**`matchrequests`** — inbox
- Two tabs: Received / Sent.
- Each request card: player info, sport, proposed date chip row (3 date options), Accept / Decline buttons.
- Accepting a request → creates a Play Date entry in `playDates` state.

**`playdates`** — scheduled play sessions
- List of confirmed play dates.
- Each card shows: player names, sport, court, date/time.
- "Kids Play Auto-Added" badge if either player has kids (mock: assume yes for demo).
- Kids Zone slot is shown as a linked add-on — tapping it navigates to `kids` booking flow.

**State to add at App level:**
```js
const [matchRequests, setMatchRequests] = useState(MATCH_REQUESTS_INIT);
const [playDates, setPlayDates] = useState([]);
```

**Mock data `MATCH_REQUESTS_INIT`:** 2 incoming requests, 1 sent request.

---

### 3 · Court & Sport Selection (Updated Flow)

**Sport selection screen (`home`):**
Replace current sport grid with:
```
Pickleball    Badminton
Cricket       Futsal        ← NEW
Kids Zone
```

**Court selection screen (`court`):**
- For Pickleball / Badminton: show Court A and Court B (rename from Court 1/2).
- For Cricket / Futsal: skip court selection (only one Field) — go straight to `slots`.
- For Kids Zone: go straight to `kids` screen.
- Remove the "30-min changeover" notice.

**Slot selection screen (`slots`):**
- Remove all `buffer` slot state and the buffer legend.
- Remove the buffer info banner.
- Slot states: `available` (white circle) and `booked` (grey filled).
- Hotspot slots: new state `hotspot` — rendered as a gold/amber outlined circle with a 🔥 label and the current bid floor.
  - Tapping a hotspot slot navigates to `bidflow` instead of adding to selection.

---

### 4 · Hotspot Bidding

**New screens:**

**`bidflow`** — bid on a specific hotspot slot
- Shows: sport, court, date, time slot, current highest bid (mock: LKR 3,500), minimum increment (LKR 500).
- Stepper to select bid amount (starts at current bid + increment).
- "Place Bid" button → navigate to `bidconfirm`.
- "Book at Standard Rate" secondary button (only if slot has standard availability too — not applicable for pure hotspot).

**`bidconfirm`** — bid placed confirmation
- Shows bid reference, slot details, amount bid, "You'll be notified if outbid" note.
- Buttons: "View My Bids" → `mybids`, "Back to Home".

**`mybids`** — my active bids (accessible from bottom nav or Home)
- List of active bids: sport, slot, your bid, current highest, status (Winning / Outbid).
- Outbid rows highlighted with ⚠️ and "Raise Bid" action.

**State to add:**
```js
const [myBids, setMyBids] = useState([]);
```

**Mock hotspot slots:** Mark hours 18, 19, 20 as `hotspot` in Court A Pickleball slots (peak evening).

---

### 5 · Recovery Area

**Logic rules (enforce in UI):**
- Only unlocked when: booking duration ≥ 2 hours (2+ slots selected) AND player count ≥ 4.
- Capacity: minimum 4, maximum 6 people.
- If conditions not met, show the recovery add-on as greyed out with a tooltip: "Select 2+ slots and 4+ players to unlock Recovery".

**In `BookingSummaryScreen`:**
- Add a Recovery Area toggle row below the Kids Zone toggle.
- Recovery fee: LKR 1,000 flat (mock rate).
- Toggle disabled and dimmed when `sorted.length < 2 || playerCount < 4`.
- When enabled, show: "Recovery Area · 1 session · LKR 1,000 · max 6 people".
- Add `recoveryAdded` to state and include in cart item and price breakdown.

**New standalone screen `recovery`:**
- Accessible from bottom nav / Home quick-links.
- Shows what the recovery area includes (mock: ice bath, stretching zone, massage chairs — placeholder text).
- "Book as Add-on" button → navigates to `home` to start a qualifying court booking.
- Shows current availability (mock: 3 of 4 sessions available today).

---

### 6 · Kids Play Area — Standalone + Add-on

**Existing:** Kids Zone already exists as a standalone screen (`kids`) and as an add-on toggle in `BookingSummaryScreen`.

**Changes needed:**

In `PackagesScreen`, add a dedicated Kids Zone package card:
```
🧒 Kids Zone — Standalone
Full session booking for children, no adult court required.
LKR 1,500 / hour · Ages 3–12
[Book Now]  ← navigates to `kids` screen
```

In `kids` screen, add a package upsell card at the bottom:
```
💡 Booking a court too? Add Kids Zone at 50% off (LKR 750) during checkout.
```

In `HomeScreen`, make Kids Zone sport card also show "From LKR 750 with court" sub-label.

---

### 7 · Packages Screen (Full Listing)

Replace current 3-item list with these 5 packages:

| # | Name | Description | Price | CTA |
|---|---|---|---|---|
| 1 | Sports + Kids Combo | 50% off Kids Zone when booked with any court | LKR 3,750 (was 4,500) | Book Now |
| 2 | Off-Peak Special | 30% off before 12:00 PM | From LKR 2,100/hr | Book Now |
| 3 | Recovery Bundle | Any 2hr+ court booking + Recovery Area included | LKR 4,000 flat | Book Now |
| 4 | Kids Zone — Standalone | Full Kids Zone session, no court required | LKR 1,500/hr | Book Now |
| 5 | Bulk 5+1 Free | Book 5 sessions, 6th free | Coming Soon | Coming Soon |

Each card: icon, title, description, price, CTA button. "Book Now" navigates to `home` (booking entry). "Coming Soon" is disabled.

---

### 8 · Bottom Navigation Update

Current 4 tabs: Home, Bookings, Packages, Profile.

Update to 5 tabs:
```
🏠 Home   |   📅 Bookings   |   🤝 Match   |   🎁 Packages   |   👤 Profile
```

---

### 9 · Remove Buffer Zone Logic

Search for every reference to `buffer`, `BUFFER_SLOTS`, `isBuffer`, and the buffer note UI in `index (3).html` and delete them. Slot states are now only `available` and `booked` (plus new `hotspot`).

---

## Feature Upgrades — Admin Dashboard (`dashboard (1).html`)

### 1 · Court Management Screen

**New nav item:** `courts` — icon 🏟️, label "Court Management".

**Screen layout:**
- Three columns: Court A, Court B, Field.
- Each column is a day timeline (08:00–21:00 in 1-hr rows).
- Booked slots shown as filled blocks with customer name + sport icon.
- Sport-switch rows: if slot N is Pickleball and slot N+1 is Badminton on the same court, add a thin amber banner between them: "⚑ Sport switch: 🏓 → 🏸 — prep needed".
- No slots blocked — purely informational.

**Tidy-up logic function:**
```js
function getSportSwitches(bookings, courtId, date) {
  const sorted = bookings
    .filter(b => b.court === courtId && b.date === date && b.status === 'Confirmed')
    .sort((a, b) => parseInt(a.start) - parseInt(b.start));
  const switches = [];
  for (let i = 0; i < sorted.length - 1; i++) {
    if (sorted[i].sport !== sorted[i+1].sport && sorted[i].end === sorted[i+1].start) {
      switches.push({ at: sorted[i].end, from: sorted[i].sport, to: sorted[i+1].sport });
    }
  }
  return switches;
}
```

Add Futsal to `SPORTS` array and courts: `COURTS['Futsal'] = ['Field']`, `COURTS['Cricket'] = ['Field']`.
Update mock bookings to include at least one Futsal booking on the Field.

---

### 2 · Recovery Area Screen

**New nav item:** `recovery` — icon 💆, label "Recovery Area".

**Screen layout:**
- Stat row: sessions today, capacity used (e.g. "18 / 24 slots filled").
- Table of recovery sessions: Time, Linked Court Booking ID, Players, Status (Available / Full).
- Each row shows which court booking unlocked it.
- Filter: by date.

**Mock recovery data:** 4 sessions linked to 2hr+ bookings from `BOOKINGS_INIT`.

---

### 3 · Hotspot Bidding Screen

**New nav item:** `bids` — icon 🔥, label "Hotspot Bids".

**Two sections:**

**Designate Hotspot Slots:**
- Table: Court, Date, Hour, Current Floor Price, Action (Set as Hotspot / Remove).
- Toggle hotspot status per slot.

**Live Bid Board:**
- Table: Slot, Bidder, Bid Amount, Time Placed, Status (Active / Accepted / Outbid).
- "Accept" button on winning bid row → marks slot as booked for that bidder.
- Admin can override and accept any bid.

**Mock bid data:** 3 bids on Court A 18:00 slot (Apr 20).

---

### 4 · Player Profiles Screen

**New nav item:** `players` — icon 🤝, label "Players".

**Screen layout:**
- Table: Name, Phone, Skill Level, Preferred Sports, Play Dates, Joined Date, Status.
- "View" button opens detail modal: full profile, match history, upcoming play dates.
- Filter by sport and skill level.

**Mock player data:** 6 players consistent with existing customer-side mock.

---

### 5 · Kids Zone Management Screen

**New nav item:** `kids` — icon 🧒, label "Kids Zone".

**Screen layout:**
- Stat row: standalone bookings today, add-on bookings today.
- Tab bar: Standalone | Add-on.
- Each tab shows a table of bookings: Customer, Date, Time, Kids Count, Linked Court (if add-on), Status.

---

### 6 · Dashboard Home Updates

Add to stat grid (expand to 2 rows of 4):
- Active Bids (new)
- Recovery Sessions Today (new)

Add sport-switch alert list below the timeline:
- "⚑ 3 sport switches today — check Court A at 14:00 and 16:00, Court B at 12:00"

---

### 7 · Updated Mock Data

**Add to `SPORTS`:** `'Futsal'`

**Update `COURTS`:**
```js
const COURTS = {
  Badminton:       ['Court A', 'Court B'],
  Pickleball:      ['Court A', 'Court B'],
  Cricket:         ['Field'],
  Futsal:          ['Field'],
  'Kids Play Area':['Kids Zone'],
  'Recovery Area': ['Recovery Room'],
};
```

**Rename** all existing `'Court 1'` → `'Court A'`, `'Court 2'` → `'Court B'` in `BOOKINGS_INIT`.

**Add mock bookings** for Futsal on Field and at least one sport-switch scenario on Court A.

**Add `MOCK_BIDS`:**
```js
const MOCK_BIDS = [
  { id:'BID-001', court:'Court A', date:'2026-04-20', hour:18, bidder:'Amal Gunaratne',    phone:'+94 71 789 0123', amount:4000, placed:'17:32', status:'Active'   },
  { id:'BID-002', court:'Court A', date:'2026-04-20', hour:18, bidder:'Priya Mendis',       phone:'+94 76 890 1234', amount:3500, placed:'17:18', status:'Outbid'   },
  { id:'BID-003', court:'Court A', date:'2026-04-20', hour:19, bidder:'Sandya Kumari',      phone:'+94 70 012 3456', amount:3500, placed:'16:55', status:'Active'   },
];
```

---

## Navigation Map (Updated)

### Customer App — Screens

```
splash → login → register → profilesetup → home
home → court → slots → (hotspot slot?) bidflow → bidconfirm
                slots → summary → cart → payment → confirm
home → kids → payment → confirm
home → recovery (info screen)
home → matchhub → playerfind → (request sent)
                → matchrequests → (accept) → playdates
home → mybookings
home → mybids
home → packages
home → tournaments
home → profile → profilesetup
```

### Admin Dashboard — Nav Items (in order)

```
dashboard | bookings | manual | courts | recovery | bids | players | kids | packages | admins
```

---

## Implementation Order

Work through files in this sequence to avoid breaking the existing prototype at each step:

### Phase 1 — Data & Court Model (both files)
1. Remove `buffer` slot type everywhere in `index (3).html`.
2. Add Futsal to `SPORTS` and `COURTS` in both files.
3. Rename Court 1/2 → Court A/B in both files.
4. Add `MOCK_BIDS` and recovery mock data to `dashboard (1).html`.
5. Update `COURTS` object in `dashboard (1).html`.

### Phase 2 — Admin New Screens
6. Add Court Management screen with sport-switch logic.
7. Add Recovery Area screen.
8. Add Hotspot Bidding screen.
9. Add Player Profiles screen.
10. Add Kids Zone Management screen.
11. Update Dashboard stats and add sport-switch alerts.
12. Update nav to include all new items.

### Phase 3 — Customer App Core Updates
13. Update `SlotSelectionScreen` — remove buffer, add `hotspot` slot state.
14. Update `CourtSelectionScreen` — rename courts, remove changeover notice, add Futsal path.
15. Update `HomeScreen` sport grid — add Futsal, update Kids Zone sub-label.
16. Update `BookingSummaryScreen` — add Recovery add-on with unlock logic, add `recoveryAdded` to state and cart.
17. Update `PackagesScreen` — full 5-package listing.
18. Update `KidsZoneScreen` — add combo upsell card.
19. Update bottom nav — add Match tab.

### Phase 4 — New Customer Screens
20. Add `profilesetup` screen and `playerProfile` state.
21. Add `matchhub`, `playerfind`, `matchrequests`, `playdates` screens.
22. Add `bidflow`, `bidconfirm`, `mybids` screens.
23. Add `recovery` standalone info screen.
24. Wire all new screens into `renderScreen()` switch and `SCREENS` list.
25. Update `SCREEN_FEATURES` with descriptions for all new screens.

---

## Key Constraints

- **No npm / no build** — all code stays in the single HTML file per prototype.
- **React state only** — no localStorage, no fetch calls.
- **Monospace / greyscale design language** — match existing CSS conventions exactly. No colour palette changes. Hotspot amber = `#b8860b` border, `#fffbf0` background (one-off).
- **Sri Lankan context** — currency is LKR, phone prefix `+94`, names from existing mock data set.
- **Booking ID format** — `BNC-YYYY-NNN` for bookings, `BID-NNN` for bids.
- **Keep files self-contained** — do not split into separate JS/CSS files.
