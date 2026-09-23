# Mollika Inn — Application Architecture

## Overview

Mollika Inn is a **Rails 8.1** full-stack hotel management system. It replaces a static WordPress site with a dynamic booking engine and a complete hotel admin dashboard.

```
Guest-facing site  →  Booking engine  →  Admin panel
(public routes)        (availability,      (dashboard,
                        payment form)       reservations,
                                            guests, reports)
```

---

## Tech Stack

| Layer            | Technology                          | Notes                                      |
|------------------|-------------------------------------|--------------------------------------------|
| Framework        | Rails 8.1.3                         | Full-stack, MVC                            |
| Frontend         | Hotwire (Turbo + Stimulus)          | SPA-like UX, no separate JS build step     |
| Styling          | Tailwind CSS (CDN in dev)           | Custom inn/gold color theme                |
| Database         | PostgreSQL                          | Custom `mollika` schema                    |
| Background jobs  | Solid Queue                         | Database-backed, no Redis needed           |
| Caching          | Solid Cache                         | Database-backed                            |
| WebSockets       | Solid Cable                         | Database-backed                            |
| File storage     | Active Storage (local in dev)       | Room photos, gallery images                |
| Asset pipeline   | Propshaft + Importmap               | No webpack/esbuild                         |
| Web server       | Puma                                | Multi-threaded                             |
| Deployment       | Kamal 2 (Docker)                    | Production deploys via SSH                 |

---

## Directory Structure

```
app/
├── controllers/
│   ├── application_controller.rb     # base; skips authentication by default
│   ├── home_controller.rb            # public homepage
│   ├── rooms_controller.rb           # public room listing & detail
│   ├── bookings_controller.rb        # public booking flow
│   ├── menu_controller.rb            # public restaurant menu
│   ├── dining_reservations_controller.rb  # public table booking
│   ├── gallery_controller.rb         # public photo gallery
│   ├── facilities_controller.rb      # public facilities page
│   ├── contacts_controller.rb        # public contact form
│   ├── reviews_controller.rb         # public review submission
│   ├── sessions_controller.rb        # admin sign-in / sign-out
│   ├── concerns/
│   │   └── authentication.rb         # session auth concern
│   └── admin/
│       ├── base_controller.rb        # all admin controllers inherit this
│       ├── dashboard_controller.rb
│       ├── bookings_controller.rb
│       ├── rooms_controller.rb
│       ├── room_types_controller.rb
│       ├── availabilities_controller.rb
│       ├── guests_controller.rb
│       ├── reviews_controller.rb
│       ├── gallery_albums_controller.rb
│       ├── gallery_images_controller.rb
│       ├── facilities_controller.rb
│       ├── menu_items_controller.rb
│       ├── dining_reservations_controller.rb
│       ├── contact_inquiries_controller.rb
│       ├── settings_controller.rb
│       └── reports_controller.rb
│
├── models/
│   ├── booking.rb          # core: statuses, state machine methods
│   ├── booking_room.rb     # join: booking ↔ room + rate snapshot
│   ├── room.rb             # physical room, status tracking
│   ├── room_type.rb        # category (Single/Double/Deluxe/Family)
│   ├── rate.rb             # seasonal pricing per room type
│   ├── availability.rb     # blocked dates per room
│   ├── guest.rb            # guest profile
│   ├── review.rb           # guest reviews (moderated)
│   ├── gallery_album.rb    # photo album
│   ├── gallery_image.rb    # individual photo (Active Storage)
│   ├── facility.rb         # hotel amenity/service listing
│   ├── menu_item.rb        # restaurant menu entry
│   ├── dining_reservation.rb  # restaurant table booking
│   ├── contact_inquiry.rb  # contact form submission
│   └── setting.rb          # key-value site configuration
│
├── views/
│   ├── layouts/
│   │   ├── application.html.erb   # guest-facing layout (dark green nav + gold CTA)
│   │   └── admin.html.erb         # admin panel layout (sidebar + top bar)
│   ├── home/                      # homepage with availability search
│   ├── rooms/                     # room listing + detail pages
│   ├── bookings/                  # multi-step booking form
│   ├── menu/                      # restaurant menu (public)
│   ├── dining_reservations/       # table reservation form
│   ├── gallery/                   # photo gallery
│   ├── facilities/                # facilities listing
│   ├── sessions/                  # admin login form
│   └── admin/                     # all admin views
│
└── javascript/
    └── controllers/               # Stimulus controllers
        ├── availability_controller.js   # AJAX availability check
        ├── date_picker_controller.js    # date range picker
        └── ...
```

---

## Data Model

```
RoomType ──< Room ──< BookingRoom >── Booking >── Guest
    |                                     |
    └──< Rate                         has_many :reviews
    └──< BookingRoom

GalleryAlbum ──< GalleryImage
Facility
MenuItem
DiningReservation
ContactInquiry
Setting (key-value store)
Availability (blocked dates per Room)
```

### Key relationships

- A **Booking** belongs to a **Guest** and can span multiple **Rooms** via `BookingRoom`
- **BookingRoom** stores a rate snapshot (`rate_per_night`, `total_amount`) at booking time
- **RoomType** has many **Rates** (seasonal pricing); `price_for(date)` picks the highest-priority applicable rate
- **Availability** records block specific dates on a Room (maintenance, etc.)

---

## Booking Status Flow

```
pending → confirmed → checked_in → checked_out
    └─────────────────────────────→ cancelled
```

State transitions use model methods:
- `booking.confirm!` — sets `confirmed_at` and marks booking rooms `occupied`
- `booking.check_in!` — marks rooms `occupied`
- `booking.check_out!` — marks rooms `available`
- `booking.cancel!(reason:)` — sets `cancelled_at`, frees rooms

---

## Authentication

- **No User model.** Authentication is session-based with a single admin account.
- Credentials are read from `ADMIN_EMAIL` / `ADMIN_PASSWORD` environment variables.
- The concern `Authentication` (in `app/controllers/concerns/`) provides:
  - `require_authentication` — before_action used by `Admin::BaseController`
  - `authenticated?` — helper available in views
  - `start_session(email)` / `end_session` — session management
- All `Admin::*` controllers inherit from `Admin::BaseController` which calls `require_admin!`
- Public controllers (`ApplicationController` subclasses) do **not** require auth

See [AUTH.md](AUTH.md) for full details.

---

## URL Structure

| Prefix      | Routes                    | Purpose                          |
|-------------|---------------------------|----------------------------------|
| `/`         | Public guest routes       | Homepage, rooms, booking, gallery|
| `/admin`    | Admin panel               | Protected by session auth        |
| `/session`  | Login/logout              | `SessionsController`             |
| `/guests/*` | Guest portal (Phase 2)    | Self-service booking management  |

---

## Configuration

- **`config/database.yml`** — uses `DATABASE_URL` env var in all environments
- **`config/environments/development.rb`** — `config.hosts.clear` enables Replit proxy access
- **`config/initializers/content_security_policy.rb`** — CSP headers
- **`app/models/setting.rb`** — runtime site config (name, phone, email, address) via `Setting["key"]`
