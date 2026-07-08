# Clipcat Skill

Clipcat is a TikTok e-commerce AI video creation skill that **any AI agent can integrate** — Claude Code, OpenClaw, Cursor, or your own custom agent. It ships as a single cross-platform `clipcat` CLI plus a `SKILL.md` manifest: the agent calls `clipcat` commands to complete viral video discovery, TikTok Shop market intelligence, video analysis, viral replication, product video generation, AI image generation, and TikTok video download — all in one workflow.

For the latest guide and examples, see: [https://clipcat.ai](https://clipcat.ai)

## How It Works

Clipcat is just a small CLI binary and a `SKILL.md` skill manifest, so any agent that can run shell commands can drive it:

- The agent reads `SKILL.md` to learn the available commands and conventions.
- It runs `clipcat <command>` and parses the JSON output (default).
- Async tasks (video / image generation) submit immediately and are polled across turns.

OpenClaw auto-installs the skill from the manifest; any other agent installs the CLI manually (see below). Either way the runtime is the same `clipcat` binary.

## Core Capabilities

- **TikTok E-commerce Data Intelligence**: Query 6 entity domains — creators, products, shops, videos, lives, and keyword/image search — covering leaderboards, multi-filter discovery, trends, detail, reviews, comments, and cross-entity relationships (the agent picks the exact command via `clipcat <entity> -h`)
- **Video Analysis**: Extract scripts, scenes, hooks, and music from TikTok or Douyin videos
- **Viral Replication**: Recreate proven viral structures with your own product assets (auto-detects TikTok/Douyin links vs direct video URLs)
- **Product to Video**: Turn product images into UGC-style TikTok videos
- **AI Image Generation**: Generate AI images from text prompts using GPT Image 2, with optional reference images (up to 5)
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

- `grok_imagine` - 10s, 15s, 20s, 30s (720p, 9:16 only) — default, longer clips
- `veo3.1fast` - 8s, 16s, 24s (720p, 1080p) — balanced quality and cost
- `sora2_official_exp` - 4s, 8s, 12s (720p, 9:16 or 16:9) — paid only, OpenAI Sora 2 official channel

Run `clipcat models` for the full available-model list with the exact per-combination credit cost and your balance; `clipcat replicate -h` also lists models.

## Supported Languages

English, Chinese, French, German, Malay, Vietnamese, Thai, Japanese, Korean, Indonesian, Filipino

## Usage Examples

### Example 1: Search for Viral TikTok Videos

```
Search for viral TikTok videos about lip gloss in the US market this week.
Show me the top 10 results sorted by likes.
```

Returns a ranked list of relevant viral videos, including core metrics and source links for further analysis.

### Example 2: Replicate a TikTok Video

```
Replicate this TikTok video with my product:
https://www.tiktok.com/@username/video/123456789

Use these product images:
- /path/to/product1.jpg
- /path/to/product2.jpg

Generate a 16-second video in English using veo3.1fast model.
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

## Links

- Homepage: https://clipcat.ai
- OpenClaw one-click install: https://clipcat.ai/tiktok/openclaw
- Command reference: See SKILL.md for the detailed command reference
