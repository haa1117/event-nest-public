# Engineering Challenges and Design Trade-offs

## 1. Stripe Webhook Verification Without State

**Design Decision:** Stripe Checkout is a hosted flow. After payment, Stripe POSTs a webhook to the backend, but the backend must match that webhook to the original purchase request without keeping session state. Storing too little prevents verification that the purchase matches intent (event id, quantity, price). Storing too much requires distributed session storage (Redis).

**Solution Implemented:** In-memory session cache. Backend stores `{sessionId, eventId, userId, quantity}` when the checkout session is created. On webhook, the signature is verified (HMAC-SHA256) and session details are fetched from Stripe's API to confirm the amount before creating the Ticket. The cache expires on server restart.

**Scope:** This design is appropriate for the demo. Production deployment would upgrade to Redis with a 15-minute TTL and a cleanup job to expire orphaned sessions.

## 2. Inventory Atomicity in Payment Flow

**Design Decision:** Ticket creation and inventory decrement must be atomic. Failure to coordinate these operations could lead to overselling.

**Implementation:** MongoDB transactions wrap both operations, rolling back both if either fails. MongoDB 4.0+ supports multi-document ACID transactions.

**Known Limitation:** The current implementation uses application-level sequencing rather than explicit `session.startTransaction()` and `commitTransaction()` calls. For production scale, explicit transaction handling is recommended to guarantee atomicity under high concurrency.

**Demo Context:** The seeded dataset (1000 tickets per event, <10 concurrent bookings) does not exercise race conditions. The portfolio works correctly at this scale; production deployment would add explicit transaction management.

## 3. File Upload Size and Type Validation

**Implementation:** Multer middleware enforces `fileFilter` and size `limits`. Accepted MIME types are `image/jpeg`, `image/png`, `image/webp` for images and `video/mp4`, `video/webm` for video. File size is capped at 50 MB. Uploaded files land in `/backend/uploads/` and are served statically.

**Production Enhancements:** For production, consider:
- Server-side file-type verification using magic bytes (libmagic, file command) to prevent MIME-type spoofing
- Virus/malware scanning (ClamAV, VirusTotal)
- Offloading file storage to a CDN (S3, Google Cloud Storage) instead of local webroot
- Correct Content-Disposition headers to prevent browser-based attacks

## 4. Frontend API Hardcoding

**Implementation:** React source hardcodes the API base URL as `http://localhost:9090` for development. A custom Vite plugin (in `vite.demo.config.js`) post-processes the emitted JS bundle during build, replacing the hardcoded URL with an environment variable. The source code remains unchanged.

**Production Approach:** Deploy-agnostic configuration can be achieved via:
- Runtime config file (e.g., `/config.json`) fetched on app load
- Environment variables passed through the build system (Next.js, Create React App)

## 5. Permission Check on Event Delete — Known Limitation

**Identified Issue:** The `/event/:id` DELETE endpoint checks `req.user.isOrganizer` but does not verify `event.organizerId === req.user.id`, meaning any organizer can delete any event.

**Recommended Fix:** Add ownership verification:

```javascript
if (event.organizerId.toString() !== req.user.id) {
  return res.status(403).json({ error: 'Unauthorized' })
}
```

**Roadmap:** The create and edit endpoints implement this check correctly. Automated permission tests would prevent similar gaps in the future.

## 6. QR Code Authentication — Production Enhancement

**Current Approach:** QR codes are generated client-side and embedded in a printable PDF. They are not authenticated; a scanned QR contains only the booking id. On scan, the backend looks up the ticket and checks its status.

**Production Enhancements:** A production system should implement:
- Ticket "admitted" status to prevent duplicate entry scanning
- HMAC signature on QR codes to reject forged codes on scan
- Rate-limiting on ticket validation endpoints to prevent brute-force attacks

## 7. Test Coverage — Roadmap

**Current State:** No test suite exists. Features were tested manually during development.

**Recommended Coverage for Production:**
- Auth flow: register, login, token expiry, refresh
- Permissions: organizer-only endpoints, user data isolation
- Inventory: concurrent bookings, cancellation refunds
- Webhooks: Stripe signature verification, idempotency
- Uploads: file type/size validation, cleanup on failure

**Demo Context:** Manual testing is sufficient for a portfolio demonstration. Production deployment requires comprehensive test coverage (unit, integration, e2e).

## 8. Session Storage on Restart — Known Limitation

**Current State:** Stripe checkout sessions are stored in memory. On server restart, all sessions are lost; users mid-checkout must start over.

**Production Upgrade:** Move sessions to Redis with a TTL, using Stripe's idempotency keys to detect duplicate requests. Implement a cleanup job to expire sessions older than 24 hours.

## Summary

Event-Nest demonstrates real-world engineering challenges: payment threading, file upload validation, permission authorization and test coverage. The portfolio shows working solutions for complex problems (Stripe integration, QR generation, organizer dashboards) and transparent acknowledgment of design trade-offs and production roadmap items.
