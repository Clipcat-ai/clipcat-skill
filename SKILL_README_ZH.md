# Clipcat Skill

Clipcat 是一个 TikTok 电商 AI 视频创作 Skill，**任何 AI Agent 都可以集成**——Claude Code、OpenClaw、Cursor，或你自研的 Agent 均可。它以一个跨平台的 `clipcat` CLI 加一份 `SKILL.md` 清单的形式提供：Agent 通过调用 `clipcat` 命令，在同一套工作流中完成爆款视频发现、TikTok Shop 市场情报分析、视频分析、爆款复刻、商品视频生成、AI 图片生成以及 TikTok 视频下载。

最新指南和示例请查看：[https://clipcat.ai](https://clipcat.ai)

## 工作原理

Clipcat 本质上只是一个小巧的 CLI 二进制 + 一份 `SKILL.md` 技能清单，任何能执行 shell 命令的 Agent 都能驱动它：

- Agent 读取 `SKILL.md` 了解可用命令与约定。
- 调用 `clipcat <命令>` 并解析其 JSON 输出（默认格式）。
- 异步任务（视频/图片生成）提交后立即返回，由 Agent 跨轮次轮询。

OpenClaw 会依据清单自动安装该 Skill；其他 Agent 则手动安装 CLI（见下文）。无论哪种方式，底层运行的都是同一个 `clipcat` 二进制。

## 核心能力

- **TikTok 电商数据情报**：覆盖达人、商品、店铺、视频、直播、关键词/图片搜索 6 大实体域，支持榜单、多维筛选发现、趋势、详情、评价、评论与跨实体关系查询（Agent 通过 `clipcat <实体> -h` 选择具体命令）
- **视频分析**：提取 TikTok 或抖音视频中的脚本、分镜、钩子和音乐信息
- **爆款复刻**：基于已验证的爆款结构，结合你的商品素材进行复刻生成（自动识别 TikTok/抖音链接与直链视频 URL）
- **商品生视频**：将商品图片生成 UGC 风格的 TikTok 视频
- **AI 图片生成**：基于 GPT Image 2 模型，根据文本提示生成 AI 图片，并可选上传参考图（最多 5 张）
- **视频下载**：通过 Clipcat API 下载 TikTok 或抖音视频

## 安装

### 1. 安装 CLI

**OpenClaw**：依据 Skill 清单自动安装，无需手动操作。

**其他 Agent / 手动安装**：安装 CLI 二进制：

```bash
# 安装 clipcat CLI
curl -fsSL https://clipcat.ai/cli | bash
```

### 2. 获取 API Key

注册或登录后，在个人中心生成 API Key：

[生成 API Key](https://clipcat.ai/workspace?modal=settings&tab=apikeys)

### 3. 配置 API Key

在 Agent 运行的机器上配置一次即可：

```bash
clipcat config --api-key your_api_key_here --base-url https://clipcat.ai
```

通过环境变量管理密钥的 Agent 也可改为设置 `CLIPCAT_API_KEY`（例如 OpenClaw：`openclaw env set CLIPCAT_API_KEY your_api_key_here`）。

## 使用方式

安装完成后，你可以直接让你的 Agent 帮你：

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
- 提交消耗算力的任务前，Agent 会先用 `clipcat quote` 报出精确算力，向你展示模型/时长/分辨率/算力，并等待你确认
- 不要手动重复提交任务，Clipcat 已内置重试处理
- 请保留完整的 TikTok 或抖音链接，尤其是带签名参数的 URL

## 支持模型

- `grok_imagine` - 10s, 15s, 20s, 30s（720p，仅 9:16）—— 默认模型，支持更长时长
- `veo3.1fast` - 8s, 16s, 24s（720p, 1080p）—— 质量与成本均衡
- `sora2_official_exp` - 4s, 8s, 12s（720p，9:16 或 16:9）—— 仅付费用户，OpenAI Sora 2 官方通道

用 `clipcat models` 查看完整可用模型列表、每个「分辨率 × 时长」组合的精确算力和你的余额；`clipcat replicate -h` 也会列出模型。

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

Generate a 16-second video in English using veo3.1fast model.
```

Agent 会先展示参数，并等待你确认后再提交任务。

### 示例 3：从零生成商品视频

```text
Create a 10-second OOTD video featuring a British girl showcasing my product.
Product image: /path/to/dress.jpg
Use veo3.1fast model, 9:16 aspect ratio, English language.
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
- OpenClaw 一键安装：https://clipcat.ai/tiktok/openclaw
- 命令参考：详细命令说明请查看 `SKILL.md`
