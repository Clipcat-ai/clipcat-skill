# OpenClaw 的 Clipcat Skill

Clipcat 是一个面向 OpenClaw 的 TikTok 电商 AI 视频创作 Skill，帮助你的 OpenClaw Agent 在同一套工作流中完成爆款视频发现、TikTok Shop 市场情报分析、视频分析、爆款复刻、商品视频生成、AI 图片生成以及 TikTok 视频下载。

最新安装指南和示例请查看：[https://clipcat.ai/tiktok/openclaw](https://clipcat.ai/tiktok/openclaw)

## 核心能力

- **爆款视频搜索**：按关键词搜索 TikTok 爆款视频，用于内容灵感获取和趋势研究
- **TikTok Shop 情报分析**：按关键词搜索 TikTok Shop 商品，挖掘市场洞察、竞品店铺和选品机会
- **商品调研**：根据商品 ID 或商品链接获取 TikTok Shop 商品详情和评论洞察
- **视频分析**：提取 TikTok 或抖音视频中的脚本、分镜、钩子和音乐信息
- **爆款复刻**：基于已验证的爆款结构，结合你的商品素材进行复刻生成（自动识别 TikTok/抖音链接与直链视频 URL）
- **商品生视频**：将商品图片生成 UGC 风格的 TikTok 视频
- **AI 图片生成**：基于 GPT Image 2 模型，根据文本提示生成 AI 图片，并可选上传参考图（最多 5 张）
- **用户视频数据分析**：拉取 TikTok 用户的视频列表，包括播放、点赞、分享、评论和电商购物车数据
- **视频下载**：通过 Clipcat API 下载 TikTok 或抖音视频

## 安装

### 1. 安装 Skill

复制下面的命令并发送给你的 OpenClaw，即可自动安装。

```bash
# Install skill
curl -fsSL https://clipcat.ai/cli | bash
```

### 2. 获取 API Key

注册或登录后，在个人中心生成 API Key：

[生成 API Key](https://clipcat.ai/workspace?modal=settings&tab=apikeys)

### 3. 配置 API Key

将 `your_api_key_here` 替换为你的真实 API Key，然后把下面的命令发送给 OpenClaw 完成配置。

```bash
# Set your Clipcat API key
openclaw env set CLIPCAT_API_KEY your_api_key_here
```

## 使用方式

安装完成后，你可以直接让 OpenClaw 帮你：

- “搜索本周关于 lip gloss 的 TikTok 爆款视频”
- “搜索 TikTok Shop 里热门宠物产品，并展示竞品店铺”
- “用我的商品图片复刻这个 TikTok 视频”
- “用这些图片生成一个商品视频”
- “生成一张模特手持我商品的 AI 图片”
- “分析这个视频并提取脚本”
- “展示这个 TikTok 用户最近的视频及互动数据”
- “下载这个 TikTok 视频”
- “拉取这个商品链接对应的 TikTok Shop 商品详情和评论亮点”

## 重要说明

- 视频生成任务是异步执行的，通常需要几分钟
- OpenClaw 会先展示参数，并在你确认后再提交任务
- 不要手动重复提交任务，Clipcat 已内置重试处理
- 请保留完整的 TikTok 或抖音链接，尤其是带签名参数的 URL

## 支持模型

- `sora2` - 10s, 15s (720p)
- `sora2_official` - 4s, 8s, 12s (720p)
- `veo3.1fast` - 8s, 16s, 24s (720p, 4K)

请始终通过 `clipcat replicate -h` 查看当前最新的模型列表。

## 支持语言

英语、中文、法语、德语、马来语、越南语、泰语、日语、韩语、印尼语、菲律宾语

## 使用示例

### 示例 1：搜索 TikTok 爆款视频

```text
Search for viral TikTok videos about lip gloss in the US market this week.
Show me the top 10 results sorted by likes.
```

会返回按热度排序的相关爆款视频列表，包含核心指标和源链接，便于进一步分析。

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

### 示例 3：从零生成商品视频

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

会返回结构化数据，包括逐镜头拆解、画面描述、口播内容和背景音乐信息。

### 示例 5：下载 TikTok 视频

```text
Download this TikTok video:
https://www.tiktok.com/@username/video/111222333
```

该操作为同步执行，会立即返回视频直链 URL。

## 使用建议

- 始终提供完整的 TikTok/抖音链接
- Prompt 越具体，结果通常越好
- 视频生成需要时间，请等待任务完成
- 带签名参数的视频链接请完整保留
- 根据时长和质量需求选择合适模型

## 相关链接

- 官网：https://clipcat.ai
- OpenClaw 页面：https://clipcat.ai/tiktok/openclaw
- API 文档：详细接口说明请查看 `SKILL.md`
