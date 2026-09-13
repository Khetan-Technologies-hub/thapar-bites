**Brief:** #5

## 📖 Story / Why
The student side of placing an order: build a cart, check out, and get a **30-second grace window** to cancel or add more items before the order reaches the vendor. This is the friction-defining part of the product — it must be fast and reassuring.

## 🧭 Context
Student app (Compose Multiplatform) + student web, driven by shared ViewModels calling the order core (5a). One court per cart. Prices are integer paise → format to ₹. The **30s hold** is modelled as a pre-dispatch state so a closed app doesn't skip it (Brief #5 `[preference]`; final mechanism agreed with the Manager).

## 🔑 Access & prerequisites
- Order core (5a) and student browsing (#4) built.
- A student account and a set-up court with a menu.

## ✅ Scope / What to build
- [ ] Cart scoped to one court; add/remove/qty; live subtotal; **min-order-value gate** with a clear "below ₹X for this court" message.
- [ ] Checkout: pick a saved hostel address, choose a payment method from the **court's accepted set** (COD path live here; UPI wired in 5d).
- [ ] On confirm → order enters the **`Held`** state with a visible **30-second countdown**, during which the student can **Cancel** or **Add more items** (re-opens the cart, recomputes total server-side).
- [ ] When the timer expires → order **dispatches** to the vendor (becomes visible/accept-able). Cancel in-window releases the order (and triggers refund in 5d for paid orders).
- [ ] Loading / empty / error / disabled states throughout.

## 🎯 Acceptance Criteria
- [ ] A student builds a cart and cannot check out below the court's minimum.
- [ ] After confirming, a 30s countdown shows; Cancel voids the order; Add-more-items updates the order and total; doing nothing dispatches it to the vendor at 0s.
- [ ] The order is not visible to the vendor during the hold.
- [ ] Totals shown always match the server's computed total (paise → ₹).

## 🖼️ UI standards
UI ticket — apply `templates/ui-standards.md`. Load-bearing: light + dark; edge-to-edge + safe-area insets; responsive (phone→tablet, both orientations); correct keyboard types; truncation; loading/empty/error/disabled; accessibility; no hardcoded strings; state preserved across config change (don't lose the cart/timer). Match the provided design exactly (ask the PM for assets at `/start-ticket`).

## 🚫 Out of scope
- Razorpay online payment + refunds (5d). Vendor queue (5c). Post-dispatch tracking (5e).

## 🔗 Dependencies
- Tickets 5a and #4.

## 📚 References
- PRD §5.1, `docs/ARCHITECTURE.md` §5, Brief #5 (30s hold), `templates/ui-standards.md`.

## 🤖 Kickoff prompt
```
/start-ticket <this-issue-number>
```
