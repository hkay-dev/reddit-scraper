# reddit-scraper — Knowledge

Observed facts and project state. See `rules.md` for actionable patterns.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **403 root cause (session)**: User hit 403 on `old.reddit.com/.../comments/....json?limit=500` and similar unauthenticated `.json` endpoints before OAuth migration. Source: OAuth migration session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Auth module design**: `auth.py` caches access token in module globals (`_token`, `_token_expires_at`), refreshes 60s before expiry via `expires_in - 60`. Uses `grant_type=client_credentials` with HTTP basic auth `(client_id, client_secret)`. Source: OAuth migration session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Scraper entry flow**: `scraper.py` -> `normalize_post_url()` -> `post_url_to_oauth_path()` -> `fetch_oauth_json()`. Response shape unchanged: `[post_listing, comments_listing]` with `data.children` nesting. Source: OAuth migration session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **localhost redirect URI**: For script apps, `http://localhost:8080` is a required Reddit prefs form field only; the scraper does not perform OAuth redirect/user login — only client_credentials. localhost = user machine (127.0.0.1). Source: OAuth migration session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Files touched in OAuth migration**: `reddit_scraper/scraper.py`, `auth.py` (new), `urls.py` (new), `pyproject.toml`, `README.md`, `scripts/README.md`, `.env.example`, `.gitignore`. Source: OAuth migration session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **openspec/project.md is stale**: Still claims "no authentication required", "append `.json` to URL", and "convert to old.reddit.com". README and implementation now document OAuth; update openspec on next spec pass. Source: OAuth migration session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Known limitations (unchanged)**: Does not expand "load more comments" (`kind: "more"`); public posts only; no built-in rate-limit backoff. Source: README + OAuth migration session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Responsible Builder Policy (~Nov 2025)**: Reddit introduced explicit approval requirements before API access. Self-service developer app creation at prefs/apps is largely unavailable for new developers; official path for non-commercial use is Devvit. Source: Responsible Builder Policy session, Reddit Help Center docs.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Session blocker (2026-06-07)**: User could not create a Reddit OAuth script app at https://www.reddit.com/prefs/apps despite completing all form fields. User had Devvit vars in `.env` (`devvit_project`, `project_id`) from `npm create devvit` — these do not satisfy `REDDIT_CLIENT_ID` / `REDDIT_CLIENT_SECRET`. Source: Responsible Builder Policy session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Unauthenticated `.json` shutdown (community reports)**: Unauthenticated `*.reddit.com/.../comments/....json` endpoints reportedly return 403 globally as of ~May 30, 2026. Aligns with session 403 on `old.reddit.com` before OAuth migration. Source: gallery-dl #8559, r/redditdev reports, Responsible Builder Policy session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Bot mitigation beyond headers**: Session debugging confirmed session cookies + browser User-Agent are insufficient; Reddit serves bot challenge pages and uses IP/TLS fingerprinting. Source: Responsible Builder Policy session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **OAuth path viability split**: Implementation (`auth.py` `client_credentials`, `scraper.py` OAuth-only) is correct for users with valid script app credentials. Blocker for new users is Reddit policy / credential onboarding, not scraper code. Source: Responsible Builder Policy session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Practical forks when OAuth credentials unavailable**:
  1. Apply for Data API access via Reddit Help Center sign-up forms (manual approval; community reports frequent denial for personal scrapers).
  2. Use pre-existing OAuth script app credentials if created before the policy change.
  3. Pivot scraper to browser-based fetch (e.g. Playwright) — no Reddit OAuth app required; different architecture.
  4. `curl_cffi` TLS/browser impersonation — fragile, may break with Reddit changes.
  5. Rebuild as on-platform Devvit app — different product model, not a drop-in for this CLI scraper.
  Source: Responsible Builder Policy session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **In-repo docs still stale on unauthenticated path**: `openspec/project.md` still claims no auth and old.reddit.com `.json` trick; `.env.example` and `auth.py` error text still instruct prefs/apps self-service without Responsible Builder Policy caveat. README documents OAuth but not the new-developer blocker. Source: Responsible Builder Policy session.

<!-- 2026-06-07 — knowledge-orchestrator-builder — Devvit vs standalone scraping -->

- **User question (session)**: Whether Devvit can be used to create a scrape tool, or if it doesn't support this use case. Answer: Devvit is NOT a replacement for the Python `reddit-scraper` OAuth CLI. Source: Devvit vs standalone scraping session.

<!-- 2026-06-07 — knowledge-orchestrator-builder — Devvit vs standalone scraping -->

- **What Devvit CAN do with Reddit data**:
  - `context.reddit` API (`RedditAPIClient`): read public posts/comments in subreddits where the app is installed.
  - Event triggers (e.g. `PostSubmit`) and scheduler jobs for periodic processing.
  - Platform-managed Reddit API — no prefs/apps OAuth script app needed for Devvit apps themselves.
  - Export data externally via HTTP fetch to allow-listed domains in `devvit.json` (max ~10, HTTPS, reviewed on upload).
  - Sources: Reddit Devvit HTTP fetch docs, Devvit PRAW migration guide, Responsible Builder Policy.
  Source: Devvit vs standalone scraping session.

<!-- 2026-06-07 — knowledge-orchestrator-builder — Devvit vs standalone scraping -->

- **What Devvit CANNOT do for reddit-scraper use case**:
  - No standalone local CLI (`uv run reddit-scraper <any_url>` from user's machine).
  - No site-wide scraping — per-installation, subreddit-scoped only (not one VPS bot across all Reddit).
  - No arbitrary HTTP — external fetch restricted to allow-listed domains (cannot wire Devvit creds into `oauth.reddit.com` from user's Python script).
  - No bulk export/scraping — Responsible Builder Policy prohibits unapproved bulk data export, scraping, commercialization.
  - Short-lived handlers only; not persistent scraping loops.
  - Private user data unavailable (saved, upvoted, etc.) — public data only.
  Source: Devvit vs standalone scraping session.

<!-- 2026-06-07 — knowledge-orchestrator-builder — Devvit vs standalone scraping -->

- **Devvit vs OAuth credential split (confirmed)**: User `.env` had Devvit vars only (`devvit_project`, `project_id`); `auth.py` requires `REDDIT_CLIENT_ID` / `REDDIT_CLIENT_SECRET`. These credential types are disjoint — Devvit project setup does not grant Data API OAuth access for the local CLI. Source: Devvit vs standalone scraping session.

<!-- 2026-06-07 — knowledge-orchestrator-builder — Devvit vs standalone scraping -->

- **Practical forks refined (Devvit session)** — see also `rules.md` decision matrix:
  1. Paste any URL → JSON locally: OAuth Data API (if credentials exist) OR browser automation — NOT Devvit.
  2. Mod bot in one subreddit: Devvit appropriate.
  3. Export to own backend: Devvit + allow-listed webhook + app review (policy caveats).
  4. Replace blocked prefs/apps OAuth for personal scripts: Devvit does NOT solve.
  Source: Devvit vs standalone scraping session.

<!-- 2026-07-11 — knowledge-orchestrator-builder -->

- **Workspace default path selected**: Due to Reddit's strict enforcement of the Responsible Builder Policy blocking new OAuth credentials, the browser path (Workflow A) utilizing a DevTools extraction script (`scripts/extract_reddit_thread.js`) and `reddit-scraper-import-json` was designated as the default workspace workflow. Standalone CLI/OAuth pathways remain in-code but are inactive for day-to-day work here. Source: Session decision on default workspace path.
