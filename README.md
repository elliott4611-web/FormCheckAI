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

> **Login is required** to use the app once Supabase keys are set.

## Friends & video sharing
Adds profiles (usernames), friend requests, and sharing recorded videos with
friends. Run this **once** in the Supabase **SQL editor** (in addition to the
`sessions` table above):

```sql
-- Profiles (usernames)
create table profiles (
  id uuid primary key references auth.users on delete cascade,
  username text unique not null,
  email text,
  created_at timestamptz default now()
);
alter table profiles enable row level security;
create policy "profiles readable" on profiles for select to authenticated using (true);
create policy "insert own profile" on profiles for insert to authenticated with check (auth.uid() = id);
create policy "update own profile" on profiles for update to authenticated using (auth.uid() = id);

-- Friend requests / friendships
create table friends (
  id uuid primary key default gen_random_uuid(),
  requester uuid references auth.users on delete cascade,
  addressee uuid references auth.users on delete cascade,
  status text default 'pending',          -- 'pending' | 'accepted'
  created_at timestamptz default now(),
  unique (requester, addressee)
);
alter table friends enable row level security;
create policy "see own friendships" on friends for select to authenticated using (auth.uid() = requester or auth.uid() = addressee);
create policy "send requests"      on friends for insert to authenticated with check (auth.uid() = requester);
create policy "respond"            on friends for update to authenticated using (auth.uid() = requester or auth.uid() = addressee);
create policy "remove"             on friends for delete to authenticated using (auth.uid() = requester or auth.uid() = addressee);

-- Shared videos (metadata; the file lives in Storage)
create table shared_videos (
  id uuid primary key default gen_random_uuid(),
  owner uuid references auth.users on delete cascade,
  recipient uuid references auth.users on delete cascade,
  path text, movement text,
  created_at timestamptz default now()
);
alter table shared_videos enable row level security;
create policy "owner or recipient see" on shared_videos for select to authenticated using (auth.uid() = owner or auth.uid() = recipient);
create policy "owner insert"           on shared_videos for insert to authenticated with check (auth.uid() = owner);
create policy "owner or recipient del" on shared_videos for delete to authenticated using (auth.uid() = owner or auth.uid() = recipient);

-- Storage bucket for the video files (public read; users upload to their own folder)
insert into storage.buckets (id, name, public) values ('videos','videos', true) on conflict (id) do nothing;
create policy "videos public read" on storage.objects for select using (bucket_id = 'videos');
create policy "videos owner upload" on storage.objects for insert to authenticated
  with check (bucket_id = 'videos' and (storage.foldername(name))[1] = auth.uid()::text);
create policy "videos owner delete" on storage.objects for delete to authenticated
  using (bucket_id = 'videos' and (storage.foldername(name))[1] = auth.uid()::text);
```

**Use:** **👥 Friends** (top-right) to add people by **username or email**, accept
requests, and watch videos shared with you. After recording, the save dialog has
**👥 Share with a friend** (uploads the clip and shares it). The `videos` bucket
is public-read (links are unguessable random paths) — switch to signed URLs if
you want them fully private.

## Tech
Vanilla HTML/CSS/JS, MediaPipe Pose (via CDN), Supabase (optional, via CDN), and
a small canvas-based 3D mannequin renderer — no build step, no install.
