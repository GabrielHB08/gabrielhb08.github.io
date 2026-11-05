# Project: private couple's memories site

A private website for two people: shared photo "moments", countdowns, a bucket list and a
map. Hosted on GitHub Pages at https://gabrielhb08.github.io (user site, served from `/`).
The repo is PUBLIC. Personal context lives in `CLAUDE.local.md` (gitignored).

## Hard rules
- **No personal content in the repo.** No names, messages, letters, photos, dates or captions
  in code, JSON or assets. All content lives in Supabase and is fetched after login.
  The repo holds only layout, styling, animations and logic.
- **Everything sits behind login.** A logged-out visitor sees only a neutral, pretty login
  screen: no names, no photos, nothing personal.
- Ask the user before making decisions not covered here. He prefers being asked over guessing.

## Stack
- **Frontend:** React + Vite (user knows React well). React Router; refreshing a deep link on
  GitHub Pages 404s, so use the `404.html` copy trick (or HashRouter).
- **Animation:** Motion (formerly Framer Motion) for scroll-driven effects.
- **Styling:** plain CSS with variables. Soft, pastel look; pink gradient background
  (gently animated) on the hero. Not bold.
- **Backend:** Supabase (Auth + Postgres + Storage), free plan.
  - Two accounts only, created in the dashboard; public sign-ups disabled.
  - Email + **password** login (no magic links; an email could spoil the surprise).
  - Row Level Security on every table: only authenticated users can read or write.
    Both users can create, edit and delete everything.
  - Photos in a **private** bucket, shown via signed URLs.
  - The anon key in frontend code is fine (public by design); RLS is the real protection.
  - Free projects pause after 7 days of inactivity: add a scheduled GitHub Action that
    pings the project every few days.
- **Deploy:** GitHub Actions builds with Vite and deploys to Pages on every push to `main`.
  Supabase URL and anon key come from repo secrets/vars at build time.
- **Map:** Leaflet with free tiles (pick a soft/pastel tile style).

## Devices
Exactly two layouts: **iPhone** (primary for viewing and uploading) and **laptop**.
- iPhone: single column, swipe gestures, nav reachable by thumb (bottom).
- Laptop: multi-column montage, small floating pill nav at the top.
- Suggest "Add to Home Screen" so Safari keeps the session.

## Photo uploads
- iPhone photos are often HEIC: convert to JPEG in the browser before upload.
- Resize in the browser (~2000px long edge, ~300–500 KB) and generate a thumbnail;
  Supabase image transforms are not on the free plan. 1 GB storage limit.
- Read the date taken from EXIF. Don't rely on GPS (iOS often strips it on browser upload):
  each moment has a location search box (geocode to lat/lng).
- No videos.

## Pages
1. **Home**: date-gated (see Anniversary mode).
   - Regular mode: full-screen pastel hero (photo collage + messages), then on scroll it
     transitions into a montage of moments (masonry, grouped by month/year, newest first).
2. **Moment (album)**: a trip/event with multiple photos, title, date, location and a short
   description. Swipeable gallery on iPhone.
3. **Countdown**: events (birthdays, holidays, trips, anything); live countdowns, soonest
   first; `repeats_yearly` events compute their next occurrence.
4. **Bucket list**: things to do together; check off, optionally link to the moment it became.
5. **Map**: pins for moments with a location; tapping a pin opens the moment.
6. **Manage**: add/edit/delete moments, photos, events, bucket items, messages (logged-in UI,
   e.g. "+ Add" buttons, not necessarily a separate page).

## Anniversary mode (fixed dates)
- **Dec 10 00:00 → Dec 17 23:59, America/Chicago.** Shown to **both** users.
- During the window the site shows ONLY the anniversary experience; the regular pages
  (montage, countdown, bucket list, map) are hidden.
- From **Dec 18** the regular site appears automatically, with no redeploy. Everything must
  be built, tested and deployed **before Dec 10**; nobody works on it during the week.
- Contents (all from Supabase, never in code):
  - a long letter
  - a photo slideshow
  - reasons I love you
  - favorite moments together
  - things I'm looking forward to
  - trips taken
- A one-time welcome animation on each user's first login during the window.
- Build a way to preview both modes before launch (e.g. a `?preview=anniversary` /
  `?preview=regular` override that only works when logged in).

## Data model (draft; refine before creating tables)
- `moments`: id, title, description, date, location_name, lat, lng, cover_photo_id, created_by, created_at
- `moment_photos`: id, moment_id, photo_path, thumb_path, taken_at, sort_order
- `events`: id, title, date, category, repeats_yearly, emoji, notes
- `bucket_items`: id, title, notes, done, done_on, moment_id (nullable)
- `messages`: id, text, sort_order (hero messages in regular mode)
- `anniversary_items`: id, section (`letter` | `reason` | `favorite_moment` |
  `looking_forward` | `trip`), title, body, photo_path, sort_order
- `anniversary_slides`: id, photo_path, caption, sort_order
- `profiles`: id (= auth user), display_name, first_seen_anniversary_at

## Plan (launch Dec 10)
| Week | Goal |
|---|---|
| Nov 5–11 | Vite + React scaffold, deploy workflow, Supabase project, tables + RLS, login screen |
| Nov 12–18 | Montage + moment album pages from Supabase; upload flow working on iPhone |
| Nov 19–25 | Anniversary mode + welcome moment + preview override; user starts adding real content |
| Nov 26–Dec 2 | Countdown, bucket list, pastel styling pass on both layouts |
| Dec 3–9 | Map; full test on a real iPhone with a test account; fixes; final deploy |

## Open questions
- Exact visual references (user is collecting Pinterest inspiration).
- How the anniversary sections are laid out (one long scroll vs. chapters).
- How she gets her password on Dec 10 (e.g. a card with a QR code to the site).

## Current state
- Repo contains only a hello-world `index.html` used to test GitHub Pages; replace it with
  the Vite app.
