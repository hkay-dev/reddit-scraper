# reddit-scraper — Hypotheses

Open questions not yet confirmed enough for `rules.md`.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **client_credentials rate limits**: Default Reddit OAuth rate limits for script apps using client_credentials on thread fetches are undocumented in-repo; may need backoff/retry if users scrape heavily. Source: OAuth migration session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Private/gated subreddits**: client_credentials may not access NSFW or private subs without user-context OAuth; behavior not validated in this project. Source: OAuth migration session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Data API manual approval success rate**: Whether Reddit Help Center / sign-up form approvals are granted for personal offline scrapers (vs. commercial or Devvit-aligned use cases) is unknown in-repo; community reports suggest frequent denial.
  - *Update (2026-07-11)*: Confirmed to be a complete blocker for this workspace. New credential onboarding is unviable, leading to the decision to default to the browser extraction path. Source: Session decision on default workspace path.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Playwright pivot durability**: Browser-based fetch may evade OAuth credential requirements but could face the same bot challenges (CAPTCHA, fingerprinting) observed with cookie + User-Agent attempts; long-term viability unvalidated.
  - *Update (2026-07-11)*: Instead of automated Playwright fetching, the workspace has pivoted to manual browser-based extraction via DevTools console copying. This bypasses automated bot detection by utilizing the user's active Chrome session. Source: Session decision on default workspace path.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **curl_cffi as interim fetch layer**: TLS impersonation might restore unauthenticated or session-based access without OAuth apps; expected to be fragile and policy-sensitive. Source: Responsible Builder Policy session.

<!-- 2026-06-07 — knowledge-orchestrator-builder -->

- **Global unauthenticated JSON cutoff date**: Community cites ~May 30, 2026 for worldwide 403 on `.json` endpoints; not independently verified against Reddit official changelog in this repo. Source: gallery-dl #8559, Responsible Builder Policy session.

<!-- 2026-06-07 — knowledge-orchestrator-builder — Devvit vs standalone scraping -->

- **Devvit webhook export approval for non-moderation use**: Whether Devvit apps that pass upload review can reliably export subreddit event data to user-owned backends via allow-listed domains for analytics/archival (vs. moderation/engagement) is unknown; Responsible Builder Policy may deny bulk-export patterns. Source: Devvit vs standalone scraping session.

<!-- 2026-06-07 — knowledge-orchestrator-builder — Devvit vs standalone scraping -->

- **Devvit scheduler coverage for historical backfill**: Whether Devvit scheduler jobs can backfill historical posts/comments in an installed subreddit (vs. only new events after install) is unvalidated in this repo. Source: Devvit vs standalone scraping session.

<!-- 2026-07-11 — knowledge-orchestrator-builder -->

- **Comment expansion extension durability**: The long-term reliability and styling selector compatibility of browser extensions or custom JS to fully expand nested Reddit comment threads remains unvalidated and subject to change when Reddit alters its desktop layout. Source: Session decision on default workspace path.
