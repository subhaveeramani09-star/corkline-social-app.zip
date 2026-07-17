# Corkline — Mini Social Media App

A small full-stack social app: user profiles, posts, comments, likes, and a
follow system. Backend is Express.js + SQLite, frontend is plain
HTML/CSS/JS (no build step, no framework).

## Features

- **Auth** — register / log in with a hashed password (bcrypt) and a JWT session token.
- **Profiles** — display name, bio, avatar URL, follower/following/post counts.
- **Posts** — create, delete, view in a global feed or a "following" feed.
- **Comments** — threaded replies under each post.
- **Likes** — like/unlike posts, with live counts.
- **Follow system** — follow/unfollow other users, view followers/following lists.

## Tech stack

- **Backend**: Node.js, Express.js, better-sqlite3 (file-based SQL database), bcryptjs, jsonwebtoken
- **Frontend**: Vanilla HTML, CSS, JavaScript (fetch API, no framework)
- **Database**: SQLite (single file, `db/social.db`, created automatically)

## Getting started

### 1. Install dependencies

```bash
cd social-app
npm install
```

### 2. Configure environment variables

```bash
cp .env.example .env
```

Open `.env` and set `JWT_SECRET` to a long random string (used to sign login tokens).

### 3. Run the server

```bash
npm start
```

The app will be available at **http://localhost:3000**. The SQLite database
file is created automatically at `db/social.db` on first run — no manual
setup needed.

For auto-restart on file changes during development:

```bash
npm run dev
```

## Project structure

```
social-app/
├── server.js              # Express app entry point
├── db/
│   └── database.js        # SQLite connection + schema (users, posts, comments, likes, follows)
├── middleware/
│   └── auth.js            # JWT verification middleware
├── routes/
│   ├── auth.js             # /api/auth/register, /api/auth/login
│   ├── users.js             # profiles, follow/unfollow, followers/following lists
│   ├── posts.js             # feed, create/delete post, like/unlike
│   └── comments.js          # list/create comments, delete a comment
└── public/                # static frontend
    ├── index.html
    ├── css/style.css
    └── js/app.js
```

## Database schema

- **users**: id, username, email, password_hash, display_name, bio, avatar_url, created_at
- **posts**: id, user_id, content, image_url, created_at
- **comments**: id, post_id, user_id, content, created_at
- **likes**: id, post_id, user_id, created_at (unique per post+user)
- **follows**: id, follower_id, following_id, created_at (unique per pair)

Foreign keys cascade on delete, so removing a user cleans up their posts,
comments, likes, and follow relationships automatically.

## API reference

All endpoints are prefixed with `/api`. Protected endpoints require an
`Authorization: Bearer <token>` header (token returned from register/login).

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | `/auth/register` | — | Create an account |
| POST | `/auth/login` | — | Log in, get a token |
| GET | `/users/:id` | optional | View a profile (includes counts, isFollowing if logged in) |
| PUT | `/users/me` | required | Update your own display name / bio / avatar |
| GET | `/users/:id/followers` | — | List followers |
| GET | `/users/:id/following` | — | List who they follow |
| POST | `/users/:id/follow` | required | Follow a user |
| DELETE | `/users/:id/follow` | required | Unfollow a user |
| GET | `/posts` | optional | Global feed (add `?feed=following` for your following feed) |
| GET | `/posts/user/:userId` | optional | Posts by one user |
| GET | `/posts/:id` | optional | A single post |
| POST | `/posts` | required | Create a post (`{ content, image_url }`) |
| DELETE | `/posts/:id` | required | Delete your own post |
| POST | `/posts/:id/like` | required | Like a post |
| DELETE | `/posts/:id/like` | required | Unlike a post |
| GET | `/posts/:postId/comments` | — | List comments on a post |
| POST | `/posts/:postId/comments` | required | Add a comment (`{ content }`) |
| DELETE | `/comments/:id` | required | Delete your own comment |

## Notes & next steps

- Passwords are hashed with bcrypt; never stored in plaintext.
- Tokens expire after 7 days (see `routes/auth.js`).
- This is a learning/demo project — for production you'd want rate limiting,
  input validation with a library like `zod`, image uploads instead of raw
  URLs, and pagination on feeds/lists.
