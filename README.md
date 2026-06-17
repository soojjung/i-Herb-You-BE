# iHerbYou — Backend

iHerbYou is a learning project — a Spring Boot 3 + MySQL backend for a Korean-market online supplement marketplace, modeled after iHerb as a thematic motif. I worked on it with a small team and own the parts described below.

> **Status:** demo / portfolio project. The catalog is populated with crawled product data (collected for development use only, not redistributed), and the Toss Payments integration runs against the sandbox keys only — no real charges are processed.

This README focuses on what I designed and built end-to-end — the shopping cart, the wishlist with snapshot-based sharing, the Spring Security / JWT foundation, and the initial domain model and ERD the rest of the project was built on top of.

---

## 🎬 Demo

https://github.com/user-attachments/assets/e80f5c67-7e12-40fe-a076-1e46e60ee2e2

---

## 🌿 Why I Built This

I picked an iHerb-style supplement marketplace as a motif because the domain is familiar to Korean shoppers and the underlying backend touches the e-commerce primitives I most wanted hands-on practice with — multi-variant SKUs, stock-aware carts, guest-to-user merges, coupon/point promotions, and a payment-to-delivery lifecycle.

My goal was to build a backend that could plausibly support a real marketplace with the architectural discipline of a production service rather than a school project — while staying honest about its demo status. The pre-login shopping flow (guest cart → merge into the user cart on sign-in without losing items) is the slice I owned end-to-end.

---

## 👩‍💻 What I Built

I owned the following slices of the system end-to-end — from schema design and API contract through implementation, refactor, and exception handling. Each section points to the PRs I shipped and the specific files I wrote.

### Shopping Cart — `cart/`

A unified cart model that lets shoppers add items before they have an account, then keeps everything intact at sign-in. One `Cart` row holds either a `user_id` or a `guest_token` (UUID), so every endpoint resolves through the same path. On login, `mergeGuestCart` folds the guest cart into the user's — summing quantities for matching `ProductVariant`s and deleting the guest cart in one transaction — so the frontend just calls `/api/cart/merge` and never diffs anything itself. Stock failures throw three distinct exceptions so the UI can render different CTAs for "sold out" vs. "only N left." Scheduled jobs purge expired guest carts and inactive user carts to keep the table bounded.

**Important points:** single model for both audiences (no duplicated business logic), transaction-safe merge, expressive stock errors instead of a generic "failed" toast.

### Wishlist with Shareable Snapshot Links — `wishlist/`

Wishlist CRUD plus a "share my wishlist" feature backed by a 72-hour public link. The 20-item cap is enforced on the aggregate root (`Wishlist.addProduct` raises `IllegalStateException`), not in the service, so the invariant follows the entity. When a user creates a share, the service serializes a JSON snapshot of the wishlist and stores it on `WishlistShare.snapshotJson`; the read endpoint deserializes that snapshot and never re-queries the live wishlist. Concurrent duplicate adds are absorbed by a unique constraint on `(wishlist_id, product_id)` — the loser's `DataIntegrityViolationException` is caught and translated into the same duplicate response the in-app check would have produced.

**Important points:** snapshot-based sharing guarantees the recipient sees exactly what was sent, the DB index is the integrity boundary for concurrent duplicates, and `EntityManager.getReference` skips a redundant user SELECT on creation.

### Security Foundation — `security/`, `config/`

Spring Security filter chain configured for a stateless JWT API: `STATELESS` session policy, CSRF off, a custom `JwtAuthenticationFilter` placed ahead of `UsernamePasswordAuthenticationFilter`, and a structured 401 response from `authenticationEntryPoint`. CORS is allowed for the React frontend (local dev + production domain) with credentialed requests. SpringDoc OpenAPI is integrated with grouped API docs and a `bearerAuth` scheme so authenticated endpoints can be exercised directly in Swagger UI.

**Important points:** this is the auth scaffolding the rest of the team built on top of — getting it wrong (e.g. leaving sessions stateful, or missing CORS credentials) would have blocked every feature behind it.

### Initial Domain Model & ERD

The first cut of the domain entities (catalog, ordering, cart, wishlist, user) and the ERD at `docs/erd.png`. The most consequential decision was pushing per-variant pricing and stock onto `ProductVariant` instead of `Product`, which is what made variant-aware carts, orders, and stock checks possible later. Seed SQL scripts load product and category data crawled by a teammate — using real names, brands, and images surfaced edge cases (long titles, missing thumbnails, duplicate brand names) that `Lorem ipsum` fixtures would have hidden.

