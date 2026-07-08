---
name: clipcat
description: Clipcat - TikTok e-commerce video creation skill. Video search, product insights, viral replication, product-to-video generation, breakdown analysis, and video download via Clipcat CLI.
user-invocable: true
metadata:
  {
    "openclaw":
      {
        "requires": { "env": ["CLIPCAT_API_KEY"] },
        "primaryEnv": "CLIPCAT_API_KEY",
      },
    "homepage": "https://clipcat.ai",
  }
---

# Clipcat CLI

Use this skill when you need TikTok e-commerce video creation through `clipcat`.

Get API key: https://clipcat.ai/workspace?modal=settings&tab=apikeys

This skill is intentionally short. Detailed flags and supported values belong to the CLI itself — always treat `clipcat -h` and `clipcat <subcommand> -h` as the primary reference.

## Installation

Skip if `clipcat version` already works. If the command is missing, install first:

macOS / Linux / Git Bash:

```bash
curl -fsSL https://clipcat.ai/cli | bash
clipcat config --api-key <your-key> --base-url https://clipcat.ai
```

Windows (PowerShell, no bash):

```powershell
irm https://clipcat.ai/cli.ps1 | iex
clipcat config --api-key <your-key> --base-url https://clipcat.ai
```

Update later with `clipcat update` (re-runs the installer; your saved config is preserved).

## What this CLI is for

`clipcat` is the local entrypoint for all Clipcat AI video generation workflows:

- Query TikTok e-commerce data: creators, products, shops, videos, lives, search
- Replicate viral videos with your product
- Generate product videos from images
- Generate AI images from text prompts using GPT Image 2 (with optional reference images)
- Analyze videos (script, scenes, music)
- Download TikTok/Douyin videos
- Query async task status

## Default agent workflow

1. Start with `clipcat -h` to see all commands.
2. Before using any command, run `clipcat <subcommand> -h` to see flags.
3. Default to JSON output.
4. Before any credit-consuming video command, quote the exact cost with
   `clipcat quote`, confirm it with the user, and submit with
   `--expected-credits` (see "Confirming cost before paid video commands").
5. If any command prints an update notice on stderr (`⬆ clipcat X is
   available … Run: clipcat update`), run `clipcat update` once, then continue.
   It self-skips when already up to date, so it is safe to run.

## Choosing the right command

### TikTok e-commerce data — entity commands

These are noun-verb commands: `clipcat <entity> <verb>`. Run `clipcat <entity> -h`
to list verbs and `clipcat <entity> <verb> -h` for flags.

- `creator <list|rank|detail|trend|videos|lives|products|followers|following|region|milestones>` — TikTok creators/influencers
- `product <list|rank|detail|trend|comments|creators|videos|lives>` — TikTok Shop products
- `seller <list|rank|detail|trend|products|creators|videos|lives>` — TikTok Shop shops
- `video <list|rank|detail|trend|comments|captions|products|hashtag>` — TikTok videos
- `live detail` — live-room detail (only while live)
- `find <creators|products|videos|lives|hashtags|music|photo|all>` — keyword/image search; `find all` is the broad fallback

**Data mode (`--mode`)**: some commands (`creator detail`, `creator videos`,
`product comments`, `seller products`, `video detail`) accept
`--mode offline|realtime`. **Default is already the safe choice — omit it unless
the user needs it.** Use `--mode realtime` for "latest / current / live", `--mode
offline` for "history / trend / cumulative / leaderboard". Never expose the words
offline/realtime to end users; phrase as historical vs. latest data.

**Pagination**: offline list/rank commands take `--page` / `--page-size` (and
`--max-pages` to auto-fetch several pages); realtime lists take `--offset` /
`--cursor` / `--scroll-param` echoed back from a prior page.

**Data-query playbook (dense):**

- **Chain ids, don't guess them.** Discover first (`<entity> list|rank`, `find …`),
  take the id from the result, then call `detail` / `trend` / relationship verbs.
  Detail verbs take **comma-separated batches** (`--user-ids`, `--product-ids`,
  `--video-ids`, ≤10).
- **Seed relationships from commerce-active entities.** Sub-resource verbs
  (`creator products|lives`, `product creators|videos|lives`, `seller lives`,
  `video products`) return `[]` for low-activity ids. Pull seeds from `… rank` or a
  sorted `… list` (top sales/followers), not an arbitrary row, or expect empties.
