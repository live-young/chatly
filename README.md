# Chatly — a simple WhatsApp-style messenger

React (Vite) frontend · Node.js (Express + Socket.IO) backend · MongoDB (Mongoose)

## Features
- Sign up / log in with **phone number + password** (JWT auth, bcrypt-hashed passwords)
- **Add contacts** by phone number and save them under your own name
- **1-to-1 real-time chat** over WebSockets
- Unread counters, sent / read ticks, online + "last seen", "typing…" indicator
- People who message you before you've saved them show up by phone number, with an "Add to contacts" banner
- Responsive: two panes on desktop, one pane at a time on phones

## Run it

```bash
npm run setup      # installs root, server and client dependencies
cp server/.env.example server/.env   # then edit MONGO_URI / JWT_SECRET
npm run dev        # API on :5000, web app on http://localhost:5173
```

**No MongoDB installed?** Leave `MONGO_URI` empty (or don't create `.env`) and the server
starts a temporary in-memory MongoDB instead. Data is wiped when the server stops.

**Demo data:** with the server running, `npm --prefix server run seed` creates three demo
accounts with a few chats (the logins are printed, and listed in `server/scripts/seed.js`).

**Testing two users on one computer:** the login is stored per browser origin, so open
`http://localhost:5173` in one tab and `http://127.0.0.1:5173` in another (or use a private window).

## Project layout

```
server/src
  index.js            Express app + HTTP server
  socket.js           Socket.IO: auth, presence, typing, emitToUser()
  db.js               Mongo connection (falls back to in-memory)
  models/User.js      phone, name, password hash, contacts[], lastSeen
  models/Message.js   from, to, text, readAt
  routes/auth.js      POST /signup, POST /login, GET /me
  routes/contacts.js  GET / (chat list), POST / (add contact)
  routes/messages.js  GET /:userId, POST /:userId, POST /:userId/read
client/src
  App.jsx             login state
  api.js              fetch wrapper + token storage
  components/         AuthScreen, ChatApp, Sidebar, ChatWindow, AddContactModal, Avatar, Icons
```

## API

| Method | Path | Body | Notes |
|---|---|---|---|
| POST | `/api/auth/signup` | `{ phone, name, password }` | returns `{ token, user }` |
| POST | `/api/auth/login` | `{ phone, password }` | returns `{ token, user }` |
| GET | `/api/auth/me` | | |
| GET | `/api/contacts` | | chat list with last message + unread count |
| POST | `/api/contacts` | `{ phone, name? }` | the number must belong to a registered user |
| GET | `/api/messages/:userId` | | last 200 messages |
| POST | `/api/messages/:userId` | `{ text }` | also pushed live via `message:new` |
| POST | `/api/messages/:userId/read` | | notifies the sender via `message:read` |

All routes except signup/login need `Authorization: Bearer <token>`.

Socket events (server → client): `message:new`, `message:read`, `presence`, `typing`.
Client → server: `typing { to, typing }`.

## Not included yet (good next steps)
- **SMS OTP verification** — phone numbers aren't verified; plug in Twilio Verify / Firebase Auth at signup
- Group chats, media/voice messages, message delete/edit
- "Delivered" (double grey tick) state, push notifications
- Pagination for older messages
