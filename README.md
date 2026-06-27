# FormCheck — AI Workout Form Analyzer

A browser-based gym form analyzer. It uses your webcam + MediaPipe Pose to track
your joints in real time, grades your form on a 0–100 scale with per-criterion
breakdowns and coaching feedback, counts reps, and produces a full session report.

## Features
- **Live form grading** for 15 gym movements (squat, lunge, deadlift, push-up,
  bench, overhead press, curl, bent-over row, plank, glute bridge, tricep
  pulldown, Bulgarian split squat, seated row, pull-up, sit-up).
- **Readiness gate** — it won't grade until the joints it needs are in frame.
- **Auto rep counting** and per-movement session stats (best / average).
- **Stop Workout report** — a full-page session summary with per-movement bullets.
- **Form Guide** — interactive 3D mannequin demos (drag to rotate) showing proper
  vs. improper technique, with the equipment for each lift.
- **Searchable sidebar** grouped by Legs / Push / Pull / Core, light/dark theme,
  larger-text mode, and keyboard shortcuts.

## Running it
It's a single self-contained file — open `index.html` in a browser.

For the **webcam** to work, serve it over `http://localhost` or HTTPS (browsers
block camera access on `file://`). For example:

```bash
npx serve .
# or
python -m http.server 8000
```

then visit the printed URL. The Form Guide 3D demos work even from `file://`.

## Cloud sync (optional, via Supabase)
Log in by email and save each workout's per-movement scores/reps to the cloud,
then view your history across devices. It's disabled until you add keys.

**Setup**
1. Create a free project at [supabase.com](https://supabase.com) → **Project
   Settings → API**, and copy the **Project URL** and **anon public key**.
2. Near the top of the `<script>` in `index.html`, set:
   ```js
   const SUPABASE_URL = 'https://YOURPROJECT.supabase.co';
   const SUPABASE_ANON_KEY = 'your-anon-public-key';
   ```
   (The anon key is public by design — security comes from Row Level Security.)
3. In the Supabase **SQL editor**, run:
   ```sql
   create table sessions (
     id uuid primary key default gen_random_uuid(),
     user_id uuid references auth.users default auth.uid(),
     movement text,
     avg_score int,
     reps int,
     best int,
     hold_s int,
     is_hold boolean,
     created_at timestamptz default now()
   );
   alter table sessions enable row level security;
   create policy "own rows" on sessions
     for all using (auth.uid() = user_id) with check (auth.uid() = user_id);
   ```
4. Under **Authentication → URL Configuration**, add your site URL (e.g. your
   GitHub Pages URL, or `http://localhost:3000`) so magic-link logins redirect back.

**Use:** the top-right **☁ Log in** sends a magic link; after a workout, the
report's **☁ Save workout** button stores it; **📋 History** shows past sessions.
Magic links don't work from `file://` — use the hosted/localhost URL.

## Tech
Vanilla HTML/CSS/JS, MediaPipe Pose (via CDN), Supabase (optional, via CDN), and
a small canvas-based 3D mannequin renderer — no build step, no install.
