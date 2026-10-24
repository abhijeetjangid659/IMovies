# IMovies
A web based movie ticket booking site made using MERN stack.
# 🎬 iMovies – Movie Ticket Booking Website

A full-stack movie ticket booking application built with the **MERN stack** (MongoDB, Express.js, React, Node.js) in plain JavaScript (no TypeScript). Customers can browse movies, pick exact seats, pay online and get their tickets by email. Administrators manage movies, theatres, shows, bookings and users from a dashboard.

---

## ✨ Features

| Area | What it does |
|---|---|
| **Authentication** | Register, login and logout with JWT (httpOnly cookie). Separate customer and admin accounts. |
| **Movie listings** | Title, genre, language, duration, certificate, cast, director, ratings and release date. Now Showing / Upcoming tabs, search and filters. |
| **Theatres & shows** | Admins create theatres, screens with custom seat layouts and schedule shows with per-seat-type prices. Overlapping shows on a screen are blocked. |
| **Seat selection** | Visual seat map (Regular / Premium / Recliner) with live availability. Seats are held for 10 minutes while paying. Double booking is impossible (atomic database update). |
| **Bookings** | Confirm bookings, view history, cancel up to 2 hours before the show. |
| **Payments** | UPI, card, net banking and wallet with transaction tracking (simulated gateway – replaceable with Razorpay/Stripe). |
| **Watchlist & favourites** | Save movies; get an email when a show is added for a saved movie. |
| **Reviews & ratings** | 1–5 star reviews; average rating updates automatically. |
| **Admin dashboard** | Revenue, tickets sold, top movies, 7-day sales chart; manage movies, theatres, shows, bookings and users. |
| **Notifications** | Booking confirmation, payment receipt, cancellation and new-show emails (Nodemailer). |
| **Error handling** | Central middleware returns consistent JSON errors with proper HTTP status codes. |

## 🧰 Tech Stack

- **Frontend:** React 19, Vite, React Router v6, Context API, plain CSS
- **Backend:** Node.js, Express.js, JWT, bcryptjs, Helmet, CORS, Morgan, express-rate-limit, Nodemailer
- **Database:** MongoDB with Mongoose

---

## 📋 Prerequisites

