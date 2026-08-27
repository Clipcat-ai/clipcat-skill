---
name: clipcat
description: Clipcat - TikTok 电商视频创作技能。通过 Clipcat CLI 进行视频搜索、商品洞察、爆款复刻、产品转视频生成、视频拆解分析和视频下载。
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

当你需要通过 `clipcat` 进行 TikTok 电商视频创作时，使用本技能。

获取 API 密钥：https://clipcat.ai/workspace?modal=settings&tab=apikeys

本技能刻意保持精简。详细的参数和支持值属于 CLI 本身 —— 始终以 `clipcat -h` 和 `clipcat <子命令> -h` 作为首要参考。

## 安装

`clipcat version` 能跑通则跳过本节；命令不存在时先安装：

macOS / Linux / Git Bash：

```bash
curl -fsSL https://clipcat.ai/cli | bash
clipcat config --api-key <your-key> --base-url https://clipcat.ai
```

Windows（PowerShell，无 bash）：

```powershell
irm https://clipcat.ai/cli.ps1 | iex
clipcat config --api-key <your-key> --base-url https://clipcat.ai
```

后续用 `clipcat update` 升级（重跑安装器，本地配置会保留）。

## 这个 CLI 用来做什么

`clipcat` 是所有 Clipcat AI 视频生成工作流的本地入口：

- 查询 TikTok 电商数据：达人、商品、店铺、视频、直播、搜索
- 用你的商品复刻爆款视频
- 从图片生成产品视频
- 用 GPT Image 2 从文本提示词生成 AI 图片（可选参考图）
- 分析视频（脚本、场景、音乐）
- 下载 TikTok/抖音视频
- 查询异步任务状态

## 默认 agent 工作流

1. 先用 `clipcat -h` 查看所有命令。
2. 使用任何命令前，先 `clipcat <子命令> -h` 查看参数。
3. 默认使用 JSON 输出。
4. 运行任何消耗算力的视频命令前，先用 `clipcat quote` 报出精确算力、与用户确认，
   再带 `--expected-credits` 提交（见「付费视频命令的算力确认」）。
5. 若任何命令在 stderr 打出更新提示（`⬆ clipcat X is available … Run: clipcat
   update`），执行一次 `clipcat update` 再继续。已是最新时它会自动跳过，可放心运行。

## 如何选择命令

### TikTok 电商数据 —— 实体命令

这些是「名词-动词」命令：`clipcat <实体> <动作>`。用 `clipcat <实体> -h`
列出动作，用 `clipcat <实体> <动作> -h` 查看参数。

- `creator <list|rank|detail|trend|videos|lives|products|followers|following|region|milestones>` —— TikTok 达人/网红
- `product <list|rank|detail|trend|comments|creators|videos|lives>` —— TikTok Shop 商品
- `seller <list|rank|detail|trend|products|creators|videos|lives>` —— TikTok Shop 店铺
- `video <list|rank|detail|trend|comments|captions|products|hashtag>` —— TikTok 视频
- `live detail` —— 直播间详情（仅直播进行时可用）
- `find <creators|products|videos|lives|hashtags|music|photo|all>` —— 关键词/图片搜索；`find all` 是兜底的宽泛搜索

**数据模式（`--mode`）**：部分命令（`creator detail`、`creator videos`、
`product comments`、`seller products`、`video detail`）接受
`--mode offline|realtime`。**默认值已是安全选择 —— 除非用户确有需要，否则省略它。**
查「最新/当前/正在直播」用 `--mode realtime`，查「历史/趋势/累计/榜单」用 `--mode
offline`。永远不要向最终用户暴露 offline/realtime 这两个词；用「历史数据」和「最新数据」来表述。

**分页**：每次调用只返回一页、各计一次费。offline 的 list/rank 命令用 `--page` /
`--page-size` 翻页；realtime 列表用 `--offset` / `--cursor` / `--scroll-param`，
这些值要从上一页结果中回传。需要更多数据就逐页多发几次命令（`--max-pages` 已废弃、被忽略）。

