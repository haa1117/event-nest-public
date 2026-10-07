# Privacy and Source

## What This Repository Is

A portfolio showcase for a production event-ticket booking application. It describes the engineering, architecture and design decisions; it is not a source distribution.

## What Is Published

- Written descriptions of product features, architecture and engineering challenges
- Original Mermaid diagrams created for this showcase
- Analysis of trade-offs and implementation choices
- Descriptions of identified bugs and missing features

## What Is Intentionally Not Published

- Production source code, in whole or in part
- API keys, tokens and credentials (including Stripe keys)
- Database connection strings and configuration
- Firebase / Google service configuration
- Environment files (.env)
- Version control history of the production repository
- Any user, financial or personal data
- Real screenshots with live data

## How The Descriptions Were Produced

The production project was inspected read-only. Statements describe behaviour observed in code, build configuration, and project documents. Where behaviour could not be confirmed it is worded conservatively or left out. Diagram labels are conceptual and do not match production identifiers or file paths.

## Demo Data

This portfolio includes a demo build with seeded fictional event and user data for illustration. Screenshots use this fictional demo data only; no real user information or transactions are shown. The demo runs locally with a seeded MongoDB dataset and does not connect to production systems.

## Sensitive Configuration

The project includes hardcoded payment processor credentials in source code. These are excluded from this portfolio. All sensitive configuration (API keys, database URLs, signing secrets) has been removed from any published materials.

## Third-Party Technologies

The following third-party technologies and services are used and documented:
- React 18, Vite 6, Tailwind CSS (frontend)
- Node.js, Express 5 (backend)
- MongoDB (database)
- Stripe (payment processing)
- Multer (file upload)
- jsonwebtoken, bcrypt (auth)
- qrcode, react-to-print (e-tickets)

Third-party names and trademarks belong to their respective owners and are used only to identify technologies used in this application.

## Publication Checks

Before publishing, all files were checked for:
- Source code or code snippets beyond conceptual descriptions
- API keys, tokens and hardcoded credentials
- Database connection strings and internal endpoints
- Email addresses and personal identifiers
- Local file-system paths
- Production database exports or real user data

None remain in the published materials.

## Access

Source access for hiring or due-diligence purposes can be discussed at the owner's discretion.
