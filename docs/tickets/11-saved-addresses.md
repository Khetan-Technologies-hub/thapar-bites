## 📖 Story / Why
A student can't have food delivered without saying **where**. This builds saved hostel delivery addresses — a prerequisite for checkout (#7). Surfaced by the Gate 1 review of brief #5: address management was deferred out of the auth ticket and never re-ticketed.

## 🧭 Context
Student app (Compose Multiplatform) + web, shared ViewModels + repositories. Addresses live on `users/{uid}.addresses[]` per `docs/DATA_MODEL.md` §2 (hostel/block + room + notes + isDefault). Campus delivery only (hostels).

## 🔑 Access & prerequisites
- Auth & roles (#2) built; a student account.

## ✅ Scope / What to build
- [ ] List, add, edit, delete saved hostel addresses (hostel/block, room, optional notes).
- [ ] Mark one as default; default is preselected at checkout.
- [ ] Validation (hostel + room required); write only to the student's own `users` doc (rules).
- [ ] Loading / empty / error states.

## 🎯 Acceptance Criteria
- [ ] A student adds, edits, deletes, and sets a default hostel address.
- [ ] The default address is preselected on the #7 checkout screen.
- [ ] A student cannot read or write another user's addresses (verified against rules).

## 🖼️ UI standards
UI ticket — apply `templates/ui-standards.md`. Load-bearing: light + dark; safe-area insets; responsive; correct keyboard types; loading/empty/error; accessibility; no hardcoded strings. Match the provided design (ask the PM at `/start-ticket`).

## 🚫 Out of scope
- Map/geolocation, address outside campus, cart/checkout itself (#7).

## 🔗 Dependencies
- Ticket #2.

## 📚 References
- `docs/DATA_MODEL.md` §2 (`users.addresses`), PRD §5.1, Brief #5 (checkout needs an address).

## 🤖 Kickoff prompt
```
/start-ticket <this-issue-number>
```