**余额不足**：读取类命令每次 6 算力；余额 <6 时直接报错、不返回数据。

**数据查询实战手册（高密度）：**

- **ID 要顺藤摸瓜，别猜。** 先发现（`<实体> list|rank`、`find …`），
  从结果里取 id，再调用 `detail` / `trend` / 关系类动作。
  detail 类动作接受**逗号分隔的批量**（`--user-ids`、`--product-ids`、
  `--video-ids`，≤10 个）。
- **关系类查询要从「电商活跃」实体出发。** 子资源动作
  （`creator products|lives`、`product creators|videos|lives`、`seller lives`、
  `video products`）对低活跃 id 会返回 `[]`。种子要从 `… rank` 或排序后的
  `… list`（销量/粉丝靠前的）里取，而不是随便挑一行，否则就会拿到空结果。
- **`… rank` 需要一个_较近_的 `--date`。** 传目标周期内的任意一天即可 —— 后端会
  自动对齐到周期锚点（周→该周周一，月→该月 1 号）。但绝不会静默返回_别的_周期的
  数据：若该周期尚未结束、或数据还没生成（T+1，通常次日中午后可用），返回的是
  `data: []` + `period`（`requested` / `latest_available` / `previous`，各含
  `anchor`/`start`/`end`）+ `hint`（直接给出该改用的 `--date`）—— 按 hint 改日期，
  不要对同一周期反复重查。日期还必须落在与 `--rank-type`
  对应的新鲜度窗口内：**day ≤30 天，week ≤6 个月，month ≤12 个月**（从_今天_往前算）。
  日期太**旧**（如去年）会被上游拒绝并报 `rant_type N only support …` —— 把日期
  **往今天方向移**，不要去改 rank-type。
- **类目筛选是数字的，且按层级拆分。** 要把 `rank` / `list` 限定到某个类目，先运行
  `category resolve --keyword <词>`（如 `lipstick` / `口红`；中日韩文会自动用 zh 树）。
  它会返回每个匹配项的层级 + 祖先 id `{l1_id, l2_id?, l3_id?}`（id 对任意地区通用）。
  把对应层级的 id 传给目标命令所接受的参数：
  **product/seller** 的 rank/list 用 **L1→`--category-id`，L2→`--category-l2-id`，
  L3→`--category-l3-id`**（`--category-id` 仅限 L1 —— 别把 L2/L3 的 id 放进去）。
  传的这几级必须是**同一条父子链**上的 id；重复填或层级不匹配会在本地被直接拒绝
  （不消耗调用次数），响应里 `issues` 指出错在哪个字段、`suggested` 给出正确的 id，
  照抄重发即可。另外：
  **creator** rank 用 `--product-category-id` 接受任意层级；**video** rank 只
  接受 L1（`l1_id`）。`hint` 置信度低时 → 运行 `category tree`（L1+L2 概览），按含义
  选定分支，再 `category tree --parent <那个 L2 id>` 下钻到它的 L3 叶子节点。纯关键词
  _搜索_（非榜单）时，`find products --keyword` 不需要 id。
- **`find products` 只返回 product_id**（它是搜索索引）。要标题/
  价格/指标，把 id 接到 `product detail` 里查。
- **空 `[]` / `null` 表示「无」，不是错误。** 同一组筛选重复查时可能返回
  `cached: true` + `retry_after`（ISO 时间戳）：后端记住了这组条件当前无数据，
  到点会自动重新探测 —— 不要轮询，换筛选条件或继续往下做。已知稀疏/特殊的情况：
  `creator region`（不可靠 → 改从 `creator detail` 读 `region`）、
  `video captions`（很多视频没有字幕）、`live detail`（仅直播间在播时可用）、
  `seller products --mode realtime`（无在播库存时为空；默认的 offline 已覆盖该需求）。
- 响应已被**服务端裁剪为有效信息**（id、核心指标、名称、关键链接；
  图片已转为可访问 URL）—— 无需处理原始二进制数据。
