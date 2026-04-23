**API Design
Food Delivery Platform**

Versioning Strategy

We use URI-based versioning (e.g., /v1/...) for all endpoints.
Any breaking change results in a new version, while non-breaking updates (like adding optional fields) are introduced without affecting existing clients.

Older versions are supported for a limited time, giving developers enough room to migrate before deprecation.

Authentication

Most APIs require authentication using a JWT:

Authorization: Bearer <JWT>

We use:

Short-lived access tokens (15 minutes)
Rotating refresh tokens for session continuity

Public endpoints (like browsing restaurants) may not require authentication.

Idempotency

To prevent duplicate actions during retries (especially for orders and payments), we use an Idempotency-Key.

Clients must send this key in request headers for critical operations.
The server stores the request hash and response for 24 hours.
If the same request is retried, the server returns the original response instead of creating duplicates.
Rate Limiting

To keep the system stable under load:

Per-user limit: 100 requests/minute
Per-IP limit: 1000 requests/minute (adjustable during high traffic)

Sensitive endpoints (like order creation or payments) have stricter limits:

Example: max 10 order attempts per hour per user

We use a token bucket approach backed by Redis for efficient rate control.

Core Endpoints
1. Search Restaurants
GET /v1/restaurants?lat={lat}&lng={lng}&cuisine={}&q={}&limit=20&offset=0

Response (200 OK):

{
  "items": [
    {
      "id": "r_1",
      "name": "The Spice",
      "rating": 4.3,
      "eta_min": 20,
      "eta_max": 30
    }
  ],
  "count": 123
}

Errors:

400 – Invalid coordinates
429 – Rate limit exceeded
2. Get Menu
GET /v1/restaurants/{restaurant_id}/menu

Returns menu sections, items, and modifiers.

Caching:

Cached via CDN + Redis (60 seconds)
Updates trigger cache invalidation using pub/sub
3. Cart (Optional Server-Side)
POST /v1/cart

Request:

{
  "userId": "u_1",
  "items": [
    {"menuItemId": "m_1", "qty": 2}
  ],
  "coupon": "SAVE50"
}

Response:
A cart preview with pricing breakdown.

4. Create Order
POST /v1/orders

Headers:

Authorization (required)
Idempotency-Key (required)

Request:

{
  "user_id": "u_1",
  "restaurant_id": "r_1",
  "items": [
    {"menu_item_id": "m_1", "quantity": 2}
  ],
  "address_id": "addr_1",
  "payment_method": "CARD",
  "tip": 200
}

Success (201 Created):

{
  "order_id": "o_123",
  "status": "CONFIRMED",
  "estimated_pickup_minutes": 12,
  "estimated_delivery_minutes": 25
}

Possible errors:

400 – Invalid request
402 – Payment failed
409 – Idempotency conflict
429 – Too many requests
500 – Server error

If the same request is retried with the same idempotency key, the same order_id is returned.

5. Payment Webhooks
POST /v1/payments/webhook

Used by the payment service provider (PSP).

Validate request using HMAC signature
Update internal payment/order status
Respond quickly with 200 OK
Handle heavy processing asynchronously via background jobs
6. Order Status & Tracking
GET /v1/orders/{order_id}
GET /v1/orders/{order_id}/tracking

For real-time tracking, we use WebSockets:

wss://api.example.com/v1/realtime?token=<ws-token>

Clients subscribe to:

order:<order_id>

This provides live updates like order status and delivery partner location.

7. Delivery Partner APIs

Update availability:

POST /v1/partners/{partner_id}/availability

Send location updates:

POST /v1/partners/{partner_id}/location
{
  "lat": 28.61,
  "lng": 77.20,
  "timestamp": 1710000000
}
Updates are rate-limited (e.g., once per second)
Data is streamed via Redis pub/sub to real-time systems
Error Handling

We follow standard HTTP status codes:

200 – Success
201 – Resource created
202 – Accepted for async processing
400 – Bad request
401 – Unauthorized
403 – Forbidden
404 – Not found
409 – Conflict
422 – Business rule violation (e.g., restaurant closed)
429 – Too many requests
500 – Internal server error
Example Order Response
{
  "order_id": "o_123",
  "status": "PREPARING",
  "items": [
    {"name": "Paneer Tikka", "qty": 1, "price": 24900}
  ],
  "pricing": {
    "subtotal": 24900,
    "tax": 1245,
    "delivery": 3000,
    "total": 29145
  }
}
Security Notes
Never store or log full card details
Use tokenized payments via PSP
Always verify webhook authenticity using signatures
