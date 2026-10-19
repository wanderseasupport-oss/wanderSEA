# WanderSEA Singapore

A working static travel-companion beta, built with HTML, CSS and ES modules. No dependencies or secret keys are required to run it.

## Run

Install Node.js 22 or later, open a terminal here, and run:

```powershell
npm run check
npm test
npm run dev
```

Open http://127.0.0.1:4173. Stop with Ctrl+C. Avoid opening `index.html` directly with `file://`: JavaScript modules require HTTP.

## Included

- Twelve curated Singapore places, with illustrative editorial artwork.
- Local keyword-based matching and explicit location/time/budget/category filters.
- Place details, official visitor links and Google walking/transit handoffs.
- Local day-plan saving, reordering/removal, estimate warnings and text download.
- Hotel area/date search handoff to Booking.com, ordinary Klook search links and Grab transport links.
- Responsive layout, keyboard-operable dialogs, policy disclosures and implementation guide.

## Limits

There is no server AI, GPS search, live map, route computation, real-time hours/weather, supplier inventory, user accounts, payment or booking confirmation. All such features require further backend work and provider credentials/agreements. The model does not parse arbitrary budgets/time/date/party details from prose: explicit form filters control those limits. Costs and durations are illustrative allowances; transfers use a 20-minute placeholder.

Plans live in browser local storage, not cloud storage. No service worker or offline-map support is claimed; the manifest allows supported browsers to offer a basic standalone experience. Imagery/fonts require network access. The skyline uses an Unsplash CDN photo; fonts use Google Fonts. Discovery cards are CSS illustrations, not venue photographs. Replace third-party media as appropriate before commercial launch.

## Implementation and deployment

Read `dist/implementation.html` in the running website and `dist/requirements.md` for the full specification, API onboarding, security, partner applications, privacy/licensing considerations, data model and staged build plan.

For static hosts, serve `dist` with no build command. Never upload `.env` or put private keys in `dist`. `.env.example` documents future backend configuration only: the current app does not load it.

Sites registration is stored in `.openai/hosting.json`; preserve its project ID on updates. Sites source sync/deployment uses the installed Sites workflow, not this local preview server. The initial hosted site is private; public access is a separate intentional change.

Before in-app booking, implement current provider-approved authentication, inventory lookup, rechecks, idempotent orders, status recovery, refunds and support procedures. Do not assume an affiliate or merchant API grants distribution booking access.