- **所有金额一律为美元 USD。** 价格/均价/GMV 等字段（`min_price`、`max_price`、
  `spu_avg_price`、`*_gmv_*_amt` 等）无论 `--region` 是哪个区，都是**美元换算值**；
  响应会带 `"currency": "USD"` 作确认。**切勿标成本地符号如 `¥`/`円`**。若报告需要本地
  货币（如日本市场调研要 JPY），从 USD 按当前汇率换算，并注明为约数。

### 视频生成与工具

- `quote` —— 返回某一次具体生成（`--model` + `--resolution` + `--duration`，社媒复刻再带 `--url`/`--social`，超分再带 `--enhance`）的精确算力。付费命令报价的首选：服务端把算术全做完，直接回 `totalCredits`（已含超分费）以及 `enhanceCredits` / `enhanceBlocked`（见「付费视频命令的算力确认」和「超分」）。
- `models` —— 浏览所有可用视频模型及其算力（离散看 `prices`、range 看 `creditsPerSecond`）和你的余额。用户还没选模型、或报了不可用模型时用它。
- `replicate` —— 用你的商品图复刻爆款视频（自动识别 URL 类型）；图片用 `--image`（本地）或 `--image-url`（URL）；本地文件和 URL 可混用；支持 `--model`、`--duration`、`--size`（仅 `9:16` 或 `16:9`）、`--lang`、`--resolution`、`--enhance`（超分，见下）、`--character-id`、`--expected-credits`
- `product_video` —— 仅用商品图生成视频（无参考视频）；图片用 `--image`（本地）或 `--image-url`（URL）；本地文件和 URL 可混用；`--size` 仅接受 `9:16` 或 `16:9`；支持 `--enhance`（超分，见下）、`--expected-credits`
- `image` —— 用 **GPT Image 2** 模型从文本提示词生成 AI 图片；可选用 `--image`（本地文件）或 `--image-url`（URL）提供最多 5 张参考图。用 `--aspect-ratio` 选 `1:1`（默认）/ `16:9` / `9:16`。**尺寸提示（9:16/16:9/1:1、portrait/landscape/square、竖版/横版/方图、banner、wallpaper）必须同时出现在 `--prompt` 和 `--aspect-ratio` 中** —— `--aspect-ratio` 设定画布，提示词里的尺寸词锚定构图。不要臆造用户没要求的尺寸。
- `list_images` —— 列出服务端的图片生成任务；支持 `--status` / `--limit` / `--page` 筛选
- `breakdown` —— 分析视频（脚本、场景、音乐）；之前分析过的会立即返回缓存结果
- `download` —— 下载 TikTok/抖音视频（返回签名 URL）；缓存结果立即返回
- `query_task` —— 按 ID 和类型查询任务状态（`--type replicate | product | breakdown | download | image`）。省略 `--task-id` 则恢复本地最近一个任务。开启超分时，`videos[]` 每项各带 `status` / `enhanceStatus`（见「超分」）。
- `list_tasks` —— 列出服务端最近的**视频相关**任务（`--type` 必填：`replicate | product | breakdown | download`）。图片任务用 `list_images`。
- `character list` —— 列出账号下保存的角色（`id`、`name`、`status`、`type`）。`id` 即 `replicate` / `product_video` 的 `--character-id` 取值；只有 `status: completed` 的角色可用。支持 `--status` / `--limit` / `--page` / `--sort-by` / `--sort-order`。免费（自有账号元数据，不扣算力）。

## 付费视频命令的算力确认

`replicate` 和 `product_video` 会消耗算力。发起前务必先确认算力——且**绝不要自己算**，
让 `clipcat quote` 返回：

1. 用将要提交的同一套参数运行 `clipcat quote`（`--model`、`--resolution`、`--duration`；
   TikTok/抖音链接复刻再带 `--url`，会自动计入下载费；要超分再带 `--enhance`）。它返回
   `totalCredits`（按秒单价、下载费、延迟扣的超分费等全由服务端算好）和你的余额
   `remainingCredits`。
