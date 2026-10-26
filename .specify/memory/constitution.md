<!--
Sync Impact Report
==================
Version change: (unversioned template) → 1.0.0
Bump rationale: Initial ratification; all placeholders replaced with project-specific content.

Modified principles (template placeholder → new title):
  - [PRINCIPLE_1_NAME] → I. Visual-First Experience
  - [PRINCIPLE_2_NAME] → II. Information, Not Transactions
  - [PRINCIPLE_3_NAME] → III. Disciplined, Extensible Scope
  - [PRINCIPLE_4_NAME] → IV. Data Stewardship & Graceful Degradation
  - [PRINCIPLE_5_NAME] → V. Performance & Accessibility Budgets
  - (added)            → VI. Type-Safe Simplicity

Added sections:
  - Technology & Platform Constraints (was [SECTION_2_NAME])
  - Development Workflow & Quality Gates (was [SECTION_3_NAME])

Removed sections: none

Dependent templates (plan/spec/tasks) read this file at runtime and were not modified.

Follow-up TODOs: none
-->

# Football Map England Constitution

## Core Principles

### I. Visual-First Experience

The product's identity is the map. Every feature MUST be judged first by how it looks and feels.

- The primary view MUST be an interactive 3D terrain map of England rendered with Mapbox GL JS;
  stadiums MUST be presented geographically on that map, never only as a list.
- Transitions between stadiums MUST use smooth, cinematic camera animations (e.g. `flyTo` with
  tuned pitch, bearing, zoom and easing) rather than instant jumps.
- UI chrome (panels, cards, buttons, typography) MUST follow a single consistent design system;
  ad-hoc one-off styling is not permitted.
- Loading, empty and error states MUST be designed, not left as browser or library defaults.
- A feature that works but degrades the visual polish of the experience is not complete.

**Rationale**: The visual experience is the differentiator for fans and the core demonstration
of capability for the portfolio. Polish is a requirement, not a stretch goal.

### II. Information, Not Transactions

The app informs and links out; it never transacts.

- The app MUST NOT implement bookings, payments, checkout, or ticket inventory.
- The app MUST NOT implement user accounts, authentication, or server-side user profiles.
- The app MUST NOT collect, store, or transmit personal data. Any client-side persistence
  (e.g. a remembered UI preference) MUST stay in the browser and be non-essential.
- "Buy tickets" MUST open the club's official website in a new tab (`rel="noopener noreferrer"`);
  links MUST point to official club domains only, never resellers or affiliates.
- No AI/LLM integration in v1. This is a visualisation and information product.

**Rationale**: Removing transactions, accounts and personal data removes whole classes of legal,
security and operational risk, and keeps focus on the visual and informational core.

### III. Disciplined, Extensible Scope

Ship each release narrowly; design the data model so later releases extend rather than rewrite.

- **v1** scope is exactly: the 20 Premier League stadiums; per-stadium upcoming fixtures for the
  next 60 days; stadium info (capacity, year built, home team); directions from the nearest major
  city; and an outbound "buy tickets" link. Anything else is out of scope for v1.
- **v2** (Championship, League One, League Two, division filtering) and **v3** (Scotland, Wales,
  Ireland, Europe) MUST NOT be built during v1, but v1 data structures MUST NOT preclude them:
  competition/division and country MUST be explicit fields on stadium and club records, never
  implied by file location or hard-coded assumptions.
- Stadium and club identity MUST be modelled separately (a stadium may host more than one club).
- Features not traceable to the current release's spec MUST be rejected or deferred.

**Rationale**: A small, finished, polished v1 is worth more than a sprawling unfinished one;
explicit extension points make v2/v3 additive.

### IV. Data Stewardship & Graceful Degradation

Live data comes from a rate-limited free tier; the app MUST treat it as scarce and fallible.

- The football-data.org API key MUST NOT be shipped to the browser. All calls to
  football-data.org MUST go through server-side code (Vercel serverless/edge functions).
- Fixture responses MUST be cached server-side (and via HTTP cache headers) so that normal
  traffic stays well within free-tier rate limits; request volume MUST NOT scale linearly
  with visitors.
- Curated static data (stadium coordinates, capacity, year built, home club, official website,
  travel directions) MUST live in version-controlled, typed data files with a recorded source
  for each fact.
- External API responses MUST be validated at the boundary (runtime schema validation) and
  mapped to internal types; raw third-party shapes MUST NOT leak into UI components.
