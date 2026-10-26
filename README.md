# SkillSwap

SkillSwap is a full-stack, peer-to-peer skill-exchange platform. Users list the skills
they can teach and the skills they want to learn, get matched with complementary people,
connect, chat, schedule learning sessions, and rate each other afterwards.

---

## 1. Features

- JWT authentication (register / login / logout / persistent session) with hashed passwords
- Profile setup with skills-to-teach, skills-to-learn, availability, bio, location
- Discover page with live backend search, filtering (skill, category, location, teaching,
  learning) and sorting (relevance / newest / rating)
- Percentage-based skill matching algorithm ("92% Skill Match")
- Connection requests (send / accept / reject / cancel) with tabs (All / Incoming / Sent / Accepted)
- Direct messaging with conversation list, unread counts, and read receipts
- Session scheduling between connected users, with accept / reject / cancel / complete flow
- Reviews & 1–5 star ratings after completed sessions, with automatic average-rating rollup
- Notifications (connection requests, acceptances, messages, session events, reviews) with
  an unread badge in the navbar
- Account settings: change name/email/password, privacy controls, notification preferences,
  and account deletion that cascades across all related data
- Fully responsive Bootstrap 5 UI (320px → 1920px), protected React routes, loading / error /
  empty states throughout
- Seed script with realistic demo data

---

## 2. Tech Stack

**Frontend:** React 18, Vite, React Router DOM, Bootstrap 5, Bootstrap Icons, Axios, Context API
**Backend:** Node.js, Express, MongoDB, Mongoose, JWT, bcryptjs
**Auth:** JWT stored in an httpOnly cookie (with a `localStorage` bearer-token fallback for
environments that block third-party cookies, e.g. some in-browser previews)

---

## 3. Folder Structure

```
skillswap/
├── client/                # React + Vite frontend
│   ├── src/
│   │   ├── components/    # Navbar, UserCard, SkillBadge, ProtectedRoute, etc.
│   │   ├── pages/         # Home, Discover, Dashboard, Messages, Sessions, Settings, ...
│   │   ├── layouts/       # MainLayout (Navbar + Footer wrapper)
│   │   ├── context/       # AuthContext, NotificationContext
│   │   ├── services/      # api.js — centralized Axios instance
│   │   ├── styles/        # theme.css — design tokens & custom styles
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── vercel.json        # SPA rewrite rule for React Router on Vercel
│   └── package.json
│
├── server/                 # Express + MongoDB backend
│   ├── config/db.js
│   ├── controllers/
│   ├── middleware/         # auth (JWT), errorHandler, asyncHandler
│   ├── models/             # User, Skill, Connection, Message, Session, Review, Notification
│   ├── routes/
│   ├── utils/               # generateToken.js, matching.js (skill-match algorithm)
│   ├── seed/seed.js
│   ├── server.js
│   └── package.json
│
├── .env.example
├── .gitignore
├── README.md
└── package.json             # convenience scripts for running both apps
```

---

## 4. MongoDB Setup

1. Create a free cluster at [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/register).
2. Create a database user and allow network access from your IP (or `0.0.0.0/0` for development).
3. Copy your connection string — it looks like:
   `mongodb+srv://<username>:<password>@cluster0.mongodb.net/skillswap?retryWrites=true&w=majority`
4. Paste it into `MONGO_URI` in your `.env` file (see below).

---

## 5. Environment Variables

Copy `.env.example` to `.env` inside `server/` and fill in the values:

```
MONGO_URI=mongodb+srv://<username>:<password>@cluster0.mongodb.net/skillswap
JWT_SECRET=replace_this_with_a_long_random_secret
JWT_EXPIRES_IN=7d
CLIENT_URL=http://localhost:5173
PORT=5000
NODE_ENV=development
```

Never commit your real `.env` file — it's already listed in `.gitignore`.

---

## 6. Installation

From the project root:

```bash
npm run install:all
```

This installs dependencies for both `server/` and `client/`. Alternatively, install each
individually:

```bash
cd server && npm install
cd ../client && npm install
```

---

## 7. Running the Backend

```bash
cd server
cp ../.env.example .env   # then fill in MONGO_URI and JWT_SECRET
npm run dev                # starts on http://localhost:5000
```

---

## 8. Running the Frontend

```bash
cd client
npm run dev                # starts on http://localhost:5173
```

The Vite dev server proxies `/api/*` requests to `http://localhost:5000` (see
`client/vite.config.js`), so no CORS configuration is needed locally beyond what's already
set up in `server.js`.

---

## 9. Database Seeding

Populate MongoDB with realistic demo data (users, skills, connections, messages, sessions,
reviews, notifications):