- **`… rank` needs a *recent* `--date`.** Pass any day in the target period — the backend
  auto-snaps it to the period anchor (week→that week's Monday, month→that month's 1st) and
  back to the latest *complete* period (data is T+1), so a mid-week / mid-month date, or even
  *today*, still resolves. But it must fall within the freshness window keyed to `--rank-type`:
  **day ≤30d, week ≤6mo, month ≤12mo** back from *today*. A too-**old** date (e.g. last year)
  is rejected upstream as `rant_type N only support …` — move it **forward toward today**;
  don't switch rank-type.
- **Category filtering is numeric and split by level.** To scope `rank` / `list` to a
  category, first run `category resolve --keyword <term>` (e.g. `lipstick` / `口红`; CJK
  auto-uses the zh tree). It returns each match's level + ancestor ids `{l1_id, l2_id?,
  l3_id?}` (ids work for any region). Pass the id for the level the target command takes:
  **product/seller** rank/list use **L1→`--category-id`, L2→`--category-l2-id`,
  L3→`--category-l3-id`** (`--category-id` is L1-only — don't put an L2/L3 id there);
  **creator** rank takes any level via `--product-category-id`; **video** rank only
  accepts L1 (`l1_id`). Low-confidence `hint` → run `category tree` (L1+L2 overview), pick
  the branch by meaning, then `category tree --parent <that L2 id>` to drill into its L3
  leaves. For plain keyword *search* (no leaderboard), `find products --keyword` needs no id.
- **`find products` returns product_id only** (it's a search index). For title /
  price / metrics, chain the ids into `product detail`.
- **Empty `[]` / `null` means "none", not an error.** Known thin/quirky:
  `creator region` (unreliable → read `region` from `creator detail` instead),
  `video captions` (many videos have none), `live detail` (only while a room is
  live), `seller products --mode realtime` (empty when no live inventory; the
  offline default already covers it).
- Responses are **server-trimmed to signal** (ids, core metrics, names, key links;
  images already converted to accessible URLs) — no raw-blob handling needed.
- **All monetary values are USD.** Every price / avg-price / GMV field (`min_price`,
  `max_price`, `spu_avg_price`, `*_gmv_*_amt`, …) is a USD-converted number, regardless
  of `--region`; the response carries `"currency": "USD"` to confirm it. Never label
  them with a local symbol like `¥`/`円`. If a report needs the local currency (e.g.
  JPY for a Japan market study), convert from USD using a current FX rate and mark the
  result approximate.

### Video generation & tools

- `quote` — return the exact credit cost of one specific generation (`--model` + `--resolution` + `--duration`, plus `--url`/`--social` for a TikTok/Douyin replicate, plus `--enhance` for super-resolution). The primary way to quote a paid command: the server does all the math and hands back `totalCredits` (already includes the enhance fee) plus `enhanceCredits` / `enhanceBlocked` (see "Confirming cost before paid video commands" and "Super-resolution").
- `models` — browse all available video models with their credit costs (discrete → `prices`, range → `creditsPerSecond`) and your balance. Use it when the user hasn't picked a model yet, or an unavailable one is reported.
- `replicate` — replicate a viral video with your product images (auto-detects URL type); images via `--image` (local) or `--image-url` (URL); local files and URLs can be mixed; supports `--model`, `--duration`, `--size` (only `9:16` or `16:9`), `--lang`, `--resolution`, `--enhance` (super-resolution, see below), `--character-id`, `--expected-credits`
- `product_video` — generate video from product images only (no reference video); images via `--image` (local) or `--image-url` (URL); local files and URLs can be mixed; `--size` only accepts `9:16` or `16:9`; supports `--enhance` (super-resolution, see below), `--expected-credits`
- `image` — generate an AI image from a text prompt using **GPT Image 2** model; optionally supply up to 5 reference images via `--image` (local file) or `--image-url` (URL). Use `--aspect-ratio` to pick `1:1` (default) / `16:9` / `9:16`. **Dimension hints (9:16/16:9/1:1, portrait/landscape/square, 竖版/横版/方图, banner, wallpaper) must appear in BOTH `--prompt` and `--aspect-ratio`** — `--aspect-ratio` sets canvas, the prompt hint anchors framing. Don't invent dimensions the user didn't ask for.
- `list_images` — list image generation tasks from server; supports `--status` / `--limit` / `--page` filters
- `breakdown` — analyze a video (script, scenes, music); returns cached result immediately if previously analyzed
- `download` — download TikTok/Douyin video (returns signed URL); cached results return immediately
- `query_task` — check status of a task by ID and type (`--type replicate | product | breakdown | download | image`). Omit `--task-id` to resume the latest local task. With `--enhance`, each `videos[]` item carries its own `status` / `enhanceStatus` (see "Super-resolution").
- `list_tasks` — list recent **video-related** tasks from server (`--type` required: `replicate | product | breakdown | download`). Image tasks use `list_images`.

## Confirming cost before paid video commands

`replicate` and `product_video` consume credits. Always confirm cost first — and
**never compute the credits yourself**, let `clipcat quote` return them:

1. Run `clipcat quote` with the SAME parameters you'll submit (`--model`,
   `--resolution`, `--duration`; for a TikTok/Douyin replicate also pass the
   `--url`, which auto-adds the download surcharge; for super-resolution also pass
   `--enhance`). It returns `totalCredits` (the server does all the math —
   per-second rates, download surcharge, deferred enhance fee) and your
   `remainingCredits`.
2. Show the user the model, duration, resolution and that `totalCredits`, and get
   explicit approval.
3. Submit with `--expected-credits <totalCredits>`. The server rejects the request
   only if the real cost is **higher** than what you pass, so you can never
   overcharge (a cheaper real cost — cache hit, promo — just goes through). On a
   rejection it returns the current cost — re-confirm that number with the user
   and resubmit with the updated `--expected-credits`.

Example — quote, then submit the confirmed cost (Seedance 2, 720p, 8s, TikTok link):

```bash
clipcat quote --model seedance2 --resolution 720p --duration 8 \
  --url "https://www.tiktok.com/@u/video/123"
