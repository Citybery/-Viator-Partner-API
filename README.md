
# Viator Partner API Integration - Requirements Document

## 🎯 Project Overview

This project is an isolated Linux-native .NET 9 application designed to integrate with the Viator Partner API. It will retrieve data such as products, bookings, availability, and cancellations from Viator and store them in a PostgreSQL database. It also provides secure API endpoints for internal service consumption and an admin UI for service status and logs.

---

## 🔧 Technologies and Architecture

| Component           | Description                                                           |
|---------------------|-----------------------------------------------------------------------|
| Backend             | .NET 9 (Native AOT, Linux binary output)                             |
| Hosting             | Linux (systemd service)                                               |
| Reverse Proxy       | Nginx (TLS termination and routing)                                  |
| Admin UI            | .NET MVC (No login, log and service status only)                     |
| Database            | PostgreSQL 16.8+                                                     |
| Logging             | Serilog (custom sink configuration)                                  |
| Authentication      | None (IP whitelisting for external access)                           |
| API Documentation   | Swagger (OpenAPI spec-based)                                         |
| CI/CD               | Azure DevOps with `azure-pipelines.yml`                              |
| Testing             | Unit Tests + Azure DevOps Test Cases (https://learn.microsoft.com/en-us/azure/devops/test/?view=azure-devops) |

---

## 🔌 I/O Interfaces

### 1. Public APIs (IP Whitelisted Access)

| Endpoint                         | Description                                               |
|----------------------------------|-----------------------------------------------------------|
| `GET /api/availability/{id}`     | Check availability of a product                          |
| `POST /api/cart/hold`            | Create a cart hold                                       |
| `POST /api/bookings`             | Create a booking                                         |
| `GET /api/cart/holds`            | List current cart holds                                  |
| `GET /api/bookings`              | List bookings                                             |
| `GET /api/bookings/status`       | Check booking status                                     |
| `POST /api/bookings/cancel`      | Cancel a booking                                         |
| `GET /api/bookings/cancellations`| List cancelled bookings                                  |

### 2. Synchronization Endpoints (Triggered from External Services)

Each sync operation is triggered separately and writes to PostgreSQL.

| Endpoint                       | Description                                                   |
|--------------------------------|---------------------------------------------------------------|
| `POST /api/sync/products`      | Fetch product list from Viator and store                     |
| `POST /api/sync/availability`  | Fetch product availability                                   |
| `POST /api/sync/bookings`      | Fetch and store booking data                                |
| `POST /api/sync/cancellations` | Fetch and store cancellation data                           |
| `POST /api/sync/cart-holds`    | Fetch cart holds from Viator                                |

> 🔁 There is no `/sync-all` endpoint. All sync calls must be triggered independently.

---

## 🧩 Database Schema

- Database: **PostgreSQL 16.8+**
- Schema: Will match Viator response structure
- Key tables:
  - `products`
  - `availabilities`
  - `bookings`
  - `booking_status`
  - `cart_holds`
  - `cancellations`

---

## 📜 Logging

- Technology: **Serilog**
- Output: File, Seq, or external log sink
- Logged Events:
  - HTTP requests
  - Synchronization operations
  - API events
  - Errors and exceptions

---

## 🧾 Admin UI

- Technology: .NET MVC
- No login/authentication
- Views:
  - Service Health Status
  - Log Viewer
- ❌ No manual trigger buttons

---

## ⚙️ Build and Deployment

- Output: Native Linux binary (not DLL)
- Deployment: systemd service
- CI/CD: Azure DevOps
  - Includes `azure-pipelines.yml`
  - Runs Unit Tests
  - Publishes binary

---

## 🧪 Testing

- Framework: xUnit or NUnit
- Azure DevOps:
  - Test Cases configured via Azure Test Plans
  - Results reported in pipeline
- Must include:
  - Mocked API calls
  - Sync logic tests
  - Validation and error case tests

---

## ✅ Deliverables

- Full source code (.NET 9, Native AOT)
- PostgreSQL-ready schema
- Admin UI (login-less)
- Swagger-enabled API endpoints
- `azure-pipelines.yml` file
- Azure-compatible Unit Tests + Test Cases
- `README.md` with setup, environment, and usage
- systemd service file example


---

## 🔄 Simple Workflow

Below is a simple, high-level workflow that describes how the system operates:

### 🔁 1. Sync Products from Viator
- `POST /api/sync/products` is triggered manually or by a scheduler.
- It fetches product list from Viator `/products/bulk` endpoint (paginated).
- Each product is stored or updated in the PostgreSQL `products` table.

### 📅 2. Check Availability
- Internal service sends `GET /api/availability/{productId}`.
- The system queries Viator's availability API.
- Response is returned directly without DB write.

### 🛒 3. Hold Cart & Create Booking
- User selects a product in the frontend.
- A call is made to `POST /api/cart/hold` to initiate the reservation.
- Once confirmed, `POST /api/bookings` is used to finalize the booking.
- Booking is stored via `/api/sync/bookings`.

### 📥 4. Scheduled Sync Jobs (Cron or External Trigger)
- Triggers:
  - `POST /api/sync/bookings`
  - `POST /api/sync/cart-holds`
  - `POST /api/sync/cancellations`
- All data fetched is stored in PostgreSQL in their respective tables.

### ❌ 5. Cancel Booking
- A cancellation request is sent to `POST /api/bookings/cancel`.
- The result is also fetched via `GET /api/bookings/cancellations`.

---

✅ Each API call to Viator uses an API key via HTTP header.
✅ All external access is protected via IP whitelisting.
✅ All data writes are persisted in PostgreSQL 16.8+.
✅ All activity is logged with Serilog and visible from the Admin UI.
