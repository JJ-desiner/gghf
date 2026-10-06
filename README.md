Store API — Sample Backend Software Engineering Project

A production-style REST API for a small online store, built with Python, FastAPI and SQLAlchemy. It covers the core skills of backend engineering: API design, authentication, authorization, database modelling, validation, business logic, automated testing and containerisation.

Features
JWT authentication — register, log in, get the current user
Role-based access — admins manage products and order status; customers place orders
Product catalogue — CRUD, search by name, filter by category, pagination
Orders — multi-item orders, stock checking and reservation, totals computed server-side, cancellation that returns stock, customers can only see their own orders
Validation — Pydantic schemas reject bad input (negative prices, short passwords, invalid status…)
17 automated tests with pytest, each on a fresh in-memory database
Docker + PostgreSQL setup via docker-compose; SQLite by default for local development
Auto-generated API docs at /docs (Swagger UI) and /redoc
Project structure
store-api/
├── app/
│   ├── main.py            # FastAPI app, router registration, startup
│   ├── config.py          # Settings from environment variables / .env
│   ├── database.py        # Engine, session, Base, get_db dependency
│   ├── models.py          # ORM models: User, Product, Order, OrderItem
│   ├── schemas.py         # Pydantic request/response schemas
│   ├── security.py        # Password hashing, JWT, auth dependencies
│   └── routers/
│       ├── auth.py        # /auth/register, /auth/login, /auth/me
│       ├── products.py    # /products CRUD + search/pagination
│       └── orders.py      # /orders place, list, view, cancel, status
├── tests/                 # pytest suite (auth, products, orders)
├── seed.py                # Sample admin user + products
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── .env.example
Running locally
bash
python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt

python seed.py                    # optional: admin@example.com / admin12345 + 5 products
uvicorn app.main:app --reload

Open http://localhost:8000/docs to explore and try every endpoint. Click Authorize and log in with the seeded admin to call protected routes.

Note: when not using seed.py, the first user to register becomes the admin.

Running the tests
bash
pytest -v
Running with Docker (PostgreSQL)
bash
docker compose up --build
API endpoints (all prefixed with /api/v1)
Method	Endpoint	Access	Description
POST	/auth/register	Public	Create an account
POST	/auth/login	Public	Get a JWT (form fields username, password)
GET	/auth/me	User	Current user's profile
GET	/products	Public	List, ?q=, ?category=, ?page=, ?size=
GET	/products/{id}	Public	Product details
POST	/products	Admin	Create product
PATCH	/products/{id}	Admin	Partial update
DELETE	/products/{id}	Admin	Delete product
POST	/orders	User	Place an order
GET	/orders	User	List my orders
GET	/orders/{id}	Owner/Admin	View an order
POST	/orders/{id}/cancel	Owner/Admin	Cancel a pending order (restores stock)
PATCH	/orders/{id}/status	Admin	Set status: pending/paid/shipped/delivered/cancelled
Example requests
bash
# Log in
curl -X POST localhost:8000/api/v1/auth/login \
  -d "username=admin@example.com&password=admin12345"

# Place an order
curl -X POST localhost:8000/api/v1/orders \
  -H "Authorization: Bearer <TOKEN>" -H "Content-Type: application/json" \
  -d '{"items":[{"product_id":1,"quantity":2}]}'
Design decisions
Layered structure — routers (HTTP), schemas (validation), models (persistence) and security are kept separate.
Money as Decimal — never floats, so totals are exact.
Server-side pricing — the client only sends product IDs and quantities; prices come from the database.
Atomic orders — stock checks and decrements happen in one transaction; if any line fails, nothing is saved.
404 instead of 403 for other users' orders — doesn't reveal that the order exists.
Dependency injection — the test suite swaps the real database for an in-memory one via dependency_overrides.
Possible extensions
Alembic migrations instead of create_all
Refresh tokens and password reset
M-Pesa / Stripe payment integration
Row-level locking (SELECT … FOR UPDATE) for high-concurrency stock updates
Rate limiting, structured logging, CI pipeline (GitHub Actions)
