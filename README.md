# Clipcat OpenClaw Skill

This skill enables OpenClaw & Claude Code to interact with Clipcat.ai's API for TikTok e-commerce video creation.

## Features

- **Video Replication**: One-click replication from TikTok/Douyin links
- **Product Video Generation**: Create videos from product images
- **Video Analysis**: Extract structured data from videos (script, scenes, music)
- **Video Download**: Download TikTok/Douyin videos

## Setup

1. Get your API key from [Clipcat.ai](https://clipcat.ai/workspace?modal=settings&tab=apikeys)
2. Set the environment variable:
   ```bash
   export CLIPCAT_API_KEY="your_api_key_here"
   ```

## Usage

Once installed, you can ask Claude Code to:

- "Replicate this TikTok video with my product images"
- "Generate a product video from these images"
- "Analyze this video and extract the script"
- "Download this TikTok video"

## Important Notes

- All video generation tasks are asynchronous and may take several minutes
- Claude will display all parameters and wait for your confirmation before submitting tasks
- Do not retry tasks manually - Clipcat has robust internal retry mechanisms
- Video URLs contain signed parameters - preserve the complete URL

## Supported Models

- `sora2` - 10s, 15s (720p)
- `sora2_pro` - 15s, 25s (720p)
- `sora2_official` - 4s, 8s, 12s (720p)
- `sora2_official_exp` - 4s, 8s, 12s (720p)
- `veo3.1fast` - 8s (720p, 4K)

## Supported Languages

English, Chinese, French, German, Vietnamese, Thai, Japanese, Korean, Indonesian, Filipino

## Usage Examples

### Example 1: Replicate a TikTok Video

```
Replicate this TikTok video with my product:
https://www.tiktok.com/@username/video/123456789

Use these product images:
- /path/to/product1.jpg
- /path/to/product2.jpg

Generate a 15-second video in English using sora2_pro model.
```

Claude will display the parameters and wait for confirmation before submitting the task.

### Example 2: Generate Product Video from Scratch

```
Create a 10-second OOTD video featuring a British girl showcasing my product.
Product image: /path/to/dress.jpg
Use sora2 model, 9:16 aspect ratio, English language.
```

### Example 3: Analyze a Video

```
Analyze this video and extract the script, scenes, and music information:
https://www.tiktok.com/@username/video/987654321
```

Returns structured data including scene-by-scene breakdown, visual descriptions, voiceover content, and background music.

### Example 4: Download a TikTok Video

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
- API Documentation: See SKILL.md for detailed API reference
