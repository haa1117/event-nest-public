# Product Overview

Event-Nest is a full-stack event ticket booking platform where attendees discover events, purchase tickets and manage their bookings, while organizers create events and monitor sales and attendee data.

## User Roles

**Attendees** browse by category (sports, concerts, workshops, networking, exhibitions), search by text, favorite events and purchase tickets via Stripe Checkout. They receive QR-code e-tickets that can be printed as PDFs or viewed on-screen, and can cancel bookings to restore inventory.

**Organizers** create events with titles, descriptions, cover images, video previews, pricing and ticket inventory. They can edit event details, upload media files and access a dashboard showing total revenue, attendees, and tickets sold per event.

## Core Features

- **Event Discovery:** Browse 17 pages including category feeds, full-text search, and favorites list
- **Ticketing:** Real-time inventory tracking, cancellation with automatic refunds, per-booking QR codes as data URLs
- **Checkout:** Stripe Checkout integration with session webhooks
- **Media Upload:** Multi-file image and video uploads via multer, stored in `/backend/uploads/`
- **E-Tickets:** QR code generation (qrcode library) and printable PDF export (react-to-print)
- **Admin Dashboard:** Sales metrics, attendee list, average revenue per ticket, event edit interface

## Platform

Built with React 18 + Vite 6 on the frontend and Node + Express 5 on the backend. Uses MongoDB for persistence, JWT for stateless auth (jsonwebtoken + bcrypt), and Stripe for payments. No built-in charts; organizer dashboards display stat cards only.

## Key Technical Choices

- **SPA Architecture:** React Router 7 for client-side routing; API calls use JWT bearer tokens
- **File Uploads:** Multer middleware processes multipart requests; images/videos served as static files
- **QR Generation:** Code-embedded in data URLs; no server-side rendering
- **Search:** Text-based event queries plus category filtering
- **Organizer Scope:** Create and edit endpoints check ownership; ticket data is organizer-only
