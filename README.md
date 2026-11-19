# BIOLOOP – backend + frontend

Express + SQLite (built into Node) API that also serves the website from `public/`.

## Run
```bash
npm install
cp .env.example .env      # then set JWT_SECRET
npm start                 # http://localhost:3000
```
Demo logins (password `demo123`): `restaurant@bioloop.demo`, `biogas@bioloop.demo`,
`compost@bioloop.demo`, `farmer@bioloop.demo`. Demo data is seeded on first start.
Needs Node 22.13 or newer (SQLite is built in, so there is nothing native to compile).

## Project layout
| File | Purpose |
|---|---|
| `server.js` | All API routes, auth, validation |
| `db.js` | SQLite connection + table definitions |
| `seed.js` | Demo accounts and sample data |
| `constants.js` | Categories, crops, locations |
| `public/` | Frontend (index.html, style.css, script.js, forest-bg.jfif) |
| `data/bioloop.db` | Database file (created automatically) |
| `uploads/` | Photo proofs (saved as files, not in the DB) |

## API (all under `/api`, JSON, header `Authorization: Bearer <token>`)
| Method & path | Who | What |
|---|---|---|
| `POST /auth/register` `{email,password,name,role,phone}` | anyone | Create account, returns `{token,user}` |
| `POST /auth/login` `{email,password,role}` | anyone | Returns `{token,user}` |
| `GET /me`, `GET /state` | signed in | Current user / everything that user may see |
| `POST /listings` `{type,cat,kg,loc,when,phone,photo}` | restaurant | New waste/surplus listing (photo = base64 data URL, saved to `uploads/`) |
| `DELETE /listings/:id` | restaurant (owner) | Delete listing and photo |
| `POST /listings/:id/request` `{pid}` | restaurant (owner) | Request collection from a producer |
| `POST /listings/:id/advance` `{stage,recv?}` | biogas/compost (assigned) | Move Waste Passport to the next stage |
| `POST /listings/:id/reject` | biogas/compost (assigned) | Reject a new request |
| `POST /compost`, `DELETE /compost/:id` | compost producer | List / remove compost |
| `POST /produce`, `PATCH /produce/:id`, `DELETE /produce/:id` | farmer | Manage produce |
| `POST /orders` `{kind:"C" or "Pr",id,qty}` | farmer buys compost, restaurant buys produce | Place order (stock checked atomically) |
| `PATCH /orders/:id` `{status}` | seller | Placed → Accepted → Delivered |
| `POST /demo/reset` | anyone, only if `DEMO_MODE=true` | Wipes DB and reseeds demo data |

## What the server enforces
- Passwords hashed with bcrypt; JWT sessions (7 days); login and register are rate-limited.
- Role and ownership checks on every write (a producer can only advance requests assigned to them, one stage at a time).
- Restaurants see only their own listings; producers see only requests assigned to them. Phone numbers and photo proofs are not sent to anyone else through the API.
- Phone must be a 10-digit Indian mobile number; photos must be real JPEG/PNG/WebP files under 600 KB; collection time must be in the future.
- Business names and locations accept letters, numbers, spaces and `. , - ( ) # / :` only, because the frontend inserts that text into HTML.

## Before going live
- Set a strong `JWT_SECRET`, `NODE_ENV=production`, and `DEMO_MODE=false`.
- Serve behind HTTPS (nginx, Caddy or your host's proxy).
- Back up `data/bioloop.db` and `uploads/`.
