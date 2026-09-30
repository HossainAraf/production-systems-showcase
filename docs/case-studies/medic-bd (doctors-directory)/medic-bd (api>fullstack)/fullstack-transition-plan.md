# MedicBD Fullstack Transition Plan

## Goal
- Transition from API-only Rails to SEO-friendly fullstack Rails without breaking existing API clients.
- Keep `/api/v1` stable while introducing server-rendered public pages.
- Launch an MVP that can prove search visibility and booking conversion impact.

## Core Business Logic (Must Preserve)
- Doctor discovery is always narrowed by both specialization and district.
- The main doctor listing flow should encourage users to select specialization first, then district.
- Chamber discovery is district-first and independent of doctor search.
- Chamber pages are for diagnosis, admission, and facility search, not primary doctor matching.
- Keep result sets narrow and relevant for Bangladesh local search behavior.

## Recommended Branch Strategy
- Dedicated foundation branch: `epic/fullstack-foundation`
- Long-running branch: `feat/fullstack-mvp`
- Feature branch pattern: `feat/fullstack-<feature>`
- Release branch pattern: `release/fullstack-m1`, `release/fullstack-m2`, `release/fullstack-mvp`

## Methodology
- Deliver in vertical slices (route + controller + view + test in each PR).
- Keep PRs small and shippable (one feature per PR).
- Build SSR-first pages; add Hotwire enhancements only after core content is crawlable.
- Keep API and web concerns separated:
  - API: JWT, JSON, stateless.
  - Web: server-rendered HTML, SEO metadata, performance, crawlability.
- Add acceptance criteria before implementation and verify with tests/review checklist.

## Proposed Milestones

### Milestone 0: Foundation (Week 1)
- Add web route namespace and base web controller.
- Add shared layout for web pages.
- Add reusable SEO helper (title, meta description, canonical).
- Add sitemap and robots setup.
- Ensure CI runs lint + tests for every PR.

### Milestone 1: Discovery Pages (Week 2-3)
- Doctor listing page (SSR, crawlable).
- Doctor detail page (SSR, crawlable, slug URL).
- Specialization listing page.
- District listing page.
- Internal links between doctors, specializations, districts, and chambers.
- Doctor search UI with required specialization + district filter pair.
- Chamber search UI with district-only filter.

### Milestone 2: Availability + Booking MVP (Week 4-5)
- Show chamber schedules on doctor detail pages.
- Add booking request form with instant confirmation feedback.
- Show schedule freshness timestamp (`Last updated`).
- Add event tracking for booking funnel.

### Milestone 3: Trust + Optimization (Week 6)
- Add structured data (JSON-LD) for doctors and listings.
- Add fragment caching for high-traffic pages.
- Improve Core Web Vitals and query performance.
- Launch monitoring dashboard for indexation and conversions.

## File Structure (Target)

```text
app/
  controllers/
    api/
      v1/
      base_controller.rb
    web/
      base_controller.rb
      home_controller.rb
      doctors_controller.rb
      specializations_controller.rb
      districts_controller.rb
  helpers/
    seo_helper.rb
  presenters/
    doctor_card_presenter.rb
  services/
    booking_requests/
      create.rb
  views/
    layouts/
      web.html.erb
    web/
      home/index.html.erb
      doctors/index.html.erb
      doctors/show.html.erb
      specializations/index.html.erb
      districts/index.html.erb
  javascript/
    controllers/
      booking_form_controller.js
config/
  routes/
    api.rb
    web.rb
  initializers/
    sitemap.rb
    seo.rb
test/
  controllers/
    web/
      doctors_controller_test.rb
      specializations_controller_test.rb
      districts_controller_test.rb
  integration/
    seo_pages_test.rb
  services/
    booking_requests/
      create_test.rb
docs/
  fullstack-transition-plan.md
  seo-mvp-spec.md
```

