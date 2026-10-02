# EpicBook — Environment Variables, Ports, Persistence, Health Check

## Internal Ports
- Backend (Node/Express): 8080 — confirmed in server.js (`process.env.PORT || 8080`) and the repo's own install guide
- Frontend (static assets, served by Nginx): 80 (internal, container-only)
- Database (MySQL): 3306 (internal only, never published to the host)
- Reverse proxy (public entry point): 80

## Environment Variables
The backend does not read discrete DB_HOST/DB_USER/DB_PASS/DB_NAME variables.
Confirmed from models/index.js and config/config.json:

- NODE_ENV — must be set to `production` to activate env-var-based DB config
- JAWSDB_URL — single MySQL connection string:
  mysql://<user>:<password>@database:3306/bookstore
- PORT — optional, defaults to 8080

MySQL container (standard Docker image variables):
- MYSQL_ROOT_PASSWORD
- MYSQL_DATABASE=bookstore   (must match config.json and db/BuyTheBook_Schema.sql exactly)
- MYSQL_USER
- MYSQL_PASSWORD

## API Route Prefix
/api — confirmed in routes/cart-api-routes.js

## Health-Check Endpoint
None exists yet. Added in Task 3 as GET /health, returning HTTP 200.

## Persistent Data Path
MySQL data directory: /var/lib/mysql — backed by named volume db_data

## Database Initialization Files
db/BuyTheBook_Schema.sql, db/author_seed.sql, db/books_seed.sql
Mounted into MySQL's /docker-entrypoint-initdb.d/.
Alphabetical execution order correctly runs schema before author seed before book seed
(books have a foreign-key dependency on authors).
