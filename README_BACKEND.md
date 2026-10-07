# SAAJNIKA — Backend

### Production-Ready Women's Fashion E-Commerce Platform

This repository contains the **backend application for Saajnika**, a production-style women's-fashion e-commerce platform.

The backend is responsible for the authoritative business logic, REST APIs, authentication, catalog, inventory, cart, checkout, payments, orders, shipping, notifications, administration, security, background processing, and production operations defined in the Saajnika Internship Project Implementation Plan.

> **Important:** This README follows the technology stack specified in the project plan exactly. The backend stack is **Django + Django REST Framework + PostgreSQL + Redis + Celery**.

---

# 🛠️ Required Backend Technology Stack

| Layer | Required Technology | Purpose |
|---|---|---|
| Backend | **Django** | Structured backend and application framework |
| API | **Django REST Framework** | REST API |
| Database | **PostgreSQL** | Production relational storage and transactional consistency |
| Authentication | **JWT / SimpleJWT** | Stateless access + refresh token authentication |
| Cache / Broker | **Redis** | Fast caching and Celery message brokering |
| Async Jobs | **Celery** | Email, SMS, cleanup, reconciliation and background jobs |
| Scheduler | **Celery Beat** | Scheduled tasks where required |
| Media | **Cloudinary** | Product/banner image storage and delivery |
| Payments | **Razorpay** | Payment gateway, signature verification and webhooks |
| Shipping | **Shiprocket or equivalent** | Shipment creation, courier selection, tracking and serviceability |
| Web Server | **Nginx + Gunicorn** | Production HTTP serving and reverse proxy |
| Containers | **Docker + Docker Compose** | Reproducible development and deployment |
| Version Control | **Git + GitHub/GitLab** | Collaboration, branching and history |
| Testing | **Pytest / Django tests + API tests** | Regression protection and release confidence |
| Observability | **Structured logs + error monitoring** | Production diagnosis and visibility |

This is the exact technology baseline specified in the project plan. fileciteturn0file0L37-L65

---

# 🚀 Backend Responsibility

The backend is the authoritative layer for:

- Pricing
- Inventory
- Payment state
- Order state
- Coupon validation
- Checkout calculations
- Authentication
- Authorization
- Customer data
- Order creation
- Stock reduction
- Payment verification
- Webhook processing
- Shipping state
- Administrative operations

The browser must never be treated as authoritative for final pricing, inventory, payment status, or order status. fileciteturn0file0L11-L20

---

# 🏗️ Production Architecture

```text
                         ┌──────────────────────┐
                         │    React Frontend    │
                         │   Customer + Admin   │
                         └──────────┬───────────┘
                                    │ HTTPS
                                    ▼
                         ┌──────────────────────┐
                         │        Nginx         │
                         │    Reverse Proxy     │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ Django REST API      │
                         │ Business Logic       │
                         └───────┬───────┬──────┘
                                 │       │
                    ┌────────────┘       └─────────────┐
                    ▼                                  ▼
          ┌──────────────────┐                ┌──────────────────┐
          │    PostgreSQL    │                │      Redis       │
          │ Persistent Data  │                │ Cache / Broker   │
          └──────────────────┘                └────────┬─────────┘
                                                       │
                                                       ▼
                                             ┌──────────────────┐
                                             │      Celery       │
                                             │ Background Jobs   │
                                             └────────┬──────────┘
                                                      │
                    ┌─────────────────────────────────┼─────────────┐
                    ▼                                 ▼             ▼
             ┌────────────┐                   ┌────────────┐ ┌────────────┐
             │ Cloudinary │                   │  Razorpay  │ │ Shiprocket │
             │   Media    │                   │  Payments  │ │  Shipping  │
             └────────────┘                   └────────────┘ └────────────┘
```

The project plan separates the web frontend, Django API, PostgreSQL database, Redis, Celery worker, scheduler and Nginx responsibilities. fileciteturn0file0L71-L83

---

# 📁 Backend Repository Structure

The implementation plan defines the backend applications by business domain:

