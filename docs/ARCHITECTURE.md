# Architecture

All diagrams are original and conceptual.

## System Overview

```mermaid
graph TB
    Client["React SPA<br/>(event discovery, checkout,<br/>e-tickets, profile)"]
    Auth["JWT Auth<br/>(token in bearer header)"]
    API["REST API<br/>(Express 5)"]
    Upload["Multer<br/>(multipart image/video)"]
    DB["MongoDB<br/>(users, events, tickets,<br/>favorites, favorites)"]
    Stripe["Stripe Checkout<br/>(hosted session)"]
    Webhook["Payment Webhook<br/>(listen for completed)"]
    Files["Static /uploads<br/>(images, videos)"]
    QR["QR Code Gen<br/>(jsonwebtoken)"]
    
    Client -->|bearer token| Auth
    Auth --> API
    Client -->|multipart| Upload
    Upload --> API
    Upload --> Files
    API --> DB
    Client -->|redirect| Stripe
    Stripe -->|POST| Webhook
    Webhook --> API
    API --> QR
    API --> Files
    
    style Client fill:#e1f5ff
    style API fill:#fff3e0
    style DB fill:#f3e5f5
    style Stripe fill:#e8f5e9
```

## Request Flow: Ticket Purchase

1. **Browse & Checkout:** Attendee selects event and quantity on detail page
2. **Stripe Session:** Backend creates session (event id, user id, quantity) and returns client ID
3. **Payment:** User completes payment on Stripe's hosted checkout
4. **Webhook:** Stripe POSTs payment completion; backend verifies and creates ticket record
5. **Confirmation:** Frontend redirects to success page, shows QR code and PDF download option
6. **Cancellation:** Attendee cancels via my-tickets; inventory restored, ticket marked cancelled

## Data Model

- **Users:** id, email, password (bcrypt), organizer flag, profile fields
- **Events:** id, title, description, organizer id, category, price, inventory (count), media urls
- **Tickets:** id, event id, user id, quantity, qr code (data URL), created at, cancelled flag
- **Favorites:** user id, event id (join table)
- **Stripe Sessions:** session id, event id, user id, quantity (stored for webhook matching)

## API Endpoints

**User Routes:**
- POST /user/register (email, password)
- POST /user/login (email, password) → token
- GET /user/profile (auth required)
- PATCH /user/profile (auth required)
- GET /user/organizer-status

**Event Routes:**
- POST /event (organizer; multipart)
- GET /event/:id
- PATCH /event/:id (organizer; multipart)
- DELETE /event/:id (organizer; **missing check**)
- GET /event/search?q=text&category=sports
- GET /event/all

**Ticket Routes:**
- POST /ticket (after payment webhook)
- GET /ticket/my-tickets (auth required)
- DELETE /ticket/:id (cancel)
- GET /ticket/event/:id (organizer-only)

**Stripe Routes:**
- POST /payment/checkout-session
- POST /payment/webhook (Stripe signature verify)

**Favorites Routes:**
- POST /favourite (auth required)
- DELETE /favourite/:eventId (auth required)
- GET /favourite (auth required)

## Security & Authorization

- JWT tokens issued on login; expiration checked on protected routes
- Organizer routes verify user.organizer flag and event ownership
- Stripe webhook verifies signature (hmac-sha256)
- Inventory decremented on ticket creation; restored on cancellation
- No delete permission check on events (vulnerability; organizer access only via code review)

## File Storage

Images and videos uploaded via multipart go to `/backend/uploads/images/` and `/backend/uploads/videos/`. Frontend serves via static route; no CDN in this build. Demo seeded with stock images copied to this directory.
