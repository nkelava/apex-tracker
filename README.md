# Apex Tracker

A small web app for looking up Apex Legends player stats. Pick a platform, enter a gamertag, and get that player's selected legend, level, lifetime kills, damage and Season 6 wins, with percentile rankings.

It is built with an Express.js backend that proxies requests to the [Tracker Network (TRN) API](https://tracker.gg/developers) and a Vue 3 frontend. I started it as a project-based way to learn Vue.js.

## How it works

```
Vue client (localhost:8080)  ──►  Express API (localhost:5000)  ──►  Tracker Network API
```

The client never talks to TRN directly. The Express server adds your `TRN-Api-Key` header, forwards the request and returns the JSON response, so the API key stays on the server.

## Features

- Search by platform (PlayStation, Xbox, Origin) and gamertag
- Profile page with avatar, selected legend and legend artwork
- Stats with percentiles: Apex level, lifetime kills, damage done, Season 6 wins (each shown only if the player has it tracked)
- Toast warning when the gamertag field is empty, and a "Profile Not Found" message for unknown players
- Request logging with `morgan` in development mode

## Tech stack

| Part     | Tools                                                    |
| -------- | -------------------------------------------------------- |
| Backend  | Node.js, Express 4, Axios, dotenv, cors, morgan           |
| Frontend | Vue 3, Vue Router 4, Vue CLI 5, Axios, @meforma/vue-toaster |
| Dev      | nodemon, concurrently, ESLint                             |

## Prerequisites

- [Node.js](https://nodejs.org/) and npm (a version supported by Vue CLI 5, e.g. Node 16 or newer)
- A Tracker Network API key — create an app at [tracker.gg/developers](https://tracker.gg/developers)

## Installation

```bash
git clone https://github.com/nkelava/apex-tracker.git
cd apex-tracker

# Server dependencies
npm install

# Client dependencies
npm install --prefix client
```

## Configuration

The server loads environment variables from a `.env` file in the project root (it is git-ignored). Create it:

```bash
# .env
NODE_ENV=development
PORT=5000
TRACKER_API_URL=https://public-api.tracker.gg/v2/apex/standard
TRACKER_API_KEY=your-trn-api-key
```

| Variable          | Required | Description                                                                                  |
| ----------------- | -------- | -------------------------------------------------------------------------------------------- |
| `TRACKER_API_KEY` | Yes      | Your TRN API key, sent as the `TRN-Api-Key` header.                                          |
| `TRACKER_API_URL` | Yes      | Base URL of the TRN Apex API. The server requests `${TRACKER_API_URL}/profile/:platform/:gamertag`. |
| `PORT`            | No       | Port for the Express server. Defaults to `3000`, but **set it to `5000`** — the client calls `http://localhost:5000`. |
| `NODE_ENV`        | No       | Set to `development` to enable `morgan` request logging.                                     |

## Running the app

Start the server and the client together:

```bash
npm run dev
```

Then open http://localhost:8080 (the Vue CLI dev server's default port).

### Scripts

Root `package.json`:

| Command          | What it does                                        |
| ---------------- | --------------------------------------------------- |
| `npm run dev`    | Runs the server (nodemon) and client together        |
| `npm run server` | Runs only the Express server with auto-reload        |
| `npm run client` | Runs only the Vue dev server                         |
| `npm start`      | Runs the Express server with plain `node`            |

Inside `client/`:

| Command         | What it does                                  |
| --------------- | --------------------------------------------- |
| `npm run serve` | Vue dev server with hot reload                |
| `npm run build` | Production build into `client/dist`           |
| `npm run lint`  | Lints the client code with ESLint             |

## API

The backend exposes a single endpoint:

```
GET /api/v1/profile/:platform/:gamertag
```

- `platform` — `psn`, `xbl` or `Origin` (the values the search form sends)
- `gamertag` — PSN ID, Xbox Live gamertag or Origin ID

Example:

```bash
curl http://localhost:5000/api/v1/profile/psn/SomePlayer
```

On success it returns the TRN profile JSON unchanged. On failure it responds with TRN's status code and `{ "message": "<status text>" }`.

## Project structure

```
apex-tracker/
├── server.js              # Express app: loads .env, CORS, logging, mounts routes
├── routes/
│   └── profile.js         # GET /api/v1/profile/:platform/:gamertag → TRN API
├── package.json           # Server dependencies and dev/start scripts
└── client/                # Vue 3 frontend (Vue CLI)
    ├── public/
    │   └── index.html     # HTML template, favicon, Font Awesome
    ├── src/
    │   ├── main.js        # Creates the app, registers router and toaster
    │   ├── router.js      # Routes: / (search) and /profile/:platform/:gamertag
    │   ├── App.vue        # Layout and global styles
    │   ├── assets/        # Logo and background image
    │   └── components/
    │       ├── Header.vue   # Logo header
    │       ├── Search.vue   # Platform + gamertag search form
    │       └── Profile.vue  # Fetches and displays player stats
    ├── vue.config.js
    └── package.json       # Client dependencies and scripts
```

## Notes

- The API base URL `http://localhost:5000` is hard-coded in `client/src/components/Profile.vue`. Change it there if you run the server on another host or port.
- There is no production setup yet: the Express server only serves the API, not the built client.
- No automated tests are included.

## License

ISC, as declared in `package.json`.
