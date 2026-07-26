# DriveFleet — Full Stack Architecture Document

---

## 1. Functional Requirements

### Authentication
- Register with name, email, photo URL, password
- Login with email + password
- Google OAuth login via Better Auth
- Logout (clears session cookie)
- Password validation: min 6 chars, at least 1 uppercase + 1 lowercase
- No email verification, no forgot password (per assignment rule)

### Public Pages
- **Home** — Banner, Available Cars (6+ from DB), 2 static extra sections
- **Explore Cars** — All cars grid, search by name ($regex), filter by type
- **Car Details** — Full car info + Book Now modal
- **Login** — Email/password form + Google OAuth button
- **Register** — Registration form + Google OAuth button
- **404** — Custom not-found page with Back to Home button

### Private Pages (authenticated users only)
- **Add Car** — Form to list a new car
- **Update Car** — Pre-filled form (owner only)
- **My Added Cars** — Grid of user's own cars with Update/Delete
- **My Bookings** — Table/cards of user's bookings

### CRUD — Cars
- Add car: name, daily rent, type, image URL (ImgBB), seats, location, description, availability
- Update car: price, description, availability, image, type, location (owner only)
- Delete car with confirmation modal (owner only)

### Booking System
- Book car from details page (modal or separate page)
- Fields: driver needed (yes/no), special note
- Total price = daily rent price
- Booking date = `new Date()`
- Increment `booking_count` on car via `$inc`
- My Bookings page shows: car name, total price, booking date, driver needed

### Search & Filter
- Search by car name using MongoDB `$regex` (case-insensitive)
- Filter by car type dropdown

---

## 2. Non-Functional Requirements

| Concern | Requirement |
|---|---|
| **Responsiveness** | Mobile, tablet, desktop — fluid grid, responsive navbar |
| **Design** | Clean, recruiter-friendly, unique (not copied from modules) |
| **SPA Reload** | No 404/error on any route reload; private routes persist auth |
| **Loading states** | Spinner/skeleton while data fetches |
| **Empty states** | Friendly message when no cars/bookings exist |
| **Error handling** | Toast notifications (React Hot Toast), no default alerts |
| **Card consistency** | Equal height/width across all car/booking cards |
| **Button consistency** | Same style across all pages |
| **No lorem ipsum** | All content must be meaningful |
| **Git commits** | Client: 15+ meaningful | Server: 8+ meaningful |
| **README** | Project name, live URL, 5+ bullet feature list |

---

## 3. Security Requirements

- **Better Auth** handles session management with HTTPOnly cookies — no manual JWT
- Server validates session on every protected API call via `auth.api.getSession()`
- Owner authorization: compare `car.addedBy` with `session.user.id` before update/delete
- MongoDB credentials in environment variables only
- CORS restricted to client origin
- Input validation on all endpoints (manual, no Zod — keep it simple)
- No secrets exposed to client

---

## 4. Overall System Architecture

```
┌──────────────────────────────────────────────────┐
│                    CLIENT                         │
│  Next.js 16 App Router (Vercel)                  │
│  ┌─────────────┐  ┌─────────────┐               │
│  │ Server      │  │ Client      │               │
│  │ Components  │  │ Components  │               │
│  │ (RSC)       │  │ ("use client")│              │
│  │ - Layout    │  │ - Forms      │               │
│  │ - Metadata  │  │ - Modals     │               │
│  │ - Data fetch│  │ - Interactive│               │
│  └──────┬──────┘  └──────┬──────┘               │
│         │                │                       │
│         │    Native fetch() calls                │
│         │    (server: direct, client: /api)      │
└─────────┼────────────────┼──────────────────────┘
          │                │
          ▼                ▼
┌──────────────────────────────────────────────────┐
│                    SERVER                         │
│  Express.js (Render)                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │  Better  │  │  Express  │  │ MongoDB  │       │
│  │  Auth    │  │  Routes   │  │ Native   │       │
│  │  Handler │  │  (Protected│  │ Driver   │       │
│  │          │  │   via     │  │          │       │
│  │          │  │  session) │  │          │       │
│  └──────────┘  └──────────┘  └──────────┘       │
│                                                  │
│  Session stored in HTTPOnly cookie               │
│  ─────────────────────────────────────           │
└──────────────────────────────────────────────────┘
          │
          ▼
┌──────────────────────────────────────────────────┐
│              MongoDB Atlas                        │
│  Collections: users, cars, bookings              │
└──────────────────────────────────────────────────┘
```

**Why this architecture:**
- Next.js App Router gives hybrid rendering — server components for SEO/perf, client components for interactivity
- Better Auth eliminates manual JWT/Cookie logic — production-grade auth out of the box
- MongoDB Native Driver avoids Mongoose overhead — lighter, more explicit queries
- Express remains the backend so auth and data APIs are unified on one server

