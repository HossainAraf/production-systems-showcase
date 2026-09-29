# Mollika Inn — Application Architecture

## 1. Overview

Mollika Inn is a Rails 8 hotel management application built as a single monolithic web app with two primary user experiences:

- a public guest-facing site for browsing rooms, checking availability, submitting booking requests, and contacting the inn
- a protected admin dashboard for managing bookings, room inventory, gallery content, dining reservations, facilities, reports, and settings

The application intentionally keeps the business domain in one codebase rather than splitting into separate services. This is a pragmatic choice for a boutique hotel operation where the main operational needs are booking, room inventory, rate management, and administrative content updates.

The architecture favors clarity, Rails conventions, and rapid iteration over distributed complexity. Core business logic lives in the model layer and controllers stay thin, with views rendered server-side via ERB and Hotwire.

---

## 2. Architectural goals

The system is designed around a few core principles:

- Keep the booking engine reliable and explicit
- Model hotel operations as domain entities, not generic CRUD objects
- Separate public guest routes from protected admin routes
- Minimize operational complexity by staying in one Rails app
- Use native Rails features where possible: Active Record, session auth, Active Storage, Solid Queue, and Hotwire
- Preserve historical booking data by snapshotting rates at booking time

---

## 3. System context

```text
Browser / guest user
        |
        v
Rails App (Mollika Inn)
  ├── public booking flows
  │     - room search
  │     - availability checks
  │     - booking creation
  │     - contact / dining / review submission
  │
  ├── protected admin flows
  │     - room management
  │     - booking operations
  │     - guest records and reports
  │     - media and settings management
  │
  ├── domain services and models
  │     - booking logic
  │     - rate selection
  │     - room availability checks
  │
  └── infrastructure
        - PostgreSQL database
        - Active Storage for uploads
        - Solid Queue for jobs
        - Solid Cache / Cable for app infra
```

The app is not built as a microservice system. Instead, it is a classic Rails monolith where the persistence model, business rules, and UI live in a single application process.

---

## 4. Technology choices

| Layer | Choice | Why it fits this app |
|---|---|---|
| Framework | Rails 8 | Full-stack MVC with strong conventions and fast iteration |
| Frontend | Hotwire + Turbo + Stimulus | Rich interactivity without a separate JavaScript build pipeline |
| CSS | Tailwind via CDN / app styles | Lightweight, easy styling for marketing + admin UI |
| Database | PostgreSQL | Strong support for relational hotel data and transactional flows |
| Schema | Custom `mollika` schema | Keeps app data organized and separate from generic app tables |
| Storage | Active Storage | Handles room photos, gallery media, and admin uploads |
| Background jobs | Solid Queue | Database-backed async jobs without Redis dependency |
| Caching | Solid Cache | Rails-native caching with minimal operational overhead |
| Realtime | Solid Cable | Native Action Cable without a separate infrastructure stack |
| Web server | Puma | Standard Rails production server |
| Deployment | Docker / Kamal / Render-ready config | Simple deploy model for a small production site |

---

## 5. Runtime architecture

### 5.1 Request flow

Guest requests and admin requests are handled by the same Rails application, but they are separated by routing and controller concerns.

Public requests flow like this:

1. Browser requests a route such as `/rooms`, `/bookings/new`, `/contact`, or `/`
2. A public controller handles the request without admin auth checks
3. The controller loads domain data from Active Record models
4. The view renders server-side HTML with ERB and Hotwire
5. Interactions such as availability checks happen through controller actions and partial rendering

Admin requests flow like this:

1. Browser requests a route under `/admin`
2. `Admin::BaseController` enforces login via session auth
3. Controller loads data for booking, room, rate, report, or content management screens
4. Admin UI renders from templates under `app/views/admin/`
5. Changes are persisted through standard Rails model validations and database transactions

### 5.2 Authentication model

The app does not use a traditional `User` model. Instead, admin access is intentionally minimal and environment-driven:

- session-based authentication through `Authentication` concern
- `Admin::BaseController` requires a valid admin session
- credentials are read from environment variables such as `ADMIN_EMAIL` and `ADMIN_PASSWORD`
- this is a deliberate trade-off to keep the app lightweight and avoid a broad app-user system for a single admin operator