# → seedance2 720p 8s → 328 credits  + 10 download → total 338 credits
clipcat replicate --url "https://www.tiktok.com/@u/video/123" \
  --image product.jpg --model seedance2 --duration 8 --resolution 720p \
  --size 9:16 --expected-credits 338
```

When the user hasn't chosen a model yet (or you need the full menu), run `clipcat
models` to list every available model and its cost, then `clipcat quote` the pick.

Premium models (e.g. `seedance2`, `happyhorse10`) require a paid plan; `clipcat
quote` flags them (`premiumBlocked`) and the server rejects them for free users.

## Super-resolution (`--enhance`)

`replicate` and `product_video` accept `--enhance 720p|1080p|2k` to upscale the
finished video. Rules:

- **Tier must be strictly higher than the generated resolution**: 480p → 720p /
  1080p / 2k, 720p → 1080p / 2k, 1080p → 2k, 2k → no option. The CLI only
  enum-checks the value; the server enforces the tier ladder.
- **Paid plans only.** Free users are rejected on submit; `clipcat quote --enhance`
  flags this as `enhanceBlocked: true` (upgrade needed).
- **Cost** = ceil(duration_sec / 10) × tier rate (`720p`=10, `1080p`=20, `2k`=30
  credits per 10s). It is **deferred** — charged only after the base video
  succeeds. `quote` returns it as `enhanceCredits`, already folded into
  `totalCredits`; submit that `totalCredits` via `--expected-credits`.
- **Status semantics** (`query_task`): once the base video is ready it appears in
  `videos[]` with `status: enhancing` and a usable `videoUrl` (the original), but
  the **task reaches its final completed state only after enhance finishes** (a
  standard 1-min video takes ~6-10 min extra). `enhanceStatus: failed` → the task
  still completes and delivers the original video, and the enhance fee is refunded.

```bash
clipcat quote --model seedance2 --resolution 480p --duration 8 --enhance 1080p
# → seedance2 480p 8s → 320 credits  + 20 enhance (1080p) → total 340 credits
clipcat product_video --image product.jpg --model seedance2 --duration 8 \
  --resolution 480p --size 9:16 --enhance 1080p --expected-credits 340
