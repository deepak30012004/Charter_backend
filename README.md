# Charter Backend

Node.js + Express + MySQL API for the charter flight booking platform.

## Setup

```bash
npm install
cp .env.example .env        # fill in DB_* and JWT_SECRET
npm run db:schema           # creates all tables (drops existing ones first!)
npm run db:seed             # optional: sample airports, operators, aircraft
npm run dev                 # http://localhost:4000
                            # API docs: http://localhost:4000/api-docs
```

## Structure

```
charter-backend/
├── database/schema.sql          MySQL schema (21 tables)
├── database/seed.sql            sample data (npm run db:seed)
├── scripts/run-sql.js           npm run db:schema / db:seed
└── src/
    ├── server.js                starts the server, checks DB connection
    ├── app.js                   express app, middleware, /api routes
    ├── config/                  env variables and business rules
    ├── db/
    │   ├── pool.js              mysql2 pool (UTC)
    │   └── transaction.js       withTransaction(conn => ...)
    ├── middlewares/             auth (JWT + roles), errors, 404
    ├── constants/statuses.js    all status values + operator panel labels
    ├── utils/                   AppError, asyncHandler, geo, time, money
    ├── routes/                  URL -> controller (one file per module)
    ├── controllers/             read request, call service, send response
    ├── services/                business logic
    └── repositories/            SQL queries only
```

Request flow: `route -> controller -> service -> repository -> MySQL`

## Modules

| Module | Status |
|---|---|
| Search (`POST /api/search`) | Done |
| Airports (`GET /api/airports?q=`) | Done |
| Auth (login, customer register, me, change password) | Done |
| Admin: create / manage operators | Done |
| Operator: aircraft, hourly rate, rules, profile | Done |
| Booking requests (customer request, list, cancel) | Done |
| Operator requests + quotes (panel list, quote, decline, customer accepts -> booking) | Done |
| Bookings (confirm / cancel / upcoming) | Stub |
| Payments + admin verification | Stub |
| Offered trips | Stub |

Stubbed endpoints return `501 Not implemented yet`.

## API

| Method | URL | Who |
|---|---|---|
| POST | /api/auth/register | public (customers only) |
| POST | /api/auth/login | public (all roles) |
| GET | /api/auth/me | logged in |
| POST | /api/auth/change-password | logged in |
| POST | /api/admin/operators | super admin |
| GET | /api/admin/operators, /api/admin/operators/:id | super admin |
| PUT | /api/admin/operators/:id | super admin |
| PATCH | /api/admin/operators/:id/status | super admin |
| POST | /api/admin/operators/:id/reset-password | super admin |
| GET | /api/operator/profile | operator |
| GET, PUT | /api/operator/rules | operator |
| GET, POST | /api/operator/aircraft | operator |
| GET, PUT | /api/operator/aircraft/:id | operator |
| PATCH | /api/operator/aircraft/:id/status | operator |
| GET | /api/airports?q= | public |
| POST | /api/search | public |
| POST | /api/booking-requests | customer |
| GET | /api/booking-requests, /api/booking-requests/:id | customer |
| GET | /api/booking-requests/:id/quotes | customer |
| POST | /api/booking-requests/:id/cancel | customer |
| POST | /api/quotes/:id/accept | customer |
| GET | /api/bookings, /api/bookings/:id | customer |
| POST | /api/bookings/:id/cancel | customer, operator, admin |
| POST | /api/payments | customer |
| POST | /api/payments/webhook | payment gateway |
| GET | /api/offered-trips, /api/offered-trips/:id | public |
| POST | /api/offered-trips/:id/book | customer |
| GET | /api/operator/booking-requests(/:id) | operator |
| POST | /api/operator/booking-requests/:id/quote | operator |
| POST | /api/operator/booking-requests/:id/decline | operator |
| GET | /api/operator/bookings/upcoming | operator |
| POST | /api/operator/bookings/:id/confirm | operator |
| GET | /api/admin/payments/pending | super admin |
| POST | /api/admin/payments/:id/verify, /reject | super admin |

## Test logins (after npm run db:seed)

| Role | Email | Password |
|---|---|---|
| Super admin | admin@skyvibe.test | Admin@123 |
| Operator | ops@delhijets.test | Operator@123 |
| Operator | ops@puneair.test | Operator@123 |
| Customer | customer@skyvibe.test | Customer@123 |

## Rules

- All times are UTC. Convert to local time only for display.
- Search never writes to the database.
- Before creating a booking, lock the aircraft (`aircraftRepo.lockById`) inside
  `withTransaction` and re-check availability.
