---
name: openclaw-support
description: Clipcat - TikTok e-commerce video creation skill. Video search, social media insights, video analysis, viral replication, product-to-video generation, and video download via Clipcat.ai API.
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

# Clipcat - TikTok E-commerce Video Creation

Create TikTok e-commerce videos using AI. Video search, social media insights, analyze viral videos, replicate successful patterns, and generate product videos.

Get your API key at: https://clipcat.ai/workspace?modal=settings&tab=apikeys

## API Overview

| API Endpoint             | Description                                                  | Task Type |
| ------------------------ | ------------------------------------------------------------ | --------- |
| `/search`                | Search for viral TikTok videos by keyword                    | Sync      |
| `/replicate_from_social` | One-click replication from TikTok/Douyin links (Recommended) | Async     |
| `/replicate`             | Replicate from direct video URL                              | Async     |
| `/product_video`         | Generate video from product images                           | Async     |
| `/breakdown`             | Analyze video content (script, scenes, music, etc.)          | Async     |
| `/task`                  | Query task status (unified endpoint)                         | Sync      |
| `/download`              | Download TikTok/Douyin videos                                | Sync      |

**Async task flow:** Submit task -> Get `taskId` -> Poll `/task` endpoint until completion

---

## Search Viral Videos

Search for viral TikTok videos by keyword. Returns video metadata including engagement metrics, author info, and direct video links.

**Search videos:**

```bash
curl "https://clipcat.ai/api/openclaw/search?query=fashion&limit=10&sort_by=likes&time_range=week&region_code=US" \
  -H "Authorization: Bearer $CLIPCAT_API_KEY"
```

**Parameters:**

- `query` (required): Search keyword
- `limit` (optional): Number of results, default `10`, must be an integer from `1` to `20`
- `sort_by` (optional): Sort method
  - `relevance` - By relevance (default)
  - `likes` - By most likes
  - Any other value returns an error
- `time_range` (optional): Time range filter
  - `any` - No limit (default)
  - `day` - Last 24 hours
  - `week` - Last 7 days
  - `month` - Last 30 days
  - `quarter` - Last 90 days
  - `half_year` - Last 180 days
  - Any other value returns an error
- `region_code` (optional): Region code, default `US`
  - Common regions: `US`, `GB`, `FR`, `DE`, `JP`, `KR`, `TH`, `VN`, `ID`, `PH`

**Response:**

```json
{
  "code": 0,
  "message": "ok",
  "data": {
    "status": "success",
    "analyzed_count": 1,
    "videos": [
      {
        "video_id": "7123456789",
        "video_url": "https://v16m.tiktokcdn-us.com/...",
        "tiktok_url": "https://tiktok.com/@fashionuser123/video/7123456789",
        "create_time": "2024-03-15T00:00:00.000Z",
        "duration_seconds": 15,
        "content": {
          "desc": "Check out this amazing product! #fashion #trending",
          "tags": ["fashion", "trending", "ootd"],
          "bgm_type": "original_sound"
        },
        "performance": {
          "play_count": 500000,
          "like_count": 25000,
          "share_count": 3500,
          "collect_count": 8000
        },
        "agent_metrics": {
          "like_rate": 0.05,
          "viral_multiplier": 3.33,
          "is_paid_traffic": false
        },
        "author": {
          "nickname": "FashionInfluencer",
          "uid": "7123000000000000000",
          "follower_count": 150000
        },
        "ecommerce_anchor": {
          "has_anchor": false,
          "product_id": null,
          "product_title": null,
          "category": null,
          "price_usd": null
        }
      }
    ]
  }
}
```

- When quota is exceeded, the API returns an error

---

## Available Models

| Model ID             | Duration Options | Resolution |
| -------------------- | ---------------- | ---------- |
| `sora2`              | 10s, 15s         | 720p       |
| `sora2_pro`          | 15s, 25s         | 720p       |
| `sora2_official_exp` | 4s, 8s, 12s      | 720p       |
| `sora2_official`     | 4s, 8s, 12s      | 720p       |
| `veo3.1fast`         | 8s               | 720p, 4K   |

**Language options:** `zh`, `en`, `fr`, `de`, `vi`, `th`, `ja`, `ko`, `id`, `fil`

---

## Replicate from Social Media (Recommended)

One-click replication from TikTok/Douyin links. Automatically downloads the video and replaces it with your product images.

**⚠️ IMPORTANT NOTES:**

- **Display all parameters to user and wait for confirmation before calling API**
- **DO NOT retry automatically before the previous task returns a result (success or failure)**
  - Video tasks are time-consuming (typically several minutes)
  - Clipcat has robust internal error handling and retry mechanisms
  - Duplicate submissions waste resources and congest the task queue

**Submit task:**