```text
backend/
│
├── apps/
│   ├── catalog/
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── services.py
│   │   └── tests/
│   │
│   ├── accounts/
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   └── tests/
│   │
│   ├── cart/
│   ├── orders/
│   ├── payments/
│   ├── shipping/
│   └── notifications/
│
├── config/
│   ├── settings/
│   ├── urls.py
│   ├── celery.py
│   └── wsgi.py
│
├── manage.py
├── requirements.txt
└── ...
```

The required business-domain apps are `catalog`, `accounts`, `cart`, `orders`, `payments`, `shipping`, and `notifications`. fileciteturn0file0L84-L96

---

# 👤 Accounts & Authentication

## Custom User

The backend must use a custom user model with:

- Stable user identifier
- Required profile fields
- Customer/admin role separation

## JWT

Authentication uses:

```text
Access Token
+
Refresh Token
```

The access token should be short-lived and the refresh strategy must be controlled.

## OTP

Applicable phone-based flows require:

- OTP generation
- OTP expiry
- Single-use OTP
- Verification
- Rate limiting

## Logout

Logout/token invalidation behaviour must be defined and implemented.

## Protected Endpoints

Every protected endpoint must enforce authentication server-side.

The customer can access only their own:

- Cart
- Addresses
- Orders
- Personal data

Customers must not be able to modify:

- Inventory
- Payment state
- Other customers' data

fileciteturn0file0L97-L109

---

# 🔐 Backend Security

The backend must implement:

### HTTPS

Production traffic must use TLS.

### Secrets

Secrets must be stored through environment variables or a secret manager.

Never commit secrets to Git.

### CORS

Only trusted frontend origins should be allowed.

### CSRF

Correct CSRF configuration must be used where cookie/session-based flows apply.

### Input Validation

Use DRF serializers and business-rule validation.

### Rate Limiting

Sensitive endpoints must be protected, especially:

- OTP
- Login
- Password/reset
- Sensitive operations

### Webhooks

Payment webhook signatures must be verified.

### SQL Safety

Use Django ORM / parameterized queries.

### Uploads

Validate file type and size before using Cloudinary.

### PII

Collect only required customer data and avoid leaking it into logs.

fileciteturn0file0L110-L123

---

# 🛍️ Catalog Backend

The catalog must behave like a real e-commerce catalog.

## Product

Required product information includes:

- Name
- Slug
- Description
- Category
- Brand/collection
- Base price
- Sale price
- Status
- Timestamps

## Variant

Variants support:

- Size
- Color
- Other required option combinations

## SKU

Every sellable variant must have a unique SKU.

## Inventory

Inventory includes:

- Available quantity
- Reserved quantity where implemented
- Low-stock threshold
- Stock status

## Media

Products support:

- Multiple images
- Thumbnails
- Image ordering
- Primary image

Product and banner media are stored through Cloudinary.

## SEO

Catalog data supports:

- Slugs
- Meta title
- Meta description
- Canonical-friendly URLs where appropriate

fileciteturn0file0L124-L132

---

# 🔎 Catalog API Features

The backend supports:

- Category/subcategory navigation
- Product search
- Category filters
- Size filters
- Price-range filters
- Availability filters
- Attribute filters
- Sorting
- Pagination
- Featured products
- New products
- Discounted products
- Product details
- Related products

Out-of-stock variants cannot be purchased.

fileciteturn0file0L133-L140

---

# 📦 Inventory Rules

Inventory is one of the most critical backend responsibilities.

### Checkout Stock Validation

Cart-time stock is not authoritative.

Stock must be revalidated during checkout.

### Transactional Order Creation

Order creation and stock reduction must be transactional.

### Failed Orders

Failed/cancelled orders must not permanently corrupt inventory.

### Concurrent Purchases

The system must prevent negative inventory when multiple users attempt to purchase the same final unit.

### Audit Trail

Admin inventory changes must have an audit trail.

fileciteturn0file0L141-L146

---

# 🛒 Cart Backend

The cart API supports:

- Add item
- Update item
- Remove item
- Variant validation
- Stock validation
- Cart subtotal
- Stock-state information

Cart-time stock is only informational; checkout performs authoritative stock validation.

---

# 🧾 Checkout Backend

