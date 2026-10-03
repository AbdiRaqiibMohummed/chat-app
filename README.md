# Supabase Realtime Chat

A real-time group chat app built with **React** and **Supabase**. Users sign in with Google, join a shared chat room, see who's online, and exchange messages instantly using Supabase Realtime (Broadcast + Presence).

## Features

- **Google sign-in** via Supabase Auth (OAuth)
- **Real-time messaging** using Supabase Realtime Broadcast
- **Online presence**: live count of users currently in the room
- **User avatars** pulled from the Google profile
- **Message timestamps** and auto-scroll to the latest message
- **Responsive UI** styled with Tailwind CSS

## Tech Stack

| Layer     | Technology                          |
| --------- | ----------------------------------- |
| Frontend  | React (Create React App), Tailwind CSS |
| Auth      | Supabase Auth (Google OAuth)        |
| Realtime  | Supabase Realtime (Broadcast, Presence) |

## Project Structure

```
src/
├── App.js                  # Chat UI, channel subscription, send/receive logic
├── hooks/
│   └── useSession.js       # Hook that tracks the current Supabase auth session
└── lib/
    └── supabaseClient.js   # Supabase client initialisation
```

## Prerequisites

- [Node.js](https://nodejs.org/) 18 or later
- A free [Supabase](https://supabase.com/) account and project
- A Google Cloud project with OAuth credentials (see below)

## Getting Started

### 1. Clone and install

```bash
git clone <your-repo-url>
cd <your-project-folder>
npm install
```

### 2. Set up environment variables

Create a `.env` file in the project root:

```dotenv
REACT_APP_SUPABASE_URL=https://your-project-ref.supabase.co
REACT_APP_SUPABASE_ANON_KEY=your-anon-public-key
```

You can find both values in your Supabase dashboard under **Project Settings → API**.

| Variable                      | Description                                         |
| ----------------------------- | --------------------------------------------------- |
| `REACT_APP_SUPABASE_URL`      | Your project URL (format: `https://<ref>.supabase.co`) |
| `REACT_APP_SUPABASE_ANON_KEY` | The public `anon` key. Safe for the browser when Row Level Security is used |

Make sure `.env` is listed in your `.gitignore`:

```gitignore
.env
.env.local
```

### 3. Configure the Supabase client

`src/lib/supabaseClient.js`:

```js
import { createClient } from "@supabase/supabase-js";

export const supabase = createClient(
  process.env.REACT_APP_SUPABASE_URL,
  process.env.REACT_APP_SUPABASE_ANON_KEY
);
```

### 4. Enable Google authentication

1. In the [Google Cloud Console](https://console.cloud.google.com/), create an **OAuth 2.0 Client ID** (type: Web application).
2. Add your Supabase callback URL as an **Authorized redirect URI**:
   `https://<your-project-ref>.supabase.co/auth/v1/callback`
3. In Supabase, go to **Authentication → Providers → Google**, enable it, and paste in the Google **Client ID** and **Client Secret**.
4. Under **Authentication → URL Configuration**, set the **Site URL** to `http://localhost:3000` for development, and add your production URL to the **Redirect URLs** list when you deploy.

### 5. Run the app

```bash
npm start
```

Open [http://localhost:3000](http://localhost:3000), sign in with Google, and open a second browser window (with a different Google account) to see messages and presence update in real time.

## How It Works

- **Channel:** every signed-in user subscribes to a Supabase Realtime channel called `room_one`.
- **Presence:** on subscribing, each user calls `track()` with their user ID. The `presence` `sync` event updates the "users online" count.
- **Broadcast:** when a user sends a message, it is added to local state immediately and broadcast to the channel as a `message` event. Other clients receive it and append it to their message list.
- **Cleanup:** the channel is unsubscribed when the user signs out or the component unmounts.

## Known Limitations

- **Messages are not persisted.** Broadcast is ephemeral, so history is lost on refresh and users only see messages sent while they're connected. To keep history, store messages in a Supabase table and load them on startup.
- **Single room.** All users share one hard-coded room (`room_one`).

## Ideas for Future Improvements

- Save messages to a Supabase Postgres table with Row Level Security
- Multiple chat rooms
- Typing indicators using Presence
- Prevent sending empty messages
- Display names instead of email addresses in the chat

## Scripts

| Command         | Description                          |
| --------------- | ------------------------------------ |
| `npm start`     | Run the app in development mode      |
| `npm run build` | Create an optimised production build |
| `npm test`      | Run tests                            |

## Deployment

Works with any static host (Vercel, Netlify, etc.). Add `REACT_APP_SUPABASE_URL` and `REACT_APP_SUPABASE_ANON_KEY` as environment variables in your host's settings, and add your deployed URL to Supabase's **Redirect URLs**.

## License

MIT