**Important points:** schema decisions early on (variant-level price/stock) shaped what the cart and order logic could express, and choosing realistic-shaped seed data caught UI/data edge cases pre-launch.

---

## 🧩 Technical Challenges & How I Solved Them

**One cart model for users and guests.** Separate guest/user tables duplicate business logic and push the merge to the frontend. A single `Cart` row holds either `user_id` or `guest_token` (UUID) and every endpoint resolves through `findCartByUserOrGuest`. On login, `mergeGuestCart` sums quantities for any matching `ProductVariant`, inserts the rest, and deletes the guest cart — all in one transaction.

**Wishlist sharing without leaking later edits.** Re-querying the live wishlist on share view would let later edits or deletions leak into a sent link. Instead, `createShare` serializes a JSON snapshot at share time and stores it on `WishlistShare.snapshotJson`; the read path never touches the live wishlist. `isExpired()` lives on the entity so `SHARE_EXPIRED` doesn't scatter wall-clock logic.

**Concurrent duplicate adds.** A double-tapped heart can race two `existsBy…` checks. A unique `(wishlist_id, product_id)` constraint makes the loser raise `DataIntegrityViolationException`, which I translate into the same duplicate response — the DB is the integrity boundary, not the app.

**Stock-aware error messages.** The original generic exception couldn't distinguish "sold out" from "you asked for 5 but only 2 left." Three exception types (`OutOfStockException`, `InsufficientStockException(requested, available)`, `InvalidQuantityException`) each map to a distinct response code, so the frontend can render a different CTA per case.

**Skipping a redundant SELECT.** Creating a `Wishlist` needs only a `User` FK reference, not a row read. `entityManager.getReference(User.class, userId)` returns a proxy that satisfies the FK without hitting the DB.

---

## 🚀 Tech Stack

| Category           | Technology                                                            |
| ------------------ | --------------------------------------------------------------------- |
| **Language**       | Java 21                                                               |
| **Framework**      | Spring Boot 3.5.5                                                     |
| **Core Modules**   | Spring Web, Spring Data JPA (Hibernate), Spring Security, Spring Mail |
| **Authentication** | JWT (JJWT 0.13.0)                                                     |
| **Database**       | MySQL 8.0                                                             |
| **API Docs**       | SpringDoc OpenAPI 2.8.11 (Swagger UI)                                 |
| **Build Tool**     | Gradle 8.5                                                            |
| **Deployment**     | Docker, Docker Compose                                                |
| **Utilities**      | Lombok                                                                |
| **Testing**        | JUnit 5, Spring Boot Starter Test                                     |

---

## 🗂️ Architecture, Modules & ERD

A conventional Spring Boot stack: React client → `JwtAuthenticationFilter` → Controller → Service → JPA Repository → MySQL.

🟢 marks modules I owned end-to-end.

```
src/main/java/com/iherbyou/
├── auth/   user/   catalog/   ordering/   payment/   promotion/   community/   banner/
├── cart/        🟢 guest + user cart with merge
├── wishlist/    🟢 wishlist with 72h snapshot-based sharing
├── security/    🟢 JWT + CORS + filter chain
└── config/      🟢 OpenAPI / Swagger
```

![ERD Diagram](./docs/erd.png)

I drew the initial ERD and designed the cart, wishlist, and catalog-variant tables; it evolved as my teammates added their modules.

**Teammates' modules:** Catalog (products, variants, search), User & Auth (signup, JWT issuance, address book), Orders & Payments (Toss Payments **sandbox**, no real charges), Promotions (coupons, points), Community (reviews, Q&A).

---

## 🔌 API Endpoints

- `/api/cart` — guest/user unified access, guest-to-user merge on login
- `/api/wishlist` — CRUD (20-item cap), 72h snapshot-based share links
- `/api/auth/**` — Spring Security filter chain, JWT bearer auth, CORS

Swagger UI: `/swagger-ui/index.html`.

---

## 🛠️ Getting Started

Requires Java 21, MySQL 8.0, Gradle 8.5. Configure DB / JWT / mail / Toss sandbox in `application.yaml`, then:

```bash
./gradlew bootRun          # local
docker-compose up -d       # Docker (requires .env)
```