---

## 5. Frontend Folder Structure

```
client/
├── public/
│   └── favicon.ico
├── src/
│   ├── app/
│   │   ├── layout.jsx                  # Root layout: html, body, providers
│   │   ├── page.jsx                    # Home page (Server Component)
│   │   ├── loading.jsx                 # Root loading skeleton
│   │   ├── not-found.jsx               # Custom 404 page
│   │   ├── error.jsx                   # Global error boundary
│   │   │
│   │   ├── explore-cars/
│   │   │   └── page.jsx                # All cars + search/filter
│   │   │
│   │   ├── car/
│   │   │   └── [id]/
│   │   │       └── page.jsx            # Car details page
│   │   │
│   │   ├── login/
│   │   │   └── page.jsx                # Login page (Client Component)
│   │   │
│   │   ├── register/
│   │   │   └── page.jsx                # Register page (Client Component)
│   │   │
│   │   ├── add-car/
│   │   │   └── page.jsx                # Add car form (Client Component)
│   │   │
│   │   ├── update-car/
│   │   │   └── [id]/
│   │   │       └── page.jsx            # Update car form (Client Component)
│   │   │
│   │   ├── my-added-cars/
│   │   │   └── page.jsx                # My cars list (Client Component)
│   │   │
│   │   └── my-bookings/
│   │       └── page.jsx                # My bookings list (Client Component)
│   │
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Navbar.jsx              # "use client" — responsive, conditional links
│   │   │   └── Footer.jsx              # Server Component — static content
│   │   │
│   │   ├── home/
│   │   │   ├── Banner.jsx              # Hero section with CTA
│   │   │   ├── AvailableCars.jsx       # Fetches 6 cars, renders CarCard grid
│   │   │   ├── WhyChooseUs.jsx         # Static section 1
│   │   │   └── RentalProcess.jsx       # Static section 2
│   │   │
│   │   ├── cars/
│   │   │   ├── CarCard.jsx             # Reusable car card (equal height via grid)
│   │   │   ├── CarGrid.jsx             # Responsive grid wrapper
│   │   │   ├── SearchFilter.jsx        # Search input + type dropdown
│   │   │   └── BookingModal.jsx        # Booking form modal ("use client")
│   │   │
│   │   ├── bookings/
│   │   │   └── BookingCard.jsx         # Booking display card
│   │   │
│   │   ├── shared/
│   │   │   ├── LoadingSpinner.jsx      # Full-page or inline spinner
│   │   │   ├── EmptyState.jsx          # "No data found" placeholder
│   │   │   ├── ErrorDisplay.jsx        # Inline error message
│   │   │   ├── ConfirmModal.jsx        # Delete confirmation dialog
│   │   │   └── ThemeToggle.jsx         # Dark/light toggle (optional)
│   │   │
│   │   └── auth/
│   │       ├── LoginForm.jsx           # Email + password form
│   │       ├── RegisterForm.jsx        # Registration form with validation
│   │       └── GoogleButton.jsx        # Google OAuth button
│   │
│   ├── lib/
│   │   ├── auth.js                     # Better Auth client instance
│   │   ├── api.js                      # Base fetch wrappers (server & client)
│   │   ├── cars.js                     # fetch cars, getCar, addCar, updateCar, deleteCar
│   │   ├── bookings.js                 # createBooking, getMyBookings
│   │   └── constants.js                # Car types, seat options
│   │
│   ├── middleware.js                    # Next.js middleware for auth redirect (optional)
│   ├── globals.css                      # Tailwind directives + custom styles
│   └── providers.jsx                    # Client provider wrapper
│
├── .env.local                           # NEXT_PUBLIC_API_URL
├── tailwind.config.js
├── next.config.js
└── package.json
```

**Why feature-based (co-located by route):**
- Next.js App Router naturally organizes by route segments
- `car/[id]/page.jsx` and `update-car/[id]/page.jsx` follow the framework convention
- Shared components live in `components/` organized by domain (cars, bookings, auth)
- `lib/` holds pure utility functions — no React, just data fetching logic

---

## 6. Backend Folder Structure

