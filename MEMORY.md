# NYCDA KC Parent Tracker — Project Memory

## What This Is
A mobile-first web app for parents at the NYCDA Kansas City dance competition. Shows the full 512-entry schedule across 2 days, lets parents favorite dances, and syncs a real-time "Now Playing" indicator via Supabase.

## Architecture
- **Static site**: Single `index.html` with all HTML/CSS/JS inline (no framework, no build)
- **Data**: `schedule.json` (96KB, 512 entries) fetched at runtime
- **Realtime**: Supabase free tier — single `competition_state` table with one row
- **Favorites**: localStorage (`nycda_favorites` key, stored as JSON array of entry_ids)
- **Hosting**: Vercel static deployment from GitHub

## Live URLs
- **Site**: https://nycda-kc-tracker.vercel.app
- **Admin**: https://nycda-kc-tracker.vercel.app?admin=true
- **GitHub**: https://github.com/derekcolling/nycda-kc-tracker

## Supabase
- **Project**: "NYCDA" under "SundayNightAi" org
- **Project ID**: `uuqfcrbsqeagjzmrgzfe`
- **URL**: `https://uuqfcrbsqeagjzmrgzfe.supabase.co`
- **Table**: `competition_state` (id=1, current_entry_index, updated_at)
- **RLS**: Public read + public update enabled
- **Realtime**: Enabled via `supabase_realtime` publication
- Anon key (legacy JWT format) is embedded in index.html

## Schedule Data Structure
- Fields: `entry_id`, `routine_title`, `studio_code`, `category`, `approx_time`
- `entry_id` is a string (some have letters like "14A", "19A")
- `category` parses to: `{ageGroup} {genre} {type}` (e.g., "Junior Musical Theatre Solo")
- **Day 1**: entries up to index 238 (entry_id 1–234), 7:30 AM – 10:14 PM
- **Day 2**: entries from index 239 (entry_id 236–506), 7:00 AM – 11:23 PM
- Day boundary detected by PM→AM time transition (entry 234→236, id 235 skipped)
- 42 unique studio codes, 4 age groups, 10 genres, 8 performance types

## Key Implementation Details
- `currentEntryIndex` is the array index (0-based) into the sorted SCHEDULE array, not the entry_id
- Supabase stores `current_entry_index` as this array index
- Admin mode activated via `?admin=true` URL parameter
- The "Jump to #" input in admin bar expects an `entry_id` string, not an index
- Category parsing handles multi-word types like "Small Production" and "Duo/Trio"
- Filter dropdowns are dynamically populated from the data on load

## Deployment
- `vercel --prod --yes` from the project directory
- Project name had to be lowercase (`nycda-kc-tracker`) — Vercel rejects uppercase
- GitHub repo connected to Vercel for auto-deploy on push

## Files
```
/Users/derekcolling/Documents/NYCDA/
├── index.html       ← Complete SPA (~600 lines, all inline)
├── schedule.json    ← 512 dance entries (96KB)
├── schedule.pdf     ← Original PDF schedule
├── vercel.json      ← Rewrites + cache headers for schedule.json
└── .vercel/         ← Vercel project config (gitignored)
```

## Lessons Learned
- Supabase SQL Editor uses Monaco — use `window.monaco.editor.getEditors()[0].setValue()` to set content programmatically
- The `type` action in browser automation merges lines when text has newlines — use JS/Monaco API for multi-line code entry
- Supabase now has "Publishable keys" (sb_publishable_...) alongside legacy JWT anon keys — supabase-js v2 from CDN works with the legacy JWT format
- Browser automation screenshot coordinates are in viewport space, not screenshot image space — use `ref` IDs for reliable clicks
