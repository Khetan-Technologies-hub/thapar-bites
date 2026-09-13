**Brief:** #5

## 📖 Story / Why
Add online payment and make the money safe: UPI via Razorpay at checkout, reliable reconciliation via webhook, and **automatic refunds** when a paid order is cancelled in the hold window or rejected by the vendor. This is the highest-risk ticket — isolated on purpose.

## 🧭 Context
Razorpay + Cloud Functions, layered onto `placeOrder` (5a) and checkout (5b). Money is integer paise. Payments settle **vendor-direct** (PRD D1). Webhook is signature-verified and idempotent (`docs/FUNCTIONS.md` `razorpayWebhook`).

**Multiplatform note:** there is no official Kotlin Multiplatform Razorpay SDK. The order-creation call and webhook are shared/server; the **checkout step is platform-specific** and must be wired via `expect/actual` in `shared/`: Android → Razorpay Android SDK; iOS → Razorpay iOS SDK; web → Razorpay Checkout.js. Keep the shared repository interface identical across all three so ViewModels stay platform-agnostic.

⚠️ **Gated by PRD Decision D1** — confirm the settlement model and that the platform can issue **refunds** on the vendor's Razorpay (Route / vendor keys) before building the refund path.

## 🔑 Access & prerequisites
- Razorpay account in **test mode**; key/secret + webhook secret stored in **Firebase secrets** (never in the repo).
- Order core (5a) + checkout (5b) built.

## ✅ Scope / What to build
- [ ] UPI path in `placeOrder`: create a Razorpay order, write the order `paymentStatus: pending`, return the Razorpay handle to the client; client completes payment.
- [ ] `razorpayWebhook`: verify signature, idempotently reconcile `payment.captured` → `paid` (and dispatch after the hold) / `payment.failed` → `failed`.
- [ ] **Auto-refund** via a Cloud Function on: student cancel within the 30s hold (paid order), vendor rejection of a paid order, and **vendor-didn't-accept auto-cancel** (#6 expiry). Set `paymentStatus` accordingly.
- [ ] Handle the paid **"add more items"** case from 5b: recompute server-side and charge/authorize the delta (mechanism agreed with the Manager).

## 🎯 Acceptance Criteria
- [ ] A student pays by UPI (test mode) on **Android, iOS, and web** and the order becomes `paid` only after the webhook confirms.
- [ ] Cancelling a paid order in the hold window issues a Razorpay refund automatically; vendor rejection of a paid order does the same.
- [ ] Webhook rejects bad signatures and is safe to receive twice (idempotent).
- [ ] No secrets in the repo; all amounts integer paise.

## 🚫 Out of scope
- COD path (5a). UIs beyond the Razorpay payment step. Tracking/notifications (5e).

## 🔗 Dependencies
- Tickets 5a, 5b. **Blocked by PRD D1.**

## 📚 References
- `docs/FUNCTIONS.md` (`placeOrder` UPI, `razorpayWebhook`), `docs/ARCHITECTURE.md` §5, PRD §7 D1, Brief #5.

## 🤖 Kickoff prompt
```
/start-ticket <this-issue-number>
```