```
server/
├── src/
│   ├── config/
│   │   ├── db.js                       # MongoDB Native Driver connection (singleton)
│   │   └── auth.js                     # Better Auth configuration + handler
│   │
│   ├── lib/
│   │   └── getDB.js                    # Returns cached DB instance
│   │
│   ├── collections/
│   │   ├── users.js                    # User collection helpers (create, findByEmail, findById)
│   │   ├── cars.js                     # Car collection helpers (CRUD + search)
│   │   └── bookings.js                 # Booking collection helpers (create, findByUser)
│   │
│   ├── middlewares/
│   │   ├── authenticate.js             # Verifies Better Auth session, attaches user to req
│   │   ├── authorizeOwner.js           # Checks car ownership before update/delete
│   │   └── errorHandler.js             # Global error handler
│   │
│   ├── routes/
│   │   ├── authRoutes.js               # Better Auth handler mount + /me endpoint
│   │   ├── carRoutes.js                # CRUD + search/filter + my-cars
│   │   └── bookingRoutes.js            # Create booking + my-bookings
│   │
│   ├── controllers/
│   │   ├── carController.js            # Business logic for car endpoints
│   │   └── bookingController.js        # Business logic for booking endpoints
│   │
│   ├── services/
│   │   ├── carService.js               # Car data operations (calls collections)
│   │   └── bookingService.js           # Booking data operations (calls collections)
│   │
│   └── utils/
│       └── validate.js                 # Simple manual input validation helpers
│
├── .env                                # MONGO_URI, BETTER_AUTH_SECRET, etc.
├── index.js                            # Express app entry point
└── package.json
```

**Why collections + services + controllers:**
- **Collections** — raw MongoDB operations (findOne, insertOne, updateOne, aggregate). Pure data access, zero business logic.
- **Services** — orchestration layer. Combines collection calls, applies business rules (e.g., increment bookingCount after booking).
- **Controllers** — HTTP layer. Parses request, calls service, formats response.
- Three layers = testable in isolation, easy to modify one without touching others.

---

## 7. Route Structure

### Next.js Pages

```
/                          → Home (Server Component)
/explore-cars              → Explore Cars (Client Component — search params)
/car/[id]                  → Car Details (Server Component, data at build-ish)
/login                     → Login (Client Component)
/register                  → Register (Client Component)
/add-car                   → Add Car (Client Component, protected)
/update-car/[id]           → Update Car (Client Component, protected)
/my-added-cars             → My Added Cars (Client Component, protected)
/my-bookings               → My Bookings (Client Component, protected)
(not-found)                → 404 page
```

### Express API Endpoints

**Auth** (`/api/auth/*`) — handled by Better Auth, no manual routes needed beyond mounting

**Cars** (`/api/cars`)

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/cars` | No | All cars. Query: `?search=&type=` |
| GET | `/api/cars/my-cars` | Yes | Cars added by current user |
| GET | `/api/cars/:id` | No | Single car by ID |
| POST | `/api/cars` | Yes | Add a car listing |
| PUT | `/api/cars/:id` | Yes | Update car (owner only) |
| DELETE | `/api/cars/:id` | Yes | Delete car (owner only) |

**Bookings** (`/api/bookings`)

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/api/bookings` | Yes | Create booking + `$inc` bookingCount |
| GET | `/api/bookings/my-bookings` | Yes | Current user's bookings with car populated |

---

## 8. Database Collections & Schema

### `users` (managed by Better Auth)
Better Auth automatically creates and manages the users collection. Schema handled by the library.

Additional fields we may need to store alongside Better Auth:

```js
{
  _id: ObjectId,
  name: String,
  email: String,         // unique, indexed
  image: String,          // photo URL
  emailVerified: Boolean, // Better Auth managed
  createdAt: Date,
  updatedAt: Date
}
```

### `cars`

```js
{
  _id: ObjectId,
  carName: String,           // required
  dailyRentPrice: Number,    // required
  carType: String,           // "SUV" | "Sedan" | "Hatchback" | "Luxury" | "Convertible" | "Truck"
  image: String,             // ImgBB URL, required
  seatCapacity: Number,      // 2-10
  pickupLocation: String,    // required
  description: String,       // required
  availability: Boolean,     // default: true
  bookingCount: Number,      // default: 0
  addedBy: ObjectId,         // references users._id, indexed
  createdAt: Date,
  updatedAt: Date
}
// Indexes: { carName: "text" } for $regex optimization, { addedBy: 1 }, { carType: 1 }
```

### `bookings`

```js
{
  _id: ObjectId,
  userId: ObjectId,          // references users._id, indexed
  carId: ObjectId,           // references cars._id
  driverNeeded: Boolean,     // default: false
  specialNote: String,
  totalPrice: Number,        // copied from car.dailyRentPrice at booking time
  bookingDate: Date,         // new Date()
}
// Indexes: { userId: 1 }, { carId: 1 }
```

**Why native driver over Mongoose:**
- Lighter, fewer abstractions, explicit queries
- Better Auth doesn't require Mongoose — uses its own adapter or native driver
- Full control over MongoDB aggregation pipelines when needed
- Less magic — easier to debug

