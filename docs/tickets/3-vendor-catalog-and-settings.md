## 📖 Story / Why
Vendors need to set up their court — menu and settings — so students have something real to browse and order. This is the vendor-facing half of the catalog, on the web dashboard.

## 🧭 Context
React + TypeScript vendor dashboard + Firestore + Storage. Vendor role comes from ticket 2. A vendor may only touch **their own court** (`courts/{courtId}.ownerUid == uid`). Data shapes in `docs/DATA_MODEL.md` §2 (`courts`, `courts/*/menuItems`). Menu/settings edits are safe direct client writes guarded by security rules (no money/state logic) — see `docs/FUNCTIONS.md` D-note.

## 🔑 Access & prerequisites
- A provisioned **vendor** account (via `setUserRole` from ticket 2) linked to a test court.
- Firebase **Storage** bucket for menu images.

## ✅ Scope / What to build
- [ ] Menu item CRUD: name, category, price (**integer paise**), image upload to Storage, `isVeg`, `isAvailable`, `sortOrder`.
- [ ] Court settings: `operatingHours` (per weekday, campus time), `isOpen` toggle, `acceptedPaymentMethods` (subset of `["upi","cod"]`), `minOrderValue` (paise).
- [ ] All screens scoped to the signed-in vendor's own court; security rules enforce ownership.
- [ ] Loading / empty / error / disabled states on every data screen.

## 🎯 Acceptance Criteria
- [ ] Vendor logs into the dashboard and creates, edits, and deletes menu items, including image upload.
- [ ] Vendor toggles item availability and it reflects in Firestore.
- [ ] Vendor sets operating hours, open/closed, accepted payment methods, and minimum order value.
- [ ] A vendor cannot read/write another vendor's court (verified against rules).
- [ ] All prices are stored and handled as integer paise (no floats).

## 🖼️ UI standards
This is a UI ticket — apply `templates/ui-standards.md` from the plugin. Load-bearing here: **light + dark themes**; **responsive with desktop reflow + sensible min width**; **correct keyboard types** (number pad for price / min-order); **correct truncation**; **loading / empty / error / disabled states**; **no hardcoded user-facing strings**; design-system tokens, no one-off styles. Match the provided design exactly (ask the PM for design assets at `/start-ticket` if not attached).

## 🚫 Out of scope
- Student-facing browsing (ticket 4). Orders / order queue (Phase 3).

## 🔗 Dependencies
- Tickets 1 and 2.

## 📚 References
- `docs/DATA_MODEL.md` §2, PRD §5.2 (vendor dashboard), `docs/FUNCTIONS.md` (D-note on direct writes).

## 🤖 Kickoff prompt (paste into Claude Code)
```
/start-ticket <this-issue-number>
```