- [Node.js](https://nodejs.org) 18 or newer
- [MongoDB Community Server](https://www.mongodb.com/try/download/community) running on port `27017` (or a MongoDB Atlas connection string)
- [MongoDB Compass](https://www.mongodb.com/try/download/compass) (optional, to view the data)
- [Postman](https://www.postman.com) (optional, to test the API)

## 🚀 Installation & Setup

### 1. Get the code
```bash
git clone <your-repository-url> IMovies
cd IMovies
```
(or extract the project zip so that the folder contains `client/` and `server/`)

### 2. Backend setup
```bash
cd server
npm install
```
Create or edit `server/.env`:

```env
PORT=5000
NODE_ENV=development
MONGO_URI=mongodb://127.0.0.1:27017/imovies
JWT_SECRET=change_this_to_a_long_random_string
JWT_EXPIRES_IN=7d
CLIENT_URL=http://localhost:5173

# minutes seats stay held while the customer pays
SEAT_HOLD_MINUTES=10

# admin account created by the seed script
ADMIN_NAME=Super Admin
ADMIN_EMAIL=admin@imovies.com
ADMIN_PASSWORD=Admin@123

# optional email settings (if SMTP_HOST is empty, emails are printed in the server console)
SMTP_HOST=
SMTP_PORT=587
SMTP_USER=
SMTP_PASS=
EMAIL_FROM=iMovies <no-reply@imovies.com>
```
> Use `127.0.0.1` instead of `localhost` in `MONGO_URI` – newer Node versions can resolve `localhost` to IPv6 and fail to connect.

Create the admin account and sample data (3 movies, 1 theatre, 2 shows):
```bash
npm run seed
```

### 3. Frontend setup
```bash
cd ../client
npm install
```
No `.env` is needed in development: `vite.config.js` proxies every `/api` request to `http://localhost:5000`, so there are no CORS problems and the login cookie works.

---

## ▶️ Running the Application

Open **two terminals** (MongoDB must be running first).

**Terminal 1 – Backend**
```bash
cd server
npm run dev        # auto-reload   (or: npm start)
```
API: http://localhost:5000 · Health check: http://localhost:5000/api/health

**Terminal 2 – Frontend**
```bash
cd client
npm run dev
```
Website: http://localhost:5173

### Default admin login
| URL | Email | Password |
|---|---|---|
| http://localhost:5173/admin/login | `admin@imovies.com` | `Admin@123` |

(Change these in `server/.env` before running the seed.) Customers create their own accounts at `/register`.

### Try the booking flow
1. Sign up → open a movie → choose a showtime.
2. Select seats → **Proceed to payment**.
3. Pay with UPI `me@upi` (card numbers ending in `0000` are declined – for testing).
4. See the ticket in **My Bookings**. The confirmation email is printed in the backend terminal if SMTP is not configured.

---

## 📁 Project Structure

```
IMovies/
├── client/                      # React frontend (Vite)
│   ├── vite.config.js           # dev server + /api proxy
│   └── src/
│       ├── main.jsx  App.jsx  api.js  utils.js  index.css
│       ├── context/AuthContext.jsx
│       ├── components/          # Navbar, MovieCard, ProtectedRoute, Toast
│       └── pages/               # Home, MovieDetails, SeatSelection, Checkout,
│           │                    # MyBookings, Watchlist, Login, Register
│           └── admin/           # Dashboard, Movies, Theatres, Shows, Bookings, Users
└── server/                      # Express backend
    ├── index.js                 # entry point
    ├── server.js                # app setup & route mounting
    ├── config/db.js
    ├── models/                  # user, admin, movie, theatre, show, booking, review, favourite
    ├── controllers/             # business logic
    ├── routes/                  # URL definitions
    ├── middleware/              # auth, error handler, ObjectId validation
    └── utils/                   # payment gateway, seat helpers, email, token, seed
```

## 🔌 API Overview

Base URL: `http://localhost:5000/api` (JSON). Protected routes use the login cookie or `Authorization: Bearer <token>`.

| Area | Endpoints |
|---|---|
| **Auth** | `POST /auth/register` · `POST /auth/login` · `POST /auth/logout` · `GET/PUT /auth/me` · `POST /admin/login` |
| **Movies** | `GET /movies` (`search`, `genre`, `language`, `status`, `sort`, `page`, `limit`) · `GET /movies/:id` · *admin:* `POST` `PUT` `DELETE` |
| **Theatres** | `GET /theatres` · `GET /theatres/:id` · *admin:* `POST` `PUT` `DELETE` and `/theatres/:id/screens[/:screenId]` |
| **Shows** | `GET /shows` (`movie`, `theatre`, `city`, `date`) · `GET /shows/:id` (seat map) · *admin:* `POST` `PUT` `DELETE` |
| **Bookings** | `POST /bookings` (hold seats) · `POST /bookings/:id/pay` · `GET /bookings/my` · `GET /bookings/:id` · `PUT /bookings/:id/cancel` |
| **Reviews** | `GET /reviews/movie/:movieId` · `POST /reviews` · `PUT/DELETE /reviews/:id` |
| **Favourites** | `GET /favourites` · `POST /favourites` · `DELETE /favourites/:movieId?type=` |
| **Admin** | `GET /admin/dashboard` · `GET /admin/users` · `GET/PUT/DELETE /admin/users/:id` · `GET /admin/bookings` |

Errors always look like `{ "success": false, "message": "..." }` with a suitable status (400, 401, 402, 403, 404, 409, 410, 500).

## 🔐 Authentication

- Passwords are hashed with bcryptjs; JWT (7 days) is stored in an httpOnly cookie and also returned in the response.
- Roles: `user` and `admin` (separate collections). Middleware: `protect`, `userOnly`, `adminOnly`.
- Login/register are rate limited (100 requests per 15 min per IP); Helmet adds security headers.

---

## 🛠 Troubleshooting

| Problem | Fix |
|---|---|
| `Cannot find module 'dotenv'` or `'nodemon' is not recognized` | Run `npm install` inside the `server` folder (and `client` folder for the frontend). |
| `Could not connect to MongoDB` / `ECONNREFUSED 127.0.0.1:27017` | Start MongoDB Server (Windows: `services.msc` → *MongoDB Server* → Start, or `net start MongoDB` as Administrator). Check `MONGO_URI`. |
| `imovies` database not visible in Compass | Run `npm run seed`, then refresh Compass. |
| Seed says "Data already present" | Drop the `imovies` database in Compass and run `npm run seed` again. |
| Frontend shows network / 500 errors | Make sure the backend is running on port 5000. |
| Admin page redirects to home | Log out of the customer account first, then use `/admin/login`. |
| Port already in use | Change `PORT` in `server/.env` (and the proxy target in `client/vite.config.js`). |

## ⚠️ Known Issues

- Payment is **simulated** (no real money moves); refunds are only recorded in the database.
- Notifications are **email only** (no SMS). Without SMTP settings emails appear in the server console.
- Expired seat holds are released when the show or bookings are next loaded, not by a background job.
- Seat availability refreshes every 15 seconds (polling), not in real time.
- No automated test suite yet; no password reset or email verification.
- For production deployment, serve the site over HTTPS and set `CLIENT_URL`, otherwise the login cookie may be blocked.

## 🔮 Future Enhancements

Real payment gateway (Razorpay/Stripe) · SMS notifications · QR-code e-tickets · real-time seat updates with Socket.IO · visual seat-layout designer · coupons and offers · food ordering · password reset · automated tests and CI/CD.