```

## replicate: URL type auto-detection

`clipcat replicate` automatically detects the URL type:

- **TikTok/Douyin link** → calls `/replicate_from_social` (costs **10 extra credits** for download)
- **Direct video URL** → calls `/replicate`

Always inform the user about the extra credits before running with a social URL.

## clipcat:// asset references

`clipcat://...` strings seen in earlier turns are stable asset references. Pass them **verbatim** to any `--image-url` / `--character-id` flag — never prepend `https://` or modify them. See subcommand `-h` for details.

## Async task rules

`replicate`, `product_video`, `image`, and `breakdown` are async. All four
**submit and return immediately** with a task ID — they never block.

Typical durations: `image` ~3 min, `breakdown` a few minutes, `product_video` /
`replicate` 10+ min. **Never try to wait synchronously inside a single tool
call** — every realistic agent harness has a tool-call timeout (commonly 60s)
that will kill the call long before the task is done. Always go submit → return
→ poll across turns.

1. Task ID is saved locally to `~/.clipcat/tasks.json` automatically.
2. Check status with `clipcat query_task --task-id <id> --type <type>`. Each
   call returns immediately with the current status. Omit `--task-id` to resume
   the latest task. Re-invoke the command across turns (suggested cadence:
   ~30s for `image`, ~1-2 min for `breakdown` / `product_video` / `replicate`)
   until `status` is `completed` or `failed`.
3. Use `clipcat list_tasks --type <replicate|product|breakdown|download>` to
   see tasks of a given type from the server.

## query_task: auto-resume

`clipcat query_task` with no flags automatically reads the latest task from `~/.clipcat/tasks.json` and resumes it. No need to remember task IDs.

## Available models

Trial models are available to all users; standard models require a paid plan.

| Model ID             | Duration              | Resolution        | Notes                                                             |
| -------------------- | --------------------- | ----------------- | ----------------------------------------------------------------- |
| `veo3.1fast`         | 8s, 16s, 24s          | 720p, 1080p       | **Trial**. Google Veo 3.1 Fast, balanced quality and cost         |
| `veo3.1pro`          | 8s, 16s, 24s          | 720p, 1080p       | **Trial**. Google Veo 3.1 Pro, high-quality variant               |
| `omini_flash`        | 10s                   | 720p, 1080p       | **Trial**. Gemini Omni Flash, Google's newest model               |
| `grok_imagine`       | 10s, 15s, 20s, 30s    | 720p              | **Trial**, default. 9:16 aspect ratio only, longer clips          |
| `seedance2`          | 4-15s (any integer)   | 480p, 720p, 1080p | Standard (paid). ByteDance Seedance 2, top quality                |
| `seedance2_fast`     | 4-15s (any integer)   | 480p, 720p, 1080p | Standard (paid). ByteDance Seedance 2 Fast, fast variant          |
| `happyhorse10`       | 3-15s (any integer)   | 720p, 1080p       | Standard (paid). Alibaba HappyHorse 1.0                           |
| `sora2_official_exp` | 4s, 8s, 12s           | 720p              | Standard (paid). OpenAI Sora 2 official channel, 9:16 or 16:9     |

Always check `clipcat replicate -h` for the current model list, and `clipcat
models` for the authoritative live per-combination credit costs and your balance.

## Supported languages (`--lang`)

`en` `zh` `fr` `de` `ms` `vi` `th` `ja` `ko` `id` `fil` `es`

## Region (`--region`)

ISO 3166-1 alpha-2, uppercase: `US` `GB` `DE` `ES` `FR` `IT` `JP` `MX` `BR` `ID` `MY` `PH` `SG` `TH` `VN`. Server-enforced; an out-of-range code returns the current allowed list.

## Good agent behavior

- Run `clipcat -h` first if unsure which command to use.
- For paid video commands (`replicate`, `product_video`): quote the exact cost with `clipcat quote` (same params you'll submit), show the user the model / duration / resolution / `totalCredits`, get explicit approval, then submit with `--expected-credits <totalCredits>`. Never compute the credits yourself — let `clipcat quote` return them.
- Keep record of task IDs; re-invoke `query_task` across turns to track long-running tasks.
- Preserve signed video URLs intact — they contain `X-Amz-*` params that break if truncated.
- Agents should prefer the default JSON output.
