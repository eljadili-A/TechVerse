# TechVerse Electronic Store ⚡

A full-stack web-based e-commerce platform for electronic devices, built as a graduation project at the Islamic University of Gaza.

**Live Demo:** [electronics-store-project.netlify.app](https://electronics-store-project.netlify.app)

---

## 📋 Table of Contents

- [About](#about)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Payment Methods](#payment-methods)
- [API Endpoints](#api-endpoints)
- [Database Schema](#database-schema)
- [Team](#team)

---

## About

TechVerse is a responsive web store that sells smartphones, laptops, tablets, accessories, wearables, TVs, and gaming products. Customers can browse products, manage a cart, save favorites, and pay using four different payment methods. The project combines a Vue 3 frontend with a plain PHP REST API backend, Firebase Realtime Database, MySQL, and Stripe.

---

## Features

### Customer
- 📦 Browse products by category, brand, and price
- 🔍 Search products
- ❤️ Save products to favorites
- 🛒 Add to cart with quantity control
- 💳 Checkout with 4 payment methods
- 📋 View order history (My Purchases)
- ✉️ Contact form
- ❌ Cancel pending Stripe orders

### Technical
- Stripe webhook for reliable payment confirmation
- Server-side price validation (prices fetched from Firebase, not frontend)
- PDO prepared statements (SQL injection protection)
- Route guards for protected pages
- Cart and favorites persist via localStorage

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend Framework | Vue 3.2 + Pinia + Vue Router |
| CSS | Bootstrap 5.3 + Bootstrap Icons |
| Product/User Database | Firebase Realtime Database |
| Order Database | MySQL (Stripe orders only) |
| Backend API | Plain PHP 8.x (no framework) |
| Payment | Stripe Checkout + Webhooks |
| Frontend Hosting | Netlify |
| Backend Hosting | Hostinger |
| Local Dev | XAMPP |

---

## Project Structure

```
/
├── frontend/                   # Vue 3 project
│   ├── src/
│   │   ├── views/              # 13 pages
│   │   │   ├── HomePage.vue
│   │   │   ├── ProductsPage.vue
│   │   │   ├── ProductDetailPage.vue
│   │   │   ├── CartPage.vue
│   │   │   ├── CheckoutPage.vue
│   │   │   ├── FavoritesPage.vue
│   │   │   ├── MyPurchasesPage.vue
│   │   │   ├── LoginPage.vue
│   │   │   ├── SignupPage.vue
│   │   │   ├── StripeSuccessPage.vue
│   │   │   ├── StripeCancelPage.vue
│   │   │   ├── AboutPage.vue
│   │   │   └── ContactPage.vue
│   │   ├── stores/             # 4 Pinia stores
│   │   │   ├── authStore.js
│   │   │   ├── productsStore.js
│   │   │   ├── cartStore.js
│   │   │   └── purchasesStore.js
│   │   └── components/
│   └── public/
│       └── _redirects          # Netlify SPA routing fix
│
└── stripe-backend/             # PHP backend (htdocs)
    ├── config.php              # Credentials + CORS headers
    ├── db.php                  # MySQL PDO connection (singleton)
    ├── firebase.php            # Fetch product prices from Firebase
    ├── creatCheckout.php       # Create Stripe session  [POST]
    ├── webhook.php             # Handle Stripe events   [POST]
    ├── cancelOrder.php         # Cancel pending order   [POST]
    ├── orderStatus.php         # Get order status       [GET]
    ├── .env                    # Secrets (never commit this)
    └── vendor/                 # Composer packages
```

---

## Getting Started

### Prerequisites
- Node.js + npm
- XAMPP (PHP 8.x + MySQL)
- Composer
- Stripe CLI (for webhook testing)

### Frontend Setup

```bash
cd frontend
npm install
npm run dev
```

### Backend Setup

```bash
cd stripe-backend
composer install
```

Copy `.env.example` to `.env` and fill in your credentials.

```bash
cp .env.example .env
```

Start XAMPP (Apache + MySQL), then place the `stripe-backend` folder inside `htdocs/`.

### Webhook Testing (Local)

```bash
stripe login
stripe listen --forward-to localhost/stripe-backend/webhook.php
```

> Copy the `whsec_...` secret shown by Stripe CLI and paste it into your `.env` as `STRIPE_WEBHOOK_SECRET`.

---

## Environment Variables

Create a `.env` file inside `stripe-backend/`:

```env
# Stripe (sandbox keys for development)
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLIC_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Firebase
FIREBASE_PROJECT_ID=your-project-id
FIREBASE_API_KEY=your-api-key

# MySQL
DB_HOST=localhost
DB_NAME=your_db_name
DB_USER=your_db_user
DB_PASS=your_db_password

# URLs
SUCCESS_URL=https://your-frontend.netlify.app/stripe-success
CANCEL_URL=https://your-frontend.netlify.app/stripe-cancel
FRONTEND_ORIGIN=https://your-frontend.netlify.app
```

> ⚠️ Never commit `.env` to GitHub. Add it to `.gitignore`.

---

## Payment Methods

| Method | Flow | Database |
|---|---|---|
| Cash on Delivery | Vue → Firebase directly | Firebase |
| Credit Card (manual) | Vue → Firebase directly | Firebase |
| Bank Transfer | Vue → Firebase directly | Firebase |
| Stripe Online | Vue → PHP → Stripe → Webhook | MySQL |

---

## API Endpoints

Base URL: `https://your-hostinger-domain.com/stripe-backend/`

| Endpoint | Method | Description |
|---|---|---|
| `creatCheckout.php` | POST | Validate cart → create Stripe session → return checkout URL |
| `webhook.php` | POST | Receive Stripe events → update order status in MySQL |
| `cancelOrder.php` | POST | Update order status to `cancelled` |
| `orderStatus.php` | GET | Return order details to Vue |

### Example Request — Create Checkout

```json
POST /creatCheckout.php

{
  "cart_items": [
    { "product_id": "abc123", "quantity": 1 }
  ],
  "firebase_uid": "user@email.com"
}
```

```json
Response:
{
  "checkout_url": "https://checkout.stripe.com/...",
  "order_id": 5
}
```

---

## Database Schema

### Firebase Realtime Database

| Node | Description |
|---|---|
| `/Sign up` | User accounts (name, email, password, createdAt) |
| `/Products/{id}` | Product catalog (name, price, stock, category, brand, specs) |
| `/Buyers` | Buyer shipping info for COD/Card/Bank orders |
| `/Sales` | Sales records with item totals |
| `/My Purchases` | Order history per user (filtered by email) |
| `/orders` | COD/Card/Bank order records |
| `/Messages` | Contact form submissions |

### MySQL — `orders` Table (Stripe Only)

| Column | Type | Description |
|---|---|---|
| `id` | INT AUTO_INCREMENT PK | Unique order ID |
| `firebase_uid` | VARCHAR(128) | Customer email |
| `product_id` | VARCHAR(100) | Comma-separated product IDs |
| `product_name` | VARCHAR(255) | Human-readable names |
| `amount` | INT | Total in cents (119900 = $1,199.00) |
| `currency` | VARCHAR(10) | Always `usd` |
| `stripe_session_id` | VARCHAR(255) UNIQUE | Stripe session ID |
| `status` | ENUM | `pending` → `paid` / `failed` / `cancelled` |
| `created_at` | TIMESTAMP | Auto-set on insert |
| `updated_at` | TIMESTAMP | Auto-updated on change |

---

## Netlify Deployment

Add a `_redirects` file to `frontend/public/`:

```
/*    /index.html    200
```

This prevents 404 errors on direct URL access with Vue Router.

---

## Team

| Name | Role |
|---|---|
| Abdulrahman Ahmed Al-Dimassi | UI/UX Designer |
| Momin Ahmed Alwawi | Frontend Lead / Scrum Master |
| Abdulrahman Alaa Al-Jadili | Backend Lead |

**Supervisor:** Dr. Hazem A. Alrakhawi  
**University:** Islamic University of Gaza — Faculty of Information Technology  
**Year:** 2026

---

## License

This project was built as a graduation project for academic purposes.
