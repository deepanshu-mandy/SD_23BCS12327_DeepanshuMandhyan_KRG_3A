API Design — Food Delivery Platform (Short)
Versioning
•	Use /v1/... for APIs 
•	Breaking changes → new version 
•	Old versions supported for a limited time 

Authentication
•	JWT-based (Authorization: Bearer <token>) 
•	Access token (15 min) + refresh token 
•	Public APIs don’t need auth 

Idempotency
•	Required for order/payment APIs 
•	Use Idempotency-Key 
•	Same request → same response (prevents duplicates) 

Rate Limiting
•	100 req/min per user 
•	1000 req/min per IP 
•	Strict limits for orders/payments (e.g., 10/hour) 

Key APIs
•	Search Restaurants
GET /v1/restaurants → list with rating, ETA 
•	Get Menu
GET /v1/restaurants/{id}/menu → cached (60s) 
•	Cart
POST /v1/cart → price preview 
•	Create Order
POST /v1/orders → returns order ID & status 
•	Payment Webhook
POST /v1/payments/webhook → validate + async process 
•	Order Tracking
GET /v1/orders/{id} + WebSocket for real-time updates 
•	Delivery Partner APIs
Availability + live location updates 

Status Codes
•	200 OK, 201 Created 
•	400 Bad Request 
•	401/403 Auth errors 
•	404 Not Found 
•	409 Conflict 
•	429 Rate Limit 
•	500 Server Error 


Security
•	No card data stored 
•	Use PSP tokens 
•	Webhooks verified with signatures
