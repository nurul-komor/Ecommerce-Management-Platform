# Well Mart — E-commerce Management Platform

> A **multi-vendor e-commerce marketplace** built with Laravel 10, supporting four distinct roles — customers, sellers, delivery men, and admins — each with its own authentication, dashboard, and permissions.

Well Mart is a full server-rendered marketplace where **sellers** list products (subject to admin approval), **customers** browse, search, and buy them with online payment, **delivery men** accept and fulfill delivery tasks, and **admins** oversee the whole operation. It features Algolia-powered search, real-time chat, SSLCommerz payments, and database/email notifications.

---

## ✨ Features

### 🛍️ Customers
- Browse products, filter by category, and view product detail pages
- Algolia-powered product search
- Session-based shopping cart (add / increment / decrement / remove)
- Checkout and online payment via **SSLCommerz**
- Order tracking dashboard
- Product ratings & reviews (with images)
- News/blog section, About, and Contact form
- Experimental **3D / "VR" product view** (glTF model)

### 🏪 Sellers / Vendors
- Self-registration → email verification → **admin approval** gate
- Seller dashboard
- Full CRUD over their own products
- Seller profile management

### 🚚 Delivery Men
- Dashboard of orders ready for delivery
- Accept delivery tasks and update their status
- Toggle availability status

### 🛠️ Admins
- Approve/reject sellers and products
- Manage customers, categories, orders, and delivery men
- Manage news/blog articles and categories
- Respond to contact messages

### 🔗 Cross-cutting
- **Multi-guard authentication** (`web`, `admin`, `seller`, `delivery_men`) with email verification
- **Real-time chat** (Chatify + Pusher)
- **Database & email notifications** with queued jobs
- SSLCommerz payment lifecycle (success / fail / cancel / IPN)

---

## 🧰 Tech stack

**Backend**
- [Laravel 10](https://laravel.com) (PHP ^8.1)
- Laravel Breeze (auth scaffolding), Laravel Sanctum (API tokens)
- Laravel Scout + **Algolia** (search)
- Chatify (real-time messaging)
- SSLCommerz (payment — bundled under `app/Library/SslCommerz`)

**Frontend**
- Blade templates + [Alpine.js 3](https://alpinejs.dev), bundled with **Vite 4**
- **Tailwind CSS 3** with **DaisyUI** and **Flowbite**

**Database:** MySQL
**Testing:** Pest + PHPUnit

---

## 🏗️ Architecture

A classic **Laravel MVC monolith** serving role-based server-rendered views plus a small JSON API.

```
app/
├── Http/Controllers/
│   ├── AdminAuth/        # admin guard
│   ├── SellerAuth/       # seller guard
│   ├── DeliveryMenAuth/  # delivery_men guard
│   ├── Auth/             # customer (web) guard
│   └── Api/              # JSON API (products, search)
├── Models/               # User, Admin, Seller, DeliveryMan, Products, Order, …
├── Jobs/ · Notifications/ · Services/
└── Library/SslCommerz/   # bundled payment integration

routes/   web.php · api.php · auth.php · admin_auth.php · seller_auth.php · delivery_men_auth.php
resources/views/  admin/ · seller/ · delivery_men/ · user/ · common/
```

Four session guards (`config/auth.php`) give each role fully separate login/registration flows.

---

## 🗄️ Data model (selected)

| Model | Notes |
|---|---|
| `User`, `Admin`, `Seller`, `DeliveryMan` | One per auth guard |
| `Products` | `product_code` (unique), price, discount, colors/options (JSON), seller + category FKs, status |
| `ProductCategories` | Per-seller product categories |
| `Order` | `invoice`, product snapshot (JSON), amount, payment, transaction id, status |
| `DeliveryMenTask` | Links a delivery man to an order |
| `Rating` | Reviews with images |
| `News`, `NewsCategory` | Blog/news section |
| `ChMessage`, `ChFavorite` | Chatify chat |

---

## 🚀 Getting started

### Prerequisites
- PHP 8.1+ and Composer
- Node.js & npm
- MySQL

### Installation

```bash
# Dependencies
composer install
npm install

# Environment
cp .env.example .env
php artisan key:generate

# Configure your MySQL database and service keys in .env, then:
php artisan migrate --seed      # seeds demo users, products, orders, etc.
php artisan storage:link        # for uploaded images
```

### Run

```bash
php artisan serve        # backend
npm run dev              # Vite dev server (second terminal)
php artisan queue:work   # process jobs & notifications
```

Build production assets with `npm run build`.

### Environment variables

Configure these in `.env` (names only — never commit real values):

- **App:** `APP_NAME`, `APP_ENV`, `APP_KEY`, `APP_DEBUG`, `APP_URL`, `APP_LOGO`
- **Database:** `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`
- **Mail:** `MAIL_MAILER`, `MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME`, `MAIL_PASSWORD`, `MAIL_ENCRYPTION`, `MAIL_FROM_ADDRESS`, `MAIL_FROM_NAME`
- **Broadcast / chat (Pusher):** `PUSHER_APP_ID`, `PUSHER_APP_KEY`, `PUSHER_APP_SECRET`, `PUSHER_APP_CLUSTER` (+ `VITE_PUSHER_*`)
- **Payment (SSLCommerz):** `SSLCZ_STORE_ID`, `SSLCZ_STORE_PASSWORD`, `SSLCZ_TESTMODE`
- **Search (Algolia):** `ALGOLIA_APP_ID`, `ALGOLIA_SECRET`, `SCOUT_DRIVER`, `SCOUT_QUEUE`
- **Maps:** `GOOGLE_MAPS_API_KEY`
- **Queue/cache/session:** `QUEUE_CONNECTION`, `CACHE_DRIVER`, `SESSION_DRIVER`, `REDIS_*`

> ⚠️ **Security:** Never commit real credentials. Keep secrets only in your local `.env` (which is git-ignored) and rotate any keys that may have been exposed.

---

## 📡 API endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/user` | Authenticated user (Sanctum) |
| `GET` | `/api/products/{sorting?}/{min?}/{max?}/{category?}` | Product listing with sort/price/category filters |
| `GET` | `/api/search/` | Algolia product search |

SSLCommerz callbacks (web): `POST /pay`, `/pay-via-ajax`, `/success`, `/fail`, `/cancel`, `/ipn`.

---

## 🚢 Deployment

CI/CD via GitHub Actions (`.github/workflows/deploy.yml`): on push to `master`, installs production dependencies and FTP-deploys to **cPanel** hosting. `laravel/sail` is available for local Docker-based development.

---

## 🧪 Testing

```bash
php artisan test
```

---

## 📄 License

MIT.
