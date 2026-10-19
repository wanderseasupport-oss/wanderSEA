# Singapore AI Travel Companion — requirements and implementation playbook

Prepared 19 October 2026. Working product name: WanderSEA, Singapore edition.

This is a proposed product specification and implementation plan, not a claim that an application or partner integration has been built. The pasted reference describes a starter ZIP, but that ZIP/source was not attached and the current workspace contains no application source. Its build, database policies, and claimed features remain unverified.

## 1. Product decision

Build one place to understand a visitor's intent, choose what to do, organize the day, and access hotel, activity, and transport services. Start with partner checkout; add embedded booking only where contracts and APIs support it.

Example: “We're near Bugis with two children, have three hours and S$120 left, and want something indoors with local food afterwards.” Return three suitable options with explanations, a realistic route, estimated costs, booking actions, and a saved plan. If the visitor changes the request, preserve fixed bookings and revise the flexible parts.

Launch assumptions: Singapore destinations only; mobile web/PWA first; English interface initially; SGD display; Asia/Singapore local schedules; optional account; manual location entry as an alternative to GPS; adults organize family trips. Flight sales, restaurant reservation inventory, insurance, payments for supplier services, automatic ride dispatch, and multi-provider packages are later scope.

“All services in one app” has three stages:

1. Plan and organize in your app; open the relevant provider to finish booking.
2. Search live partner inventory in your app; use provider checkout.
3. Complete supported bookings in your app, with payment, servicing, cancellation and reconciliation obligations implemented.

The first stage can launch without every commercial API approval. A redirect is not a confirmed booking. Do not advertise a single checkout until it exists.

## 2. Requirements and acceptance criteria

| ID | Requirement | Acceptance criterion |
|---|---|---|
| F01 | Natural-language intent | Extract origin, time window, dates, party size/child ages where needed, budget scope, interests, transport preferences and exclusions into a validated object. Ask only for missing information that changes feasibility. |
| F02 | Discover nearby | User can enter an MRT station/hotel/address or grant location access; results stay in Singapore and disclose the search area. |
| F03 | Explain recommendations | Show three useful options with why each fits, travel time, duration, price status, opening-hours status, source and relevant uncertainty. |
| F04 | Practical itinerary | Calculate routes through a routing service; schedule visits, meals and buffers; flag infeasible constraints rather than inventing a plan. |
| F05 | Edit and replan | Add/remove/reorder stops; shorten the day; replace outdoor stops when requested; retain locked bookings. Saving a proposed change requires an explicit user action. |
| F06 | Hotels | Collect check-in/out dates, rooms, adults and child ages before shopping. Filter by route distance and budget; display live rates only from authorized hotel inventory. |
| F07 | Activities | Match place to exact product, option and dates where available. Separate the attraction from the ticket/package being sold. |
| F08 | Transport | Show walking/public transport directions and an Open Grab action. Label estimated travel time and distinguish it from live ride fare/ETA. |
| F09 | Trip hub | Save itinerary, user-added reservations, source links and notes; distinguish planned, externally opened, user-reported booked and provider-confirmed statuses. |
| F10 | Location control | Works without GPS; permission denial has a useful fallback. Request fresh location only when needed. |
| F11 | Weather | Show forecast issue/validity times and geography. A forecast is not a guarantee. Old/failed weather data must not appear live. |
| F12 | Account and sharing | Guest planning; optional email sign-in for sync. Private by default. Share via revocable, scoped links without exposing reservation references or private notes. |
| F13 | Feedback | “Closed”, “wrong information”, “not suitable” and broken-booking-link reports reach an admin queue. |
| F14 | Admin | Curate place facts and licensed media, match provider IDs, disable products/links and review source freshness. |
| F15 | Commercial transparency | Disclose affiliate relationships and sponsored placements. Commercial yield cannot silently override user constraints. |

Recommended first screens: Ask; results/list and map; place detail; Today's Plan; hotel results; saved trips/reservations; account/privacy; minimal admin.

