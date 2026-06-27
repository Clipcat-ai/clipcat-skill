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

```bash
curl -fsSL https://clipcat.ai/cli | bash
clipcat config --api-key <your-key>
```

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
4. Warn the user before running commands that consume credits.

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

### Video generation & tools

- `replicate` — replicate a viral video with your product images (auto-detects URL type); images via `--image` (local) or `--image-url` (URL); local files and URLs can be mixed; supports `--model`, `--duration`, `--size` (only `9:16` or `16:9`), `--lang`, `--resolution`, `--character-id`
- `product_video` — generate video from product images only (no reference video); images via `--image` (local) or `--image-url` (URL); local files and URLs can be mixed; `--size` only accepts `9:16` or `16:9`
- `image` — generate an AI image from a text prompt using **GPT Image 2** model; optionally supply up to 5 reference images via `--image` (local file) or `--image-url` (URL). Use `--aspect-ratio` to pick `1:1` (default) / `16:9` / `9:16`. **Dimension hints (9:16/16:9/1:1, portrait/landscape/square, 竖版/横版/方图, banner, wallpaper) must appear in BOTH `--prompt` and `--aspect-ratio`** — `--aspect-ratio` sets canvas, the prompt hint anchors framing. Don't invent dimensions the user didn't ask for.
- `list_images` — list image generation tasks from server; supports `--status` / `--limit` / `--page` filters
- `breakdown` — analyze a video (script, scenes, music); returns cached result immediately if previously analyzed
- `download` — download TikTok/Douyin video (returns signed URL); cached results return immediately
- `query_task` — check status of a task by ID and type (`--type replicate | product | breakdown | download | image`). Omit `--task-id` to resume the latest local task.
- `list_tasks` — list recent **video-related** tasks from server (`--type` required: `replicate | product | breakdown | download`). Image tasks use `list_images`.

## replicate: URL type auto-detection

`clipcat replicate` automatically detects the URL type:

- **TikTok/Douyin link** → calls `/replicate_from_social` (costs **1 extra credit** for download)
- **Direct video URL** → calls `/replicate`

Always inform the user about the extra credit before running with a social URL.

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

Trial channels are per-clip pricing; standard channels are per-second pricing.

| Model ID             | Duration              | Resolution        | Notes                                                             |
| -------------------- | --------------------- | ----------------- | ----------------------------------------------------------------- |
| `sora2_official_exp` | 4s, 8s, 12s           | 720p              | **Trial**, default. OpenAI Sora 2 official channel, 9:16 or 16:9  |
| `veo3.1fast`         | 8s, 16s, 24s          | 720p, 1080p       | **Trial**. Google Veo 3.1 Fast, balanced quality and cost         |
| `veo3.1pro`          | 8s, 16s, 24s          | 720p, 1080p       | **Trial**. Google Veo 3.1 Pro, high-quality variant               |
| `omini_flash`        | 10s                   | 720p, 1080p       | **Trial**. Gemini Omni Flash, Google's newest model               |
| `grok_imagine`       | 10s, 15s, 20s, 30s    | 720p              | **Trial**. 9:16 aspect ratio only, longer clips                   |
| `seedance2`          | 4-15s (any integer)   | 480p, 720p, 1080p | Standard. ByteDance Seedance 2, top quality                       |
| `seedance2_fast`     | 4-15s (any integer)   | 480p, 720p, 1080p | Standard. ByteDance Seedance 2 Fast, fast variant                 |
| `happyhorse10`       | 3-15s (any integer)   | 720p, 1080p       | Standard. Alibaba HappyHorse 1.0                                  |

Always check `clipcat replicate -h` for the current model list.

## Supported languages (`--lang`)

`en` `zh` `fr` `de` `ms` `vi` `th` `ja` `ko` `id` `fil` `es`

## Region (`--region`)

ISO 3166-1 alpha-2, uppercase: `US` `GB` `DE` `ES` `FR` `IT` `JP` `MX` `BR` `ID` `MY` `PH` `SG` `TH` `VN`. Server-enforced; an out-of-range code returns the current allowed list.

## Good agent behavior

- Run `clipcat -h` first if unsure which command to use.
- Show parameters to user and get confirmation before running paid commands.
- Keep record of task IDs; re-invoke `query_task` across turns to track long-running tasks.
- Preserve signed video URLs intact — they contain `X-Amz-*` params that break if truncated.
- Agents should prefer the default JSON output.
