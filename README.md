# Clipcat Skill

Clipcat is a TikTok e-commerce AI video creation skill that **any AI agent can integrate** — Claude Code, OpenClaw, Cursor, or your own custom agent. It ships as a single cross-platform `clipcat` CLI plus a `SKILL.md` manifest: the agent calls `clipcat` commands to complete viral video discovery, TikTok Shop market intelligence, video analysis, viral replication, product video generation, AI image generation, and TikTok video download — all in one workflow.

For the latest guide and examples, see: [https://clipcat.ai](https://clipcat.ai)

中文文档：[README_ZH.md](README_ZH.md)

## How It Works

Clipcat is just a small CLI binary and a `SKILL.md` skill manifest, so any agent that can run shell commands can drive it:

- The agent reads `SKILL.md` to learn the available commands and conventions.
- It runs `clipcat <command>` and parses the JSON output (default).
- Async tasks (video / image generation) submit immediately and are polled across turns.

OpenClaw auto-installs the skill from the manifest; any other agent installs the CLI manually (see below). Either way the runtime is the same `clipcat` binary.

## Core Capabilities

- **TikTok E-commerce Data Intelligence**: Query 6 entity domains — creators, products, shops, videos, lives, and keyword/image search — covering leaderboards, multi-filter discovery, trends, detail, reviews, comments, and cross-entity relationships (the agent picks the exact command via `clipcat <entity> -h`)
- **Video Analysis**: Extract scripts, scenes, hooks, and music from TikTok or Douyin videos
- **Viral Replication**: Recreate proven viral structures with your own product assets — reference a TikTok/Douyin link, a direct video URL, or a local video file (up to 100MB)
- **Product to Video**: Turn product images into UGC-style TikTok videos
- **AI Image Generation**: Generate AI images from text prompts using GPT Image 2, with optional reference images (up to 5)
- **Super-Resolution**: Upscale a generated video to 720p, 1080p or 2K with `--enhance`
- **Reusable Characters**: Keep the same on-screen character across videos via `--character-id`
- **Video Download**: Download TikTok or Douyin videos through the Clipcat API

## Installation

### 1. Install the CLI

**OpenClaw** — auto-installed from the skill manifest; no manual step needed.

**Any other agent / manual** — install the CLI binary:

```bash
# Install the clipcat CLI
curl -fsSL https://clipcat.ai/cli | bash
```

### 2. Get Your API Key

Sign up or log in, then generate an API key in your personal center:

[Generate API Key](https://clipcat.ai/workspace?modal=settings&tab=apikeys)

### 3. Configure Your API Key

Configure the key once on the machine the agent runs on:

```bash
clipcat config --api-key your_api_key_here --base-url https://clipcat.ai
```

Agents that manage secrets via environment variables can instead set `CLIPCAT_API_KEY` (e.g. OpenClaw: `openclaw env set CLIPCAT_API_KEY your_api_key_here`).

## Usage

Once installed, you can ask your agent to:

- "Search viral TikTok videos about lip gloss this week"
- "Search TikTok Shop for trending pet products and show me competitor shops"
- "Replicate this TikTok video with my product images"
- "Generate a product video from these images"
- "Generate an AI image of a model holding my product"
- "Analyze this video and extract the script"
- "Show me this TikTok user's recent videos with engagement stats"
- "Download this TikTok video"
- "Fetch TikTok Shop product detail and review highlights for this product URL"

## Important Notes

- Video generation tasks are asynchronous and may take several minutes
- Before submitting a task that consumes credits, the agent quotes the exact cost with `clipcat quote`, shows you the model / duration / resolution / credits, and waits for your confirmation
- Do not retry tasks manually; Clipcat already includes retry handling
- Preserve complete TikTok or Douyin URLs, including signed parameters when present

## Supported Models

Trial models are open to all users; standard models require a paid plan.

| Model ID             | Duration            | Resolution        | Notes                                                     |
| -------------------- | ------------------- | ----------------- | --------------------------------------------------------- |
| `grok_imagine`       | 10s, 15s            | 480p, 720p        | **Trial**, default. xAI Grok Imagine 1.5, 9:16 only       |
| `veo3.1fast`         | 8s, 16s, 24s        | 720p              | **Trial**. Google Veo 3.1 Fast, balanced quality and cost |
| `omini_flash`        | 10s, 20s            | 720p, 1080p       | **Trial**. Gemini Omni Flash, Google's newest model       |
| `seedance2_mini`     | 4-15s (any integer) | 480p, 720p        | **Trial**. Seedance 2 Mini, value tier (free plans: 480p) |
| `mmh3_promo`         | 10s, 15s            | 480p, 720p, 2K    | **Trial**. Subsidized MiniMax H3, open to free plans      |
| `seedance2`          | 4-15s (any integer) | 480p, 720p, 1080p | Paid. ByteDance Seedance 2, top quality                   |
| `seedance2_5`        | 4-30s (any integer) | 480p, 720p        | Paid. ByteDance Seedance 2.5, clips up to 30s             |
| `seedance2_fast`     | 4-15s (any integer) | 480p, 720p        | Paid. ByteDance Seedance 2 Fast, fast variant             |
| `wan30`              | 5-30s (any integer) | 480p, 720p, 1080p | Paid. Alibaba Wan 3.0, clips up to 30s                    |
| `minimax_h3`         | 10s, 15s            | 768p, 2K          | Paid. MiniMax H3                                          |
| `happyhorse10`       | 3-15s (any integer) | 720p, 1080p       | Paid. Alibaba HappyHorse 1.1                              |

The table is what each model *offers*; run `clipcat models` for the live listing with the exact per-combination credit cost and your balance — it is the authority on which models still exist, and `-h` lags behind it between releases. Prefer `mmh3_promo` over `minimax_h3` whenever it is listed: same model, subsidized channel, a fraction of the credits.

## Supported Languages

English, Chinese, French, German, Malay, Vietnamese, Thai, Japanese, Korean, Indonesian, Filipino, Spanish

## Supported Regions

`US` `GB` `DE` `ES` `FR` `IT` `JP` `MX` `BR` `ID` `MY` `PH` `SG` `TH` `VN`

All monetary values in e-commerce data responses are USD-converted regardless of region.

## Usage Examples

### Example 1: Search for Viral TikTok Videos

```
Search for viral TikTok videos about lip gloss in the US market this week.
Show me the top 10 results sorted by likes.
```

Returns a ranked list of relevant viral videos, including core metrics and source links for further analysis.

### Example 2: Replicate a TikTok Video

```
Please use Clipcat’s replication feature to fully replicate the original video’s script and visuals, replacing only the original product with my product. If there is voiceover in the original video, please adapt it to match my product; if there is no voiceover, then no voiceover is needed. The replicated video should not include subtitles. The duration of the new script must be controlled within 15 seconds. Please use the model seedance2.0, and the voiceover language should be English.
This is the TikTok link to replicate:https://www.tiktok.com/@username/video/123456789
```

The agent will display the parameters and wait for confirmation before submitting the task.

### Example 3: Generate Product Video from Scratch

```
Create a 10-second OOTD video featuring a British girl showcasing my product.
Product image: /path/to/dress.jpg
Use veo3.1fast model, 9:16 aspect ratio, English language.
```

### Example 4: Analyze a Video

```
Analyze this video and extract the script, scenes, and music information:
https://www.tiktok.com/@username/video/987654321
```

Returns structured data including scene-by-scene breakdown, visual descriptions, voiceover content, and background music.

### Example 5: Download a TikTok Video

```
Download this TikTok video:
https://www.tiktok.com/@username/video/111222333
```

Synchronous operation, returns direct video URL immediately.

## Tips

- Always provide complete TikTok/Douyin URLs
- Be specific with prompts for better results
- Wait for task completion - video generation takes time
- Preserve complete video URLs with all signed parameters
- Choose appropriate models based on duration and quality needs

## Repository Structure

This repo follows the Agent Skills convention — one skill per directory under `skills/`, each containing a `SKILL.md` manifest:

```
clipcat-skill/
├── README.md                  # this file
├── README_ZH.md               # Chinese version
├── docs/
│   └── SKILL_ZH.md            # Chinese translation of the skill manifest
└── skills/
    └── clipcat/
        └── SKILL.md           # the skill manifest agents load
```

## Links

- Homepage: https://clipcat.ai
- OpenClaw one-click install: https://clipcat.ai/tiktok/openclaw
- Command reference: [`skills/clipcat/SKILL.md`](skills/clipcat/SKILL.md)
