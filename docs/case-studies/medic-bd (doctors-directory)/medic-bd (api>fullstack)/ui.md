# UI Plan (SEO-First Fullstack Rails)

## Objective
- Build server-rendered, crawlable pages first.
- Keep UI modular and reusable with clear partial boundaries.
- Support conversion flow: discovery -> doctor profile -> appointment request.
- Enforce domain behavior: doctor search uses specialization + district together; chamber search is district-first.

## Proposed View Structure

```text
app/views/
├── layouts/
│   ├── web.html.erb        # Main layout shell
│   ├── _nav.html.erb               # Global navigation
│   ├── _footer.html.erb            # Global footer
│   └── _seo.html.erb               # Title/meta/canonical/open graph tags
│
├── shared/
│   ├── _flash.html.erb             # Shared flash messages
│   ├── _empty_state.html.erb       # Shared no-data state
│   └── _pagination.html.erb        # Shared pagination component
│
├── home/
│   ├── index.html.erb              # Home page with navigation to core entities
│   └── _slides.html.erb            # Hero/slide content block
│
├── specializations/
│   ├── index.html.erb              # Specialization grid page
│   ├── show.html.erb               # Specialization detail + related doctors
│   ├── _card.html.erb              # Specialization card
│   └── _json_ld.html.erb           # Structured data for specialization pages
│
├── districts/
│   ├── index.html.erb              # District directory page
│   ├── show.html.erb               # District detail + related doctors/chambers
│   └── _card.html.erb              # District card
│
├── doctors/
│   ├── index.html.erb              # Doctor listing page
│   ├── show.html.erb               # Doctor profile detail page
│   ├── _filters.html.erb           # Required filters: specialization + district
│   ├── _card.html.erb              # Doctor card
│   ├── _profile.html.erb           # Doctor profile details block
│   ├── _availability.html.erb      # Schedule and freshness status
│   └── _json_ld.html.erb           # Structured data for doctor pages
│
├── appointments/
│   ├── new.html.erb                # Appointment booking form page
│   ├── create.turbo_stream.erb     # Optional future response when Hotwire/Turbo is introduced
│   └── _form.html.erb              # Reusable booking form
│
├── chambers/
│   ├── index.html.erb              # Chambers listing page (district-first)
│   ├── show.html.erb               # Chamber detail page
│   └── _card.html.erb              # Chamber card
│
├── static_pages/
│   ├── about.html.erb              # About page
│   ├── disclaimer.html.erb         # Medical/legal disclaimer
│   ├── contact.html.erb            # Contact page
│   └── privacy.html.erb            # Privacy policy
│
└── errors/
	├── 404.html.erb                # Not found page
	└── 500.html.erb                # Server error page
```

## Page Priority for MVP
- Priority 1: `doctors/index`, `doctors/show`
- Priority 2: `specializations/index`, `specializations/show`
- Priority 3: `districts/index`, `districts/show`
- Priority 4: `chambers/index`, `chambers/show`
- Priority 5: `appointments/new` with Turbo response

## Discovery Flow Rules
- Doctor discovery flow:
	- Select specialization
	- Select district
	- Show narrowed doctor list
- Chamber discovery flow:
	- Select district first
	- Optionally refine by chamber category
	- Show diagnosis/admission-focused chamber list

## SEO Requirements Per Public Page
- Unique title tag
- Meta description
- Canonical URL
- Structured data (JSON-LD) where relevant
- Internal links to related specialization, district, chamber, and doctor pages

## UX Requirements Per Public Page
- Mobile-first layout
- Fast first paint with server-rendered content
- Empty-state handling via `shared/_empty_state`
- Pagination for long lists via `shared/_pagination`

## Done Criteria for UI Slice
- Route + controller + view are complete
- Metadata is wired through `layouts/_seo`
- Page is crawlable without JavaScript execution
- Tests added for render/status/content basics
- Reusable partials extracted when duplication appears