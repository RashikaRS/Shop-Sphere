# ShopSphere — Complete (Phases 1-10)

A production-style e-commerce app: **React + Vite** frontend, **Spring Boot** backend,
**MySQL** database, **Razorpay** test-mode payments. Built in phases; this README covers
everything delivered end to end.

## Feature summary
- **Auth**: JWT register/login, BCrypt hashing, ROLE_USER/ROLE_ADMIN, protected routes
  (backend + frontend)
- **Catalog**: search, category filter, price filter, sort, pagination, product detail,
  image gallery, 12 demo products auto-seeded
- **Cart & Wishlist**: full CRUD, stock-capped quantities, duplicate wishlist protection
- **Addresses, Checkout, Orders**: multi-address support, transactional checkout with
  stock validation, order history with a visual status tracker
- **Payments**: real Razorpay test-mode flow with **server-side signature verification**
  (never trusts the frontend alone), retry-on-failure flow
- **Reviews & Ratings**: one review per user per product, auto-recalculated product rating
- **Coupons**: percent/flat discounts, min order value, per-user usage limits, date ranges,
  applied for real at checkout
- **Recommendations**: same-category, top-rated-first suggestions
- **Admin dashboard**: revenue/order/user/product stats, low-stock alerts, order status
  management, user enable/disable, product & coupon CRUD — all in one tabbed UI
- **UI polish**: global toast notifications, mobile hamburger nav, error boundary, 404 page,
  consistent JSON error responses (401/403/etc.) from the backend
- **Security hardening**: disabling a user immediately invalidates their live JWT (not just
  future logins), custom auth entry point / access denied handler return clean JSON instead
  of Spring's default HTML/empty responses
- **Testing**: backend unit tests for coupon discount logic (`mvn test`)
- **Docs**: this README + `API.md` (full endpoint reference)

## Project structure
```
shopsphere/
  backend/     Spring Boot REST API (Java 17, Maven)
    src/main/java/com/shopsphere/
      controller/   REST endpoints
      service/      Business logic
      repository/   Spring Data JPA repositories
      entity/       JPA entities
      dto/          Request/response objects
      security/     JWT filter, UserDetails, entry points
      config/       Security, CORS, Razorpay, demo data seeder
      exception/    Global exception handling
    src/test/java/  Unit tests
  frontend/    React + Vite SPA
    src/
      components/   Navbar, ProductCard, ReviewsSection, ErrorBoundary, etc.
      pages/        One file per route
      context/      Auth, Cart, Toast providers
      services/     API call wrappers (axios)
  database/
    schema.sql   Full DB schema + seed roles/categories
  API.md         Full endpoint reference
  README.md      This file
```

## Setup — Replit (recommended for mobile)

### A. Database (MySQL)
Replit doesn't host MySQL directly — use a free managed instance (Railway, Aiven, or
Clever Cloud all have free tiers with a web SQL console, no local `mysql` CLI needed):
1. Create the database, copy its host/port/username/password/db-name.
2. Paste the contents of `database/schema.sql` into its web SQL console and run it.

### B. Backend Repl
1. New Repl → import `backend/` (Java/Maven).
2. In Replit **Secrets**, set everything from `backend/.env.example`:
   `DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD, CORS_ALLOWED_ORIGINS, JWT_SECRET,
   RAZORPAY_KEY_ID, RAZORPAY_KEY_SECRET`
3. Run `mvn spring-boot:run`.
4. Verify: `https://<backend-repl-url>/api/health` → `{"status":"UP",...}`

### C. Frontend Repl
1. New Repl → import `frontend/` (Node.js).
2. Copy `.env.example` → `.env`, set `VITE_API_BASE_URL=https://<backend-repl-url>`.
3. Run `npm install && npm run dev`.
4. Open the preview — the homepage should show the backend status in green.

### Local alternative
```
# Backend
cd backend && cp .env.example .env   # edit values
mvn spring-boot:run

# Frontend
cd frontend && cp .env.example .env  # edit values
npm install && npm run dev
```

## Setting up Razorpay test mode
1. Sign up at https://dashboard.razorpay.com, toggle **Test Mode** (top right).
2. Settings → API Keys → Generate Test Key → copy Key ID + Key Secret.
3. Set `RAZORPAY_KEY_ID` / `RAZORPAY_KEY_SECRET` in backend Secrets.
4. Test card: **4111 1111 1111 1111**, any future expiry, any CVV. No real money moves.

## Making yourself an admin
Register normally on the site, then run against your database:
```sql
INSERT INTO user_roles (user_id, role_id)
SELECT u.id, r.id FROM users u, roles r WHERE u.email = 'your@email.com' AND r.name = 'ROLE_ADMIN';
```
Log out and back in — an **Admin** link appears in the Navbar.

