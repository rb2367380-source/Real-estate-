# Real-estate-
For Real estate clients to show my real estate properties

# PRD: Client Property Portal

**Version:** 1.0 | **Owner:** You (Admin) | **Status:** Draft for build

---

## 1. Overview

A cross-platform app (web + iOS + Android) where **invited clients** browse a **single shared property catalog** and interact with it: inquiries, offers, and documents. **You are the sole administrator**: you control who gets in, what is listed, and how every inquiry and offer is handled. **No payments** are processed in v1.

### Goals
1. Give clients a polished, self-service way to explore properties in depth.
2. Give you full control over access, content, and client communication.
3. Replace email/phone back-and-forth for inquiries, offers, and documents.

### Non-Goals (v1)
- Payment processing, rent collection, e-signatures
- Public (anonymous) browsing or self-registration
- Multi-admin / agent teams
- Per-client private catalogs (all clients see the same catalog)

---

## 2. Users & Roles

| Role | Description | Access |
|---|---|---|
| **Admin (you)** | Single owner | Full CRUD on everything; invites/revokes clients; manages inquiries, offers, documents |
| **Client** | Invited customer | Read catalog; create inquiries/offers; upload/download own documents; manage own profile |

**Access model:** Invite-only. No public signup. Admin can suspend any client instantly.

---

## 3. Core User Stories

**Client**
- C1: Accept my invite and set up my account.
- C2: Browse and search/filter the catalog.
- C3: View a property's photos, details, map, virtual tour, neighborhood data, and financials.
- C4: Save favorites.
- C5: Send an inquiry about a property and see your replies.
- C6: Submit an offer, track its status, and respond to counteroffers.
- C7: Upload and download documents (my own + ones you share with me).
- C8: Get notifications on replies and status changes.

**Admin**
- A1: Invite, suspend, and remove clients.
- A2: Create/edit/archive properties with all content.
- A3: Reply to inquiries.
- A4: Accept, reject, or counter offers.
- A5: Share documents to specific clients and review client uploads.
- A6: See an activity overview (new inquiries, pending offers).

---

## 4. Functional Requirements

### 4.1 Authentication & Access
- Invite by email (magic link), with the client setting a password or using passwordless login.
- Roles enforced server-side (never UI-only).
- Admin can suspend a client; suspended users are logged out and blocked.

### 4.2 Property Catalog
- List view with photo, price, address, status badge (Available / Under Offer / Sold or Leased / Archived).
- Search by keyword; filters: price range, type, beds/baths, location, status.
- Sort by newest, price, and so on.

### 4.3 Property Detail
- Photo gallery
- Core details: address, price, type, beds, baths, size, description, status
- **Map:** pin plus nearby points of interest
- **Virtual tour:** embedded third-party tour URL (e.g., Matterport) or video
- **Neighborhood data:** walk/transit scores, schools, amenities, crime/demographics (provider-based)
- **Financials:** rent, expenses, ROI/yield, maintenance history log
- Actions: Favorite, Inquire, Make Offer

### 4.4 Inquiries
- Client sends a message tied to a property.
- Threaded conversation between client and admin.
- Statuses: Open → Answered → Closed.

### 4.5 Offers
- Client submits: amount, terms/notes, optional attached documents.
- Statuses: Submitted → Under Review → Countered → Accepted / Rejected / Withdrawn.
- Counteroffer creates a new linked round; the full history is visible to both sides.
- Accepting an offer can auto-set the property to "Under Offer" (admin-confirmed).

### 4.6 Documents
- Clients upload files (PDF, images, Office docs; size limit applies).
- Admin shares documents to a specific client or attaches them to a property or offer.
- Private storage; access only via short-lived signed URLs.

### 4.7 Notifications
- Push (mobile) and email for: new reply, offer status change, new document shared, new invite.
- Admin gets notified of new inquiries, offers, and uploads.

### 4.8 Admin Console (web)
- Dashboard: counts of open inquiries, pending offers, new uploads.
- Tables for Clients, Properties, Inquiries, Offers, Documents.
- Audit log of key actions.

---

## 5. Non-Functional Requirements
- **Security:** row-level security, encrypted at rest and in transit, signed URLs for files, rate limiting, audit log.
- **Performance:** catalog loads under 2s on 4G; images lazy-loaded and CDN-served.
- **Reliability:** daily backups; error monitoring.
- **Privacy:** consent for data processing; ability to delete a client and their data.
- **Accessibility:** WCAG 2.1 AA target.
- **Compatibility:** latest 2 major iOS/Android versions; modern browsers.

---

## 6. Recommended Architecture

