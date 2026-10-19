# Daily Recall

A responsive quiz website with a Vinext Worker backend, account-linked D1 storage, and private cross-device sync. UI and image assets live in `public/quiz/`; root redirects there on the same origin.

## Five daily categories

- Software engineering: 72 authored questions, 24 concepts, 8 topics, 36 Java debugging questions. Every eight-question session includes at least three Java questions, balances topics, and favors alternate variants.
- SNL: 16 cast-member portraits with four name choices.
- LOST: 16 actor portraits asking which LOST character the actor portrays; choices are character names.
- U.S. capitals: all 50 states, with forward/reverse variants (100 questions).
- Original Pokémon: all 151 species with local artwork and four name choices.

Portraits and Pokémon artwork are local. Credits are visible beside images, with detailed credits after submission and source records in the image folders. Portrait attribution files identify creator, license, and source. Pokémon artwork remains copyrighted by its respective owners; see `public/quiz/pokemon-images/ATTRIBUTION.txt`.

## Learning and quiz rules

The main calendar uses the browser's local calendar date. Each category/date has an independent deterministic daily session, persisted after creation. Future days are unavailable. Past days are available for catch-up. Unlimited extra sessions are independent of daily sessions.

Answers remain editable drafts until the full quiz is submitted. Draft selection never grades or updates learning statistics. Submission is idempotent, produces explanations for every question, and records quiz grade, category accuracy, coverage, mastery, and progress change. Reloading resumes saved drafts.

A concept becomes mastered after correct submitted answers on three distinct **actual answer dates**, regardless of the calendar date selected for catch-up. A wrong answer resets that concept's correct-day sequence; correcting it again on the same day does not rebuild that day. Same-day extra sessions cannot accelerate mastery. A category is “tested out” only when all concepts in its current bank are mastered; optional maintenance remains available. This is an application learning milestone, not professional certification.

Due and recently missed concepts are prioritized, unseen concepts come before other reviewed concepts, and variants avoid the most recent question where possible. Review intervals are 1, 2, 4, 7, 14, and 30 days. A repeated same-day correct answer does not advance the interval level.

Engineering quizzes require acknowledging a three-paragraph pre-quiz lesson. Lessons prioritize engineering weaknesses only. The learning library contains official documentation links, topic accuracy, and seven YouTube recommendations with runtimes verified as less than one hour. Other categories do not impose a lesson. Video provenance is recorded in `videos-research.json`.

## Account data and offline behavior

The Sites authenticated user header is the only account authority. `/api/progress` requires authentication for reads and writes; writes also validate Origin, JSON content type, and an expected-account guard to prevent queued data from crossing an account switch. No email is stored.

D1 `user_progress` stores a validated account document with revision-based compare-and-swap. Concurrent writes merge sessions instead of replacing the account. Daily completed submissions are first-accepted and immutable. Extra sessions use UUIDs. Drafts merge by per-question edit time; already accepted answers are immutable. Grades and mastery are recomputed from known question banks and actual answer dates. Imported history can include claimed historical dates; this is a learning tool, not anti-cheat certification.

The browser opens an account cache only after authenticated GET succeeds. Changes are queued and retry after network restoration, focus, visibility, and background retry while the page remains open. First-load offline waits for verified identity. If device storage fails, online cloud saves still work; if both fail, the UI warns to keep the tab open and export a backup. The server account guard rejects stale queued writes after account switches.

Existing `daily-practice-v2` history migrates after authentication, with deterministic legacy extra IDs; the original key remains unchanged as a backup. Migration ownership is marked only after cloud acknowledgement, with a prior account reservation for safe retries. Imports merge rather than replace saved results. Export remains available.

## Validation and deployment

`npm test` runs engine/render tests and independent multi-device, idempotency, migration, offline, and account-isolation checks. `node tests/http-smoke.mjs` exercises a running production Worker on port 4173 with actual local D1. Browser screenshots were not taken in this update.

Use the Sites build helper. `.openai/hosting.json` retains the Site identity and enables `DB`. Drizzle migration `drizzle/0000_true_weapon_omega.sql` creates the table; Sites applies hosted migrations during deployment. Never replay an applied migration.

## Reading list and book settings

The Reading list tab includes a shared, researched catalog (initially 36 books: 20 core and 16 recent), source/edition notes, rolling two-year and new-addition filters, search and suggested reading priorities. Reading status is stored per authenticated account in a separate D1 table. Settings → Book settings saves enabled named search templates per account; only validated Open Library, WorldCat and Barnes & Noble canonical searches are supported, and book tokens are URL-encoded. Quiz banks, quiz history, account sync and scoring are unchanged.

`GET /api/reading-list` and `POST /api/reading-list/update` are shared catalog operations. The writer relies on the current owner-private Sites dispatch boundary. Do not make this Site public without adding independent writer authorization. `/api/reading-list/status` and `/api/reading-list/settings` additionally require trusted visitor identity; mutations enforce same-origin JSON and the expected-account header. No shared updater reads or mutates these personal tables.

The new schema-only migrations are 0001–0003; the applied original 0000 is unchanged. Initial data is seeded outside migrations via `research/reading-catalog-seed.json`, generated by `node scripts/build-reading-catalog.mjs`. A bundled snapshot supplies reading content if the live catalog is unavailable; personal status editing requires a live authenticated catalog.

Recurring maintenance is documented in `READING-MAINTENANCE.md`, also hosted at `/quiz/reading-maintenance.md`. Data-only updates require no redeployment. No schedule is claimed or created by the code itself. A routine review preserves previous records, original added dates, personal settings and reading status, and may add zero books when no candidate meets the bar.
