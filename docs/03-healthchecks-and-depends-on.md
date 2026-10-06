# Health Checks and Startup Dependencies

## Health-check methods

- **database (MySQL):** `mysqladmin ping` against localhost, authenticated as root — a real
  readiness check confirming MySQL accepts connections, not just that the process exists.
- **backend (Node/Express):** `wget --spider` against `http://localhost:8080/health`.
  The `/health` route itself calls `db.sequelize.authenticate()`, so a 200 response means
  both Express and the database connection are genuinely working.
- **frontend (Nginx):** `wget --spider` against `http://localhost:80/`, confirming Nginx
  is serving content.
- **reverse-proxy (Nginx):** `wget --spider` against `http://localhost:80/health`, routed
  through to the backend's own health endpoint — this proves the full proxy chain works,
  not just that Nginx itself is alive.

wget is used instead of curl because the Alpine-based images in this stack do not include
curl by default.

## Startup dependency order

1. database starts first; backend's `depends_on: database: condition: service_healthy`
   blocks backend from starting until MySQL actually responds to `mysqladmin ping`,
   not merely until the container process has started.
2. frontend has no dependency, since it only serves static files with no backend
   requirement.
3. reverse-proxy depends on both frontend and backend reaching `service_healthy`
   before it starts, ensuring it never routes traffic to a service that isn't
   actually ready yet.

This matters because a plain `depends_on` list only waits for a container to start,
not for the application inside it to be ready. Using `condition: service_healthy`
throughout closes that gap at every stage of the stack.