| Layer | Choice | Why |
|---|---|---|
| Client app | **Expo (React Native) + React Native Web** | One codebase for iOS, Android, web |
| Admin console | **Next.js (web)** or admin routes in the same app | Table-heavy UI is easier on desktop web |
| Backend | **Supabase** (Postgres, Auth, Storage, Realtime, Edge Functions) | Fast to build, RLS for security |
| Maps | Mapbox or Google Maps | Pins, POI |
| Virtual tours | Embedded URL (Matterport etc.) | No custom 3D build |
| Neighborhood data | Walk Score / Google Places / local data API | Avoid building datasets |
| Notifications | Expo Push + transactional email (Resend/Postmark) | Cross-platform |
| Monitoring | Sentry + basic analytics | Catch errors early |

### Data Model (core tables)
- `profiles` (id, role, name, phone, status)
- `invites` (email, token, expires_at, accepted_at)
- `properties` (id, title, address, lat, lng, price, type, beds, baths, size, description, status, tour_url)
- `property_media` (property_id, url, order)
- `property_financials` (property_id, rent, expenses, roi, notes)
- `maintenance_logs` (property_id, date, description, cost)
- `neighborhood_data` (property_id, provider, payload JSON, fetched_at)
- `favorites` (client_id, property_id)
- `inquiries` + `inquiry_messages`
- `offers` + `offer_rounds`
- `documents` (owner_id, property_id?, offer_id?, shared_with?, path, type)
- `notifications` (user_id, type, payload, read_at)
- `audit_log` (actor_id, action, entity, entity_id, at)

---

## 7. Build Plan: Small Steps

Each step is small, shippable, and testable. Check off as you go.

### Phase 0: Foundations
- [ ] **0.1** Create repo, set up TypeScript, linting, and formatting.
- [ ] **0.2** Initialize Expo app (with web support) and verify it runs on iOS sim, Android emulator, and browser.
- [ ] **0.3** Create Supabase project (dev + prod environments).
- [ ] **0.4** Set up environment variables and secrets handling.
- [ ] **0.5** Set up CI (lint + type-check + tests on each push).
- [ ] **0.6** Add Sentry error tracking.
- **Done when:** a blank app deploys to all 3 platforms from CI.

### Phase 1: Auth & Roles
- [ ] **1.1** Create `profiles` table with a `role` column.
- [ ] **1.2** Seed your admin account manually.
- [ ] **1.3** Build login screen (email + password or magic link).
- [ ] **1.4** Build the `invites` table and an Edge Function to send invite emails.
- [ ] **1.5** Build the accept-invite flow.
- [ ] **1.6** Add RLS policies: clients read only their own profile; admin reads all.
- [ ] **1.7** Add suspend/reactivate client (blocks login).
- [ ] **1.8** Add route guards (client vs. admin screens).
- **Done when:** only invited users can sign in, and a suspended user is blocked.

### Phase 2: Admin Property Management
- [ ] **2.1** Create `properties` and `property_media` tables.
- [ ] **2.2** Build the admin property list.
- [ ] **2.3** Build the add/edit property form (core fields).
- [ ] **2.4** Add image upload to Storage with ordering and a cover photo.
- [ ] **2.5** Add the status field and archive action.
- [ ] **2.6** Add the virtual tour URL field with validation.
- [ ] **2.7** Add financials form (rent, expenses, ROI).
- [ ] **2.8** Add maintenance log entries (add/edit/delete).
- **Done when:** you can fully create a property with photos, tour, and financials.

### Phase 3: Client Catalog
- [ ] **3.1** Build the catalog list screen with property cards.
- [ ] **3.2** Add pagination or infinite scroll.
- [ ] **3.3** Add keyword search.
- [ ] **3.4** Add filters (price, type, beds/baths, status).
- [ ] **3.5** Add sorting.
- [ ] **3.6** Add RLS: clients read non-archived properties only.
- [ ] **3.7** Add empty, loading, and error states.
- **Done when:** a client can find a property in under 3 taps/filters.

### Phase 4: Property Detail
- [ ] **4.1** Build the detail screen with core info.
- [ ] **4.2** Add the photo gallery (swipe and zoom).
- [ ] **4.3** Add the map with a pin.
- [ ] **4.4** Add the virtual tour embed (in-app webview, with a fallback link).
- [ ] **4.5** Build the financials section (rent, ROI, expenses).
- [ ] **4.6** Build the maintenance history list.
- [ ] **4.7** Integrate the neighborhood data provider via an Edge Function, caching results in `neighborhood_data`.
- [ ] **4.8** Display the neighborhood section (scores, amenities, schools).
- [ ] **4.9** Add the Favorite toggle and a Favorites screen.
- **Done when:** a property page shows all content sections correctly on all platforms.