The checkout service is responsible for:

1. Validating the cart
2. Validating variants
3. Validating stock
4. Validating address
5. Validating coupon
6. Calculating subtotal
7. Calculating discounts
8. Calculating shipping
9. Calculating applicable taxes/fees
10. Calculating final total
11. Creating the internal order/payment intent

The final amount must be calculated server-side.

fileciteturn0file0L147-L163

---

# 🎟️ Coupon Backend

Coupons must support:

- Code validation
- Expiry validation
- Eligibility rules
- Usage limits
- Minimum cart value
- Discount rules

The browser must not be trusted to calculate the final discount.

---

# 📦 Order Backend

Orders preserve historical information so that later catalog changes do not alter historical purchases.

An order stores:

- Order items
- Purchased quantities
- Purchased prices
- Discounts
- Shipping address snapshot
- SKU/variant information
- Payment reference
- Gateway order ID
- Status
- Timestamps

---

# 🔄 Order State Machine

The backend enforces valid order transitions:

```text
CREATED
   ↓
PAYMENT_PENDING
   ↓
PAID
   ↓
PROCESSING
   ↓
PACKED
   ↓
SHIPPED
   ↓
OUT_FOR_DELIVERY
   ↓
DELIVERED
```

Exception states:

```text
PAYMENT_FAILED
CANCELLED
RETURN_REQUESTED
RETURNED
REFUNDED
```

Arbitrary status changes are not permitted.

fileciteturn0file0L164-L175

---

# 💳 Razorpay Integration

## Payment Flow

```text
React Client
    ↓
Checkout API
    ↓
Validate Cart
    ↓
Validate Stock
    ↓
Calculate Server Amount
    ↓
Create Razorpay Order
    ↓
Return Order Details
    ↓
Razorpay Checkout
    ↓
Payment
    ↓
Frontend Sends Payment IDs
    ↓
Backend Signature Verification
    ↓
Razorpay Webhook
    ↓
Idempotent Payment Processing
    ↓
Order Paid
```

## Backend Responsibilities

- Create Razorpay order
- Use server-calculated amount
- Verify payment signature
- Verify webhook signature
- Process webhook events
- Maintain payment status
- Maintain payment audit records
- Prevent duplicate processing
- Reconcile payment state

The backend must never trust the final amount supplied by the browser.

The backend must never mark an order as paid only because the frontend reports success.

fileciteturn0file0L176-L198

---

# 🔁 Payment Idempotency

Webhook processing must be idempotent.

Example:

```text
Razorpay
   │
   ├── Webhook #1
   │
   └── Webhook #2
          ↓
    webhook_event_id
          ↓
    Idempotency Check
          ↓
 Process Only Once
```

Payment audit data includes:

- `internal_order_id`
- `razorpay_order_id`
- `razorpay_payment_id`
- `signature`
- `status`
- `webhook_event_id`
- `created_at`
- `updated_at`

fileciteturn0file0L187-L198

---

# ⚡ Redis & Celery

Redis is used for:

- Caching
- Celery message brokering

Celery is responsible for slow, retryable, scheduled or asynchronous work.

> Redis is **not** the email system. Celery workers consume queued jobs and send email through the configured SMTP/email provider.

fileciteturn0file0L67-L70

---

# 📧 Background Jobs

| Job | Mode |
|---|---|
| Product listing cache | Cache hot queries where useful |
| Order confirmation email | Async Celery |
| OTP SMS | Async Celery |
| Shipping update email | Async Celery |
| Payment reconciliation | Scheduled Celery |
| Abandoned-cart cleanup | Celery Beat |
| Cache invalidation | Event-driven |

fileciteturn0file0L199-L211

---

# 🔁 Celery Retry Rules

Background jobs must:

- Use bounded retries
- Use exponential backoff for temporary failures
- Avoid infinite retries for permanent validation errors
- Be idempotent when affecting orders/payments/stock/notifications
- Record failures in logs/error monitoring
- Make failed jobs inspectable

fileciteturn0file0L212-L218

---

# 📧 Notification System

Transactional notifications include:

