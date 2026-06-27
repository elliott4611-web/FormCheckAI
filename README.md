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

## Tech
Vanilla HTML/CSS/JS, MediaPipe Pose (via CDN), and a small canvas-based 3D
mannequin renderer — no build step, no dependencies to install.