Quality requirements: mobile keyboard and screen-reader support; readable touch targets; progress indicator during generation; partial results when one provider fails; per-user and per-session request limits; API call/time/token limits; structured errors; HTTPS; backups and restore check; no secrets in client bundles or logs. Proposed beta targets: ordinary search under 5 seconds and an initial useful planning response under 15 seconds, measured rather than promised.

## 3. Integration access matrix

| Service | Launch use | What you must obtain | Approval/fallback |
|---|---|---|---|
| Google Maps JavaScript | Interactive map | Cloud project, billing, restricted browser key, map ID if required by selected marker implementation | Self-service; use a list and Maps links if unavailable |
| Google Places API (New) | Search and place detail UI | Enabled API and appropriate credentials | Self-service; restricted storage/display rules apply |
| Google Routes API | Route times and feasibility | Enabled API and server credentials | Self-service; offer Maps directions if down |
| Google Maps Grounding Lite | Google-supported AI grounding tools | Enabled service, credentials, compatible LLM/data configuration | Verify terms and retention before production; use owned/licensed data for AI if unresolved |
| Klook affiliate/distribution | Activity links, then authorized content/inventory | Affiliate approval; separately confirm API/data-feed/white-label access and rights | Approval-dependent; official attraction links work without affiliate earnings |
| Booking.com Demand | Hotel search and redirect, possibly booking later | Managed Affiliate Partner agreement, Partner Centre, token and affiliate ID | Approval-dependent; official hotel booking pages initially |
| Expedia Rapid | Alternative hotel inventory and booking | Partnership and integration/launch approval | Alternative to Booking.com, not an additional MVP dependency |
| Grab | Open ride service; partner integration later | Confirm supported links and Singapore use case; commercial access if required | Do not assume a public ride-booking API; fallback to open provider plus copy destination |
| data.gov.sg/NEA | Singapore weather | Dataset API; optional development/production key for higher limits | Public testing access; respect dataset terms and quotas |
| OneMap | Optional barrier-free routing | Follow API registration/authentication for selected endpoints | Coverage varies; not a blanket accessibility guarantee |
| Supabase | Auth, Postgres, private uploads | Project, credentials, RLS policies, email delivery configuration | Self-service |
| OpenAI | Intent parsing, planning and explanations | API project/key/billing and chosen model | Self-service subject to account access; curated browsing fallback |

