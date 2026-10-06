# Proxy Routing and CORS

## Reverse proxy selected
Nginx (nginx:1.27-alpine), configured in proxy/nginx.conf.

## Routing rules
- /assets/* -> proxied to the frontend service (Nginx container serving static files)
- Everything else (/, /cart, /gallery, /category/*, /api/*, /health) -> proxied to the
  backend service (Node/Express), which handles both page rendering and API requests.

Both proxy_pass targets use Docker Compose service names (frontend, backend), resolved
via Docker's internal DNS - no static IP addresses are used.

## CORS
Not required and not configured. The browser only ever communicates with one origin:
http://<VM_PUBLIC_IP> on port 80, served entirely through the reverse proxy. The proxy
internally forwards requests to two different containers (frontend, backend), but this
is invisible to the browser - from its perspective every request goes to the same
origin, so no cross-origin request is ever made and no CORS headers are needed.
