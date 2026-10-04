# reddit-scraper — Rules

Confirmed patterns agents should follow when working on this project or Reddit API integration.

### Unauthenticated JSON endpoints are blocked

- Do not fetch `*.reddit.com/.../comments/....json` without OAuth; unauthenticated access returns 403 on datacenter/residential IPs.
- Session cookies and browser User-Agent headers alone are insufficient (Reddit uses IP blocking, bot challenges, TLS fingerprinting).
- Confirmed by: OAuth migration session (2026-06-07)
- Promoted: 2026-06-07

### OAuth fetch layer is the canonical API access path (Not preferred for this workspace)

- For users with pre-existing, valid credentials, the codebase supports API-based thread fetching via `oauth.reddit.com` with a bearer token.
- This is NOT the preferred path for day-to-day work in this workspace because Reddit blocks new developer registrations.
- Token endpoint: `https://www.reddit.com/api/v1/access_token`
- API base: `https://oauth.reddit.com`
- Implementation: `reddit_scraper/auth.py` (`get_access_token`, `fetch_oauth_json`); path builder in `reddit_scraper/urls.py` (`post_url_to_oauth_path`).
- Post path pattern: `/r/{subreddit}/comments/{post_id}.json?limit=500`
- Confirmed by: OAuth migration session (2026-06-07), Default workspace path session (2026-07-11)
- Promoted: 2026-06-07

### Credential type: script app OAuth — not Devvit

- **Devvit credentials** (`devvit_project`, `project_id` from `npm create devvit`) are NOT Reddit Data API / OAuth credentials and will not work for this scraper.
- Devvit credentials are platform-managed; they cannot populate `REDDIT_CLIENT_ID` / `REDDIT_CLIENT_SECRET` in `reddit_scraper/auth.py` or substitute for prefs/apps OAuth script apps.
- Required credential type for `reddit-scraper`: **script** OAuth app (`REDDIT_CLIENT_ID`, `REDDIT_CLIENT_SECRET`) — NOT a web app, NOT Devvit.
- Script app form fields (when creation still works): name = anything; type = **script**; redirect uri = `http://localhost:8080` (required placeholder, unused for `client_credentials`); description optional. Client ID = short string under app name; secret = field labeled "secret".
- Confirmed by: OAuth migration session, Devvit vs standalone scraping session (2026-06-07)
- Promoted: 2026-06-07

### Devvit scope: subreddit-installed apps only — not standalone scrapers

- Devvit is for moderation/engagement apps **installed per subreddit** on Reddit's platform — not general-purpose data harvesting or local CLI tools.
- Cannot run `uv run reddit-scraper <any_url>` from the user's machine; no standalone local CLI equivalent.
- Data access is **per-installation, subreddit-scoped** via `context.reddit` (RedditAPIClient): public posts/comments in subreddits where the app is installed — not site-wide scraping across all of Reddit from one VPS.
- Handlers are short-lived (event triggers, scheduler jobs); not persistent scraping loops.
- Private user data (saved, upvoted, etc.) is unavailable — public data only.
- Official guidance: non-commercial developers should use Devvit for apps **on Reddit**; if the use case is not supported, file a ticket for manual Data API access.
- Confirmed by: Devvit vs standalone scraping session (2026-06-07); Reddit Devvit HTTP fetch docs, PRAW migration guide, Responsible Builder Policy
- Promoted: 2026-06-07

### Devvit external export: allow-listed webhooks only — with policy caveats

- Devvit can export data externally via HTTP `fetch` to **allow-listed domains** declared in `devvit.json` (max ~10, HTTPS, reviewed on app upload).
- Viable fork for subreddit-scoped use cases: Devvit app + event/scheduler + webhook to user's backend — **not** a drop-in for paste-any-URL local JSON scraping.
- Responsible Builder Policy prohibits unapproved bulk data export, scraping, and commercialization; external fetch domains are reviewed.
- Devvit cannot call `oauth.reddit.com` on behalf of the user's Python script; arbitrary outbound HTTP is restricted to the allow-list.
- Confirmed by: Devvit vs standalone scraping session (2026-06-07)
- Promoted: 2026-06-07

### Use-case decision matrix: Devvit vs OAuth CLI vs browser