---

## 9. Authentication Flow (Better Auth)

### Setup

```js
// server/src/config/auth.js
import { betterAuth } from "better-auth";
import { mongodbAdapter } from "better-auth/adapters/mongodb";

export const auth = betterAuth({
  database: mongodbAdapter(client.db()),
  emailAndPassword: { enabled: true },
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    },
  },
  session: {
    cookieCache: { enabled: true, maxAge: 7 * 24 * 60 * 60 }, // 7 days
  },
});
```

### Flow

```
┌──────────┐          ┌──────────────┐          ┌──────────┐
│  Client  │          │   Express    │          │ MongoDB  │
│ (Next.js)│          │  + Better    │          │          │
│          │          │    Auth      │          │          │
└────┬─────┘          └──────┬───────┘          └────┬─────┘
     │                       │                      │
     │  POST /api/auth/      │                      │
     │  sign-in/email        │                      │
     │  {email, password}    │                      │
     │──────────────────────>│                      │
     │                       │  Verify credentials  │
     │                       │─────────────────────>│
     │                       │<─────────────────────│
     │  Set-Cookie:          │                      │
     │  better-auth.session  │                      │
     │  (HTTPOnly, Secure)   │                      │
     │<──────────────────────│                      │
     │                       │                      │
     │  GET /api/auth/       │                      │
     │  get-session          │                      │
     │  Cookie: session      │                      │
     │──────────────────────>│                      │
     │                       │  Validate session    │
     │                       │─────────────────────>│
     │  { user: {...},       │<─────────────────────│
     │    session: {...} }   │                      │
     │<──────────────────────│                      │
     │                       │                      │

Google OAuth:
     │  GET /api/auth/       │                      │
     │  sign-in/google       │                      │
     │──────────────────────>│                      │
     │  ← Redirect to Google │                      │
     │<──────────────────────│                      │
     │  Google OAuth flow    │                      │
     │  (user consents)      │                      │
     │─────────────────────────────────────────────>│
     │  ← Callback with code │                      │
     │<──────────────────────│                      │
     │  Set-Cookie: session  │                      │
     │<──────────────────────│                      │
```

### Client-side session check (Next.js)

```js
// src/lib/auth.js
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
});

export const { signIn, signUp, signOut, useSession } = authClient;
```

**Page reload persistence:**
- Session cookie is HTTPOnly — survives refresh by default
- On app mount, `useSession()` calls `GET /api/auth/get-session` automatically
- If session valid → user state populated → stay on private route
- If session invalid/missing → user is null → redirect to login

**Why Better Auth over manual JWT:**
- No need to write token generation, verification, refresh logic
- Google OAuth handled with zero custom code
- Session management with HTTPOnly cookies is production-grade
- Type-safe, well-tested, actively maintained

---

## 10. Express API Design

### Server Entry (`index.js`)

```js
import express from "express";
import cors from "cors";
import { auth } from "./src/config/auth.js";
import carRoutes from "./src/routes/carRoutes.js";
import bookingRoutes from "./src/routes/bookingRoutes.js";
import { errorHandler } from "./src/middlewares/errorHandler.js";

const app = express();

app.use(cors({ origin: process.env.CLIENT_URL, credentials: true }));
app.use(express.json());

// Better Auth handles all /api/auth/* routes
app.all("/api/auth/*", (req, res) => auth.handler(req, res));

// Protected API routes
app.use("/api/cars", carRoutes);
app.use("/api/bookings", bookingRoutes);

// Health check
app.get("/api/health", (req, res) => res.json({ status: "ok" }));

app.use(errorHandler);

app.listen(process.env.PORT, () => {
  console.log(`Server running on port ${process.env.PORT}`);
});
```

### authenticate middleware

```js
import { auth } from "../config/auth.js";

export const authenticate = async (req, res, next) => {
  try {
    const session = await auth.api.getSession({
      headers: new Headers({ cookie: req.headers.cookie || "" }),
    });

    if (!session) {
      return res.status(401).json({ error: "Not authenticated" });
    }

    req.user = session.user;
    req.session = session.session;
    next();
  } catch (error) {
    next(error);
  }
};
```

### Controller → Service → Collection pattern

```
Route receives request
  → authenticate middleware (if protected)
    → authorizeOwner middleware (if update/delete)
      → Controller: parses req, validates input
        → Service: orchestrates business logic
          → Collection: raw MongoDB operations
            → MongoDB
```

**Express route example (car routes):**