This is enforced by the `Authentication` concern and the admin base controller, which redirect unauthenticated users to the login page.

---

## 6. Layered application structure

### 6.1 Controllers

The app uses Rails controllers to coordinate HTTP traffic and orchestrate business operations.

Public controllers handle guest-facing flows, including:

- `HomeController` — homepage and landing content
- `RoomsController` — room listing and room detail pages
- `BookingsController` — availability checks, hold logic, booking creation, booking confirmation
- `ContactsController`, `FacilitiesController`, `GalleryController`, `MenuController` — static and content-driven pages
- `ReviewsController` — guest review submission
- `DiningReservationsController` — restaurant table reservations

Admin controllers are namespaced under `Admin::*` and manage operational systems:

- `DashboardController` — hotel overview and admin homepage
- `BookingsController` — reservation lifecycle management
- `RoomsController`, `RoomTypesController`, `AvailabilitiesController` — room inventory and rates
- `GuestsController`, `ReviewsController`, `ReportsController` — guest and reporting operations
- `GalleryAlbumsController`, `GalleryImagesController` — media management
- `FacilitiesController`, `MenuItemsController`, `DiningReservationsController` — content and service management
- `ContactInquiriesController`, `SettingsController` — support and configuration

The admin controllers share a common base controller that enforces auth and loads notifications.

### 6.2 Models

Models are the domain layer for hotel operations. The key entities reflect the actual booking domain rather than generic admin CRUD resources.

Core domain model:

```text
Guest
  ├── has_many :bookings
  ├── has_many :reviews
  └── has_many :dining_reservations (if present in app scope)

Booking
  ├── belongs_to :guest
  ├── has_many :booking_rooms
  ├── has_many :rooms, through: :booking_rooms
  ├── has_many :room_types, through: :booking_rooms
  └── has_many :reviews

BookingRoom
  ├── belongs_to :booking
  ├── belongs_to :room
  └── belongs_to :room_type

Room
  ├── belongs_to :room_type
  ├── has_many :booking_rooms
  ├── has_many :bookings, through: :booking_rooms
  └── has_many :availabilities

RoomType
  ├── has_many :rooms
  ├── has_many :rates
  └── has_many :booking_rooms

Rate
  └── belongs_to :room_type

Availability
  └── belongs_to :room
```

This domain model is deliberately built around actual hotel operations:

- room types define what inventory exists
- room records represent physical rooms
- booking rooms act as the booking-to-room join and preserve price snapshots
- the rate model controls seasonal pricing logic
- availability blocks represent maintenance or closed dates

### 6.3 Views and presentation

The app uses server-rendered ERB templates with strong Rails conventions. Public pages are designed for guest conversion, while admin pages are operational and data-dense.

- public views live under `app/views/`
- admin views live under `app/views/admin/`
- layouts separate public and admin presentation concerns
- Turbo and Stimulus are used to add richer interactions without a separate frontend app

This is a key engineering decision: the app keeps a single, simple UI stack and avoids building a separate React or Vue layer.

---

## 7. Domain model and business logic

### 7.1 Booking flow

The central domain object is the booking. A booking is more than a row in a table; it is the operational unit for guest stay lifecycle management.

The key behavior in the model includes:

- status lifecycle: `pending`, `confirmed`, `checked_in`, `checked_out`, `cancelled`
- date validation to ensure check-out is after check-in
- room availability checks across overlapping date ranges
- rate snapshot storage on each booking room
- notification triggers for admin updates

The model also contains state-change helpers such as:

- `confirm!`
- `check_in!`
- `check_out!`
- `cancel!(reason:)`

These methods are intentionally part of the domain model because they encode the hotel operational workflow.

### 7.2 Pricing and rate snapshot strategy

The app records pricing in a way that protects historical integrity:

- `RoomType` defines a base price and a set of rate records
- `Rate` records represent seasonal or date-based pricing overrides
- `BookingRoom` stores `rate_per_night` and `total_amount` at the moment of booking

This is essential for a hotel system because future rate changes must not retroactively alter previously confirmed bookings. The system chooses the applicable rate for a date via the `price_for` logic and then snapshots the actual rate associated with that booking.

This is a strong engineering decision: preserve financial history over convenience.

### 7.3 Inventory and availability logic

Availability is not just a simple room count. It is derived from multiple inputs:

- room status (`available`, `maintenance`, `occupied`)
- blocked dates in `Availability`
- overlapping bookings in the reservation history

The `Room#available_between?` logic checks whether a room is available for a date range while avoiding overlapping reservation windows. This keeps availability checks consistent with the booking lifecycle.

---

## 8. Data persistence and schema strategy

The app uses PostgreSQL with a custom schema named `mollika`.

Key persistence characteristics:

- relational data model for reservations, room inventory, guest records, and content management
- transactional updates for booking creation and room assignment
- supporting app infrastructure tables for queue, cache, and cable state
- strong schema ownership in Rails migrations and generated `schema.rb`

This makes the system easier to reason about for hotel operations without requiring a NoSQL or distributed database layer.

---

## 9. Background jobs and async work

The app uses Solid Queue for asynchronous tasks and notification processing. This allows operational events to happen without blocking the main request cycle.

Examples include:

- booking notification jobs
- confirmation and reminder jobs
- admin update notifications
- future operational automation around bookings and messaging

This keeps the main booking flow responsive while still supporting background operational communication.

---

## 10. Content and admin management

Beyond hotel operations, the app also manages guest-facing content through domain records such as:

- `GalleryAlbum` and `GalleryImage` for photo assets
- `Facility` for amenity listings
- `MenuItem` for restaurant content
- `DiningReservation` for restaurant table bookings
- `ContactInquiry` for guest contact messages
- `Setting` for key-value configuration values

This keeps content editing close to the operational use cases instead of forcing a CMS layer into the architecture.

---

## 11. Route structure and separation of concerns

The app deliberately separates public and admin flows in the router:

```ruby
root "home#index"

resources :rooms, param: :slug
resources :bookings, only: [ :new, :create, :show ]

namespace :admin do
  root "dashboard#index"
  resources :bookings
  resources :rooms
  resources :room_types
  resources :guests
  resources :reports
  resources :settings
end
```

The route map reflects an important design decision: the public booking flow is intentionally not mixed with the admin management flow. This reduces risk and clarifies responsibility boundaries.

---

## 12. Security and operational practices

The app currently applies a focused security model rather than a broad enterprise identity framework:

- admin access is session-based and protected with an admin authentication concern
- public pages are intentionally open for guest browsing and booking requests
- admin-only resources are behind `Admin::BaseController`
- direct domain logic stays in models and controller actions rather than client-side security assumptions

Additional operational considerations include:

- environment-based configuration for admin credentials
- database-backed state for background tasks and sessions
- Rails defaults for CSRF, request validation, and form safety

---

## 13. Key engineering decisions

Several design choices define the app’s architecture:

1. Monolith first, not microservices
   - simpler deployment, easier debugging, lower operational overhead

2. Rails MVC with domain models as the core business layer
   - hotel logic remains readable and easy to evolve with the business

3. Rate snapshotting on each booking room
   - keeps historical pricing correct even when room rates change later

4. Single admin authentication model
   - small team, minimal operational burden, no need for a general-purpose user system

5. Hotwire over a heavy JavaScript frontend
   - faster delivery and lower frontend complexity for a boutique hotel website

6. One codebase for guest and admin workflows
   - easier shared model relationships and operational consistency

---

## 14. Strengths and trade-offs

### Strengths

- Clear domain model centered on bookings, rooms, and rates
- Good fit for a boutique accommodation business with moderate complexity
- Simple deployment and maintenance model
- Easy to extend for admin features without introducing service boundaries

### Trade-offs

- Business logic is concentrated in the Rails app, which can become large over time
- Admin and guest flows are closely coupled to the same database and app runtime
- Without further domain decomposition, the app may need clearer service boundaries as the business grows

---

## 15. Future architecture direction

If the system scales beyond a boutique inn, the most natural evolution would be to keep the monolith but split domain responsibilities into service objects and smaller module boundaries, rather than introducing a distributed architecture prematurely.

Potential next steps:

- formalize more business logic in dedicated service objects for booking creation and room assignment
- add more explicit domain validations around guest identity and payment states
- separate public and admin query patterns more clearly through read models or reporting scopes
- expand automated test coverage around booking, pricing, and room availability edge cases

This keeps the architecture simple while allowing it to scale in a controlled, Rails-native way.