```bash
cd server
npm run seed
```

**Demo login:** `demo@skillswap.com` (or username `demo`) / password `demo1234`
All other seeded users share the password `password123` (their emails follow the
`firstname.lastname@example.com` pattern printed in the seed output).

---

## 10. API Overview

All endpoints are prefixed with `/api`.

| Method | Endpoint                      | Description                          | Auth |
|--------|--------------------------------|---------------------------------------|------|
| POST   | /auth/register                 | Create an account                     | No   |
| POST   | /auth/login                    | Log in                                | No   |
| POST   | /auth/logout                   | Log out                               | No   |
| GET    | /auth/me                       | Get current user                      | Yes  |
| GET    | /users                         | Search / filter / sort users          | Yes  |
| GET    | /users/matches                 | Recommended matches                   | Yes  |
| GET    | /users/:id                     | Get a user's profile                  | Yes  |
| PUT    | /users/:id                     | Update your profile                   | Yes  |
| PUT    | /users/:id/password            | Change password                       | Yes  |
| PUT    | /users/:id/email               | Change email                          | Yes  |
| DELETE | /users/:id                     | Delete account (cascades)             | Yes  |
| GET    | /skills                        | List / search skills                  | No   |
| POST   | /skills                        | Add a custom skill                    | Yes  |
| POST   | /connections                   | Send a connection request             | Yes  |
| GET    | /connections?tab=              | List connections (all/incoming/sent/accepted) | Yes |
| PUT    | /connections/:id                | Accept / reject / cancel              | Yes  |
| DELETE | /connections/:id                | Remove a connection                   | Yes  |
| GET    | /messages                       | List conversations                    | Yes  |
| GET    | /messages/:conversationId       | Get messages in a thread              | Yes  |
| POST   | /messages                       | Send a message                        | Yes  |
| POST   | /sessions                       | Request a session                     | Yes  |
| GET    | /sessions                       | List your sessions                    | Yes  |
| PUT    | /sessions/:id                   | Accept / reject / cancel / complete   | Yes  |
| DELETE | /sessions/:id                   | Delete a session                      | Yes  |
| POST   | /reviews                        | Review a completed session            | Yes  |
| GET    | /reviews/:userId                | Get reviews for a user                | No   |
| GET    | /notifications                  | List notifications                    | Yes  |
| PUT    | /notifications/:id/read         | Mark one as read                      | Yes  |
| PUT    | /notifications/read-all         | Mark all as read                      | Yes  |

---

## 11. Security Notes

- Passwords are hashed with bcrypt before being stored — never in plain text.
- JWTs are signed with `JWT_SECRET` and sent as an httpOnly cookie (falls back to a
  bearer token in `localStorage` for environments that strip cookies).
- All mutating routes are protected by the `protect` middleware, which verifies the JWT
  and attaches the current user to `req.user`.
- MongoDB credentials only ever live in `server/.env`, never in the frontend bundle.
- Centralized error handling avoids leaking stack traces or internal errors to clients in
  production.

---

## 12. Deployment

### Frontend (Vercel)

1. Push the repo to GitHub.
2. In Vercel, "Import Project" and set the **root directory** to `client/`.
3. Build command: `npm run build`. Output directory: `dist`.
4. Add an environment variable if you choose to point Axios at an absolute API URL instead
   of a relative `/api` proxy (e.g. `VITE_API_URL=https://your-api.onrender.com/api`), and
   update `client/src/services/api.js`'s `baseURL` accordingly.
5. `client/vercel.json` already contains the SPA rewrite rule needed for React Router to
   work on refresh/deep-links.

### Backend (Render, Railway, Fly.io, or any Node host)

1. Set the root directory to `server/`.
2. Build command: `npm install`. Start command: `npm start`.
3. Add the environment variables from `.env.example`, pointing `CLIENT_URL` at your deployed
   Vercel URL so CORS allows requests from production.
4. Make sure your MongoDB Atlas cluster's network access allows connections from your
   host's IP (or `0.0.0.0/0`).

---

## 13. Development Notes

- Everything is wired end-to-end — there are no static/mock datasets standing in for API
  calls, and no localStorage-as-database. `localStorage` is used only to cache the JWT
  bearer token as a cookie fallback.
- Real-time messaging currently uses polling-free REST (fetch-on-navigate); the
  `Message` model and conversation-ID scheme are structured so Socket.IO can be layered in
  later without a schema change.
- The admin role (`User.role === 'admin'`) exists in the schema and route middleware
  (`adminOnly`) is scaffolded in `middleware/auth.js`, ready for an admin dashboard to be
  built on top of it.
