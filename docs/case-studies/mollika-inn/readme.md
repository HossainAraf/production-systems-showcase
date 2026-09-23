# Mollika Inn — Documentation Index

**Mollika Inn** is a Rails 8 hotel management system for a boutique inn in Naogaon, Bangladesh. It provides a guest-facing booking engine and a full hotel admin dashboard.

---

## Guides

| Document                        | Description                                              |
|---------------------------------|----------------------------------------------------------|
| [SETUP.md](SETUP.md)            | Installation, dependencies, database setup               |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Tech stack, data model, directory structure, URL map  |
| [AUTH.md](AUTH.md)              | How authentication works, protecting controllers         |
| [DEVELOPMENT.md](DEVELOPMENT.md)| Day-to-day dev workflow, adding features, Rails tips     |
| [ADMIN_GUIDE.md](ADMIN_GUIDE.md)| Admin panel walkthrough for each section                 |
| [QUICKSTART.md](QUICKSTART.md)  | Minimal steps to get a local dev environment running     |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to contribute and testing workflow                |
| [CHANGELOG.md](CHANGELOG.md)    | Repository changelog and release notes                  |

---

## Quick Start (Replit)

1. Press **Run** — the app starts on port 5000
2. Visit `/admin` and sign in:
   - Email: `admin@mollikainn.com`
   - Password: `mollika2025`
3. Visit `/` to see the public guest-facing site

---

## Public Routes

| URL                      | Description                          |
|--------------------------|--------------------------------------|
| `/`                      | Homepage with availability search    |
| `/rooms`                 | All room types                       |
| `/rooms/:slug`           | Room type detail + booking CTA       |
| `/bookings/new`          | Booking form                         |
| `/bookings/:id`          | Booking confirmation                 |
| `/menu`                  | Restaurant menu                      |
| `/dining_reservations/new` | Table reservation form             |
| `/gallery`               | Photo gallery                        |
| `/facilities`            | Hotel amenities                      |
| `/contact`               | Contact form                         |

---

## Admin Routes

All require sign-in at `/session/new`.

| URL                           | Description                    |
|-------------------------------|--------------------------------|
| `/admin`                      | Dashboard                      |
| `/admin/bookings`             | Booking management             |
| `/admin/rooms`                | Room management                |
| `/admin/room_types`           | Room type & rate management    |
| `/admin/guests`               | Guest profiles                 |
| `/admin/reviews`              | Review moderation              |
| `/admin/gallery_albums`       | Photo gallery management       |
| `/admin/facilities`           | Facilities management          |
| `/admin/menu_items`           | Restaurant menu management     |
| `/admin/dining_reservations`  | Table reservation management   |
| `/admin/contact_inquiries`    | Contact form inbox             |
| `/admin/reports`              | Occupancy, revenue, CSV export |
| `/admin/settings`             | Site-wide settings             |

---

## Stack Summary

- **Ruby 3.2** / **Rails 8.1**
- **PostgreSQL** with custom `mollika` schema
- **Hotwire** (Turbo + Stimulus) for frontend interactivity
- **Tailwind CSS** (CDN) for styling
- **Solid Queue + PostgreSQL Action Cable** — no Redis dependency
- **Active Storage** for file uploads
- **Propshaft + Importmap** — no Node.js build step

---

## Key Design Decisions

- **No User model** — single admin via env vars (`ADMIN_EMAIL`, `ADMIN_PASSWORD`)
- **Custom PostgreSQL schema** (`mollika`) — all tables namespaced, clean separation
- **Rate snapshots on BookingRoom** — `rate_per_night` and `total_amount` stored at booking time so historical prices are preserved even when rates change
- **Hotwire over React/Vue** — keeps the stack pure Rails, fast to iterate, no JS build pipeline
