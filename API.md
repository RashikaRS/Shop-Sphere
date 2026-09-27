# ShopSphere API Reference

Base URL: `http://localhost:8080` (or your deployed backend URL). All request/response
bodies are JSON. Protected endpoints require `Authorization: Bearer <jwt>`.

## Auth (`/api/auth`) — public
| Method | Path | Body | Notes |
|---|---|---|---|
| POST | `/api/auth/register` | `{fullName, email, password, phone}` | Returns JWT + user info |
| POST | `/api/auth/login` | `{email, password}` | Returns JWT + user info |

## Users (`/api/users`) — logged in
| Method | Path | Notes |
|---|---|---|
| GET | `/api/users/me` | Current user's profile |

## Products (`/api/products`) — GET public, write admin-only
| Method | Path | Query params | Notes |
|---|---|---|---|
| GET | `/api/products` | `search, categoryId, minPrice, maxPrice, sortBy(createdAt|price|rating|name), sortDir(asc|desc), page, size` | Paginated catalog |
| GET | `/api/products/{id}` | | Product detail with images |
| GET | `/api/products/{id}/recommendations` | `limit` | Related products |
| GET | `/api/products/{id}/reviews` | `page, size` | Public, paginated |
| POST | `/api/products/{id}/reviews` | body `{rating, comment}` | Logged in — upserts your review |
| POST | `/api/products` *(admin)* | `{name, description, price, stockQuantity, categoryId, imageUrls[]}` | |
| PUT | `/api/products/{id}` *(admin)* | same shape | |
| DELETE | `/api/products/{id}` *(admin)* | | |

## Categories (`/api/categories`) — GET public, write admin-only
| Method | Path | Notes |
|---|---|---|
| GET | `/api/categories` | List all |
| POST / PUT / DELETE | `/api/categories[/{id}]` *(admin)* | CRUD |

## Reviews (`/api/reviews`)
| Method | Path | Notes |
|---|---|---|
| DELETE | `/api/reviews/{id}` | Owner or admin only |

## Cart (`/api/cart`) — logged in
| Method | Path | Body | Notes |
|---|---|---|---|
| GET | `/api/cart` | | Current user's cart |
| POST | `/api/cart/items` | `{productId, quantity}` | Adds/merges, capped at stock |
| PUT | `/api/cart/items/{itemId}?quantity=N` | | Update quantity (0 removes) |
| DELETE | `/api/cart/items/{itemId}` | | Remove one line |
| DELETE | `/api/cart` | | Clear cart |

## Wishlist (`/api/wishlist`) — logged in
| Method | Path | Notes |
|---|---|---|
| GET | `/api/wishlist` | |
| POST | `/api/wishlist/items/{productId}` | 409 if already present |
| DELETE | `/api/wishlist/items/{itemId}` | |

## Addresses (`/api/addresses`) — logged in, scoped to user
| Method | Path | Notes |
|---|---|---|
| GET / POST / PUT / DELETE | `/api/addresses[/{id}]` | Full CRUD, one "default" enforced |

## Coupons (`/api/coupons`)
| Method | Path | Access | Notes |
|---|---|---|---|
| GET | `/api/coupons` | admin | List all |
| POST / PUT / DELETE | `/api/coupons[/{id}]` | admin | CRUD |
| POST | `/api/coupons/apply` | logged in | `{code, cartTotal}` → preview discount |

## Orders (`/api/orders`) — logged in, scoped to user
| Method | Path | Body | Notes |
|---|---|---|---|
| POST | `/api/orders/checkout` | `{addressId, couponCode?}` | Validates stock, creates order (`PENDING`) |
| GET | `/api/orders?page=&size=` | | Order history |
| GET | `/api/orders/{id}` | | Order detail with tracker status |

## Payments (`/api/payments`) — logged in
| Method | Path | Notes |
|---|---|---|
| POST | `/api/payments/create-order/{orderId}` | Creates Razorpay test order |
| POST | `/api/payments/verify` | `{orderId, razorpayOrderId, razorpayPaymentId, razorpaySignature}` — server-verifies signature |
| GET | `/api/payments/status/{orderId}` | Current payment/order status |

## Admin (`/api/admin`) — ROLE_ADMIN only
| Method | Path | Notes |
|---|---|---|
| GET | `/api/admin/dashboard/stats` | Users, products, orders, revenue, pending orders, low stock, orders-by-status |
| GET | `/api/admin/products/low-stock?page=&size=` | Products at/under 10 units |
| GET | `/api/admin/orders?status=&page=&size=` | All orders, optional status filter |
| PUT | `/api/admin/orders/{id}/status` | `{status}` — PENDING/PAID/SHIPPED/DELIVERED/CANCELLED |
| GET | `/api/admin/users` | All users with roles/status |
| PUT | `/api/admin/users/{id}/enable` \| `/disable` | Toggle account access (disabling also invalidates their live JWT) |

## Error format
All errors return:
```json
{
  "timestamp": "2026-08-27T10:00:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Human-readable explanation"
}
```
`401` = not logged in / expired token. `403` = logged in but missing the required role.
`404` = resource not found or not owned by you. `409` = conflict (duplicate email, duplicate
wishlist item, duplicate coupon code).
