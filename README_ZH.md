# Clipcat Skill —— TikTok Shop 带货视频 AI 技能

**把任意 AI Agent 变成 TikTok Shop 视频制作人。** 搜索 TikTok 爆款视频，调研 TikTok Shop 商品、店铺、达人和直播间，拆解爆款视频为什么能卖，一键生成可直接出片的爆款带货提示词，用自己的商品复刻爆款，把商品图生成 AI 带货 / UGC / 真人口播视频，用文字提示生成电商图片，下载 TikTok 或抖音视频——全部通过一个 CLI 完成。

**任何能执行 shell 命令的 AI Agent 都能用**——Claude Code、Codex、WorkBuddy、OpenClaw、Cursor，或你自研的 Agent。它以一个跨平台的 `clipcat` CLI 加一份 `SKILL.md` 清单的形式提供：Agent 读取清单、执行 `clipcat` 命令并解析 JSON 输出。

最新指南和示例请查看：[https://clipcat.ai](https://clipcat.ai)

English: [README.md](README.md)

## 工作原理

Clipcat 本质上只是一个小巧的 CLI 二进制 + 一份 `SKILL.md` 技能清单，任何能执行 shell 命令的 Agent 都能驱动它：

- Agent 读取 `SKILL.md` 了解可用命令与约定。
- 调用 `clipcat <命令>` 并解析其 JSON 输出（默认格式）。
- 异步任务（视频/图片生成）提交后立即返回，由 Agent 跨轮次轮询。

OpenClaw 会依据清单自动安装该 Skill；其他 Agent 则手动安装 CLI（见下文）。无论哪种方式，底层运行的都是同一个 `clipcat` 二进制。

## 核心能力

- **TikTok 电商数据情报**：覆盖达人、商品、店铺、视频、直播、关键词/图片搜索 6 大实体域，支持榜单、多维筛选发现、趋势、详情、评价、评论与跨实体关系查询（Agent 通过 `clipcat <实体> -h` 选择具体命令）
- **爆款带货提示词生成器**：检索全球最大的真实 AI 带货视频与提示词库（覆盖 TikTok 全部国家与品类、真正卖爆的高销量 AI 带货视频），匹配与你需求相关的内容，一键生成可直接出片的爆款带货视频提示词，并附上它逆向自哪支真实视频
- **视频分析**：提取 TikTok 或抖音视频中的脚本、分镜、钩子和音乐信息
- **爆款复刻**：基于已验证的爆款结构，结合你的商品素材进行复刻生成——参考视频可以是 TikTok/抖音链接、直链视频 URL，或本地视频文件（最大 100MB）
- **商品生视频**：将商品图片生成 UGC 风格的 TikTok 视频
- **AI 图片生成**：基于 GPT Image 2 模型，根据文本提示生成 AI 图片，并可选上传参考图（最多 5 张）
- **超分**：用 `--enhance` 把成片提升到 720p、1080p 或 2K
- **可复用角色**：通过 `--character-id` 让多条视频里的出镜角色保持一致
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
- “我卖的是便携榨汁机，去爆款库里找几条美国市场的真人测评，改写成我的带货视频提示词”
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

## 爆款库里有什么

`clipcat prompt search` 不是凭空写脚本，而是从一个真实、可检索的「真的卖爆过的 AI 带货视频库」里检索匹配：

| 资产                        | 能给你什么                                                                   |
| --------------------------- | ---------------------------------------------------------------------------- |
| **5000+ 真实爆款 AI 带货视频** | 每一条都真的在 TikTok Shop 卖爆过 —— 不是 demo，不是概念                      |
| **15 个市场在线**           | 美英及其余各大 TikTok Shop 市场，提示词按你的目标地区精准匹配                 |
| **30 个一级品类在线**       | 美妆、家居、数码、食品……按你产品的品类匹配                                    |
| **提示词可复制**            | 每条爆款都附带产出它的原始提示词，可复制，也能一键复刻同款                    |
| **真实销量背书**            | 每条都带原视频真实的 GMV / 销量 / 播放，基于真赢过的视频来做                  |
| **每日更新**                | 天天有新爆款进来 —— 美国市场日更，其余市场周更                                |
| **语义检索**                | 描述你想要的效果（如「暖光居家 手持特写 真人出镜」）即可检索到对应爆款        |

因为每条提示词都基于一个真的卖爆过的视频，`clipcat prompt search` 给你的是一条**建立在验证过的爆款之上、可直接出片**的提示词，而不是靠猜。每条结果都会带上原视频链接与公开详情页，随时能看清这条提示词逆向自哪支视频。

## 支持模型

试用模型所有用户可用；标准模型需付费套餐。

| 模型 ID              | 时长                | 分辨率            | 备注                                          |
| -------------------- | ------------------- | ----------------- | --------------------------------------------- |
| `grok_imagine`       | 10s, 15s            | 480p, 720p        | **试用**，默认。xAI Grok Imagine 1.5，仅 9:16  |
| `veo3.1fast`         | 8s, 16s, 24s        | 720p              | **试用**。Google Veo 3.1 Fast，质量与成本均衡  |
| `omini_flash`        | 10s, 20s            | 720p, 1080p       | **试用**。Gemini Omni Flash，Google 最新模型   |
| `seedance2_mini`     | 4-15s（任意整数）   | 480p, 720p        | **试用**。Seedance 2 Mini，高性价比档（免费用户仅 480p） |
| `mmh3_promo`         | 10s, 15s            | 480p, 720p, 2K    | **试用**。MiniMax H3 补贴渠道，免费用户可用    |
| `seedance2`          | 4-15s（任意整数）   | 480p, 720p, 1080p | 付费。字节 Seedance 2，顶级质量                |
| `seedance2_5`        | 4-30s（任意整数）   | 480p, 720p        | 付费。字节 Seedance 2.5，最长 30s              |
| `seedance2_fast`     | 4-15s（任意整数）   | 480p, 720p        | 付费。字节 Seedance 2 Fast，快速版本           |
| `wan30`              | 5-30s（任意整数）   | 480p, 720p, 1080p | 付费。阿里 Wan 3.0，最长 30s                   |
| `minimax_h3`         | 10s, 15s            | 768p, 2K          | 付费。MiniMax H3                               |
| `happyhorse10`       | 3-15s（任意整数）   | 720p, 1080p       | 付费。阿里 HappyHorse 1.1                      |

表格列的是各模型「能提供」的档位；实时可用列表、每个「分辨率 × 时长」组合的精确算力和你的余额，请以 `clipcat models` 为准 —— 它也是「模型是否还在架」的唯一权威，`-h` 在两次发版之间必然滞后。`clipcat models` 里有 `mmh3_promo` 时优先用它而不是 `minimax_h3`：同一个模型走补贴渠道，算力只要一小部分。

## 支持语言

英语、中文、法语、德语、马来语、越南语、泰语、日语、韩语、印尼语、菲律宾语、西班牙语

## 支持地区

`US` `GB` `DE` `ES` `FR` `IT` `JP` `MX` `BR` `ID` `MY` `PH` `SG` `TH` `VN`

电商数据接口返回的所有金额字段，无论查询哪个地区，均已换算为美元。

## 使用示例

### 示例 0：生成爆款带货提示词

```
我卖的是便携榨汁机（图片：/path/to/juicer.jpg），目标市场美国。
去爆款提示词库里找几条相关的真实爆款，挑最合适的一条改写成我的带货视频提示词，
告诉我它来自哪支视频，我确认后再生成 8 秒视频。
```

Agent 会先用 `clipcat prompt search` 检索库里逆向自真实高 GMV 视频的结构化提示词，保留原视频的钩子、分镜节奏与口播语气，把商品和卖点换成你的，并附上原视频详情页链接；确认后再进入生成。

### 示例 1：搜索 TikTok 爆款视频

```text
Search for viral TikTok videos about lip gloss in the US market this week.
Show me the top 10 results sorted by likes.
```

会返回按热度排序的相关爆款视频列表，包含核心指标和源链接，便于进一步分析。

### 示例 2：复刻 TikTok 视频

```text
Please use Clipcat’s replication feature to fully replicate the original video’s script and visuals, replacing only the original product with my product. If there is voiceover in the original video, please adapt it to match my product; if there is no voiceover, then no voiceover is needed. The replicated video should not include subtitles. The duration of the new script must be controlled within 15 seconds. Please use the model seedance2.0, and the voiceover language should be English.
This is the TikTok link to replicate:https://www.tiktok.com/@username/video/123456789
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

## 仓库结构

本仓库遵循 Agent Skills 规范——每个 skill 一个目录，放在 `skills/` 下，各自包含一份 `SKILL.md` 清单：

```
clipcat-skill/
├── README.md                  # 英文说明
├── README_ZH.md               # 本文件
├── docs/
│   └── SKILL_ZH.md            # 技能清单的中文翻译
└── skills/
    └── clipcat/
        └── SKILL.md           # Agent 实际加载的技能清单
```

## 相关链接

- 官网：https://clipcat.ai
- OpenClaw 一键安装：https://clipcat.ai/tiktok/openclaw
- 命令参考：[`skills/clipcat/SKILL.md`](skills/clipcat/SKILL.md)