2. 把模型、时长、分辨率和这个 `totalCredits` 展示给用户，获得明确确认。
3. 带 `--expected-credits <totalCredits>` 提交。仅当实扣**高于**你传的值才会被拒，
   绝不会多扣（实扣更低——缓存命中、促销——直接放行）；被拒时会返回当前算力，按新数字
   与用户重新确认后，用更新的 `--expected-credits` 重新提交。

用户还没选模型（或需要完整菜单）时，用 `clipcat models` 列出所有可用模型及其算力，
再对选定项 `clipcat quote`。

付费专享模型（如 `seedance2`、`happyhorse10`）需付费套餐；`clipcat quote` 会标记
（`premiumBlocked`），免费用户提交会被服务端拒绝。

## 超分（`--enhance`）

`replicate` 和 `product_video` 支持 `--enhance 720p|1080p|2k`，对成片做超分（画质提升）。规则：

- **目标档位必须严格高于生成分辨率**：480p → 720p / 1080p / 2k，720p → 1080p / 2k，
  1080p → 2k，2k → 无可选。CLI 只做非空枚举校验，档位阶梯由服务端强校验。
- **仅付费套餐可用。** 免费用户提交会被拒；`clipcat quote --enhance` 会标记
  `enhanceBlocked: true`（需升级会员）。
- **算力** = ceil(视频时长秒 / 10) × 档位单价（`720p`=10、`1080p`=20、`2k`=30 每 10 秒）。
  **延迟扣费**——只在原片生成成功后才实扣。`quote` 以 `enhanceCredits` 返回，且已并入
  `totalCredits`；把该 `totalCredits` 用 `--expected-credits` 提交即可。
- **状态语义**（`query_task`）：原片就绪后先以 `status: enhancing` 出现在 `videos[]`，此时
  `videoUrl` 为原片（可先用），但**整个任务要等超分完成才到终态 completed**（标准档 1 分钟
  视频超分约 6-10 分钟）。`enhanceStatus: failed` → 任务照常完成并交付原片，超分费已退。

```bash
clipcat quote --model seedance2 --resolution 480p --duration 8 --enhance 1080p
# → seedance2 480p 8s → 160 credits  + 20 enhance (1080p) → total 180 credits
clipcat product_video --image product.jpg --model seedance2 --duration 8 \
  --resolution 480p --size 9:16 --enhance 1080p --expected-credits 180
```

## replicate：URL 类型自动识别

`clipcat replicate` 会自动识别 URL 类型：

- **TikTok/抖音链接** → 调用 `/replicate_from_social`（下载会**额外消耗 10 个算力**）
- **可直接下载的视频 URL** → 调用 `/replicate`

用社媒链接运行前，务必告知用户这笔额外算力消耗。

## clipcat:// 资产引用

之前轮次出现的 `clipcat://...` 字符串是稳定的资产引用。把它们**原样**传给任意 `--image-url` / `--character-id` 参数 —— 绝不要加 `https://` 前缀或做任何修改。详见子命令 `-h`。

保存的角色，其 `--character-id` 从 `clipcat character list` 的 `id` 列取 —— 不要臆造 id。

## 异步任务规则

`replicate`、`product_video`、`image`、`breakdown` 都是异步的。这四个命令
**提交后立即返回**任务 ID —— 它们不会阻塞。

典型耗时：`image` 约 3 分钟，`breakdown` 几分钟，`product_video` /
`replicate` 10 分钟以上。**绝不要在单次工具调用内同步等待** —— 任何现实中的
agent 框架都有工具调用超时（通常 60 秒），会在任务完成前就把调用杀掉。一律
走「提交 → 返回 → 跨轮次轮询」。

1. 任务 ID 会自动保存到本地 `~/.clipcat/tasks.json`。
2. 用 `clipcat query_task --task-id <id> --type <type>` 查状态。每次
   调用都立即返回当前状态。省略 `--task-id` 则恢复最近一个任务。跨轮次重复
   调用（建议节奏：`image` 约 30 秒，`breakdown` / `product_video` / `replicate`
   约 1-2 分钟），直到 `status` 为 `completed` 或 `failed`。