```js
// src/routes/carRoutes.js
import { Router } from "express";
import { authenticate } from "../middlewares/authenticate.js";
import { authorizeOwner } from "../middlewares/authorizeOwner.js";
import * as carController from "../controllers/carController.js";

const router = Router();

router.get("/", carController.getAllCars);            // Public
router.get("/my-cars", authenticate, carController.getMyCars);  // Protected
router.get("/:id", carController.getCarById);          // Public
router.post("/", authenticate, carController.addCar);  // Protected
router.put("/:id", authenticate, authorizeOwner, carController.updateCar);
router.delete("/:id", authenticate, authorizeOwner, carController.deleteCar);

export default router;
```

---

## 11. CRUD Flow

### Add Car
```
Client (form) → POST /api/cars { carName, dailyRentPrice, ... }
  → authenticate middleware (validates session)
    → carController.addCar()
      → carService.addCar(userId, carData)
        → validate(carData)
        → carsCollection.insertOne({ ...carData, addedBy: userId, ... })
      → Response: 201 + created car
```

### Update Car
```
Client (pre-filled form) → PUT /api/cars/:id { price, description, ... }
  → authenticate middleware
    → authorizeOwner middleware (checks car.addedBy === session.user.id)
      → carController.updateCar()
        → carService.updateCar(id, updateData)
          → validate(updateData)
          → carsCollection.updateOne({ _id: id }, { $set: updateData })
        → Response: 200 + updated car
```

### Delete Car
```
Client (confirm modal) → DELETE /api/cars/:id
  → authenticate middleware
    → authorizeOwner middleware
      → carController.deleteCar()
        → carService.deleteCar(id)
          → carsCollection.deleteOne({ _id: id })
        → Response: 200 + { message: "Deleted" }
```

### authorizeOwner middleware

```js
export const authorizeOwner = async (req, res, next) => {
  try {
    const { getDB } = await import("../lib/getDB.js");
    const db = await getDB();
    const car = await db.collection("cars").findOne({
      _id: new ObjectId(req.params.id),
    });

    if (!car) return res.status(404).json({ error: "Car not found" });
    if (car.addedBy.toString() !== req.user.id) {
      return res.status(403).json({ error: "Not authorized" });
    }

    req.car = car; // Pass to controller so we don't re-fetch
    next();
  } catch (error) {
    next(error);
  }
};
```

---

## 12. Booking Flow

```
Client (CarDetails page, "Book Now" button → BookingModal)
  → User fills: driverNeeded (yes/no), specialNote
  → POST /api/bookings { carId, driverNeeded, specialNote }

  Server:
    1. authenticate middleware
    2. bookingController.createBooking()
    3. bookingService.createBooking(userId, bookingData):
       a. Fetch car to get dailyRentPrice and verify availability
       b. If !car.availability → 400 error
       c. totalPrice = car.dailyRentPrice
       d. Insert booking: { userId, carId, driverNeeded, specialNote, totalPrice, bookingDate: new Date() }
       e. Increment car bookingCount: $inc { bookingCount: 1 }
    4. Response: 201 + created booking
```

### $inc for booking count

```js
// Inside bookingService.createBooking
await db.collection("cars").updateOne(
  { _id: new ObjectId(carId) },
  { $inc: { bookingCount: 1 } }
);
```

---

## 13. Search & Filter Strategy

### API: `GET /api/cars?search=toyota&type=Sedan`

```js
// carService.getAllCars(query)
const filter = {};

if (query.search) {
  filter.carName = { $regex: query.search, $options: "i" }; // case-insensitive
}

if (query.type) {
  filter.carType = query.type; // exact match on enum value
}

const cars = await db.collection("cars")
  .find(filter)
  .sort({ createdAt: -1 })
  .toArray();
```

### Client (Explore Cars page)

- URL search params used (`useSearchParams`) so filter survives page reload and is shareable
- Debounced search input (300ms) to avoid excessive API calls
- Type dropdown sends filter as query param

**Why `$regex` not `$text`:**
- `$regex` with `$options: "i"` is simpler for substring matching
- `$text` requires a text index and handles word-level search (overkill for car names)
- Assignment specifically asks for `$regex` operator

---

## 14. Authorization Rules

| Action | Who can do it |
|---|---|
| View all cars | Anyone (no auth) |
| View car details | Anyone |
| Add car | Authenticated users |
| Update car | Car owner only (addedBy === user.id) |
| Delete car | Car owner only |
| Book a car | Authenticated users |
| View own bookings | Authenticated user (their own) |
| View own added cars | Authenticated user (their own) |

Server enforces all rules. Client UI hides buttons but the server is the authority.

---

## 15. Rendering Strategy (Server vs Client Components)