## Testing
Backend unit tests (coupon discount logic — no DB required):
```
cd backend
mvn test
```
Frontend: no test runner is wired up yet by default. To add one, install Vitest +
React Testing Library (`npm install -D vitest @testing-library/react jsdom`) and add a
`test` script to `package.json` — the component structure (one file per page/component,
services separated from UI) is already test-friendly.

## Full verification checklist (all phases)
1. **Auth**: register → auto-logged in → refresh stays logged in → logout → login again works
2. **Catalog**: search/filter/sort/paginate on `/products`; product detail loads images
3. **Cart/Wishlist**: add to cart updates Navbar badge instantly; wishlist blocks duplicates;
   "Move to Cart" works
4. **Checkout**: add/edit/delete addresses; place an order; stock decrements; cart clears
5. **Payment**: Razorpay popup opens with correct amount; test card succeeds → order shows
   `PAID`; cancelling → order stays `PENDING` with a working **Retry Payment**
6. **Reviews**: submit a review → product rating updates; resubmitting updates instead of
   duplicating; recommendations strip shows related products
7. **Coupons**: `WELCOME10` applies a real discount at checkout; reuse is blocked; under
   minimum order value is blocked
8. **Admin**: `/admin` shows live stats, low-stock list, order status updates, user
   enable/disable (disabling immediately blocks that user's existing session too), product
   and coupon management — all blocked for non-admins (403 with clean JSON)
9. **Mobile**: resize below ~700px — Navbar collapses into a hamburger menu; layouts reflow
10. **Resilience**: kill the backend and try any action — a toast reports the network error
    instead of a silent failure; visiting a bad URL shows a 404 page instead of a blank screen

## GitHub upload steps
```
cd shopsphere
git init
git add .
git commit -m "ShopSphere - full e-commerce app (phases 1-10)"
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```
`.gitignore` already excludes `node_modules/`, `target/`, and any local `.env` files, so
secrets are never committed. Double-check `git status` before your first commit if unsure.

## Deployment (beyond Replit)
**Backend** (Spring Boot / Maven) — any of these work well on free tiers:
- **Railway**: New Project → Deploy from GitHub → it detects the Maven build automatically →
  add the same env vars as your `.env.example` in its Variables tab.
- **Render**: New Web Service → connect the repo → Build Command `mvn clean package -DskipTests`,
  Start Command `java -jar target/shopsphere-backend-1.0.0.jar` → add env vars.

**Frontend** (Vite/React) — any static host works:
- **Vercel** or **Netlify**: import the repo, set root directory to `frontend/`, build
  command `npm run build`, output directory `dist/`, add `VITE_API_BASE_URL` pointing at
  your deployed backend URL.

**Getting a live URL**: once both are deployed, update the backend's
`CORS_ALLOWED_ORIGINS` to your live frontend URL (no trailing slash), and the frontend's
`VITE_API_BASE_URL` to your live backend URL, then redeploy both. Test `/api/health` on
the backend URL and the homepage on the frontend URL to confirm they're talking to each other.

## Security notes
- No secrets are committed — everything sensitive comes from environment variables
  (`backend/.env.example`, `frontend/.env.example`).
- Passwords are BCrypt-hashed; password hashes are never returned in any API response.
- JWT-based stateless auth; role checks enforced both by URL pattern (`SecurityConfig`) and
  ownership checks in the service layer (e.g., you can't view someone else's order/address).
- Razorpay payments are verified **server-side** using the key secret — the frontend
  callback is never trusted on its own.
- Custom `AuthenticationEntryPoint`/`AccessDeniedHandler` return consistent JSON errors
  instead of Spring's default HTML/empty error pages.
- Disabling a user (admin action) immediately blocks their already-issued JWT, not just
  future logins.
- **Known limitations / good next steps if hardening further**: no rate limiting on
  auth endpoints (consider adding e.g. bucket4j or a reverse-proxy rate limiter in
  production), no email verification on registration, no refresh-token rotation (JWTs are
  long-lived — 24h by default via `JWT_EXPIRATION_MS`), no CSRF tokens (acceptable here
  since the API is stateless JSON with no cookie-based auth, but worth knowing).

## Roadmap — all phases complete
1. ✅ Architecture, DB schema, backend/frontend setup
2. ✅ Authentication, JWT, roles
3. ✅ Product management, search, filters, pagination
4. ✅ Cart, wishlist
5. ✅ Addresses, checkout, orders
6. ✅ Razorpay test payment, verification, status
7. ✅ Reviews, ratings, coupons, recommendations
8. ✅ Admin dashboard, analytics, inventory, order management
9. ✅ UI polishing, responsive design, error handling, security improvements
10. ✅ Testing, GitHub upload, deployment instructions
