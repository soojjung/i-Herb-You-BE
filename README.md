## 🎧 About Project

Project Name: **iHerbYou 🌿**

iHerbYou is an online platform designed for modern health-conscious consumers to **purchase nutritional supplements quickly and affordably**. <br />
Inspired by the strengths of iHerb, it aims to provide a tailored shopping experience optimized for Korean users with convenient payment and shipping services.

## 🚀 Tech Stack

| Category | Technology |
|----------|-----------|
| **Language** | Java 21 |
| **Framework** | Spring Boot 3.5.5 |
| **Core Modules** | Spring Web, Spring Data JPA (Hibernate), Spring Security, Spring Mail |
| **Authentication** | JWT (JJWT 0.13.0) |
| **Database** | MySQL 8.0 |
| **API Docs** | SpringDoc OpenAPI 2.8.11 (Swagger UI) |
| **Build Tool** | Gradle 8.5 |
| **Deployment** | Docker, Docker Compose |
| **Utilities** | Lombok |
| **Testing** | JUnit 5, Spring Boot Starter Test |

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Client (React)                    │
└──────────────────────┬──────────────────────────────┘
                       │ REST API (JWT Bearer Token)
┌──────────────────────▼──────────────────────────────┐
│                Spring Security Filter                │
│           (JwtAuthenticationFilter + CORS)            │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                   Controller Layer                    │
│        (Request/Response DTO-based API Endpoints)     │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│              Service / Facade Layer                   │
│        (Business Logic & Transaction Management)      │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│              Repository Layer (JPA)                   │
│          (Spring Data JPA + QueryMethod)              │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                  MySQL 8.0 Database                   │
└─────────────────────────────────────────────────────┘
```

## 📦 Module Structure

```
src/main/java/com/iherbyou/
├── auth/           # Authentication (token refresh)
├── user/           # User management (signup, profile, addresses)
├── catalog/        # Product catalog (products, categories, brands, search)
├── cart/           # Shopping cart (guest & user cart with merge)
├── ordering/       # Order management (orders, delivery, refunds)
├── payment/        # Payment processing (Toss Payments integration)
├── promotion/      # Promotions (coupons, points)
├── community/      # Community (reviews, Q&A)
├── wishlist/       # Wishlist (with sharing feature)
├── banner/         # Banner management
├── common/         # Shared codes / enums
├── security/       # Security config (JWT, filters)
├── config/         # App config (OpenAPI, properties)
└── exception/      # Exception handling
```

## ✨ Key Features

### Product Catalog
- Hierarchical category and brand-based product management
- Per-variant (ProductVariant) pricing and stock tracking
- Keyword search with auto-complete suggestions
- Bestsellers / New arrivals / Top-rated product listings

### User & Authentication
- Email verification-based signup
- JWT authentication & authorization (Access Token: 30 min, Refresh Token: 24 hrs)
- Token-based password reset
- Multiple shipping address management with default address
- Multi-tier admin roles (MASTER, BUSINESS, DEVELOP, MARKETING)

### Orders & Payments
- Cart-to-order conversion with guest cart merge support
- Toss Payments gateway integration
- Order status history tracking (OrderStatusHistory)
- Delivery status management and delivery confirmation
- Refund request and processing

### Promotions
- Coupon issuance / locking / redemption / release (concurrency-safe)
- Points earning / usage / restoration / expiration management
- Point rewards on review submission and order completion

### Community
- Product reviews (1-5 star rating, soft delete)
- Review reporting
- Product Q&A (questions & answers)

### Wishlist
- Save favorite products
- Shareable wishlist via snapshot-based links

## 🔌 API Endpoints

| Domain | Endpoint | Description |
|--------|----------|-------------|
| **Auth** | `/api/auth` | Token refresh |
| **User** | `/api/users` | Signup, login, logout, profile, password management |
| **Address** | `/api/users/me/addresses` | Address CRUD, set default address |
| **Product** | `/api/catalog/products` | Product list, detail, bestsellers, new arrivals, top-rated |
| **Category** | `/api/catalog/categories` | Category tree, products by category |
| **Brand** | `/api/catalog/brands` | Brand list, products by brand |
| **Search** | `/api/catalog/search` | Search, auto-complete suggestions |
| **Cart** | `/api/cart` | Cart CRUD, guest cart merge |
| **Order** | `/api/orders` | Create order, list orders, confirm delivery |
| **Payment** | `/api/payments` | Payment request / complete / cancel, Toss integration |
| **Refund** | `/api/payments/{id}/refunds` | Refund request |
| **Review** | `/api/reviews` | Create, list, summary |
| **Q&A** | `/api/qna` | Question & answer CRUD |
| **Wishlist** | `/api/wishlist` | Wishlist management, sharing |
| **Coupon** | `/api/users/me/coupons` | Coupon issuance, list, redemption |
| **Point** | `/api/users/me/points` | Point balance, history |
| **Banner** | `/api/banner` | Banner management |
| **Admin** | `/api/admin/orders` | Admin order management |

> Swagger UI: http://localhost:8080/swagger-ui/index.html

## 🛠️ Getting Started

### Prerequisites
- Java 21
- MySQL 8.0
- Gradle 8.5

### Local Setup

```bash
# 1. Clone the repository
git clone https://github.com/iHerbYou/i-Herb-You-BE.git
cd i-Herb-You-BE

# 2. Configure environment variables (refer to application.yaml)
#    - Database connection
#    - JWT secret
#    - Mail server settings
#    - Toss Payments API key

# 3. Build and run
./gradlew bootRun
```

### Docker Setup

```bash
# Create a .env file and configure environment variables
docker-compose up -d
```

## 📊 ERD

![ERD Diagram](./docs/erd.png)

## 👨🏻‍💻 Team Members

<!-- TEAM-MEMBERS-LIST:START - Do not remove or modify this section -->
<table>
  <tbody>
    <tr>
      <td align="center" valign="top">
        <a href="https://github.com/soojjung">
          <img src="https://github.com/soojjung.png" width="100px" alt="soojjung" />
          <br />
          <sub><b>Soojin Jung</b></sub>
        </a>
        <br />
        Frontend / Backend
      </td>
      <td align="center" valign="top">
        <a href="https://github.com/shawnchoi8">
          <img src="https://github.com/shawnchoi8.png" width="100px" alt="shawnchoi8" />
          <br />
          <sub><b>Seunghyun Choi</b></sub>
        </a>
        <br />
        Backend
      </td>
      <td align="center" valign="top">
        <a href="https://github.com/jaeaeee">
          <img src="https://github.com/jaeaeee.png" width="100px" alt="jaeaeee" />
          <br />
          <sub><b>Jaehee Choi</b></sub>
        </a>
        <br />
        Backend
      </td>
      <td align="center" valign="top">
        <a href="https://github.com/ye0nuu">
          <img src="https://github.com/ye0nuu.png" width="100px" alt="ye0nuu" />
          <br />
          <sub><b>Yeonwoo Jang</b></sub>
        </a>
        <br />
        Backend
      </td>
      <td align="center" valign="top">
        <a href="https://github.com/juncity-kim">
          <img src="https://github.com/juncity-kim.png" width="100px" alt="juncity-kim" />
          <br />
          <sub><b>Junhwi Kim</b></sub>
        </a>
        <br />
        Backend
      </td>
    </tr>
  </tbody>
</table>
<!-- TEAM-MEMBERS-LIST:END -->
