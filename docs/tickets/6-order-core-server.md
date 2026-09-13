**Brief:** #5

## 📖 Story / Why
The correctness backbone of the order loop: create orders on the server, own the money and the state machine, and let orders move forward legally. Everything else (student checkout, vendor queue, payments, tracking) consumes this. COD only in this ticket — Razorpay comes in 5d.

## 🧭 Context
Firebase Cloud Functions (TypeScript) + Firestore. Server is the source of truth for money and state (`CLAUDE.md`). Order lifecycle and shapes: `docs/ARCHITECTURE.md` §4/§5, `docs/DATA_MODEL.md` §2 (`orders`), `docs/FUNCTIONS.md` (`placeOrder`, `transitionOrder`). The order **state machine** is defined once in `shared/` and mirrored/enforced server-side.

## 🔑 Access & prerequisites
- Firebase project + emulators (ticket #1); auth & roles (ticket #2).
- No secrets in the repo.

## ✅ Scope / What to build
- [ ] `orders` collection + shared domain model, with the enums (`status`, `paymentMethod`, `paymentStatus`) defined once in `shared/`.
- [ ] Order **state machine** in `shared/` (`Held/Placed → Accepted → Preparing → OutForDelivery → Delivered`, plus `Rejected`/`Cancelled`).
- [ ] `placeOrder` callable (**COD path**): load court + menu (authoritative prices), reject if court closed/outside hours, reject if method not accepted, recompute subtotal/total, enforce `minOrderValue`, **snapshot prices**, write the order.
- [ ] `transitionOrder` callable: validate the requested transition against the state machine **and** the caller's role before writing; append `statusHistory`.
- [ ] Security rules: deny client writes to `total`/`subtotal`/`status`/`paymentStatus`; students read own orders, vendors read own court's orders.

## 🎯 Acceptance Criteria
- [ ] A COD order can be placed via `placeOrder` with server-computed totals (integer paise); a below-minimum or closed-court or wrong-method order is rejected.
- [ ] Illegal transitions are rejected (e.g. student marking own order `Delivered`, or `Placed → Delivered`).
- [ ] Prices on the order are snapshots — editing the menu afterward doesn't change a placed order.
- [ ] Rules (verified on the emulator) block direct client writes to authoritative fields.
- [ ] Unit tests cover the state machine and validation in `shared/`; function tests run against the emulator.

## 🚫 Out of scope
- Any UI (5b/5c). Razorpay/online payment + refunds (5d). Notifications/tracking (5e). The 30s hold UI (5b) — but the `Held` state may be included in the machine here.

## 🔗 Dependencies
- Tickets #1, #2.

## 📚 References
- `docs/FUNCTIONS.md`, `docs/DATA_MODEL.md` §2/§5, `docs/ARCHITECTURE.md` §4/§5, Brief #5.

## 🤖 Kickoff prompt
```
/start-ticket <this-issue-number>
```
