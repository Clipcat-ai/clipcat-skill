# Clipcat OpenClaw Skill

Clipcat 是一个面向 OpenClaw 的 TikTok AI 视频创作 Skill，帮助你的 OpenClaw Agent 一站式完成爆款发现、视频分析、爆款复刻、商品生视频和 TikTok 视频下载。

最新安装说明与示例可查看：
[https://clipcat.ai/tiktok/openclaw](https://clipcat.ai/tiktok/openclaw)

## 核心能力

- **视频分析**：提取 TikTok 或抖音视频中的脚本、分镜、钩子和音乐信息
- **爆款复刻**：基于已验证的爆款结构，结合你的商品素材生成新视频
- **商品生视频**：将商品图转成适用于 TikTok 的 UGC 风格短视频
- **视频下载**：通过 Clipcat API 下载 TikTok 或抖音视频

## 安装方法

### 1. 安装 Skill

复制以下命令发送给你的 OpenClaw，即可自动安装。

```bash
# Create skill directory
mkdir -p ~/.openclaw/skills/clipcat-ai

# Download skill file
curl -sL https://static.clipcat.ai/public/skills/SKILL.md -o ~/.openclaw/skills/clipcat-ai/SKILL.md
```

### 2. 获取 API Key

注册或登录后，在个人中心生成 API Key：

[生成 API Key](https://clipcat.ai/workspace?modal=settings&tab=apikeys)

### 3. 配置 API Key

复制以下内容，将 `your_api_key_here` 替换为你的 API Key，然后发送给 OpenClaw，即可完成最后一步配置并开始使用。

```bash
# Set your Clipcat API key
openclaw env set CLIPCAT_API_KEY your_api_key_here
```

## 使用方式

安装完成后，你可以直接让 OpenClaw 帮你：

- “复刻这个 TikTok 视频，并替换成我的商品图”
- “用这些商品图生成一条短视频”
- “分析这个视频并提取脚本”
- “下载这个 TikTok 视频”

## 重要说明

- 视频生成任务为异步执行，通常需要几分钟
- OpenClaw 会先展示参数，并在你确认后再提交任务
- 不要手动重复提交任务，Clipcat 内部已包含重试机制
- 请保留完整的 TikTok 或抖音链接，尤其是带签名参数的 URL

## 支持模型

- `sora2` - 10s, 15s (720p)
- `sora2_pro` - 15s, 25s (720p)
- `sora2_official` - 4s, 8s, 12s (720p)
- `sora2_official_exp` - 4s, 8s, 12s (720p)
- `veo3.1fast` - 8s (720p, 4K)

## 支持语言

英文、中文、法语、德语、越南语、泰语、日语、韩语、印尼语、菲律宾语

## 使用示例

### 示例 1：搜索爆款 TikTok 视频

```text
Search for viral TikTok videos about lip gloss in the US market this week.
Show me the top 10 results sorted by likes.
```

会返回相关爆款视频列表，包含核心指标和来源链接，便于进一步分析与复刻。

### 示例 2：复刻 TikTok 视频

```text
Replicate this TikTok video with my product:
https://www.tiktok.com/@username/video/123456789

Use these product images:
- /path/to/product1.jpg
- /path/to/product2.jpg

Generate a 15-second video in English using sora2_pro model.
```

OpenClaw 会先展示参数，并等待你确认后再提交任务。

### 示例 3：从商品图直接生成视频

```text
Create a 10-second OOTD video featuring a British girl showcasing my product.
Product image: /path/to/dress.jpg
Use sora2 model, 9:16 aspect ratio, English language.
```

### 示例 4：分析视频

```text
Analyze this video and extract the script, scenes, and music information:
https://www.tiktok.com/@username/video/987654321
```

返回结构化分析结果，包括逐镜头拆解、画面描述、口播内容和背景音乐。

### 示例 5：下载 TikTok 视频

```text
Download this TikTok video:
https://www.tiktok.com/@username/video/111222333
```

该操作为同步返回，可直接拿到视频 URL。

## 使用建议

- 始终提供完整的 TikTok/抖音链接
- Prompt 越具体，生成结果通常越稳定
- 视频生成需要时间，请等待任务完成
- 带签名参数的视频链接请完整保留
- 根据时长和质量要求选择合适模型

## 相关链接

- 官网：https://clipcat.ai
- OpenClaw 页面：https://clipcat.ai/tiktok/openclaw
- API 文档：详细接口说明见 `SKILL.md`
