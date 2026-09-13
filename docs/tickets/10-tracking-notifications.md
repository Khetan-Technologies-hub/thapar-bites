**Brief:** #5

## 📖 Story / Why
Close the loop for the student: a live status timeline, a push notification on every change so they don't have to keep checking, and order history with reorder. This is what makes the order feel *tracked* rather than sent into a void.

## 🧭 Context
Student app (Android + iOS via Compose MP) + web, Firestore listeners (surfaced as Flows from shared repositories), and FCM. Status transitions come from 5a/5c; `onOrderStatusChange` fans out notifications (`docs/FUNCTIONS.md`, `docs/ARCHITECTURE.md` §6). FCM tokens stored per user in `users/{uid}`.

**Multiplatform note:** the live status timeline (Firestore listeners → Flows) is fully shared. **Push delivery is platform-specific** and must be set up per target: Android → FCM directly; **iOS → APNs via FCM** (needs the APNs key + Push Notifications capability in Xcode, done on the Mac); web → FCM Web Push with a service worker. Token registration/refresh is wired via `expect/actual`; the `onOrderStatusChange` fan-out is shared/server.

## 🔑 Access & prerequisites
- Order core (5a), checkout (5b), and vendor queue (5c) built so real transitions exist.
- FCM configured (from ticket #1).

## ✅ Scope / What to build
- [ ] Live order status timeline on the student side (Placed → … → Delivered), updating in real time.
- [ ] `onOrderStatusChange` trigger → FCM push to the student on each transition (and to the vendor on a new post-hold order).
- [ ] **Rejection / auto-cancel are clearly communicated:** on `Rejected` or vendor-didn't-accept auto-cancel, notify the student with a clear message that includes **refund status** ("order rejected — refund on its way") for paid orders. A refund must never be silent.
- [ ] FCM token registration/refresh stored on `users/{uid}`.
- [ ] Order **history** list + **reorder** (re-creates a cart from a past order).
- [ ] Loading / empty / error states.

## 🎯 Acceptance Criteria
- [ ] A student watching an order sees each status change within seconds.
- [ ] The student receives a push notification on each transition on **Android, iOS, and web**; the vendor receives one on a new order.
- [ ] On rejection or auto-cancel of a paid order, the student sees a clear message including refund status (no silent refunds).
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