### Phase 5: Inquiries
- [ ] **5.1** Create `inquiries` and `inquiry_messages` tables with RLS.
- [ ] **5.2** Client: "Inquire" button creates a thread.
- [ ] **5.3** Client: thread view with message sending.
- [ ] **5.4** Admin: inbox listing all inquiries with status filters.
- [ ] **5.5** Admin: reply in thread.
- [ ] **5.6** Add realtime updates to threads.
- [ ] **5.7** Add status changes (Open/Answered/Closed).
- **Done when:** a full client↔admin conversation works in realtime.

### Phase 6: Offers
- [ ] **6.1** Create `offers` and `offer_rounds` tables with RLS.
- [ ] **6.2** Client: "Make Offer" form (amount, terms) with validation.
- [ ] **6.3** Client: My Offers list and detail with status timeline.
- [ ] **6.4** Admin: offers queue with filters.
- [ ] **6.5** Admin: accept / reject / counter actions.
- [ ] **6.6** Client: respond to counteroffer (accept / counter / withdraw).
- [ ] **6.7** Enforce a valid status state machine (server-side).
- [ ] **6.8** Prompt the admin to set the property to "Under Offer" on acceptance.
- [ ] **6.9** Write audit log entries for all offer transitions.
- **Done when:** a negotiation round-trip works end to end, with invalid transitions rejected.

### Phase 7: Documents
- [ ] **7.1** Create the `documents` table and a private Storage bucket with RLS.
- [ ] **7.2** Client: upload a document (type and size validation).
- [ ] **7.3** Client: Documents screen (mine and shared with me).
- [ ] **7.4** Signed-URL download flow.
- [ ] **7.5** Admin: view client uploads.
- [ ] **7.6** Admin: share a document to a client / attach to a property or offer.
- [ ] **7.7** Delete and replace documents (with audit).
- **Done when:** a client can never access another client's files.

### Phase 8: Notifications
- [ ] **8.1** Register for push tokens (iOS/Android).
- [ ] **8.2** Create the `notifications` table and in-app notification list.
- [ ] **8.3** Trigger on: new message, offer status change, document shared.
- [ ] **8.4** Send email via a transactional provider.
- [ ] **8.5** Admin alerts for new inquiries, offers, and uploads.
- [ ] **8.6** Add notification preferences (per type).
- **Done when:** key events reach the right person within a minute.

### Phase 9: Admin Console & Dashboard
- [ ] **9.1** Dashboard with counts and recent activity.
- [ ] **9.2** Clients table (invite, suspend, delete).
- [ ] **9.3** Audit log viewer.
- [ ] **9.4** Basic exports (clients, offers as CSV).
- **Done when:** you can run daily operations without touching the database.

### Phase 10: Hardening & Launch
- [ ] **10.1** Security review of every RLS policy (test as client A vs. client B).
- [ ] **10.2** Add rate limiting on invites, messages, and uploads.
- [ ] **10.3** Performance pass (image sizing, CDN, query indexes).
- [ ] **10.4** Accessibility pass (labels, contrast, screen reader).
- [ ] **10.5** Add privacy policy, terms, and data deletion flow.
- [ ] **10.6** Backup and restore test.
- [ ] **10.7** UAT with 2-3 real clients; fix findings.
- [ ] **10.8** Submit to the App Store and Google Play (or use TestFlight / private distribution first).
- [ ] **10.9** Deploy the web app and set up the domain, SSL, and monitoring.
- **Done when:** the pilot clients complete all core flows without help.

---

## 8. Acceptance Criteria Summary
1. Only invited, active clients can sign in.
2. Clients see the full shared catalog and nothing admin-only.
3. Property pages show map, virtual tour, neighborhood data, and financials.
4. Inquiries and offers follow defined state machines with full history.
5. Documents are private; cross-client access is impossible.
6. Admin can operate everything from the console.
7. Works on iOS, Android, and web.

---

## 9. Risks & Open Questions

| # | Item | Impact | Suggested Action |
|---|---|---|---|
| R1 | **Financials are visible to all clients** because the catalog is shared. | Business privacy | Add a per-property "show financials" toggle (recommended) |
| R2 | Neighborhood data provider costs/licensing | Budget | Pick provider early; cache aggressively |
| R3 | App Store review for a private-client app | Timeline | Use TestFlight / unlisted distribution if needed |
| R4 | Offers are not legally binding in-app | Legal | Add disclaimer; get legal review of wording |
| R5 | Document sensitivity (IDs, contracts) | Compliance | Encryption, retention policy, deletion flow |
| Q1 | Do clients need to see each other's activity? | Privacy | Assume no |
| Q2 | Currency/locale/region? | Formatting, data providers | Confirm before Phase 4 |
| Q3 | Branding (name, logo, colors)? | UI | Needed before Phase 3 |

---

## 10. Future Roadmap (post v1)
- Payments (deposits, fees) via Stripe
- E-signatures for offers and contracts
- Per-client private/assigned properties
- Multi-admin / agent roles
- Viewing scheduling with calendar sync
- Saved searches and new-listing alerts
