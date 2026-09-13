# Cloud Functions — Server Contract

> The server-authoritative operations. Anything touching **money, order state, or roles** goes here — clients call these, they don't write those fields directly. Companion to [ARCHITECTURE.md](ARCHITECTURE.md) and [DATA_MODEL.md](DATA_MODEL.md).

Functions live in `functions/`. Callable functions use Firebase **callable** (auth context attached automatically); webhooks are HTTPS endpoints; triggers react to Firestore/Auth events.

---

## 1. Callable functions (client → server)

### `placeOrder`
Creates an order after full server-side validation.

**Auth:** `student`
**Request:**
```jsonc
{ "courtId": "...", "items": [{ "itemId": "...", "qty": 2 }], "addressId": "addr_1", "paymentMethod": "upi" }
```
**Server does:**
1. Load court + referenced menu items (authoritative prices).
2. Reject if court closed / outside operating hours.
3. Reject if `paymentMethod` not in `court.acceptedPaymentMethods`.
4. Recompute `subtotal`/`total`; reject if `subtotal < court.minOrderValue`.
5. **UPI:** create a Razorpay order, write the order as `paymentStatus: pending`, return the Razorpay handle. **COD:** write order `Placed`, `paymentStatus: cod_due`.
6. Notify vendor (COD) or wait for payment webhook (UPI) before notifying.

**Response:** `{ orderId, status, razorpay?: { orderId, keyId, amount } }`

---

### `transitionOrder`
Advances an order's status, validated against the state machine and caller role.

**Auth:** `vendor` (own court) or `student` (cancel pre-accept only)
**Request:** `{ "orderId": "...", "toStatus": "Preparing" }`
**Server does:** verify caller owns/placed the order → check transition is legal in the state machine → check role is allowed to make it → write `status` + append `statusHistory` → trigger notification.
**Response:** `{ orderId, status }`

---

### `setUserRole` *(admin only)*
Grants `vendor`/`admin` custom claims and creates/links the court for a vendor.
**Auth:** `admin`
**Request:** `{ "uid": "...", "role": "vendor", "courtId?": "..." }`

---

### `upsertMenuItem` / `updateCourtSettings`
Convenience callables for vendors to edit menu items and court settings (`operatingHours`, `isOpen`, `acceptedPaymentMethods`, `minOrderValue`). Enforce `court.ownerUid == caller`. (These can alternatively be direct Firestore writes guarded by security rules — see D-note below.)

> **D-note:** menu/settings edits are safe to do as **direct client writes** guarded by security rules (no money/state logic). Order and role operations must be functions. Decide per-team; the rules matrix in [DATA_MODEL.md](DATA_MODEL.md) already permits vendor self-service writes to their own court.

---

## 2. HTTPS webhooks (external → server)

### `razorpayWebhook`
Receives Razorpay payment events.
- **Verifies the signature** using the Razorpay webhook secret (reject otherwise).
- On `payment.captured`: find the order by Razorpay order id → set `paymentStatus: paid`, `status: Placed` → notify vendor.
- On `payment.failed`: set `paymentStatus: failed`; the order is not sent to the vendor.
- **Idempotent:** safe to receive the same event twice.

---

## 3. Firestore / Auth triggers (event-driven)

### `onUserCreate` (Auth trigger)
On new Firebase Auth user: verify the email domain is `thapar.edu`, set default custom claim `student`, create `users/{uid}`.

### `onOrderStatusChange` (Firestore trigger, `orders/{orderId}` onUpdate)
When `status` changes: send the appropriate FCM push — to the **vendor** on a new `Placed` order, to the **student** on `Accepted`/`Preparing`/`OutForDelivery`/`Delivered`/`Rejected`.

---

## 4. Conventions

- **Language/runtime:** Node.js (TypeScript) or Kotlin — team choice; TypeScript is the common Firebase default. *(Open: align with D2 web decision.)*
- **Money:** all amounts integer paise; never floats.
- **Validation:** re-validate everything server-side even if the client already checked (client checks are UX, not security).
- **Secrets:** Razorpay key/secret and webhook secret stored in Firebase secrets/config, never in the repo.
- **Idempotency:** webhooks and payment handling must tolerate retries.
- **Errors:** throw typed `HttpsError` with codes clients can map to messages (e.g. `failed-precondition` for below-minimum).

---

## 5. Function summary table

| Function | Type | Auth | Purpose |
|---|---|---|---|
| `placeOrder` | callable | student | Validate + create order, kick off payment |
| `transitionOrder` | callable | vendor/student | Legal, role-checked status change |
| `setUserRole` | callable | admin | Grant vendor/admin claims, link court |
| `upsertMenuItem` | callable | vendor | Edit menu (optional; may be direct write) |
| `updateCourtSettings` | callable | vendor | Edit court settings (optional; may be direct write) |
| `razorpayWebhook` | https | signature | Reconcile payment status |
| `onUserCreate` | auth trigger | — | Domain check, default role, user doc |
| `onOrderStatusChange` | firestore trigger | — | Push notifications |
