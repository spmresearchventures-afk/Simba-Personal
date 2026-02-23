# Project Tracker

A single-page cross-device project tracker with Supabase backend, built as a static HTML file deployed on Cloudflare Pages.

## Purpose

Personal project management across two domains:
- **Ventures** (`#ventures`): Career, research, and aspirational projects (focused, ~10 max)
- **Life** (`#life`): Everything else — health, creative, spiritual, fashion, etc. (many, categorised)

Designed for ADHD-friendly workflow: low friction, glanceable status + next action, works on iPhone/iPad/Windows desktop without passwords after initial setup.

## Architecture

- **Frontend**: Single `index.html` file (vanilla HTML/CSS/JS, no build step)
- **Backend**: Supabase (Postgres + auth + row-level security)
- **Hosting**: Cloudflare Pages (static site from GitHub repo)
- **Auth**: Supabase magic link (email, no password). Session persists per device.
- **Sync**: Auto-saves to Supabase on every edit. Real-time cross-device.

## Tech Stack

- Supabase JS client v2 (CDN)
- Google Fonts: Playfair Display, DM Sans, JetBrains Mono
- No frameworks, no build tools, no dependencies beyond the CDN scripts

## Visual Design

- **Ventures mode**: Dark/deep theme with gold accents (#c8a44e) — focused, ambitious
- **Life mode**: Warm/earthy theme with green accents (#7aaa6e) — organic, expansive
- Mode determined by URL hash: `#ventures` or `#life`
- Grain texture overlay, subtle animations, mobile-responsive
- Save each mode as a separate home screen shortcut on mobile

## Data Model

```sql
CREATE TABLE projects (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  domain TEXT NOT NULL DEFAULT 'ventures' CHECK (domain IN ('ventures', 'life')),
  title TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'planning' CHECK (status IN ('active', 'planning', 'paused', 'dormant', 'done')),
  category TEXT DEFAULT 'general',
  next_action TEXT DEFAULT '',
  notes TEXT DEFAULT '',
  motivation TEXT DEFAULT '',
  sort_order INT DEFAULT 0,
  updates JSONB DEFAULT '[]'::jsonb,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users manage own projects"
  ON projects FOR ALL
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

CREATE INDEX idx_projects_user_domain ON projects(user_id, domain);
```

## Features

- Status filters: active, planning, paused, dormant, done
- Life mode categories: life, health, creative, fashion, spiritual (colour-coded)
- Quick update log per project (timestamped progress history)
- Motivation/why field (preserve reason for starting)
- Next action field (concrete next step)
- Notes field (working notes, links)
- Keyboard shortcuts: Cmd+N new project, Escape to close
- Mobile: bottom sheet modal, optimised touch targets, FAB for new project

## Setup

1. Create Supabase project (free tier, Asia-Pacific region)
2. Run the SQL script above in Supabase SQL Editor
3. Get Project URL and anon key from Settings → API
4. Deploy `index.html` to Cloudflare Pages via GitHub
5. Visit the deployed site — enter Supabase URL + anon key on setup screen (once per device)
6. Sign in via magic link email (persists)
7. Save `yoursite.pages.dev/#ventures` and `yoursite.pages.dev/#life` as separate bookmarks/shortcuts

## Supabase Project

- **Org**: Simba-Personal (free tier)
- **Region**: Asia-Pacific
- **Auth**: Magic link (email)
- **RLS**: Enabled — each user can only access their own projects

## Future Ideas

- Apple Shortcuts integration for one-tap access
- URL parameter to open directly to new project modal
- State-aware daily dashboard (different UI for medicated vs unmedicated states)
- Friction-scaled impulse buffer
- Tab limiter browser extension