## Route Design (Example)
- Keep current API routes under `/api/v1` unchanged.
- Add web routes for crawlable pages:
  - `/` -> home
  - `/doctors` -> doctor index (requires specialization + district query params for filtered results)
  - `/doctors/:slug` -> doctor detail
  - `/specializations` -> specialization index
  - `/specializations/:slug` -> specialization detail
  - `/districts` -> district index
  - `/districts/:slug` -> district detail
  - `/chambers` -> chamber index (district filter first)
  - `/chambers/:id` -> chamber detail

## Filtering Contract (Web + API)
- Doctors:
  - Required filters: `specialization_id` and `district_id`
  - Optional filters can be added later, but must not replace the required pair in the main flow.
- Chambers:
  - Required filter: `district_id`
  - Optional filter: `category` (diagnostic, hospital, clinic) for refinement.
- UX rule:
  - If required filters are missing, show guided empty state and filter prompts instead of broad unfiltered results.

## MVP Scope (Must Have)
- SSR doctor index and doctor detail pages.
- SSR specialization and district pages.
- Chamber index page filtered by district.
- Unique `<title>`, meta description, canonical on each page.
- XML sitemap and robots.txt configured.
- Schedule block with freshness timestamp on doctor page.
- Booking request form on doctor detail page.
- Doctor search form enforcing specialization + district.
- Analytics events:
  - `doctor_profile_view`
  - `doctor_filter_applied`
  - `chamber_filter_applied`
  - `booking_click`
  - `booking_submit`
  - `booking_confirm`

## MVP Scope (Not in First Release)
- Payments.
- Telemedicine video calls.
- Full patient account system.
- AI recommendations.
- Complex multilingual content system.

## Data and SEO Standards
- One canonical URL per resource page.
- Use slug URLs everywhere for web entities.
- Avoid thin pages (add descriptive content blocks).
- Add breadcrumbs and internal links for crawl depth.
- Pagination on large lists; avoid rendering huge datasets in one response.
- Keep filtered URLs crawl-safe (index pages canonicalized, detail pages indexable).
- Preserve intent-driven landing pages for specialization + district combinations.

## Engineering Checklist Per Feature
- Route, controller, view, and tests included.
- SEO metadata present and verified.
- Query performance checked (no obvious N+1).
- Mobile layout verified.
- API compatibility confirmed (no `/api/v1` regressions).
- CI passing (`bundle exec rubocop --color`, `bundle exec rails test`).

## Definition of Done
- Feature is SSR and crawlable.
- Core flows work on mobile and desktop.
- Tests added and passing.
- Metadata/canonical present.
- No critical security or performance regressions.

## Metrics to Track from Day 1
- Indexed pages count.
- Organic landing pages.
- CTR from search results.
- Doctor profile views.
- Booking submissions.
- Booking confirmation rate.
- North-star metric: confirmed appointments per 100 doctor profile views.

## Initial Execution Tasks
These are the first implementation steps for the foundation phase.
1. Create branch: `git checkout -b epic/fullstack-foundation`
2. Split routes into API and web route files.
3. Add `Web::BaseController` and web layout.
4. Build `Web::DoctorsController#index` + view.
5. Build `Web::DoctorsController#show` + view.
6. Add SEO helper and page metadata wiring.
7. Build specialization and district listing/detail pages.
8. Add sitemap and robots configuration.
9. Add booking request form on doctor detail page.
10. Add tests and CI checks for web pages.

## Risks and Mitigation
- Risk: Breaking existing API behavior.
  - Mitigation: keep API controllers/routes untouched, add web in parallel.
- Risk: Thin content hurts SEO.
  - Mitigation: add descriptive copy and structured data for each page type.
- Risk: Slow pages due to large associations.
  - Mitigation: pagination, selective eager loading, fragment caching.

## Quick Start Command Set
```bash
git checkout -b epic/fullstack-foundation
bundle install
bin/rails routes
```