### Server Components (RSC — no "use client")
| Component | Reason |
|---|---|
| `layout.jsx` | Static shell, wraps everything |
| `page.jsx` (Home) | Banner + static sections need no interactivity |
| `card/[id]/page.jsx` | Car details mostly static, fetch data server-side |
| `Footer.jsx` | Purely presentational |
| `Banner.jsx` | Static hero section |
| `WhyChooseUs.jsx` | Static content |
| `RentalProcess.jsx` | Static content |
| `CarCard.jsx` | Presentational — receives props, renders UI |
| `CarGrid.jsx` | Layout wrapper — receives children |
| `EmptyState.jsx` | Presentational placeholder |
| `BookingCard.jsx` | Presentational — receives booking data |

### Client Components (with "use client")
| Component | Reason |
|---|---|
| `Navbar.jsx` | Needs session state (user dropdown), mobile menu toggle |
| `LoginForm.jsx` | Form interactions, validation, submit |
| `RegisterForm.jsx` | Form interactions, validation, submit |
| `GoogleButton.jsx` | Opens OAuth popup/redirect |
| `SearchFilter.jsx` | Controlled inputs, debounce, URL params |
| `BookingModal.jsx` | Modal open/close state, form submit |
| `ConfirmModal.jsx` | Delete confirmation dialog, open/close state |
| `ThemeToggle.jsx` | Dark/light toggle state |
| `LoadingSpinner.jsx` | CSS animation (needs browser APIs) |
| `add-car/page.jsx` | Entire form page — many controlled inputs |
| `update-car/[id]/page.jsx` | Pre-filled controlled form |
| `my-added-cars/page.jsx` | Needs session, fetch + delete interaction |
| `my-bookings/page.jsx` | Needs session, fetch interaction |
| `explore-cars/page.jsx` | Search params, filter state, loading states |
| `login/page.jsx` | Redirects if already logged in |
| `register/page.jsx` | Redirects if already logged in |

**Rule of thumb:**
- If it needs `useState`, `useEffect`, `onClick`, or `onSubmit` → Client Component
- If it just renders data passed as props → Server Component
- Fetch data in Server Components where possible, pass down to Client Components as props
- For protected pages, fetch session client-side (Better Auth `useSession`)

---

## 16. Data Fetching Strategy

### Server Components (RSC) — direct fetch

```js
// Home page fetches available cars server-side
export default async function HomePage() {
  const res = await fetch(`${process.env.API_URL}/api/cars?limit=6`);
  const cars = await res.json();

  return (
    <>
      <Banner />
      <AvailableCars cars={cars} />
      <WhyChooseUs />
      <RentalProcess />
    </>
  );
}
```

### Client Components — native fetch in useEffect

```js
// AddCar page — session check then form
"use client";
import { useSession } from "@/lib/auth";
import { useState, useEffect } from "react";
import { useRouter } from "next/navigation";

export default function AddCarPage() {
  const { data: session, isPending } = useSession();
  const router = useRouter();

  useEffect(() => {
    if (!isPending && !session) router.push("/login");
  }, [session, isPending, router]);

  // ... form logic
}
```

**Why native `fetch()` over Axios:**
- Built into Node.js and browsers — zero dependencies
- Next.js extends `fetch()` with automatic caching, revalidation
- Simpler, no need for interceptors (Better Auth handles auth cookies natively)
- Fewer KB in bundle

### Data flow for protected data (My Cars, My Bookings)

```
Client Component mounts
  → useSession() → get session (or null while loading)
  → if session → fetch("/api/cars/my-cars") with credentials
  → if no session after loading → redirect to /login
  → on success → render data
  → on error → render ErrorDisplay
```

**Loading states:**
- `isPending` from `useSession()` → show spinner
- `loading` state from manual fetch → show spinner/skeleton
- Combined: show spinner until both session AND data resolve

---

## 17. Error Handling Strategy

### Server
- `errorHandler.js` middleware catches all uncaught errors
- Controllers use try/catch → pass to `next(error)`
- Error responses always JSON: `{ error: "Human-readable message" }`
- Never leak stack traces in production

```js
export const errorHandler = (err, req, res, next) => {
  console.error(err);
  const status = err.status || 500;
  const message =
    process.env.NODE_ENV === "production" && status === 500
      ? "Internal server error"
      : err.message;
  res.status(status).json({ error: message });
};
```

### Client
- Every fetch wrapped in try/catch
- On error → show `ErrorDisplay` component or toast (React Hot Toast)
- Network errors → "Unable to connect. Please try again."
- API errors → show `error.message` from response body

```js
try {
  const res = await fetch("/api/cars", { credentials: "include" });
  if (!res.ok) {
    const data = await res.json();
    throw new Error(data.error || "Something went wrong");
  }
  const cars = await res.json();
  setCars(cars);
} catch (err) {
  toast.error(err.message);
  setError(err.message);
}
```