Sources: [Google setup](https://developers.google.com/maps/documentation/javascript/get-api-key), [Grounding Lite](https://developers.google.com/maps/ai/grounding-lite), [Klook affiliate](https://affiliate.klook.com/home), [Booking prerequisites](https://developers.booking.com/demand/docs/getting-started/prerequisites), [Expedia partnership](https://innovation.expediagroup.com/en-us/solutions/build-your-travel-experience/rapid-api), [data.gov.sg overview](https://guide.data.gov.sg/developer-guide/api-overview), [OneMap BFA](https://www.onemap.gov.sg/apidocs/bfa).

Do not build against the old STB Tourism Information and Services Hub simply because its API documentation appears in search. STB's public discontinuation notice says TIH ended on 31 July 2025. Confirm any replacement directly with STB. [TIH discontinuation notice](https://tih.stb.gov.sg/content/dam/tih/09_Knowledge_Centre/Documents/XR%20guide.pdf)

## 4. Step-by-step: Google accounts and keys

1. Open [Google Cloud Console](https://console.cloud.google.com/) using an account you control. Set up recovery and multifactor authentication. Create separate development and production projects when preparing public release.
2. Create a project, e.g. `wandersea-sg-dev`, and attach a billing account. Record project ID and billing owner.
3. In APIs & Services → Library enable Maps JavaScript API, Places API (New), and Routes API. Enable Maps Grounding Lite separately if using that integration. Do not enable unrelated products by default.
4. In APIs & Services → Credentials create a browser API key. Name it `sg-web-dev`.
5. Apply Websites restrictions for your actual development origin, e.g. `http://localhost:3000/*`, and later a separate production key restricted to your real HTTPS domain. Restrict the key to Maps JavaScript API, plus Places API (New) only if calling the supported browser Places library/widget.
6. Create a different server credential for server-side services. Restrict it to the services actually used. Use IP restrictions with known static outbound IPs, or supported OAuth credentials where appropriate. Website restrictions are not server restrictions.
7. If hosting on serverless infrastructure with changing outbound IPs, decide on supported authentication or a secured proxy with stable egress. Do not enter your website domain as an IP restriction or remove protections merely to make requests work.
8. Set project quotas and application-level call limits. Add billing notifications at chosen thresholds. A budget alert does not itself stop spending. Use the current SKU pricing page rather than assuming an old monthly credit applies.
9. Store the browser key in `NEXT_PUBLIC_GOOGLE_MAPS_BROWSER_KEY`. Browser map keys are visible by design and must be restricted. Store server credentials in deployment secrets, never a `NEXT_PUBLIC_` variable.
10. Create a map ID if the selected Google map/marker feature requires it. Map IDs are identifiers, not secret API credentials.
11. Test: map loads on your allowed domain; Places resolves a Singapore location; route service returns a plausible route; an unauthorized origin is rejected; private keys never appear in browser network responses.

Use Google-provided field masks and request only fields displayed by your UI; some requested fields change the billed SKU. Debounce autocomplete, use the documented session-token flow where applicable, limit candidate counts before route calculations, and avoid repeating calls on every map movement.

Important content design: standard Places results belong in the permitted UI with required attribution. Do not turn them into a permanent place/review/photo database or assume that arbitrary copying into an LLM is allowed. Keep owned or separately licensed editorial facts distinct. Google's AI grounding products have their own allowed uses and retention/display conditions. Confirm compatibility with the chosen model and app configuration. `store: false` alone does not establish zero retention across an AI provider's logs, caching and tools.

Sources: [setup](https://developers.google.com/maps/documentation/javascript/get-api-key), [security](https://developers.google.com/maps/api-security-best-practices), [Places policies](https://developers.google.com/maps/documentation/places/web-service/policies), [pricing](https://developers.google.com/maps/billing-and-pricing/pricing), [Grounding Lite terms guidance](https://developers.google.com/maps/ai/grounding-lite).

## 5. Step-by-step: Klook

1. Prepare a simple live demo, domain, support email, privacy policy, product description and company details if applicable. Be honest about prelaunch status and traffic.
2. Apply through the [Klook Affiliate Program](https://affiliate.klook.com/home). Describe Singapore recommendations with contextual activity links.
3. Request tracked affiliate links first. Confirm permitted channels: website/PWA, future native app, messaging, email, paid advertising and AI recommendations.
4. Ask separately for distribution API/data-feed/white-label eligibility, supported Singapore inventory, application requirements, sandbox access, rate limits and certification.
5. Obtain the actual approved integration documentation and credentials. Klook's public GitBook describes an API for merchants, reservation systems and channel managers; it is not evidence that your app can retrieve and resell Klook inventory. Do not implement against it without Klook confirming the direction of integration.
6. Create your own mapping between a curated place and a provider product. For example, an attraction may have multiple admission products, bundles and time-slot options; do not substitute one for another.
7. Generate URLs with Klook's approved tooling; do not guess tracking parameters. Store approved link/product identifiers and the rights/freshness rules supplied by Klook.
8. If authorized live inventory is unavailable, show “Check availability on Klook” rather than a fabricated current price. A manually curated price must be clearly indicative and sourced.
9. Test links on mobile and desktop, date/currency behavior, unavailable products, attribution, and whatever conversion reporting Klook supplies. Record clicks separately from confirmed sales.
10. Add API search, option selection and availability only after access is granted; add in-app booking only if explicitly supported by contract and technical scope.

Questions for Klook: commission and eligible products; attribution window; app-to-app tracking; cancellation/clawbacks; payout/currency; permitted content/photo storage; AI usage; rate refresh; customer support owner; booking confirmations/webhooks; whether chat-generated links qualify.

Draft enquiry, for you to send:

> We are developing a Singapore-first AI travel companion that recommends attractions based on dates, interests, party composition, budget and location. We would like to start with approved affiliate links and understand eligibility for distribution APIs or data feeds. Please confirm permitted AI recommendation use, Singapore inventory, mobile/PWA attribution, content rights, sandbox access, booking capabilities and commercial requirements. Our product is prelaunch; demo: [URL].

Sources: [affiliate options](https://affiliate.klook.com/home), [merchant API scope](https://klook.gitbook.io/openapi).

## 6. Step-by-step: nearby hotels and booking

Nearby discovery and room availability are different systems. A map result tells you where a hotel is; a hotel supplier tells you what room is bookable for particular dates, occupants and policies.

### Recommended path: one inventory partner

1. Launch discovery with independently sourced hotel entries or permitted Places UI and links to official hotel booking engines.
2. Choose one hotel supplier to apply to first. Booking.com Demand offers content/redirect, search-and-redirect, and search-and-book integration patterns, but access requires becoming a Managed Affiliate Partner and completing its onboarding.
3. For Booking.com: secure the agreement/account manager, access Partner Centre, generate the API token and obtain `X-Affiliate-Id`; use sandbox before production. Do not assume an ordinary consumer account or affiliate registration grants Demand API access.
4. If that partnership is unavailable, apply to Expedia Rapid as an alternative. Partnership and launch requirements still apply. Its lodging flow includes shopping, price check, booking and retrieval; each stage must be implemented for in-app checkout.
5. Collect dates, room allocation, adult count, child ages and applicable traveler residency/booker context required by the supplier. Ask whether the requested budget is nightly or total.
6. Retrieve candidate properties by area, map official supplier IDs to your records, shop live availability, and compare totals with the same occupancy, dates and currency. Do not name-match automatically without checking address/coordinates.
7. Display total stay cost and the provider's tax/fee treatment, pay-at-property amounts, room/bed and meal plan, cancellation deadline/time zone and payment timing. If totals are not comparable, explain why rather than claiming cheapest.
8. Recheck the chosen rate before booking or use the provider's current checkout. If price changes, present the new amount for consent.
9. For redirect: use the approved booking URL; label it “Continue with [provider]”. Your app records an outbound action, not a reservation.
10. For embedded booking later: use idempotent order creation, pending/unknown states after timeouts, supplier status retrieval, confirmation numbers, cancellation and refund handling. A successful payment alone is not proof of a supplier reservation.

“Nearby” should use usable route distance/time where available, not only a straight-line radius. Offer a user-selected radius/time threshold and explain when no room meets the constraints.

Sources: [Booking onboarding](https://developers.booking.com/demand/docs/getting-started/prerequisites), [integration types](https://developers.booking.com/demand/docs), [Expedia lodging flow](https://developers.expediagroup.com/rapid/lodging).

### Connecting directly to Singapore hotels

1. Approach the hotel's sales/e-commerce team with a working demo and a short distribution proposal.
2. Initially request permission for a tracked link or an agreed referral code to its existing booking engine. Agree attribution/reporting and who handles guest support.
3. If volume justifies direct booking, ask which booking engine, CRS, PMS or channel manager it uses and whether it supports authorized distribution to your app.
4. Agree rates, inventory allocation, cancellation/no-show terms, payment responsibility, taxes/fees, image rights and reconciliation in writing.
5. Implement the supported interface; test room/rate mapping, booking duplication prevention, modifications and cancellations. Confirm inventory remains synchronized.

There is no universal API key that connects every nearby hotel. An aggregator avoids implementing a different commercial/technical relationship for each property.

## 7. Step-by-step: Grab and public transport

1. At launch provide an “Open Grab” action and a destination card containing the place name, address, verified coordinates where licensed, and pickup instructions sourced independently or under permission.
2. Request current supported transport deep-link/universal-link documentation from [Grab Developer](https://developer.grab.com/) or Grab's partnership channel. Confirm Singapore consumer rides, PWA/native behavior and commercial permission. Public material reviewed did not establish self-service access to consumer ride booking.
3. Only prefill pickup/drop-off using a supported documented method. Do not rely on copied, undocumented `grab://` examples from forums as a production contract.
4. Test iOS and Android, installed/not installed, browser fallback and location permission denial. If prefill fails, let the user copy the address and open Grab manually.
5. The user chooses the vehicle, sees the actual fare and confirms in Grab. Do not label your own driving estimate a Grab quote or promise a vehicle ETA without authorized live data.
6. Do not mark a ride booked based on a link click or app return. Ask the user or consume an authorized status integration if available.
7. For deeper integration, ask Grab whether booking, quoting, status, cancellation and payment APIs are offered for your exact business use case. GrabPay, delivery, business travel and consumer ride booking are separate products.
8. Provide walking and public transport alternatives through routing services and Google Maps directions links. Ticket purchase for transit is separate scope.

Grab already offers pickup meeting points and venue guides within its app; use the provider experience rather than duplicating unsupported proprietary data. [Grab pickup guide](https://www.grab.com/sg/inside-grab/stories/the-guide-to-pick-up-feature-will-help-travellers-navigate-airports/)

## 8. Step-by-step: AI and recommendation logic

1. Create an account/project on [OpenAI Platform](https://platform.openai.com/), configure API billing and create a project API key. A ChatGPT subscription is not a substitute for application API access.
2. Keep `OPENAI_API_KEY` server-side. Set a configurable model identifier after verifying current availability and testing quality/cost; do not hardcode an old starter's model choice as a requirement.
3. Use the Responses API with structured output/tool calling. Define narrow server tools, e.g. `search_owned_catalog`, `get_weather`, `compute_route`, `search_activity_offers`, `search_hotels`, `propose_itinerary` and `save_approved_itinerary`.
4. Put Google AI grounding through a dedicated adapter governed by the applicable service terms and approved data controls. If compatibility is unresolved, keep Google content out of LLM inputs and use your owned/licensed catalog for AI selection, with separate permitted map/place UI.
5. Parse the request to a validated intent object. Distinguish hard requirements from preferences. Child ages are necessary for some ticket/room searches; disability or health diagnoses are generally unnecessary for route preferences.
6. Search a bounded set of real candidates. Apply hard constraints first: geography, time, occupancy, explicit exclusions, known opening hours and reliable dietary/accessibility requirements. Unknown is not the same as verified suitable.
7. Score feasible candidates using interest match, usable travel time, duration, budget fit, current weather compatibility, data confidence and variety. These weights are product choices; tune them with user feedback. Keep affiliate commissions out of hidden relevance scoring.
8. Compute routes and time/budget arithmetic in code. Include visit duration and buffers; respect timed bookings and the user's end location/time. Distinguish missing cost from zero cost.
9. Ask the model to explain the validated shortlist, not invent businesses, quotes, booking references or certification claims. Only offer tool-backed or curated IDs and server-approved links.
10. For each recommendation preserve source type and retrieval/verification timestamp where permitted. User-facing labels should distinguish live quote, indicative cost, unknown hours and forecast.
11. Treat supplier descriptions, websites and uploaded confirmations as untrusted data. They cannot alter system instructions, invoke payments, change destinations or expose secrets. Limit tool calls, timeouts and spend per request.
12. Keep proposals separate from actions. Require the user's approval before editing locked plans, submitting personal data, paying, booking or cancelling.

Example intent fields: `origin`, `date`, `start_time`, `end_time`, `party`, `budget_amount`, `budget_scope`, `interests`, `max_walking_minutes`, `transport_modes`, `indoor_preference`, `food_requirements`, `locked_items`, `end_location`.

Fallback rules: no AI → browse curated places; no supplier → honest provider link; no route → show unavailable and offer navigation; unknown dietary/accessibility information → state unknown; impossible request → suggest which constraint to relax.

Sources: [Responses API](https://developers.openai.com/api/reference/responses/overview), [function calling](https://developers.openai.com/api/docs/guides/function-calling), [OpenAI data controls](https://developers.openai.com/api/docs/guides/your-data), [Google grounding requirements](https://developers.google.com/maps/ai/grounding-lite).

## 9. Singapore data and content preparation

Start with a proposed 50–100 independently curated places across a manageable set of precincts: Marina Bay, Chinatown, Kampong Glam/Bugis, Little India, Orchard and Sentosa. Include free activities and food choices so the product remains useful without bookings.

For each owned/licensed record track: stable internal ID; source and usage rights; independent description; location provenance; categories; expected visit duration; indoor/outdoor; indicative cost with source/date; applicable child policies; confirmed accessibility facts or unknowns; official link; media licence; editor verification date; optional Google Place ID and partner product IDs.

Keep hotel, attraction, ticket offer and restaurant as distinct entities. A partner product ID is not a place ID. Keep Google data subject to its rules instead of copying it into your permanent catalog. Use your own photos, licensed media or supplier assets under agreed rights.

Weather: visit [data.gov.sg](https://data.gov.sg/), sign in with email, select Create API Key and request development/production access as appropriate. Select the NEA two-hour forecast dataset, copy its current API specification, and test area matching and timestamps. Do not infer a neighborhood forecast solely from island-wide rain. [Key setup](https://guide.data.gov.sg/developer-guide/api-overview/how-to-request-an-api-key), [weather dataset](https://developers.data.gov.sg/datasets?formats=API&resultId=d_3f9e064e25005b0e42969944ccaf2e7a).

## 10. Application architecture and storage

Recommended foundation: Next.js/TypeScript mobile PWA; Supabase Auth/Postgres and private storage; server-side provider adapters; Google map UI; OpenAI planning adapter; Vercel deployment or another suitable host. This is a recommendation, not an audit of the missing starter ZIP.

Flow: browser → authenticated/rate-limited backend → intent parsing → permitted data/provider adapters → deterministic feasibility checks → structured recommendation → user-approved saved trip. Backend builds partner links from approved IDs; model output is never used as an arbitrary URL or API endpoint.

Suggested data entities:

| Entity | Purpose |
|---|---|
| profiles | Optional preferences, locale and consent settings |
| trips / trip_members | Owner, dates, time zone, sharing permissions |
| itinerary_items | Internal/allowed place references, scheduled time, duration, locked status |
| editorial_places | Independently sourced or licensed catalog only |
| provider_mappings | Internal place/property → exact supplier ID; confidence and verification |
| quote_sessions | Permitted short-lived supplier references, expiry and context; supplier retention rules apply |
| reservations | User-entered or verified bookings; status provenance, supplier reference and support owner |
| outbound_events | Provider/action/time and pseudonymous attribution ID; no fabricated conversion |
| feedback / audit_events | Corrections, admin actions and operational diagnostics |

User data must be owner/member-scoped with row-level security. Use a Supabase publishable key in the client with appropriate policies; secret/service-role keys stay on the server. Revoke unnecessary grants. Test that user B cannot read, modify or delete user A's trip, including files and shared links.

Suggested backend endpoints, not supplier API paths: `/api/intent`, `/api/recommendations`, `/api/routes`, `/api/weather`, `/api/hotels/search`, `/api/activities/search`, `/api/booking-links`, `/api/trips`, `/api/feedback`. Create order/cancellation endpoints only for approved embedded booking integrations.

Environment-variable names are application conventions, not values to obtain from these exact provider field names:

```dotenv
NEXT_PUBLIC_APP_URL=
NEXT_PUBLIC_GOOGLE_MAPS_BROWSER_KEY=
NEXT_PUBLIC_GOOGLE_MAP_ID=
GOOGLE_MAPS_SERVER_KEY=
GOOGLE_MAPS_GROUNDING_KEY=
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=
SUPABASE_SECRET_KEY=
OPENAI_API_KEY=
OPENAI_MODEL=
DATA_GOV_SG_API_KEY=
HOTEL_PROVIDER=links_only
ACTIVITY_PROVIDER=links_only
GRAB_INTEGRATION_MODE=open_only
```

Add Klook/hotel credentials only when issued, using the actual approved authentication scheme. Never commit `.env.local`; commit a blank `.env.example`. Use separate dev/production credentials. Do not paste keys into chat.

Supabase setup: create project → choose suitable region → set Auth site/redirect URLs for local/staging/production → configure email sender → create migrations → apply RLS/grants → test two users → backups. [Next.js guide](https://supabase.com/docs/guides/getting-started/quickstarts/nextjs), [RLS guide](https://supabase.com/docs/guides/database/postgres/row-level-security).

Deployment: create a private Git repository → add source and blank env template → run the project's defined lint/typecheck/test/build commands → import into hosting → add secrets by environment → deploy staging → test allowed map origins/auth redirects → configure HTTPS domain → deploy production → monitor errors, quotas and cost. Hosting a frontend on Vercel does not solve stable server egress automatically.

## 11. Trust, privacy and commercial requirements

Implement a privacy notice explaining location, trip/preferences, provider disclosures, AI processing, retention and account deletion. Allow manual origin entry and avoid continuous location history in the MVP. Set documented retention periods and minimize logs; record operational IDs rather than full prompts, precise locations, reservation documents or credentials. Restrict admin access and provide an incident response contact.

Review Singapore PDPA obligations including notification/consent as applicable, purpose and retention limits, security and overseas transfers. Designate responsibility for data protection and publish a contact. Singapore hosting alone does not mean no overseas processing. [PDPC obligations](https://www.pdpc.gov.sg/-/media/Files/PDPC/PDF-Files/Resource-for-Organisation/Data-Protection-Obligations-under-the-PDPA.ashx?la=en)

Before selling travel services, arranging tours/packages or accepting supplier payments, assess your exact flow against STB's platform/travel-agent guidance; confirm uncertainties with STB or a qualified adviser. Do not assume that calling yourself an affiliate, using redirects, or selling only Singapore trips automatically resolves licensing. Verify listed travel-agent/tourist-guide licences where applicable. [STB platform guidance](https://isomer-user-content.by.gov.sg/1/694e58c7-eb05-425c-a7f1-99595de6753a/STB%20Guidelines%20for%20Platform%20Operators.pdf), [Travel Agents](https://www.stb.gov.sg/industries-experience-development/travel-agents/).

For embedded booking add: merchant/payment responsibility; hosted/tokenized payment handling; supplier terms and cancellation acknowledgement; amount/currency consent; idempotency; webhook signature verification where supported; timeout reconciliation; refund state tracking; support procedures; ledger and supplier/affiliate reconciliation. Do not collect raw card details in your own database.

## 12. Ordered delivery plan

Indicative 8–12 week pilot plan for an experienced developer with part-time product/content help. It is an estimate, not a delivery commitment; partner approval and embedded booking can extend it substantially.

| Stage | Owner | Work and exit condition |
|---|---|---|
| 1: Scope and applications | Founder, week 1 | Decide visitor group, link-based launch and budget; own accounts/domain; initiate Klook/hotel/Grab enquiries. Exit: written scope and access tracker. |
| 2: Foundation | Developer, weeks 1–2 | Scaffold app, staging, secrets, auth/RLS, catalog admin, error/cost monitoring. Exit: private trips stay private and staging deploys reproducibly. |
| 3: Discovery | Developer/content, weeks 2–3 | Seed catalog, place UI, map, location fallback, routing, weather. Exit: visitor can find real places and navigate without AI. |
| 4: Intent and planning | Developer/product, weeks 3–5 | Structured intent, constrained ranking, feasible schedule, explanations, save/edit/replan. Exit: benchmark scenarios pass. |
| 5: Service actions | Developer, weeks 5–6 | Official/approved activity and hotel links; Grab handoff; reservation status labels and support ownership. Exit: mobile handoffs work and clicks never become fake bookings. |
| 6: Live inventory if approved | Developer/partners, weeks 6–8 | One hotel provider and activity feed/API only where access exists. Exit: correct dates/occupancy/totals and expired-offer handling. Otherwise retain links. |
| 7: Beta and launch | Founder/developer, weeks 8–12 | Test with proposed 20–30 visitors; fix failures, optimize calls; publish policies/support. Exit: release criteria below. |

Embedded booking is a separate phase after pilot evidence and commercial access, not an assumed completion item within this estimate.

## 13. Release tests and success measures

Minimum scenario suite:

- Bugis, rain, family with children, three hours, explicit total budget.
- Parents prefer short walks; unknown accessibility information is not presented as verified.
- Late-night request where attractive venues are closed.
- Impossible budget/time combination: app proposes an honest relaxation.
- Nearby hotel has no suitable family room or child rate; no invented availability.
- Taxes/pay-at-property fees make an initially cheap hotel more expensive.
- User denies GPS, loses network, or a provider times out.
- Klook product is unavailable or mismatched; official link fallback works.
- Grab missing from device; user can copy destination and continue.
- Fixed booking remains intact after rain replanning.
- Two users and a revoked shared link: authorization boundaries hold.
- Duplicate order/timeout/cancel/refund tests if embedded checkout is introduced.
- Supplier text includes malicious instructions: no unauthorized tool/action follows.
- API usage ceilings, leaked-secret scan, account deletion and backup restoration.

Proposed pilot measures: percentage completing a useful plan; time to first useful option; recommendations rejected and why; route/hour/price errors; saved plans; successful service handoffs; return use during the trip; cost per completed plan. Track actual conversion only when a provider confirms it. Establish targets after observing initial usage rather than claiming unsupported conversion benchmarks.

## 14. Budget worksheet

No fixed provider costs or commissions are promised here. Hosting, database, model and Maps pricing depend on plan, SKU, region, currency and usage; partnership economics are contractual.

Create a spreadsheet with: component; request unit; calls per completed plan; plans per month; unit price/free allowance; minimum subscription; currency; alert threshold; actual spend. Calculate each SKU separately, subtract only applicable allowances and add taxes/exchange costs where relevant.

Include map loads; autocomplete sessions; place details fields; route/matrix elements; grounding calls; model input/output/cache tokens; database/storage/egress; email; domain; error monitoring; support; content updates; refunds/payment fees in later booking stages. Estimate clicks and paid commissions separately from gross booking value.

Set a founder-selected pilot spending cap, implement app-side metering and provider quota controls where available, and review cost per completed plan weekly. Avoid treating billing alerts as hard caps.

## 15. Founder checklist: what to do first

1. Confirm the product scope above; choose a working name/domain and support email.
2. Create Google Cloud development project/billing, restricted map/server credentials and usage controls.
3. Create Supabase and an AI API project; store credentials in a password manager/deployment secrets.
4. Apply to Klook affiliate and one hotel inventory partner; request Grab integration details. Keep a tracker: applied, awaiting documents, sandbox, production approved, fallback.
5. Prepare the first 50 curated Singapore places and official booking links with rights/source records.
6. Build discovery → intent → feasible plan → save/replan → provider handoff in that order.
7. Pilot with visitors before expanding to embedded bookings or native apps.

Partner application pack: product one-pager; demo/screenshots; channels and target countries; current stage and truthful traffic expectations; company/contact details; support model; privacy/terms URLs; requested inventory/actions; payment/merchant role; data rights/AI use questions; test plan and target launch window.

Nothing in this document grants API access. Keys, commercial agreements and credentials remain under your control; this preparation does not submit applications or contact providers on your behalf.