- **Paste any Reddit URL → JSON locally**: OAuth Data API (if script credentials exist) OR browser automation — **NOT Devvit**.
- **Mod bot processing posts in one installed subreddit**: Devvit is appropriate.
- **Export subreddit events to own backend**: Devvit + allow-listed webhook + app review — possible with policy caveats.
- **Replace blocked prefs/apps OAuth for personal offline scripts**: Devvit does **NOT** solve this.
- Confirmed by: Devvit vs standalone scraping session (2026-06-07)
- Promoted: 2026-06-07

### Responsible Builder Policy blocks new self-service app creation

- As of ~Nov 2025, Reddit's **Responsible Builder Policy** requires explicit approval before API access; self-service script app creation at https://www.reddit.com/prefs/apps is largely closed for new developers.
- Community and session reports: prefs/apps redirects to policy page, CAPTCHA loops, silent form failures after filling all fields, "cannot create more applications", API requests denied or pending for personal scripts/scrapers.
- Official guidance: non-commercial developers **should use Devvit for on-platform apps**; Devvit does not replace standalone OAuth CLI scrapers. If the use case is not supported by Devvit, file a ticket via [Reddit Help Center](https://support.reddithelp.com/hc/en-us/articles/42728983564564-Responsible-Builder-Policy) / [Developer Platform — Accessing Reddit Data](https://support.reddithelp.com/hc/en-us/articles/14945211791892-Developer-Platform-Accessing-Reddit-Data).
- The OAuth fetch layer in this repo **works when credentials exist** (pre-existing or manually approved); new users are typically blocked at **credential acquisition**, not at the fetch implementation.
- Agents must **not** assume prefs/apps self-service still works; verify whether the user has pre-existing OAuth credentials or needs an alternative fetch strategy (see `knowledge.md` practical forks).
- Confirmed by: OAuth migration session, Responsible Builder Policy session (2026-06-07)
- Promoted: 2026-06-07

### Required .env variables

- `REDDIT_CLIENT_ID`, `REDDIT_CLIENT_SECRET`, `REDDIT_USER_AGENT` — loaded via `python-dotenv` from project-root `.env`.
- `REDDIT_USER_AGENT` format (Reddit-required): `platform:app_name:version (by /u/reddit_username)` — e.g. `windows:reddit-scraper:1.0 (by /u/your_reddit_username)`.
- Missing ID/secret raises `RedditAuthError`: `"Missing REDDIT_CLIENT_ID or REDDIT_CLIENT_SECRET in .env"` — usually means `.env` still has Devvit vars or is empty.
- Reference template: `.env.example`
- Confirmed by: OAuth migration session (2026-06-07)
- Promoted: 2026-06-07

### URL normalization before OAuth fetch

- Accept `reddit.com`, `www.reddit.com`, `old.reddit.com` post URLs; normalize to `https://www.reddit.com/r/{sub}/comments/{id}/` via `normalize_post_url()` before building OAuth path.
- Confirmed by: OAuth migration session (2026-06-07)
- Promoted: 2026-06-07

### Package tooling and CLI

- Dependency manager: **uv** (`pyproject.toml`, `uv.lock`); runtime dep includes `python-dotenv>=1.0.0`.
- Setup: `cd reddit-scraper && uv sync && cp .env.example .env` (fill credentials).
- Scrape CLI: `uv run reddit-scraper <reddit_post_url>`
- Server CLI: `uv run reddit-scraper-server` (port 8001)
- Confirmed by: OAuth migration session (2026-06-07)
- Promoted: 2026-06-07

### Browser-based extraction (Workflow A) is the default workspace path

- Due to Reddit blocking new OAuth script app creations (Responsible Builder Policy), browser-based extraction (Workflow A) is the default day-to-day workflow.
- Do not push or expect OAuth app onboarding for new setups in this workspace.
- The step-by-step browser workflow is:
  1. Open the target Reddit post in Chrome.
  2. Expand all comments (optionally utilizing a comment expansion extension).
  3. Paste the contents of `scripts/extract_reddit_thread.js` into the Chrome DevTools Console to extract the thread and copy its JSON representation to the clipboard.
  4. Run `Get-Clipboard -Raw | uv run reddit-scraper-import-json -` in PowerShell.
  5. The thread is processed and output to `reddit_md_files/<subreddit>/<post_slug>.md`.
- Confirmed by: Session decision on default workspace path (2026-07-11)
- Promoted: 2026-07-11