```bash
curl -X POST "https://clipcat.ai/api/openclaw/replicate_from_social" \
  -H "Authorization: Bearer $CLIPCAT_API_KEY" \
  -F "item_image=@/path/to/product1.jpg" \
  -F "item_image=@/path/to/product2.jpg" \
  -F "tiktok_douyin_url=https://www.tiktok.com/@username/video/123" \
  -F "prompt=Fully replicate the original video script and visuals, only replace the product in the original video with my product. Adjust any voice-over content to match my product (omit voice-over if there was none). The duration of the replicated new script must be controlled within the specified time range." \
  -F "model=sora2" \
  -F "duration=10" \
  -F "size=9:16" \
  -F "lang=en" \
  -F "resolution=720p"
```

**Parameters:**

- `item_image` (required): Product image files (can upload multiple files)
- `tiktok_douyin_url` (required): TikTok/Douyin video URL
- `prompt` (required): Generation instructions
- `model` (optional): Model ID, default `sora2`
- `duration` (optional): Video duration in seconds, must match model's supported durations
- `size` (optional): Aspect ratio, default `9:16`
- `lang` (optional): Video language, default `en`
- `resolution` (optional): `720p` or `4K`, default `720p`
- `character_id` (optional): Sora2 character ID (e.g., `ygns74cxe.clararue90`) or image URL (for non-Sora models)

**Response:**

```json
{
  "code": 0,
  "message": "ok",
  "data": {
    "taskId": 456,
    "status": "pending"
  }
}
```

**Poll status:** Poll every 20 seconds using `/task` endpoint (see below)

---

## Replicate from Direct URL

Replicate from a direct video URL (suitable for already downloaded videos).

**⚠️ IMPORTANT NOTES:**

- **Display all parameters to user and wait for confirmation before calling API**
- **DO NOT retry automatically before the previous task returns a result (success or failure)**
  - Video tasks are time-consuming (typically several minutes)
  - Clipcat has robust internal error handling and retry mechanisms
  - Duplicate submissions waste resources and congest the task queue

**Submit task:**

```bash
curl -X POST "https://clipcat.ai/api/openclaw/replicate" \
  -H "Authorization: Bearer $CLIPCAT_API_KEY" \
  -F "item_image=@/path/to/product.jpg" \
  -F "video_url=https://clipcat-private.example.com/video.mp4" \
  -F "prompt=Fully replicate the original video script and visuals, only replace the product in the original video with my product." \
  -F "model=sora2" \
  -F "duration=10" \
  -F "size=9:16" \
  -F "lang=en" \
  -F "resolution=720p"
```

**Parameters:** Same as `/replicate_from_social`, but use `video_url` instead of `tiktok_douyin_url`

**Response:** Same as `/replicate_from_social`

**Poll status:** Poll every 20 seconds using `/task` endpoint (see below)

---

## Product to Video

Generate video directly from product images and prompt.

**⚠️ IMPORTANT NOTES:**

- **Display all parameters to user and wait for confirmation before calling API**
- **DO NOT retry automatically before the previous task returns a result (success or failure)**
  - Video tasks are time-consuming (typically several minutes)
  - Clipcat has robust internal error handling and retry mechanisms
  - Duplicate submissions waste resources and congest the task queue

**Submit task:**

```bash
curl -X POST "https://clipcat.ai/api/openclaw/product_video" \
  -H "Authorization: Bearer $CLIPCAT_API_KEY" \
  -F "item_image=@/path/to/product.jpg" \
  -F "prompt=Use this product to create a 15-second OOTD outfit video featuring a British girl." \
  -F "model=sora2" \
  -F "duration=15" \
  -F "size=9:16" \
  -F "lang=en" \
  -F "resolution=720p"
```

**Parameters:** Same as `/replicate`, but use `item_image` and `prompt`, no `video_url` needed

**Response:** Same as `/replicate_from_social`

**Poll status:** Poll every 20 seconds using `/task` endpoint (see below)

---

## Video Analysis

Analyze video content to extract structured data including script, scene descriptions, music information, and more.

**Submit task:**

```bash
curl -X POST "https://clipcat.ai/api/openclaw/breakdown" \
  -H "Authorization: Bearer $CLIPCAT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "video_url": "https://www.tiktok.com/@username/video/123"
  }'
```

**Parameters:**

- `video_url` (required): Video URL (supports TikTok/Douyin links or direct video URLs)

**Response:**

```json
{
  "code": 0,
  "message": "ok",
  "data": {
    "taskId": 789,
    "status": "pending"
  }
}
```

If the same video was already analyzed before, the API may return the existing `taskId`, current `status`, and cached `result` directly instead of creating a new pending task.

**Poll status:** Poll every 20 seconds using `/task` endpoint (see below), use `breakdown` for the `type` parameter

**Completed data structure:** Contains analysis results including video script, scene descriptions, music information, visual elements, etc.

---

## Task Status (Unified Endpoint)

Query the status of all async tasks.

**Query status:**

```bash
curl "https://clipcat.ai/api/openclaw/task?taskId=<taskId>&type=<type>" \
  -H "Authorization: Bearer $CLIPCAT_API_KEY"
```

**Parameters:**

- `taskId` (required): Task ID
- `type` (required): Task type
  - `project` - Replication/product video tasks
  - `breakdown` - Video analysis tasks

**Response Example 1 (type=project, completed):**