### Global error boundary (`error.jsx` in App Router)
- Catches rendering errors
- Shows "Something went wrong" + "Try again" button

---

## 18. Loading & Empty States

### Loading strategy

| Scenario | UI |
|---|---|
| **Page navigation** | Next.js `loading.jsx` — skeleton/spinner as page loads |
| **Session check** | Full-screen `LoadingSpinner` until `useSession` resolves |
| **Data fetch (client)** | Conditional `LoadingSpinner` inline where data would render |
| **Form submit** | Button shows "Submitting..." + disabled state |
| **Image upload (ImgBB)** | Show upload progress or spinner |

### Empty states

| Scenario | UI |
|---|---|
| **No cars available** | EmptyState: "No cars available right now. Check back later!" |
| **No search results** | EmptyState: "No cars match your search. Try different keywords." |
| **No bookings** | EmptyState: "You haven't booked any cars yet. Explore cars to get started!" with link |
| **No added cars** | EmptyState: "You haven't added any cars. Add your first listing!" with link |

### Per-component loading pattern

```jsx
export default function MyAddedCarsPage() {
  const { data: session, isPending: sessionLoading } = useSession();
  const [cars, setCars] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!session) return;
    fetchMyCars()
      .then(setCars)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [session]);

  if (sessionLoading || loading) return <LoadingSpinner />;
  if (error) return <ErrorDisplay message={error} />;
  if (cars.length === 0) return <EmptyState message="..." linkTo="/add-car" />;

  return <CarGrid>{cars.map(car => <CarCard key={car._id} car={car} />)}</CarGrid>;
}
```

---

## 19. Deployment Strategy

### Frontend — Vercel

```
vercel.json (no file needed — Next.js auto-detected by Vercel)
```

- Next.js App Router handled natively by Vercel
- Server Components run on Vercel Edge/Serverless
- API calls to Render backend proxied or direct
- Environment variables set in Vercel dashboard

### Backend — Render (Web Service)

```
Build Command: npm install
Start Command: node index.js
```

- Express app on Render Web Service
- Better Auth session cookies work cross-origin (Vercel ↔ Render)
- CORS: origin = Vercel domain, credentials = true

### Database — MongoDB Atlas