- If live fixtures are unavailable (rate limit, outage, invalid response), the map and static
  stadium info MUST still work, and the fixtures panel MUST show a designed fallback state.
- Displayed dates and kick-off times MUST be shown in UK local time (Europe/London).

**Rationale**: The free tier's limits and the risk of key exposure are the project's main
operational hazards; a stale or missing fixture list must never break the map.

### V. Performance & Accessibility Budgets

Cinematic MUST NOT mean slow or exclusionary.

- Camera animations and map interaction MUST target 60fps on desktop, 30fps minimum on mobile 
- The initial page MUST reach an interactive map in under 3 seconds on a fast 4G connection;
  Largest Contentful Paint MUST be ≤ 2.5s and Cumulative Layout Shift ≤ 0.1 on the production
  build.
- The layout MUST be responsive from 360px mobile width to desktop.
- Users with `prefers-reduced-motion` MUST receive shortened or non-animated transitions.
- All non-map UI MUST be keyboard operable with visible focus, meet WCAG 2.1 AA contrast, and
  expose stadium selection through an accessible alternative (e.g. a searchable list) in
  addition to map markers.

**Rationale**: A portfolio piece is judged on real devices and by real reviewers; performance
and accessibility are measurable signals of engineering quality.

### VI. Type-Safe Simplicity

- TypeScript MUST run in `strict` mode. `any` is prohibited outside of narrowly commented,
  justified exceptions; untrusted data MUST enter as `unknown` and be validated.
- Prefer the simplest solution that meets the spec (YAGNI). New dependencies, state-management
  libraries, or abstractions MUST be justified in the plan against a concrete need.
- Map logic (Mapbox setup, layers, camera control) MUST be isolated from presentational React
  components so that each can be reasoned about and tested independently.

**Rationale**: Strict types and minimal complexity keep a solo-built project maintainable and
make the codebase itself a credible portfolio artefact.

## Technology & Platform Constraints

- **Frontend**: React + TypeScript; Mapbox GL JS for the map and 3D terrain.
- **Data**: football-data.org free tier for fixtures; curated in-repo data for stadium details
  and travel directions.
- **Hosting**: Vercel (static frontend + serverless/edge functions for API proxying and caching).
- **Secrets**: All secrets MUST be supplied via environment variables and MUST NOT be committed.
  The Mapbox public access token is browser-visible by design and MUST be URL-restricted to the
  project's production and preview domains.
- **Attribution & licensing**: Mapbox and football-data.org attribution MUST be displayed as
  their terms require. Club crests and trademarks MUST NOT be used unless their licence permits.
- **Cost**: The project MUST operate within free tiers (Mapbox, football-data.org, Vercel Hobby)
  under expected traffic; any change that risks paid usage MUST be flagged in the plan.

## Development Workflow & Quality Gates

- Work proceeds spec-first via Spec Kit: specify → (clarify) → plan → tasks → implement. Each
  plan MUST include a Constitution Check against the principles above.
- Every merge to `main` MUST pass: type-check, lint, unit tests, and a production build.
- Data-transformation logic (API mapping, date-window filtering, nearest-city/travel lookup,
  stadium data validation) MUST have automated unit tests. A test MUST verify that the static
  dataset contains exactly the expected v1 stadiums with valid coordinates and required fields.
- Visual and interaction changes MUST be reviewed on a Vercel preview deployment, on both
  desktop and mobile viewport sizes, before merging.
- Performance and accessibility budgets (Principle V) MUST be checked (e.g. Lighthouse) before
  each release.

## Governance

- This constitution supersedes other project practices and conventions. Where a spec, plan or
  task conflicts with it, the constitution wins until it is amended.
- **Amendments**: Proposed via `/speckit-constitution`, documented with a Sync Impact Report,
  and committed with a message describing the change and version bump.
- **Versioning**: Semantic versioning applies to this document.
  MAJOR — removal or redefinition of a principle or a backward-incompatible governance change;
  MINOR — a new principle or section, or materially expanded guidance;
  PATCH — clarifications and wording fixes with no change in meaning.
- **Release-scope changes**: Moving to v2 or v3 scope requires a MINOR (or MAJOR, if principles
  change) amendment to Principle III before related specs are written.
- **Compliance**: Every plan's Constitution Check and every review MUST verify compliance.
  Any justified deviation MUST be recorded in the plan's Complexity Tracking section with the
  reason and the simpler alternative that was rejected.

**Version**: 1.0.0 | **Ratified**: 2026-10-26 | **Last Amended**: 2026-10-26