- Welcome/account verification
- OTP/login verification
- Order placed
- Payment successful
- Payment failed
- Order shipped
- Order delivered
- Password/reset/security notifications where applicable

These should be handled asynchronously where appropriate.

---

# 🚚 Shipping Backend

The shipping layer supports:

- Pincode serviceability
- Shipment creation
- Courier selection
- Shipment ID
- Tracking number
- Tracking updates
- Delivery status
- Provider failures
- Safe retries

The shipping provider should be abstracted so the system can use Shiprocket or an equivalent provider.

fileciteturn0file0L226-L233

---

# 🔌 REST API

## Authentication

```text
/api/auth/send-otp/
/api/auth/verify-otp/
/api/auth/refresh/
/api/auth/logout/
```

## Profile

```text
/api/profile/
/api/profile/update/
```

## Products

```text
/api/products/
/api/products/{id}/
/api/products/home/
```

## Categories

```text
/api/categories/
/api/categories/{slug}/
```

## Cart

```text
/api/cart/
/api/cart/add/
/api/cart/update/
/api/cart/remove/
```

## Address

```text
/api/addresses/
/api/addresses/{id}/
```

## Wishlist

```text
/api/wishlist/
/api/wishlist/toggle/
```

## Checkout

```text
/api/checkout/
/api/buy-now/
```

## Payments

```text
/api/payments/create/
/api/payments/verify/
/api/payments/webhook/
```

## Orders

```text
/api/orders/
/api/orders/{id}/
/api/orders/{id}/cancel/
```

## Shipping

```text
/api/shipping/serviceability/
/api/shipping/track/{id}/
```

## Admin

```text
/api/admin/products/
/api/admin/orders/
/api/admin/inventory/
```

The API should use consistent HTTP status codes, DRF serializer validation, actionable errors, pagination, filtering/search and documented authentication requirements. fileciteturn0file0L247-L271

---

# 🧑‍💼 Admin Backend

Admin capabilities include:

### Products

- Create
- Edit
- Archive
- Variants
- Prices
- Images
- Stock

### Categories

- Create
- Reorder
- Visibility

### Orders

- Search
- Inspect
- Update permitted states
- Cancel
- Refund where supported

### Payments

- Payment references
- Reconciliation status

### Coupons

- Codes
- Limits
- Validity
- Minimum order
- Discount rules

### Banners

- Homepage banners
- Carousel content
- Ordering

### Customers

- Customer search
- Relevant account/order information

### Inventory

- Stock adjustment
- Low-stock visibility
- History

### Audit

- Who changed what
- When it changed
- Which admin action caused the change

fileciteturn0file0L234-L246

---

# 🗄️ Database

The project uses **PostgreSQL** as the production relational database.

The database must provide durable transactional state for:

```text
Users
Products
Categories
Variants
SKUs
Inventory
Cart
Cart Items
Addresses
Wishlist
Orders
Order Items
Payments
Shipping
Coupons
Banners
Audit Records
```

The exact schema should be represented in the project's ER diagram.

---

# 🔐 Environment Variables

Example backend configuration:

```env
# Django
SECRET_KEY=your_secret_key
DEBUG=False

# PostgreSQL
POSTGRES_DB=saajnika
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_password
POSTGRES_HOST=db
POSTGRES_PORT=5432

# Redis
REDIS_URL=redis://redis:6379/0

# JWT
JWT_SECRET_KEY=your_jwt_secret

# Cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# Razorpay
RAZORPAY_KEY_ID=your_key_id
RAZORPAY_KEY_SECRET=your_key_secret

# Email / SMTP
EMAIL_HOST=your_smtp_host
EMAIL_PORT=587
EMAIL_HOST_USER=your_email
EMAIL_HOST_PASSWORD=your_password

# Shipping
SHIPPING_API_KEY=your_shipping_key
```

> **Never commit `.env` files, database passwords, API keys, payment secrets or production credentials to Git.**

---

# 🐳 Docker Development

The local architecture is designed around Docker Compose.

Expected services:

```text
frontend
backend
postgres
redis
celery
nginx
```

Start services:

```bash
docker compose up --build
```

Run Django migrations:

```bash
docker compose exec backend python manage.py migrate
```

