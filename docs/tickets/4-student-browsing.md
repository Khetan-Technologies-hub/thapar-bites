## 📖 Story / Why
Students need to discover food courts and view their menus — the read side of the ordering loop, and the first thing a student actually sees. No ordering yet; this proves the catalog end-to-end from the student's device.

## 🧭 Context
Student app (Compose Multiplatform) + student web. Reads `courts` and `courts/*/menuItems` through the shared repositories (the only Firebase boundary; UI/ViewModels never touch Firestore directly). Open/closed state comes from `isOpen` + `operatingHours` (campus time, Asia/Kolkata). Prices are integer paise — format to ₹ for display. Menu browsing should work from Firestore cache when offline.

## 🔑 Access & prerequisites
- A `student` account (ticket 2).
- At least one court with a menu set up (ticket 3), or seeded emulator data.

## ✅ Scope / What to build
- [ ] Courts list: name, image, and correct open/closed state + hours.
- [ ] Court detail: categorized menu with item name, price (₹, from paise), image, veg marker, and availability (unavailable items clearly marked).
- [ ] Loading / empty / error states; offline shows cached data.
- [ ] Read-only — no cart or ordering.

## 🎯 Acceptance Criteria
- [ ] Student sees a list of courts with correct open/closed status.
- [ ] Opening a court shows its categorized menu with correctly formatted prices and images.
- [ ] Unavailable items are shown as unavailable.
- [ ] Browsing works offline from cache; a clear state shows when there's no data / an error.
- [ ] UI is driven by a shared ViewModel; no Firebase calls in the UI layer.

## 🖼️ UI standards
This is a UI ticket — apply `templates/ui-standards.md` from the plugin. Load-bearing here: **light + dark themes**; **edge-to-edge + safe-area/notch/nav-bar insets**; **responsive across phone → tablet, both orientations**; **correct truncation**; **loading / empty / error states**; **accessibility** (labels, touch targets, dynamic type); **no hardcoded strings** (i18n); native components; design-system tokens. Match the provided design exactly (ask the PM for design assets at `/start-ticket` if not attached).

## 🚫 Out of scope
- Cart, checkout, payment, order placement/tracking (Phase 3).

## 🔗 Dependencies
- Tickets 1, 2, and 3.

## 📚 References
- PRD §5.1 (student app), `docs/DATA_MODEL.md` §2, `docs/ARCHITECTURE.md` (layering, real-time), `templates/ui-standards.md`.

## 🤖 Kickoff prompt (paste into Claude Code)
```
/start-ticket <this-issue-number>
```
