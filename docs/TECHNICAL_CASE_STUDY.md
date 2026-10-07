# Technical Case Study

## End-to-End Payment Flow

The trickiest part of Event-Nest is threading the Stripe Checkout session through purchase confirmation without keeping server state.

**Challenge:** Stripe Checkout is hosted and client-side cannot verify the outcome reliably. Backend must wait for the webhook and match it to the purchase request.

**Solution:** On checkout initiation, backend generates a Stripe session and stores `{sessionId, eventId, userId, quantity}` in a temporary cache (in-memory map; production would use Redis). When Stripe POSTs the payment completion webhook (with `session.completed` event), we verify the signature (HMAC-SHA256) and fetch the session object from Stripe's API to confirm amount and currency. Only then do we create the Ticket record and decrement inventory.

This avoids:
- Creating tickets on client redirect (unverified)
- Trusting the webhook amount (we re-verify against Stripe's session)
- Leaving orphaned sessions if the user closes the browser

**Result:** Atomic ticket creation on verified payment; inventory never oversells.

## QR Code Distribution

Attendees need printable, offline e-tickets. The team chose client-side QR generation over server-side rendering.

**Choice:** `react-qr-code` renders QR codes as SVG on the detail page; `react-to-print` captures the whole ticket card (name, event, QR, barcode metadata) and opens the browser print dialog. No backend PDF generation.

**Trade-off:** No backend load, instant preview, but requires the browser to run JS to generate the code. Offline access works only if the user visits the ticket page while online.

**Implementation:** Ticket data (event id, user id, quantity) is hashed into the QR; no server lookup needed to validate a scan. Organizer dashboards simply count tickets per event.

## Inventory Accounting

Stock management happens at two points: ticket creation (decrement) and cancellation (restore).

**Problem:** What if a user creates a session but never pays? Inventory stays decremented.

**Current State:** No cleanup job exists. Sessions live in memory and are lost on server restart. Production should add a 15-minute session expiry and a background task to restore orphaned inventory.

**Demo Limitation:** This demo runs with seeded events (1000 tickets each); depletion is not realistic concern. Real deployment would need persistent session storage (Redis) and garbage collection.

## Multipart Upload

Images and videos are uploaded via `multer` middleware. No client-side validation; backend accepts jpg/png/webp for images, mp4/webm for video, up to 50MB.

**No CDN:** Files are stored locally in `/uploads/` and served via Express static middleware. Production deployment scales this to S3 or Google Cloud Storage, but the portfolio runs locally.

**Demo State:** Cover images for seeded events are placed in `/uploads/images/demo-*.{avif,jpg}` at setup time. No real uploads are made during demo use.

## Search & Filter

The dashboard aggregates search and category filter into a single query:

```
GET /event/search?q=music&category=concert
```

Backend uses MongoDB regex for text search (case-insensitive substring match, not full-text index). Categories are exact match on the enum field. Results paginate via skip/limit (not shown in this snippet; could be improved).

**Trade-off:** Simple regex search is slow on large collections (100k+ events). Production would add MongoDB text indexes or switch to Elasticsearch. The demo dataset is small (< 50 seeded events), so latency is imperceptible.

## Organizer Dashboard

My Events page shows title, date, status, revenue and "tickets sold". The last metric reads a non-existent field and always displays 0.

**Bug:** Code attempts to sum `event.ticketsSold` across tickets, but the field is never populated. Correct implementation would be:

```javascript
const sold = tickets.reduce((sum, t) => sum + t.quantity, 0)
```

This was identified in source review but not fixed in the portfolio build. It demonstrates a real bug caught during code inspection.

## Node 26 Compatibility

The `jsonwebtoken` dependency (JWT signing/verification) ships with a SlowBuffer shim for Node versions < 14. Node 26 removed SlowBuffer, causing the require to fail.

**Workaround:** A shim is preloaded at startup:

```bash
node --require ./analysis/demo-seed/slowbuffer-shim.js backend/server.js
```

The shim defines `Buffer.allocUnsafe()` as an alias, restoring compatibility without upgrading dependencies. This is a quick fix; production should upgrade jsonwebtoken.

## Frontend API Hardcoding

React code hardcodes the API base URL as `http://localhost:9090`. This breaks on deployment (different domain/port).

**Demo Solution:** Vite config (frontend/vite.demo.config.js) includes a plugin that rewrites the URL at build time:

```javascript
const apiUrl = process.env.API_URL || 'http://localhost:9090'
// plugin finds fetch('http://localhost:9090/event') and replaces the URL
```

Production deployment would use environment variables or a config file instead. The demo avoids modifying source code by intercepting the build.

## State of Testing

No test suite exists. The project was manually tested during development. Proper coverage would include:

- Auth flow (register, login, token refresh)
- Event CRUD with organizer checks
- Inventory decrement/restore on purchase/cancel
- Stripe webhook signature validation
- File upload validation

This is typical for a portfolio build; production apps have comprehensive test coverage.
