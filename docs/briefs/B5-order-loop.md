## 🎯 What we're building & why
The **order loop** — the heart of Thapar Bites. A student turns a browsed menu into a placed, paid order; the vendor sees it, prepares it, and delivers it; and both sides watch it move in real time. Without this, everything else is just a catalog. This brief covers order placement, payment, the vendor's order queue, live status, and the safety valves around cancellation and refunds.

## 👤 Who it's for
- **Students** — hungry, in a hostel, want food from a campus court with the least friction and full confidence their order and money are handled correctly.
- **Vendors** — running a busy counter; need new orders to arrive loudly and clearly, and to move each order forward with one tap.

## ✅ What it must do (capabilities)
- Build a cart from **one court**, see the live subtotal, and check out — choosing a payment method the court accepts (UPI via Razorpay, or COD).
- Enforce the court's **minimum order value** before checkout can complete.
- After checkout, a **30-second hold** before the order reaches the vendor, during which the student can **cancel** or **add more items** to the same order.
- Once the hold ends, the order **dispatches to the vendor**, who gets a loud new-order alert and can **accept or reject** it.
- Vendor advances the order: **Accepted → Preparing → Out for delivery → Delivered** (vendor/staff marks Delivered).
- Both sides see **live status** and get a **push notification** on each change.
- **Refunds happen automatically** (Razorpay) when a student cancels a paid order in the hold window, or when a vendor rejects a paid order.
- Student can see **order history** and **reorder**.

## 🌟 What "good" looks like
- A student can go from "I'm hungry" to "order placed" in well under a minute, and never wonder where their order or money is.
- A vendor never misses an order and never has to think about *how* to move it forward — the next action is always obvious.
- Nobody is ever charged for food they didn't get: cancellations and rejections refund without a human chasing it.

## 🚫 Non-negotiables
- **The server is the source of truth for money and state** — totals, minimum-order enforcement, allowed payment methods, and every status transition are validated/performed server-side. The client's cart is a request, never the final word.
- **All money is integer paise; never floats.**
- **A paid order that is cancelled (in the hold window) or rejected is refunded** — the student is never left out of pocket for an order that didn't proceed.
- **The 30-second hold is the only student-cancel window** — once dispatched to the vendor, the student cannot cancel (food may be cooking).
- **Payment method must be one the court accepts.**

## 🧭 Technical steers
- `[hard]` Order placement and status transitions go through **Cloud Functions**; clients cannot write `total`, `status`, or `paymentStatus` directly.
- `[hard]` Item prices are **snapshotted** onto the order at placement; later menu edits never change a placed order.
- `[hard]` Razorpay integration is **idempotent** and payment status is reconciled via a signature-verified **webhook**.
- `[preference]` Model the 30-second hold as a pre-dispatch order state (e.g. `Held`) rather than a client-only timer, so a closed app doesn't skip the hold. Manager to decide the exact mechanism.
- `[preference]` "Add more items" re-opens the same order and recomputes the total server-side; for UPI, decide whether the delta is re-charged or the order is re-authorized (Manager to resolve during planning).

## 🧊 Happy to defer
- Ratings/reviews, tipping, promotions/coupons, wallet, scheduled/pre-orders, live GPS tracking, dedicated delivery-staff logins. (All post-v1 per the PRD.)

## 📎 References
- `docs/PRD.md` §4.4 (order lifecycle), §5 (student app / vendor dashboard), §7 (open decisions D1 payment settlement — gates auto-refund).
- `docs/ARCHITECTURE.md` §4 (state machine), §5 (place-order critical path), §6 (real-time & notifications).
- `docs/DATA_MODEL.md` §2 (`orders`), `docs/FUNCTIONS.md` (`placeOrder`, `transitionOrder`, `razorpayWebhook`).
- Depends on tickets #1–#4 (scaffolding, auth, catalog) being built first.

---
**Non-negotiable open dependency:** auto-refund depends on **PRD Decision D1** (payment settlement / whether the platform can issue refunds on the vendor's Razorpay). Confirm D1 before the payment/refund ticket.
