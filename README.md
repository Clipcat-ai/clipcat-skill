# Clipcat Skill for OpenClaw

Clipcat is a TikTok AI video creation skill for OpenClaw. It helps your OpenClaw agent complete viral discovery, video analysis, viral replication, product video generation, and TikTok video download in one workflow.

For the latest install guide and examples, see: [https://clipcat.ai/tiktok/openclaw](https://clipcat.ai/tiktok/openclaw)

## Core Capabilities

- **Video Analysis**: Extract scripts, scenes, hooks, and music from TikTok or Douyin videos
- **Viral Replication**: Recreate proven viral structures with your own product assets
- **Product to Video**: Turn product images into UGC-style TikTok videos
- **Video Download**: Download TikTok or Douyin videos through the Clipcat API

## Installation

### 1. Install the Skill

Copy the commands below and send them to your OpenClaw for automatic installation.

```bash
# Create skill directory
mkdir -p ~/.openclaw/skills/clipcat-ai

# Download skill file
curl -sL https://static.clipcat.ai/public/skills/SKILL.md -o ~/.openclaw/skills/clipcat-ai/SKILL.md
```

### 2. Get Your API Key

Sign up or log in, then generate an API key in your personal center:

[Generate API Key](https://clipcat.ai/workspace?modal=settings&tab=apikeys)

### 3. Configure Your API Key

Replace `your_api_key_here` with your real API key, then send the command below to OpenClaw to finish setup.

```bash
# Set your Clipcat API key
openclaw env set CLIPCAT_API_KEY your_api_key_here
```

## Usage

Once installed, you can ask OpenClaw to:

- "Replicate this TikTok video with my product images"
- "Generate a product video from these images"
- "Analyze this video and extract the script"
- "Download this TikTok video"

## Important Notes

- Video generation tasks are asynchronous and may take several minutes
- OpenClaw will display parameters and wait for your confirmation before submitting tasks
- Do not retry tasks manually; Clipcat already includes retry handling
- Preserve complete TikTok or Douyin URLs, including signed parameters when present

## Supported Models

- `sora2` - 10s, 15s (720p)
- `sora2_pro` - 15s, 25s (720p)
- `sora2_official` - 4s, 8s, 12s (720p)
- `sora2_official_exp` - 4s, 8s, 12s (720p)
- `veo3.1fast` - 8s (720p, 4K)

## Supported Languages

English, Chinese, French, German, Vietnamese, Thai, Japanese, Korean, Indonesian, Filipino

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

Generate a 15-second video in English using sora2_pro model.
```

OpenClaw will display the parameters and wait for confirmation before submitting the task.

### Example 3: Generate Product Video from Scratch

```
Create a 10-second OOTD video featuring a British girl showcasing my product.
Product image: /path/to/dress.jpg
Use sora2 model, 9:16 aspect ratio, English language.
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
- OpenClaw landing page: https://clipcat.ai/tiktok/openclaw
- API Documentation: See SKILL.md for detailed API reference