Create an admin user:

```bash
docker compose exec backend python manage.py createsuperuser
```

---

# 🧪 Testing & Quality

The project requires multiple testing layers.

## Unit Tests

Test:

- Pricing
- Coupon rules
- Inventory rules
- Order transitions
- Utility/service logic

## API Tests

Test:

- Authentication
- Products
- Cart
- Checkout
- Orders
- Payment verification
- Admin permissions

## Integration Tests

Test:

- PostgreSQL
- Redis
- Celery
- Payment sandbox/mocks
- Shipping sandbox/mocks

fileciteturn0file0L315-L328

---

# 🧪 Critical Backend Test Scenarios

### 1. Concurrent Last-Unit Purchase

Two users attempt to purchase the last unit simultaneously.

Expected result:

```text
Available Stock = 1

User A ──┐
         ├── Backend Transaction ──> One Successful Purchase
User B ──┘
                         └──────────> Safe Failure
```

No negative inventory must occur.

### 2. Payment Success + Frontend Failure

Payment succeeds but the frontend callback is interrupted.

The backend must rely on verified payment state/webhooks rather than the browser alone.

### 3. Duplicate Webhook

The same payment webhook arrives multiple times.

Expected:

```text
First Webhook  → Process
Second Webhook → Ignore duplicate
```

### 4. Payment Failure

Payment fails after an internal order is created.

The system must maintain a valid order/payment state.

### 5. Coupon Validation

Test:

- Expired coupon
- Invalid coupon
- Overused coupon
- Below minimum cart value

### 6. JWT Expiry

Access token expires during checkout.

### 7. Email Failure

Email provider temporarily fails.

Celery retries according to retry rules.

### 8. Shipping Timeout

Shipping provider times out.

The backend handles the failure safely.

### 9. Authorization

Admin attempts a customer-only operation or customer attempts an admin-only operation.

fileciteturn0file0L329-L338

---

# 🚀 Production Deployment

Production environment:

```text
Internet
   ↓
HTTPS
   ↓
Nginx
   ↓
Gunicorn
   ↓
Django REST API
   ↓
PostgreSQL
   +
Redis
   +
Celery Worker
   +
Celery Beat
```

Production requirements include:

- HTTPS
- Nginx
- Gunicorn
- PostgreSQL
- Redis
- Celery workers
- Secure environment variables
- Cloudinary
- Backups
- Logs
- Error monitoring
- Automatic worker restart
- Deployment rollback capability

fileciteturn0file0L339-L358

---

# 💾 PostgreSQL Backup

The backend production environment must:

- Take regular PostgreSQL backups
- Store backups separately from the application server
- Retain multiple restore points
- Document restoration procedures
- Test restoration

> A backup that has never been restored is not a proven recovery strategy.

---

# 📊 Observability

Production observability includes:

- Application logs
- Celery worker logs
- Nginx/proxy logs
- Error monitoring
- Failed background jobs
- Service health visibility
- Payment reconciliation visibility

The implementation plan requires structured logs and error monitoring for production diagnosis. fileciteturn0file0L61-L65

---

# 🌿 Git Workflow

Recommended backend branches:

```text
main
 │
 ├── feature/accounts
 ├── feature/catalog
 ├── feature/cart
 ├── feature/checkout
 ├── feature/payments
 ├── feature/orders
 ├── feature/shipping
 ├── feature/notifications
 └── feature/admin
```

Development rules:

- Use feature branches
- Keep commits focused
- Use descriptive commits
- Open Pull Requests / Merge Requests
- Never commit `.env`
- Never commit database dumps
- Keep documentation updated
- Track work using issues/tasks
- Resolve test/lint failures before review

The project plan explicitly requires focused commits, feature branches, PR/MR review and no secrets/database dumps in Git. fileciteturn0file0L359-L367

---

# 🗺️ Backend Implementation Roadmap

