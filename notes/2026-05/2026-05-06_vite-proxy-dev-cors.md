# Why Vite dev server needs a proxy (and production doesn't)

In development, your frontend runs on `localhost:5173` and your API server on `localhost:3000`. The browser blocks cross-origin requests — that's CORS. Vite's dev proxy solves this by forwarding `/api/*` requests to the backend, making them look same-origin to the browser:

```js
// vite.config.ts
server: {
  proxy: {
    '/api': 'http://localhost:3000'
  }
}
```

In production this proxy disappears. The built frontend is just static files (HTML/JS/CSS) — no port, no process. Nginx or a CDN serves them, and reverse-proxies `/api/*` to the backend the same way Vite did in dev:

```
user → Nginx :80
         ├── /api/*  → backend :3000
         └── /*      → dist/ static files
```

One port from the user's perspective, no CORS issue. The backend port is always real and needed; the frontend "port" only exists during development because Vite's dev server is a live process.