3. 用 `clipcat list_tasks --type <replicate|product|breakdown|download>`
   查看服务端某类型的任务。

## query_task：自动恢复

`clipcat query_task` 不带参数时，会自动从 `~/.clipcat/tasks.json` 读取最近一个任务并恢复它。无需记忆任务 ID。

## 可用模型

试用模型所有用户可用；标准模型需付费套餐。

| 模型 ID              | 时长                  | 分辨率            | 备注                                                              |
| -------------------- | --------------------- | ----------------- | ----------------------------------------------------------------- |
| `veo3.1fast`         | 8s, 16s, 24s          | 720p, 1080p       | **试用**。Google Veo 3.1 Fast，质量与成本均衡                     |
| `veo3.1pro`          | 8s, 16s, 24s          | 720p, 1080p       | **试用**。Google Veo 3.1 Pro，高质量版本                          |
| `omini_flash`        | 10s, 20s              | 720p, 1080p       | **试用**。Gemini Omni Flash，Google 最新模型                      |
| `grok_imagine`       | 10s, 15s, 20s, 30s    | 720p              | **试用**，默认。仅 9:16 宽高比，支持更长片段                      |
| `seedance2`          | 4-15s（任意整数）     | 480p, 720p, 1080p | 标准（付费）。字节 Seedance 2，顶级质量。**默认 480p**            |
| `seedance2_5`        | 4-30s（任意整数）     | 480p, 720p        | 标准（付费）。字节 Seedance 2.5，新一代模型，最长 30s。**默认 480p** |
| `seedance2_fast`     | 4-15s（任意整数）     | 480p, 720p        | 标准（付费）。字节 Seedance 2 Fast，快速版本。**默认 480p**       |
| `happyhorse10`       | 3-15s（任意整数）     | 720p, 1080p       | 标准（付费）。阿里 HappyHorse 1.0                                 |
| `sora2_official_exp` | 4s, 8s, 12s           | 720p              | OpenAI Sora 2 官方渠道，9:16 或 16:9               |

当前模型列表请始终查 `clipcat replicate -h`；权威的实时「分辨率 × 时长」算力和你的余额查 `clipcat models`。

**`seedance2` / `seedance2_5` / `seedance2_fast` 默认 `--resolution 480p`**（CLI 在 `quote`、`replicate`、`product_video` 省略 `--resolution` 时自动套用）。只有用户明确要求更高分辨率才传别的值，且 `quote` 与提交必须用同一个值。

## 支持的语言（`--lang`）

`en` `zh` `fr` `de` `ms` `vi` `th` `ja` `ko` `id` `fil` `es`

## 地区（`--region`）

ISO 3166-1 alpha-2，大写：`US` `GB` `DE` `ES` `FR` `IT` `JP` `MX` `BR` `ID` `MY` `PH` `SG` `TH` `VN`。服务端强校验；超出范围的代码会返回当前允许的列表。

## 良好的 agent 行为

- 不确定用哪个命令时，先运行 `clipcat -h`。
- 付费视频命令（`replicate`、`product_video`）：先用 `clipcat quote`（同一套参数）报出精确算力，向用户展示模型/时长/分辨率/`totalCredits` 并获得明确确认，再带 `--expected-credits <totalCredits>` 提交。绝不自己算算力——让 `clipcat quote` 返回。
- 分辨率：报价和提交一律用模型默认档位（`seedance2` / `seedance2_5` / `seedance2_fast` 为 `480p`），除非用户明确要求更高分辨率。绝不静默升到 720p/1080p——更高分辨率会多扣算力。
- 记录任务 ID；跨轮次重复调用 `query_task` 来跟踪长耗时任务。
- 保持签名视频 URL 完整 —— 它们含有 `X-Amz-*` 参数，截断后会失效。
- agent 应优先使用默认的 JSON 输出。