- Free-tier M0 cluster
- IP whitelist: `0.0.0.0/0` (Render's IPs are dynamic)
- Connection string in Render env vars

### Environment Variables

**Client (`.env.local` / Vercel)**

```
NEXT_PUBLIC_API_URL=https://drivefleet-api.onrender.com
```

**Server (`.env` / Render)**

```
PORT=5000
MONGO_URI=mongodb+srv://...
BETTER_AUTH_SECRET=your-secret-key
BETTER_AUTH_URL=https://drivefleet-api.onrender.com
CLIENT_URL=https://drivefleet.vercel.app
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
NODE_ENV=production
```

### CORS config

```js
app.use(cors({
  origin: process.env.CLIENT_URL, // "https://drivefleet.vercel.app"
  credentials: true,              // Required for cookies
}));
```

### SPA routing on Vercel

Next.js handles this natively — no `vercel.json` rewrites needed. The App Router file-based routing prevents 404 on reload for all defined routes.

---

## 20. Development Roadmap

### Phase 1: Foundation (Day 1)
1. Initialize Next.js project with Tailwind + HeroUI
2. Initialize Express server with Better Auth setup
3. Set up MongoDB Atlas + native driver connection
4. Configure CORS, env vars, health check endpoint
5. Set up folder structures (client + server)
6. Git: Initial commit + project structure

### Phase 2: Authentication (Day 1-2)
7. Configure Better Auth on server (email/password + Google)
8. Mount Better Auth handler on Express
9. Create `authenticate` middleware
10. Build Login page with LoginForm + GoogleButton
11. Build Register page with RegisterForm + GoogleButton
12. Build Navbar with conditional links + session-based dropdown
13. Build `providers.jsx` + `auth.js` client setup
14. Git: Auth commits on both client and server

### Phase 3: Car CRUD — Server (Day 2)
15. Create cars collection helper
16. Create carService (business logic)
17. Create carController
18. Create car routes (all 6 endpoints)
19. Create `authorizeOwner` middleware
20. Test all car endpoints
21. Git: Server car CRUD commits

### Phase 4: Car CRUD — Client (Day 2-3)
22. Build Home page (Banner + AvailableCars + 2 static sections)
23. Build CarCard + CarGrid components
24. Build Explore Cars page with SearchFilter
25. Build Car Details page with full info
26. Build Add Car form page
27. Build Update Car form page
28. Build My Added Cars page with edit/delete
29. Build ConfirmModal for delete
30. Git: Client car CRUD commits

### Phase 5: Booking System (Day 3)
31. Create bookings collection helper
32. Create bookingService + bookingController (server)
33. Create booking routes (server)
34. Build BookingModal component
35. Build My Bookings page with BookingCard
36. Git: Booking commits on both sides

### Phase 6: Polish & Deploy (Day 3-4)
37. Build custom 404 page
38. Add LoadingSpinner, EmptyState, ErrorDisplay everywhere
39. Add React Hot Toast for success/error messages
40. Add Framer Motion animations (page transitions, hover effects)
41. Theme toggle (optional)
42. Responsive testing (mobile, tablet, desktop)
43. Live site testing — all routes, reload, auth persistence
44. Deploy server to Render
45. Deploy client to Vercel
46. Write README.md with project name, live URL, 5+ features
47. Git: Final commits, cleanup

---

## 21. Git Commit Plan

### Client commits (15+)

| # | Commit message | What it covers |
|---|---|---|
| 1 | `chore: initialize next.js project with tailwind and heroui` | Project setup, config files |
| 2 | `feat: add layout, navbar, and footer components` | PublicLayout, Navbar, Footer |
| 3 | `feat: implement better auth client and providers` | auth.js, providers.jsx, session hook |
| 4 | `feat: build login page with email/password and google oauth` | Login page, LoginForm, GoogleButton |
| 5 | `feat: build register page with password validation` | Register page, RegisterForm, validation |
| 6 | `feat: add navbar session state — conditional links and dropdown` | Navbar profile dropdown |
| 7 | `feat: build home page with banner, available cars, and static sections` | Home, Banner, AvailableCars, static sections |
| 8 | `feat: create car card and responsive grid components` | CarCard, CarGrid |
| 9 | `feat: implement explore cars page with search and filter` | ExploreCars, SearchFilter, debounce |
| 10 | `feat: build car details page with full info display` | Car details page |
| 11 | `feat: add booking modal with form and submit logic` | BookingModal |
| 12 | `feat: build add car form page with imgbb image upload` | Add car page |
| 13 | `feat: build update car page with pre-filled form` | Update car page |
| 14 | `feat: build my added cars page with edit and delete` | MyAddedCars, delete confirm |
| 15 | `feat: build my bookings page with booking cards` | MyBookings, BookingCard |
| 16 | `feat: add custom 404 not found page` | 404 page |
| 17 | `feat: add loading spinner, empty state, and error display components` | Shared components |
| 18 | `style: add framer motion animations and responsive polish` | Animations, responsive fixes |
| 19 | `docs: add readme with project info and live link` | README.md |

### Server commits (8+)

| # | Commit message | What it covers |
|---|---|---|
| 1 | `chore: initialize express server with better auth and mongodb` | Express setup, Better Auth, DB connection |
| 2 | `feat: implement car collection, service, and controller` | CRUD operations for cars |
| 3 | `feat: add authentication and authorization middlewares` | authenticate, authorizeOwner |
| 4 | `feat: add car routes with search and filter support` | All 6 car endpoints |
| 5 | `feat: implement booking collection, service, and controller with $inc` | Booking creation + bookingCount increment |
| 6 | `feat: add booking routes — create and my-bookings` | Booking endpoints |
| 7 | `feat: add global error handler middleware` | errorHandler |
| 8 | `chore: configure cors, env vars, and production settings` | Deployment-ready config |

---

## Key Architecture Decisions Summary

| Decision | Why |
|---|---|
| **Next.js App Router** | Hybrid rendering, file-based routing, SEO-friendly, native Vercel support |
| **Better Auth** | Production-grade auth (sessions, OAuth, HTTPOnly cookies), zero custom auth code |
| **MongoDB Native Driver** | Lightweight, explicit, no schema overhead, works with Better Auth adapter |
| **Native `fetch()`** | Zero dependencies, Next.js extended caching, no bundle bloat |
| **No React Hook Form** | Simple forms — native controlled inputs are sufficient and easier to debug |
| **No TanStack Query** | `fetch()` in Server Components + manual fetch in Client Components keeps it simple |
| **Feature-based structure** | Co-located by domain. App Router naturally supports this pattern |
| **Services layer** | Separates business logic from HTTP and data access — testable, maintainable |
| **URL params for search/filter** | Shareable URLs, survives refresh, no state management needed |
| **React Hot Toast** | Lightweight, customizable, assignment rule: no default alerts |
| **HeroUI** | Tailwind-compatible component library, cleaner than DaisyUI, recruiter-friendly aesthetic |
