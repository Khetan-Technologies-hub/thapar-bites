**Brief:** #5

## 📖 Story / Why
Close the loop for the student: a live status timeline, a push notification on every change so they don't have to keep checking, and order history with reorder. This is what makes the order feel *tracked* rather than sent into a void.

## 🧭 Context
Student app + web, Firestore listeners (surfaced as Flows from shared repositories), and FCM. Status transitions come from 5a/5c; `onOrderStatusChange` fans out notifications (`docs/FUNCTIONS.md`, `docs/ARCHITECTURE.md` §6). FCM tokens stored per user in `users/{uid}`.

## 🔑 Access & prerequisites
- Order core (5a), checkout (5b), and vendor queue (5c) built so real transitions exist.
- FCM configured (from ticket #1).

## ✅ Scope / What to build
- [ ] Live order status timeline on the student side (Placed → … → Delivered), updating in real time.
- [ ] `onOrderStatusChange` trigger → FCM push to the student on each transition (and to the vendor on a new post-hold order).
- [ ] FCM token registration/refresh stored on `users/{uid}`.
- [ ] Order **history** list + **reorder** (re-creates a cart from a past order).
- [ ] Loading / empty / error states.

## 🎯 Acceptance Criteria
- [ ] A student watching an order sees each status change within seconds.
- [ ] The student receives a push notification on each transition; the vendor receives one on a new order.
- [ ] Order history lists past orders; reorder rebuilds the cart (subject to current availability / prices).
- [ ] UI is driven by shared ViewModels; no Firebase calls in the UI layer.

## 🖼️ UI standards
UI ticket — apply `templates/ui-standards.md`. Load-bearing: light + dark; edge-to-edge + safe-area insets; responsive; loading/empty/error; accessibility; no hardcoded strings; notifications respect the platform's permission flow. Match the provided design exactly (ask the PM for assets at `/start-ticket`).

## 🚫 Out of scope
- Live GPS tracking (deferred, PRD §7 D4). Ratings (deferred). Payment mechanics (5d).

## 🔗 Dependencies
- Tickets 5a, 5b, 5c.

## 📚 References
- `docs/ARCHITECTURE.md` §6, `docs/FUNCTIONS.md` (`onOrderStatusChange`), PRD §5.1, Brief #5.

## 🤖 Kickoff prompt
```
/start-ticket <this-issue-number>
```
