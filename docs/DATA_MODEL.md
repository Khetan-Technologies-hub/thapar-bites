# Data Model — Thapar Bites

> Firestore schema, document shapes, indexes, and the security-rule matrix. Companion to [ARCHITECTURE.md](ARCHITECTURE.md).

All monetary values are stored as **integer paise** (₹1 = 100) to avoid floating-point errors. Timestamps are Firestore `Timestamp`. The campus timezone is **Asia/Kolkata**.

---

## 1. Collections overview

```
users/{uid}
courts/{courtId}
courts/{courtId}/menuItems/{itemId}
orders/{orderId}
```

Orders are kept in a **top-level** collection (not nested under a court) so both the student and the vendor can query them by their own id efficiently.

---

## 2. Document shapes

### `users/{uid}`
```jsonc
{
  "role": "student",              // "student" | "vendor" | "admin"  (mirrors custom claim)
  "name": "Aditi Sharma",
  "email": "aditi@thapar.edu",
  "phone": "+91XXXXXXXXXX",        // optional
  "addresses": [                    // saved hostel delivery locations
    {
      "id": "addr_1",
      "label": "Hostel J",
      "hostel": "J",
      "room": "214",
      "notes": "Call on arrival",
      "isDefault": true
    }
  ],
  "fcmTokens": ["<token>"],        // for push
  "createdAt": "<Timestamp>"
}
```

### `courts/{courtId}`
```jsonc
{
  "name": "Cettle Cafe",
  "description": "North Indian & snacks",
  "ownerUid": "<vendor uid>",
  "isOpen": true,                   // manual open/closed toggle
  "operatingHours": {               // per weekday, 24h "HH:mm", campus time
    "mon": { "open": "09:00", "close": "23:00" },
    "tue": { "open": "09:00", "close": "23:00" }
    // ...
  },
  "acceptedPaymentMethods": ["upi", "cod"],   // subset of ["upi","cod"]
  "minOrderValue": 15000,           // paise — min for hostel delivery (varies per court)
  "imageUrl": "https://.../court.jpg",
  "createdAt": "<Timestamp>"
}
```

### `courts/{courtId}/menuItems/{itemId}`
```jsonc
{
  "name": "Paneer Roll",
  "category": "Rolls",
  "price": 8000,                    // paise
  "imageUrl": "https://.../item.jpg",
  "isVeg": true,
  "isAvailable": true,
  "sortOrder": 3
}
```

### `orders/{orderId}`
```jsonc
{
  "studentUid": "<uid>",
  "courtId": "<courtId>",
  "items": [                        // price snapshot taken at placement
    { "itemId": "<id>", "name": "Paneer Roll", "unitPrice": 8000, "qty": 2 }
  ],
  "subtotal": 16000,                // paise, server-computed
  "deliveryFee": 0,                 // free delivery in v1
  "total": 16000,                   // paise, server-computed
  "deliveryAddress": {              // snapshot of chosen address
    "hostel": "J", "room": "214", "notes": "Call on arrival"
  },
  "paymentMethod": "upi",           // "upi" | "cod"
  "paymentStatus": "paid",          // "pending" | "paid" | "failed" | "cod_due" | "cod_collected"
  "razorpay": {                     // present for UPI orders
    "orderId": "<rzp order>",
    "paymentId": "<rzp payment>"
  },
  "status": "Preparing",            // order lifecycle state (see ARCHITECTURE §4)
  "statusHistory": [
    { "status": "Placed", "at": "<Timestamp>" },
    { "status": "Accepted", "at": "<Timestamp>" }
  ],
  "createdAt": "<Timestamp>",
  "updatedAt": "<Timestamp>"
}
```

---

## 3. Enumerations

| Enum | Values |
|---|---|
| `role` | `student`, `vendor`, `admin` |
| `status` (order) | `Placed`, `Accepted`, `Rejected`, `Cancelled`, `Preparing`, `OutForDelivery`, `Delivered` |
| `paymentMethod` | `upi`, `cod` |
| `paymentStatus` | `pending`, `paid`, `failed`, `cod_due`, `cod_collected` |

These are defined once in the shared KMP module and mirrored here.

---

## 4. Indexes (composite)

Firestore needs composite indexes for the common queries:

| Query | Fields |
|---|---|
| Vendor's live order queue | `courtId` ==, `status` in [...], order by `createdAt` desc |
| Student's order history | `studentUid` ==, order by `createdAt` desc |
| Admin recent orders | order by `createdAt` desc |

Declared in `firebase/firestore.indexes.json`.

---

## 5. Security rules (matrix)

Enforced in `firebase/firestore.rules`. Business-logic writes (placing/transitioning orders) go through Cloud Functions; direct client writes are restricted to safe fields.

| Collection | Student | Vendor | Admin |
|---|---|---|---|
| `users/{uid}` | read/write **own** doc (safe fields: name, addresses, fcmTokens) | read/write own | read all |
| `courts/{courtId}` | **read** only | read/write **own court** (`ownerUid == uid`) | read/write all |
| `courts/*/menuItems/*` | **read** only | read/write for **own court** | read/write all |
| `orders/{orderId}` | **read** own (`studentUid == uid`); create via Function only; cancel pre-accept | **read** own court's; status transitions via Function only | read all |

Key rules:
- **Role** is read from the auth token's custom claim (`request.auth.token.role`), never from the client-writable `users` doc.
- **Order creation and status transitions** are performed by Cloud Functions (using the Admin SDK), so clients cannot forge totals or illegal transitions. Rules deny direct client writes to authoritative fields (`total`, `status`, `paymentStatus`).
- **Court ownership**: a vendor may only touch `courts/{courtId}` where `resource.data.ownerUid == request.auth.uid`.

---

## 6. Data integrity notes

- **Price snapshotting:** order `items[].unitPrice` and `subtotal`/`total` are snapshots computed by the server at placement, independent of later menu edits.
- **Min-order enforcement:** checked server-side against `courts/{courtId}.minOrderValue` before an order is accepted.
- **Payment/court method consistency:** an order's `paymentMethod` must be within the court's `acceptedPaymentMethods` — validated server-side.
- **Soft state:** orders are never hard-deleted; terminal states (`Delivered`, `Rejected`, `Cancelled`) remain for history and reporting.