```json
{
  "code": 0,
  "message": "ok",
  "data": {
    "taskId": 1773566718092,
    "type": "project",
    "status": "completed",
    "videoAgentStatus": "completed",
    "createdAt": "2026-03-15T09:25:18.092Z",
    "projectUrl": "https://clipcat.ai/project/1773566718092",
    "videos": [
      {
        "videoUrl": "https://clipcat-dev.abf2960315eb4cc152b0fa9490b38b0b.r2.cloudflarestorage.com/generated/59b10174-624c-48f4-8a70-ee1fc9dc482d/1773566718092/videos-1773566718092000.mp4?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=49ee30e197758013626e6abb645ff45f%2F20260315%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20260315T093755Z&X-Amz-Expires=604800&X-Amz-Signature=5e600b46161ce4d186dbbb62a7540089b9a3cb09a7d16592e5c70ff23e49e482&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
        "completedAt": "2026-03-15T09:37:57.759Z",
        "scriptIndex": 0
      }
    ]
  }
}
```

**⚠️ IMPORTANT:** `videoUrl` is a signed URL containing `X-Amz-*` signature parameters. **You MUST preserve the complete URL with all parameters**, otherwise users cannot access the video.

**Response Example 2 (type=breakdown, completed):**

```json
{
  "code": 0,
  "message": "ok",
  "data": {
    "taskId": 9,
    "type": "breakdown",
    "status": "completed",
    "result": {
      "shots": [
        {
          "timestamp": "0.0sec - 2.0sec",
          "scenetype": "Highlighting pain points",
          "visual": "East Asian female aged 20-25 in trendy outfit, standing by indoor door in minimalist home, checking phone then putting it away, switching to underarm bag, leaning against door, expression changing from neutral to disgusted, with relevant subtitles.",
          "voiceover": ""
        },
        {
          "timestamp": "2.0sec - 5.0sec",
          "scenetype": "Emotional connection",
          "visual": "The woman maintains leaning position, slowly squatting down to floor, looking at phone, showing dejected expression, conveying sense of loss.",
          "voiceover": ""
        },
        {
          "timestamp": "5.0sec - 9.5sec",
          "scenetype": "Emotional connection",
          "visual": "Woman squatting, puts down phone, hand on ground gripping bag handle, touches chin then raises hand to chest, showing disgusted and helpless expression, presenting complaining state.",
          "voiceover": ""
        }
      ],
      "bg_music": "Upbeat mid-to-fast tempo R&B pop background music, full volume with rhythmic feel, catchy melody with emotionally rich vocals, suitable for emotional tension + product conversion short video atmosphere.",
      "videotype": "Interactive drama performance"
    },
    "errorMessage": null,
    "createdAt": "2026-03-14T16:09:49.289Z",
    "completedAt": "2026-03-14T16:13:31.912Z"
  }
}
```

**Status flow:** `pending` → `processing` → `completed` / `failed`

**Polling interval:** Poll every 20 seconds

---

## Video Download

Download TikTok/Douyin videos to get direct URL (synchronous, no polling needed).

**Download video:**

```bash
curl -X POST "https://clipcat.ai/api/openclaw/download" \
  -H "Authorization: Bearer $CLIPCAT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tiktok_douyin_url": "https://www.tiktok.com/@username/video/123"
  }'
```

**Parameters:**

- `tiktok_douyin_url` (required): TikTok/Douyin video URL

**Response:**

```json
{
  "code": 0,
  "message": "ok",
  "data": {
    "download": "https://clipcat-dev.abf2960315eb4cc152b0fa9490b38b0b.r2.cloudflarestorage.com/videos/tiktok/59b10174-624c-48f4-8a70-ee1fc9dc482d/video-7603873743257079070.mp4?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=49ee30e197758013626e6abb645ff45f%2F20260315%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20260315T055427Z&X-Amz-Expires=604800&X-Amz-Signature=6fe18d305bc646ae0ce6818d76080e450b90b57ca665a14e6c03cfc416d98f17&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject",
    "creditId": "8641b5a2-3945-4ae1-800b-9213822d5bb0",
    "cached": false
  }
}
```

**⚠️ IMPORTANT:** `download` is a signed URL containing `X-Amz-*` signature parameters. **You MUST preserve the complete URL with all parameters**, otherwise users cannot download the video.

**Video caching:** Downloading the same video again returns cached version (`cached: true`)

---

## Error Handling

When API returns `"code": -1`, display the error message. Common errors:

- `Unauthorized: Invalid API Key`: API key is invalid
- `limit must be an integer between 1 and 20`: Invalid `limit`
- `sort_by is invalid. Allowed values: relevance, likes`: Invalid `sort_by`
- `time_range is invalid. Allowed values: any, day, week, month, quarter, half_year`: Invalid `time_range`
- `video_url is required`: Missing required parameter
- `tiktok_douyin_url is required`: Missing required parameter
- `URL must be a TikTok or Douyin link`: Invalid URL format
- `Failed to download TikTok/Douyin video`: Video download failed
- `Task not found`: Task does not exist or unauthorized access