| Phase | Backend Deliverable |
|---|---|
| 1 | Django, DRF, PostgreSQL, Redis, Docker and environment foundation |
| 2 | Custom user, OTP, JWT, refresh and permissions |
| 3 | Catalog, variants, inventory and Cloudinary |
| 4 | Cart, stock validation and pricing |
| 5 | Address, coupons, checkout and order creation |
| 6 | Razorpay order creation, verification, webhooks and idempotency |
| 7 | Orders, state transitions and admin operations |
| 8 | Redis, Celery, notifications, retries and scheduled jobs |
| 9 | Shipping, serviceability, shipment and tracking |
| 10 | Nginx, Gunicorn, backups, tests, logs and monitoring |

This follows the exact implementation sequence in the project plan. fileciteturn0file0L293-L314

---

# ✅ Backend Definition of Done

### Authentication

- [ ] Custom user model
- [ ] JWT access/refresh
- [ ] OTP verification
- [ ] OTP expiry/single-use
- [ ] Logout/token invalidation
- [ ] Customer/admin authorization

### Catalog

- [ ] Products
- [ ] Categories
- [ ] Variants
- [ ] SKUs
- [ ] Inventory
- [ ] Cloudinary media
- [ ] Search
- [ ] Filtering
- [ ] Sorting
- [ ] Pagination

### Commerce

- [ ] Cart
- [ ] Server-side pricing
- [ ] Coupon validation
- [ ] Address handling
- [ ] Checkout
- [ ] Transactional order creation
- [ ] Safe inventory reduction

### Payments

- [ ] Razorpay order creation
- [ ] Signature verification
- [ ] Webhook verification
- [ ] Idempotency
- [ ] Payment audit data
- [ ] Reconciliation

### Orders

- [ ] Order history
- [ ] Controlled state transitions
- [ ] Historical item prices
- [ ] Address snapshot
- [ ] Payment references
- [ ] Cancellation/refund states where supported

### Async

- [ ] Redis
- [ ] Celery
- [ ] Celery Beat where required
- [ ] Email jobs
- [ ] SMS/OTP jobs
- [ ] Shipping notifications
- [ ] Payment reconciliation
- [ ] Abandoned-cart cleanup
- [ ] Retry handling

### Shipping

- [ ] Pincode serviceability
- [ ] Shipment creation
- [ ] Courier information
- [ ] Tracking
- [ ] Delivery status
- [ ] Provider failure handling

### Admin

- [ ] Products
- [ ] Categories
- [ ] Orders
- [ ] Payments
- [ ] Coupons
- [ ] Banners
- [ ] Customers
- [Inventory
- [ ] Audit visibility

### Production

- [ ] Dockerized environment
- [ ] Nginx
- [ ] Gunicorn
- [ ] HTTPS
- [ ] PostgreSQL backups
- [ ] Restore procedure
- [ ] Logs
- [ ] Error monitoring
- [ ] Rollback procedure
- [ ] Secrets outside source control

### Testing

- [ ] Unit tests
- [ ] API tests
- [ ] Integration tests
- [ ] Critical purchase-flow tests
- [ ] Payment failure tests
- [ ] Webhook idempotency tests
- [ ] Inventory concurrency tests
- [ ] Permission tests

---

# 📦 Backend Handoff

The backend handoff should contain:

```text
Source Code
├── Django Project
├── Django Apps
├── Configuration
├── Celery
└── Deployment Configuration

Documentation
├── README
├── API Documentation
├── ER Diagram
├── Architecture Diagram
├── Testing Report
└── Deployment Guide
```

The project plan requires the final handoff to include source code, README, API documentation, ER diagram, architecture diagram, test report and deployment guide. fileciteturn0file0L398-L412

---

# 🎯 Backend Goal

The Saajnika backend is designed to provide a production-oriented foundation for:

```text
Authentication
      +
Catalog
      +
Inventory
      +
Cart
      +
Checkout
      +
Payments
      +
Orders
      +
Shipping
      +
Notifications
      +
Administration
      +
Caching
      +
Background Jobs
      +
Security
      +
Testing
      +
Production Operations
```

---

# 📌 Project Status

🚧 **Backend Under Active Development**

Implementation follows:

**Foundation → Authentication → Catalog → Cart → Checkout → Payments → Orders → Async Processing → Shipping → Production**

---

## 📄 License

Add the applicable project license here.
