<div align="center">

<img src="images/logo.png" alt="HotelsQuad logo" width="150" />

# HotelsQuad — Hotel Property Management System

Front desk, rooms, inventory and reporting in a single React + Firebase web app: bookings with live
room availability, check-in / check-out, folio and payments, stock control, and a real-time
operational dashboard.

[![React](https://img.shields.io/badge/React-16.14-61DAFB?logo=react&logoColor=white)](https://reactjs.org/)
[![Redux](https://img.shields.io/badge/Redux-4-764ABC?logo=redux&logoColor=white)](https://redux.js.org/)
[![Firebase](https://img.shields.io/badge/Firebase-Firestore%20v8-FFCA28?logo=firebase&logoColor=black)](https://firebase.google.com/docs/firestore)
[![Webpack](https://img.shields.io/badge/Webpack-5-8DD6F9?logo=webpack&logoColor=black)](https://webpack.js.org/)
[![Jest](https://img.shields.io/badge/tests-196%20total-C21325?logo=jest&logoColor=white)](#testing)
[![License](https://img.shields.io/badge/license-MIT-blue)](#license)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Application Routes](#application-routes)
- [Firestore Data Model](#firestore-data-model)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Available Scripts](#available-scripts)
- [Sample Data](#sample-data)
- [Configuration](#configuration)
- [Testing](#testing)
- [Deployment](#deployment)
- [Known Limitations and Roadmap](#known-limitations-and-roadmap)
- [Contributors](#contributors)
- [License](#license)

## Overview

HotelsQuad is a business-class property management system (PMS) for a single hotel property. It
replaces the spreadsheet-and-notebook workflow of a small front desk with one screen per job:

- **Bookings** are the source of truth. Each booking owns its guest details, assigned room, agreed
  nightly rate, payments and status (`pending` → `confirmed` → `arrived` → `departed`, or
  `cancelled`).
- **Rooms** are the physical inventory (number, type, AC, bed capacity, nightly rent, amenities,
  status). The booking form can only assign rooms that exist, are not in maintenance and are free
  for the selected dates.
- **Inventory** tracks the consumables and equipment the property keeps in stock, with automatic
  low-stock and out-of-stock badges.
- **Dashboard** and **Reports** read the three collections and compute KPIs, growth charts, status
  distribution and revenue directly from live Firestore snapshots.

There is **no mock or demo data layer**. Every screen subscribes to Firestore with `onSnapshot`, so
two browsers looking at the app see the same data, and empty collections render explicit empty
states instead of placeholder rows.

## Features

### Booking management

- Create, edit and delete bookings; booking IDs are validated and unique.
- Full form validation in [`utils/bookings.js`](utils/bookings.js): required fields, e-mail / phone
  format, guests ≥ 1, departure on or after arrival, no negative rate or payment, payment never
  exceeding `nights × rate`, and an `arrived` booking must have a room.
- **Room assignment with availability checking** — the picker only lists rooms that are not in
  maintenance and are free for the chosen dates. Date ranges are half-open (the departure day can be
  resold), and only `pending` / `confirmed` / `arrived` bookings block a room.
- **Folio and payments** — nights, total, balance and a payment badge (`unbilled`, `unpaid`,
  `partial`, `paid`, `void`) are derived from the booking, never invented.
- **Front-desk actions** — check-in, check-out and cancel update the booking *and* the room in one
  Firestore batch, so the room status can never drift from the booking status.
- **CSV export** of exactly what the current search and filters show.

### Rooms

- Create, edit, delete and search rooms; room numbers are unique.
- Track type, AC / non-AC, bed capacity, nightly rent (IDR), cancellation policy, amenities and
  status (`available`, `occupied`, `maintenance`).
- Filter by free text and by minimum bed capacity.

### Inventory

- Create, edit, delete and search items; item codes are unique.
- Categories: room, bathroom, kitchen, laundry, maintenance, electronics.
- Stock badges derived from quantity: *out of stock* (0), *low stock* (1–5), *in stock* (> 5), plus
  an **Only low stock** filter.

### Dashboard and reports

- Live KPI cards (total bookings, total guests, arrivals today, departures today).
- Booking growth per month, booking status donut, check-in / check-out progress and guests per month.
- `/reports` aggregates bookings, rooms by status, low-stock items and monthly revenue with a
  billed / collected / outstanding summary.

### Profile and property settings

- The header avatar, sidebar property card and profile page are driven by the Firestore `Settings`
  collection (`profile` and `property` documents) — there is no hard-coded persona or property code.

### User experience

- Navy + lavender design system shared by every screen (tokens in `docs/css/mixin.scss`, matching
  Material UI theme in `material-theme.js`), Lucide icons and pill controls.
- Responsive down to 390 px: the sidebar becomes an off-canvas drawer, tables scroll inside their
  card and the animations collapse under `prefers-reduced-motion: reduce`.
- Explicit loading, empty and error states (for example when Firestore security rules reject a read).

## Tech Stack

| Layer        | Choice                                                                   |
| ------------ | ------------------------------------------------------------------------ |
| UI           | React 16.14 (hooks), Material UI 4, `styled-components`, Lucide icons     |
| Charts       | Recharts 2                                                               |
| State        | Redux 4, `react-redux` 6, `redux-saga` 1, `reselect`                     |
| Routing      | `react-router-dom` 5 + `connected-react-router` 6                        |
| Backend      | Firebase 8 (Cloud Firestore, namespaced SDK)                             |
| i18n         | `react-intl` 2 with catalogs in `translations/` (en, de, fr, pt)         |
| Styling      | SCSS compiled by dart-sass through `sass-loader`                          |
| Build        | Webpack 5 + Babel 7 (`webpack.config.js`, `babel.config.js`)             |
| Tests        | Jest 26, `babel-jest`, Enzyme + React Testing Library                    |
| Linting/CI   | None configured — `npm test` is the gate before pushing                  |

## Application Routes

| Route                 | Screen        | Firestore collection | What it does                                          |
| --------------------- | ------------- | -------------------- | ----------------------------------------------------- |
| `/`                   | —             | —                    | Redirects to `/dashboard`                             |
| `/dashboard`          | Dashboard     | `AddBooking`         | Live KPIs, growth chart, status donut, progress bars  |
| `/add-booking`        | Add Booking   | `AddBooking`         | Validate and create a booking                         |
| `/view-booking`       | View Booking  | `AddBooking`         | Live list, search, check-in / check-out, delete, CSV  |
| `/edit-booking/:id`   | Edit Booking  | `AddBooking`         | Load by document ID, update, return to the list       |
| `/add-rooms`          | Add Room      | `Rooms`              | Validate and create a room                            |
| `/view-rooms`         | View Rooms    | `Rooms`              | Live list, search, capacity filter, delete, CSV       |
| `/edit-rooms/:id`     | Edit Room     | `Rooms`              | Load by document ID, update, return to the list       |
| `/add-item`           | Add Item      | `Inventory`          | Validate and create an inventory item                 |
| `/view-inventory`     | View Inventory| `Inventory`          | Live list, search, low-stock filter, delete, CSV       |
| `/edit-item/:id`      | Edit Item     | `Inventory`          | Load by document ID, update, return to the list       |
| `/reports`            | Reports       | all three            | Aggregated operations and revenue report              |
| `/profile`            | Profile       | `Settings`           | Staff profile and property identity                   |
| anything else         | 404           | —                    | `components/NotfoundPage`                             |

Every route except the redirect and the 404 page is wrapped in `containers/_PrivateRoute`, the
application shell (sidebar, header, breadcrumb, page heading).

## Firestore Data Model

The Firebase project is configured in **`firebase.config.json`**, imported by `utils/firebase.js`
(application) and by the seed scripts (Node), so the identifiers live in one place. The app uses the
Firebase **v8 namespaced SDK** (`firebase.firestore()`); the console snippet generated for the v9
modular SDK is passed to `firebase.initializeApp()` unchanged.

| Collection   | Written by                                            | Read by                                                         |
| ------------ | ----------------------------------------------------- | --------------------------------------------------------------- |
| `AddBooking` | `/add-booking`, `/edit-booking/:id` (`add` / `update`) | dashboard, `/view-booking`, `/reports`, header notifications     |
| `Rooms`      | `/add-rooms`, `/edit-rooms/:id` (`add` / `update`)     | booking room picker, `/view-rooms`, `/reports`                   |
| `Inventory`  | `/add-item`, `/edit-item/:id` (`add` / `update`)       | `/view-inventory`, `/reports`, header notifications              |
| `Settings`   | `/profile` (`set` with `merge: true`)                  | sidebar (`property`), header and profile page (`profile`)        |

### `AddBooking` document

```js
{
  bkId, fname, lname, email, phone,
  guestno,                    // number of guests
  selectedValue,              // gender: female | male | others
  roomType,                   // single | double | quad | king | queen
  roomId, roomNumber,         // assigned physical room ('' when unassigned)
  rate,                       // agreed nightly rate in IDR
  paid,                       // amount received so far
  statusValue,                // confirmed | arrived | departed | pending | cancelled
  arrivalDate, arrivalTime,   // 'YYYY-MM-DD' / 'HH:mm'
  departureDate, departureTime,
  checkedInAt, checkedOutAt,  // set by the front-desk actions
  createdAt, updatedAt,       // Firestore server timestamps
}
```

### `Rooms` document

```js
{
  roomNumber,                 // unique reference, e.g. '101'
  roomType,                   // single | double | quad | king | queen
  acType,                     // ac | nonAc
  bedCapacity,                // number, 1-20
  rentPerNight,               // number, Indonesian Rupiah
  phone,
  status,                     // available | occupied | maintenance
  cancellation,               // free | small | nil
  details,                    // free text
  amenities: [],              // bed, pillow, chair, table, tv, wifi, locker, swimmingPool
  createdAt, updatedAt,
}
```

### `Inventory` document

```js
{
  itemCode,                   // unique reference, e.g. 'ITM-101'
  name,
  category,                   // room | bathroom | kitchen | laundry | maintenance | electronics | other
  location,                   // free text, e.g. 'Shelf 4'
  quantity,                   // number, 0 or more
  createdAt, updatedAt,
}
```

### `Settings` documents

| Document            | Fields                                        | Used by                          |
| ------------------- | --------------------------------------------- | -------------------------------- |
| `Settings/profile`  | `name`, `role`, `email`, `phone`, `address`   | header avatar, `/profile`        |
| `Settings/property` | `name`, `code`                                | sidebar property card, `/profile`|

### Security rules

The app has no authentication yet, so the rules must allow the four collections while it is in use.
The snippet below is **development only** — tighten it (or add Firebase Auth) before exposing the app
publicly:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /AddBooking/{document} { allow read, write: if true; }
    match /Rooms/{document}      { allow read, write: if true; }
    match /Inventory/{document}  { allow read, write: if true; }
    match /Settings/{document}   { allow read, write: if true; }
  }
}
```

## Project Structure

```
app.js                  Application entry point (store, router, i18n, render)
configureStore.js       Store factory with saga and router middleware
reducers.js             Root reducer (router + language)
i18n.js                 react-intl locale data and translation messages
global-styles.js        Global SCSS imports
material-theme.js       Material UI theme mirroring the design tokens
index.html              Webpack HTML template
firebase.config.json    Firebase web app identifiers (shared with the scripts)
components/             Reusable presentational components
  BookingForm/          Shared booking form (add + edit)
  RoomForm/             Shared room form (add + edit)
  InventoryForm/        Shared inventory form (add + edit)
  Dashboard/            KPI cards, Recharts widgets
  Card/ Header/ Footer/ SidebarNav/ NotfoundPage/
containers/             Connected containers, pages and layouts
  __Routes/             Route table
  _PrivateRoute/        Application shell (sidebar, header, breadcrumb, page heading)
  Dashboard/Analytic/   Live dashboard
  Pages/                AddBooking, ViewBooking, EditBooking, AddRooms, ViewRooms, EditRooms,
                        AddItem, ViewInventory, EditItem, Reports, ProfilePage
  LanguageProvider/     Locale provider and persistence
utils/                  Domain helpers (see below)
docs/css/               Theme tokens and global partials
sass/elements/          Shared element styles (buttons, modal, dropdown, alerts)
images/                 Static assets
translations/           Locale catalogs (en, de, fr, pt)
scripts/                Node seeding / backfill scripts
tests/                  Jest setup and shared mocks
webpack.config.js       Webpack 5 configuration (aliases, SCSS, assets, dev server)
babel.config.js         Babel 7 presets and plugins
jest.config.js          Jest configuration
vercel.json             Vercel build/output settings and SPA fallback
```

### Domain helpers (`utils/`)

| File            | Responsibility                                                              |
| --------------- | --------------------------------------------------------------------------- |
| `bookings.js`   | Validation, nights / total / balance, payment status, availability, search   |
| `rooms.js`      | Room validation, normalisation, filtering, amenity labels                   |
| `inventory.js`  | Item validation, stock levels (`STOCK_LOW_THRESHOLD`), filtering            |
| `dashboard.js`  | KPI metrics, monthly growth, status distribution                            |
| `settings.js`   | Profile and property validation and defaults                                |
| `currency.js`   | IDR formatting (`formatCurrency`, `formatCompactCurrency`)                  |
| `csv.js`        | CSV export with `;` separator and UTF-8 BOM                                 |
| `firebase.js`   | `initializeApp` guard, shared `db`, `serverTimestamp`                       |
| `dates.js` etc. | Date helpers and the template's store / reducer / saga injectors            |

### Path aliases

Webpack and Jest resolve the same aliases, so imports such as
`import Card from 'components/Card'` or `import logo from 'images/logo.png'` work in both:

`components`, `containers`, `utils`, `images`, `sass`

## Getting Started

### Prerequisites

- **Node.js 18 or newer** (tested with Node 20) and npm 9+
- A **Firebase project with Cloud Firestore enabled** (the default config points at
  `tantekos-7036c`) and security rules that allow the four collections described above
- Basic familiarity with JavaScript, React hooks and Firebase

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/tantekosmail-gif/hotel-reservation.git
cd hotel-reservation

# 2. Install dependencies
npm install

# 3. Point the app at your Firebase project (optional if you use the bundled one)
#    Edit firebase.config.json → apiKey, authDomain, projectId, ...

# 4. Start the development server
npm start
```

The app is served on [http://localhost:3000](http://localhost:3000) with hot module replacement and
`historyApiFallback`, so deep links such as `/view-booking` work in development.

### Production build

```bash
npm run build          # emits a static site into build/
```

## Available Scripts

| Script                   | Description                                                        |
| ------------------------ | ------------------------------------------------------------------ |
| `npm start`              | Start the webpack dev server on port 3000 (HMR + history fallback)  |
| `npm run build`          | Create a production build in `build/`                               |
| `npm test`               | Run the Jest suite once                                            |
| `npm run test:watch`     | Run Jest in watch mode                                             |
| `npm run seed:rooms`     | Insert the 10 sample rooms into Firestore                          |
| `npm run seed:bookings`  | Insert the 15 sample bookings into Firestore                       |
| `npm run seed:inventory` | Insert the 14 sample inventory items into Firestore                |
| `npm run seed:rates`     | Backfill a nightly `rate` on bookings that do not have one         |

## Sample Data

Three idempotent seed scripts write **real documents** to Firestore (they are normal records, editable
from the UI — not an in-app mock layer). They call the Firestore REST API using
`firebase.config.json`, so no Admin SDK or service-account key is required, and records that already
exist are skipped.

```bash
npm run seed:rooms      # 10 rooms: 101-105, 201-203, 301, 401
npm run seed:bookings   # 15 bookings: BK-1001 ... BK-1015
npm run seed:inventory  # 14 inventory items: ITM-101 ... ELC-502
npm run seed:rates      # backfill nightly rates from the matching room type
```

- **Rooms** cover every room type, AC / non-AC, bed capacities 1–4, all three statuses and nightly
  rents from **Rp 350.000 to Rp 1.750.000**.
- **Bookings** use dates relative to the day the script runs, so the dashboard immediately shows
  arrivals and departures today, all four statuses and ~5 months of history for the growth charts.
- **Inventory items** cover all categories with mixed quantities (in stock, low stock, out of stock),
  so the stock badges and the *Only low stock* filter are visible right away.
- Re-running a script only reports what it skipped, e.g.
  `Done. Created 0, skipped 15 existing booking(s).`

## Configuration

| Setting                | File                | Notes                                                                 |
| ---------------------- | ------------------- | --------------------------------------------------------------------- |
| Firebase identifiers   | `firebase.config.json` | Shared by `utils/firebase.js` and `scripts/*.js`. These values are public client keys — access is controlled by Firestore rules. |
| Currency               | `utils/currency.js` | `CURRENCY_CODE = 'IDR'`, `CURRENCY_SYMBOL = 'Rp'`, `CURRENCY_LOCALE = 'id-ID'`. Every price in the UI goes through `formatCurrency`. |
| Low-stock threshold    | `utils/inventory.js`| `STOCK_LOW_THRESHOLD = 5`.                                            |
| Page headings/copy     | `containers/_PrivateRoute/index.js` | `PAGE_META` maps each route to its title and description. |
| Languages              | `i18n.js` + `translations/` | `en`, `de`, `fr`, `pt` catalogs.                              |

## Testing

```bash
npm test           # single run
npm run test:watch # watch mode
```

Current state of the suite:

```
Test Suites: 5 skipped, 34 passed, 34 of 39 total
Tests:       16 skipped, 180 passed, 196 total
```

- Configuration lives in `jest.config.js`: jsdom environment, `tests/setupTests.js` (Enzyme adapter +
  `@testing-library/jest-dom`), and the same path aliases as webpack.
- Styles and binary assets are mapped to `tests/__mocks__/{styleMock,fileMock}.js`.
- Coverage focuses on the domain helpers (`utils/tests/*`) — validation, folio maths, availability,
  dashboard metrics, CSV output — and on the connected containers, routes and app shell.
- Snapshots are committed under each container's `tests/__snapshots__/` folder; run
  `npx jest -u` deliberately when a UI change is intended.

## Deployment

`npm run build` produces a static site in **`build/`** (HTML, JavaScript, CSS, images, fonts and the
`.htaccess` file). The output directory and the single-page fallback are declared in `vercel.json`,
so the repository can be imported into Vercel without changing anything in the dashboard:

```json
{
  "framework": null,
  "buildCommand": "npm run build",
  "outputDirectory": "build",
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

### Vercel

1. **Add New → Project** in Vercel and import the repository; Vercel installs dependencies with npm.
2. Keep the *Framework Preset* on **Other** — `vercel.json` sets the build command (`npm run build`)
   and the output directory (`build`, **not** `public`).
3. Deploy. No environment variables are required: the Firebase identifiers come from
   `firebase.config.json`.

If a build still fails with `STATIC_BUILD_NO_OUT_DIR: No Output Directory named "public" found`, the
project-level *Output Directory* was overridden in the dashboard — set it to `build` (or clear the
override so `vercel.json` wins).

### Other static hosts

1. Upload the contents of `build/` to any static host (Apache, Nginx, Netlify, GitHub Pages,
   S3 + CloudFront, …).
2. Make sure unknown paths fall back to `index.html`, otherwise a refresh on `/view-booking` returns
   404. Ready-to-use configurations are included:
   - **Apache** — `.htaccess` (copied into `build/` by the webpack build)
   - **Nginx** — `.nginx.conf`
3. The Firebase project must allow the deployed origin. Web API keys can be restricted by HTTP
   referrer in the Google Cloud console, and the Firestore rules stay the real security boundary.

## Known Limitations and Roadmap

- **No authentication.** The routes are wrapped in `_PrivateRoute`, but that component is a layout,
  not an auth guard. The template's login / signup / forgot-password screens were removed instead of
  being left as dead UI, because this Firebase project has no Email/Password provider enabled. To add
  real auth: enable the provider, add a guard listening to `firebase.auth().onAuthStateChanged`, sign
  in with `signInWithEmailAndPassword`, and tighten the rules to
  `allow read, write: if request.auth != null;`.
- **Firestore rules are open** for the collections the app uses — development only.
- **Room photos are not persisted.** The old uploader posted files to a public `httpbin.org` test
  endpoint and discarded them, so it was removed. Wiring it up needs Firebase Storage (bucket +
  rules) and a `photoUrl` field on the room document.
- **No CI or linter** is configured; `npm test` is the current quality gate.
- Possible next steps: housekeeping board, invoice / PDF export, audit log of front-desk actions,
  role-based permissions for reception vs. manager.

## Contributors

- Sheena K Sasi
- Sherin Shaju
- Shijith M S
- Anjana Paul

Contributions are welcome: fork the repository, create a feature branch, keep `npm test` green and
open a pull request describing the change.

## License

Released under the **MIT License** (declared in `package.json`). Add a `LICENSE` file if you need the
full text distributed with the source.
