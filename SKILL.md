---
name: openclaw-support
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

- Search viral TikTok videos by keyword
- Search TikTok Shop products by keyword (market intelligence)
- Get TikTok Shop product details and reviews
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

- `search` — find viral TikTok videos by keyword; supports `--region`, `--sort-by relevance|likes`, `--time-range any|day|week|month|quarter|half_year`, `--require-shop`
- `search_items` — search TikTok Shop products by keyword; returns market insights, competitor shops, and product intelligence; supports `--region`, `--offset`, `--page-token` for pagination
- `product_detail` — get product info by `--input <ID or URL>`; supports `--region`
- `product_comment` — get product reviews by `--input <ID or URL>`; supports `--region`, `--sort-rule`, `--filter-type`, `--filter-value`
- `replicate` — replicate a viral video with your product images (auto-detects URL type); images via `--image` (local) or `--image-url` (URL); local files and URLs can be mixed; supports `--model`, `--duration`, `--size`, `--lang`, `--resolution`, `--character-id`
- `product_video` — generate video from product images only (no reference video); images via `--image` (local) or `--image-url` (URL); local files and URLs can be mixed
- `image` — generate an AI image from a text prompt using **GPT Image 2** model; optionally supply up to 5 reference images via `--image` (local file) or `--image-url` (URL). Use `--aspect-ratio` to pick `1:1` (default) / `16:9` / `9:16`
- `list_images` — list image generation tasks from server; supports `--status` / `--limit` / `--page` filters
- `breakdown` — analyze a video (script, scenes, music); returns cached result immediately if previously analyzed
- `download` — download TikTok/Douyin video (returns signed URL); cached results return immediately
- `user_videos` — get a TikTok user's video list with analytics (plays, likes, shares, comments, e-commerce cart data); `--unique-id` required; pass `--sec-user-id` to skip ID resolution and speed up response; supports `--max-cursor` pagination and `--sort-type 0|1`
- `query_task` — check status of a task by ID and type (`--type replicate | product | breakdown | download | image`). Omit `--task-id` to resume the latest local task.
- `list_tasks` — list recent **video-related** tasks from server (`--type` required: `replicate | product | breakdown | download`). Image tasks use `list_images`.

## replicate: URL type auto-detection

`clipcat replicate` automatically detects the URL type:

- **TikTok/Douyin link** → calls `/replicate_from_social` (costs **1 extra credit** for download)
- **Direct video URL** → calls `/replicate`

Always inform the user about the extra credit before running with a social URL.

## Async task rules

`replicate`, `product_video`, `image`, and `breakdown` are async. All four
**submit and return immediately** with a task ID — they never block.

1. Task ID is saved locally to `~/.clipcat/tasks.json` automatically.
2. Poll with `clipcat query_task --task-id <id> --type <type> --poll <seconds>`
   (e.g. `--poll 600` = wait up to 10 min). Omit `--task-id` to resume the latest.
3. If poll times out, re-run `query_task` later.
4. Use `clipcat list_tasks --type <replicate|product|breakdown|download>` to see tasks of a given type from the server.

## query_task: auto-resume

`clipcat query_task` with no flags automatically reads the latest task from `~/.clipcat/tasks.json` and resumes it. No need to remember task IDs.

## Available models

| Model ID         | Duration     | Resolution |
| ---------------- | ------------ | ---------- |
| `sora2`          | 10s, 15s     | 720p       |
| `sora2_official` | 4s, 8s, 12s  | 720p       |
| `veo3.1fast`     | 8s, 16s, 24s | 720p, 4K   |

Always check `clipcat replicate -h` for the current model list.

## Supported languages (`--lang`)

`en` `zh` `fr` `de` `ms` `vi` `th` `ja` `ko` `id` `fil` `es`

## Good agent behavior

- Run `clipcat -h` first if unsure which command to use.
- Show parameters to user and get confirmation before running paid commands.
- Keep record of task IDs; use `query_task` to resume if poll times out.
- Preserve signed video URLs intact — they contain `X-Amz-*` params that break if truncated.
- Agents should prefer the default JSON output.
