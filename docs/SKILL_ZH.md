---
name: clipcat
description: 面向任意 AI Agent（Claude Code、Codex、WorkBuddy、OpenClaw）的 TikTok Shop 带货视频一站式技能。搜索 TikTok 爆款视频，调研 TikTok Shop 商品、店铺、达人和直播间，拆解爆款视频的脚本、分镜、钩子和音乐，检索全球最大的真实 AI 带货视频与提示词库并把最匹配的一条改写成可直接出片的爆款带货提示词，复刻爆款，把商品图生成 AI 带货 / UGC / 真人口播 / 商品演示视频，用文字提示生成电商图片，成片超分到 1080p 或 2K，下载 TikTok 或抖音视频。关键词 —— AI 带货视频、TikTok 爆款复刻、TikTok Shop 选品调研、竞品店铺分析、达人榜单、AI 带货提示词库、商品转视频、UGC 视频生成、AI 商品图、TikTok 视频下载。当用户需要 TikTok 电商数据、爆款视频调研或 AI 视频/图片生成时使用。
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

本技能刻意保持精简。详细的参数和支持值属于 CLI 本身 —— 始终以 `clipcat -h` 和 `clipcat <子命令> -h` 作为首要参考。唯一例外是模型清单：模型在两次发版之间就会上下架，`-h` 必然滞后，当前有哪些模型、哪些分辨率和时长一律以 `clipcat models` 为准。

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
- 从爆款提示词库生成可直接出片的带货视频提示词
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

- `creator <list|rank|profile|enrich|trend|posts|sales-videos|lives|products|followers|following|region|milestones>` —— TikTok 达人/网红
- `product <list|rank|detail|trend|reviews|live-comments|creators|videos|lives>` —— TikTok Shop 商品
- `seller <list|rank|detail|trend|catalog|inventory|creators|videos|lives>` —— TikTok Shop 店铺
- `video <list|rank|snapshot|sales|trend|comments|captions|products|hashtag>` —— TikTok 视频
- `live detail` —— 直播间详情（仅直播进行时可用）
- `find <creators|products|videos|lives|hashtags|music|photo|all>` —— 关键词/图片搜索；`find all` 是兜底的宽泛搜索

**两个数据源，命令名已经替你选好了**：没有 `--mode` 需要判断，按你要什么结果挑命令：

| 你要什么 | 命令 | 能拿到 | 拿不到 |
|---|---|---|---|
| 达人最近发了什么 | `creator posts` | 任意公开达人，最新在前 | 没有单条视频的销量/GMV |
| 达人的带货视频表现 | `creator sales-videos` | 每条视频的销量+GMV，可排序 | 只覆盖历史库已收录的达人 |
| 达人当前主页信息 | `creator profile` | 任意公开达人 | 没有累计带货指标 |
| 批量补全达人指标 | `creator enrich` | 一次 ≤10 个，累计指标 | 只覆盖已收录达人 |
| 单条视频的现状 | `video snapshot` | 任意公开视频 | 没有销量/GMV |
| 已有 id 的视频卖得怎样 | `video sales` | 一次 ≤10 个，销量+GMV | 只覆盖已收录视频 |
| 按评分筛评论 | `product reviews` | 评分筛选、翻页 | 略滞后 |
| 最新的评论 | `product live-comments` | 最新，需 `--region` | 没有评分筛选 |
| 店铺历史（含已下架） | `seller catalog` | 销量+GMV，可排序 | 不是当前在售清单 |
| 店铺当前在售 | `seller inventory` | 实时，需 `--region` | 没有销量/GMV |

**历史数据源并不覆盖全部内容**（采集范围受成本限制），所以 `sales` / `catalog` /
`enrich` 这一侧经常回答「未收录」—— id 来自实时搜索时，十次里有四到六次查不到。
而来自 `… rank` / `… list` 的 id 天然就在库内，几乎次次命中。这一侧的空结果意味着
**未收录**，不是**不存在**，改用实时那条命令去核实，别原样重试。永远不要向最终用户
暴露 offline/realtime 这两个词；用「历史数据」和「最新数据」来表述。

**分页**：每次调用只返回一页、各计一次费。历史数据的 list/rank 命令用 `--page` /
`--page-size` 翻页；最新数据的列表用 `--offset` / `--cursor` / `--scroll-param`，
这些值要从上一页结果中回传。需要更多数据就逐页多发几次命令（`--max-pages` 已废弃、被忽略）。

**两种数据源的翻页方式不通用**：历史侧命令（`creator sales-videos`、`product reviews`、
`seller catalog`）按页码翻，对应的实时命令（`creator posts`、`product live-comments`、
`seller inventory`）按游标翻，页码换不成游标。对实时命令用 `--page` 会被 CLI 直接拒绝；
老版本客户端仍会发出去，此时响应带 `pagination_ignored`——
意思是**本次返回的是该源的第一页**，不是你请求的那一页。别再按原页码翻，改用 `next` 里给的
游标；重复原页码只会拿到同一批数据并再次扣 6 算力。

**空结果是答案，不是故障**：历史数据源并不覆盖全部内容，查不到通常意味着「不在该库收录
范围」而非「不存在」。此时响应会带 `try_instead`，里面是可直接执行的换源命令、换过去能拿到
什么（最新源：覆盖全但没有销量/GMV；历史源：有销量/GMV 和排序但只覆盖已收录对象）、以及
还缺哪些参数。换源是另一次独立计费的调用，确实需要那些字段再换。**不要原样重试同一个空
查询。**

**报错会告诉你该不该重试**：失败响应带 `error_kind` 与 `retryable` 两个字段。
`transient`＝限流或短暂抖动，过几秒原样重试即可；`invalid_params`＝报错文案已写明哪个参数
不对，改参数后再发，别原样重试；`temporarily_unavailable`＝此刻重试无用，换个查询或稍后再来。

**余额不足**：读取类命令每次 6 算力（`prompt search` 是 3 算力，且只在套餐自带的免费次数用完后才扣）；余额不足时直接报错、不返回数据。

**数据查询实战手册（高密度）：**

- **ID 要顺藤摸瓜，别猜。** 先发现（`<实体> list|rank`、`find …`），
  从结果里取 id，再调用详情 / 趋势 / 关系类动作。批量类动作（`creator enrich`、
  `video sales`、`product detail`）接受逗号分隔的 id（`--user-ids`、`--video-ids`、
  `--product-ids`，≤10）；`creator profile` / `video snapshot` 是单个 id。
- **id 从哪来，决定了哪条命令答得上。** 来自 `find …`（实时搜索）的 id 是任意公开对象，
  后续要接实时命令 —— `video snapshot`、`creator posts`、`creator profile`；来自
  `… rank` / `… list` 的 id 天然在历史库内，才适合接 `video sales`、
  `creator sales-videos`、`creator enrich`、`seller catalog`。把实时搜来的 id 直接喂给
  带销量的命令，是最常见的白烧算力方式 —— `find videos` 的 id 有约四成查不到销量数据。
  如果用户要的就是实时搜到的那条内容的销量，直说数据未收录，别为了拿到本就不存在的数据
  反复翻页。
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
  `creator region`（不可靠 → 改从 `creator profile` 读 `region`）、
  `video captions`（很多视频没有字幕）、`live detail`（仅直播间在播时可用）、
  `seller inventory`（店铺当前没有在售商品时为空 —— 查历史用 `seller catalog`）。
- 响应已被**服务端裁剪为有效信息**（id、核心指标、名称、关键链接；
  图片已转为可访问 URL）—— 无需处理原始二进制数据。
- **所有金额一律为美元 USD。** 价格/均价/GMV 等字段（`min_price`、`max_price`、
  `spu_avg_price`、`*_gmv_*_amt` 等）无论 `--region` 是哪个区，都是**美元换算值**；
  响应会带 `"currency": "USD"` 作确认。**切勿标成本地符号如 `¥`/`円`**。若报告需要本地
  货币（如日本市场调研要 JPY），从 USD 按当前汇率换算，并注明为约数。

### 爆款带货提示词生成器 —— `clipcat prompt search`

Clipcat 自有的**结构化提示词库**：每一条都逆向自一支真实卖爆过的 TikTok 带货视频，
覆盖 TikTok 全部国家与品类，按真实 GMV 排序。它不是 TikTok 搜索 —— 库里的内容已经拆解
并改写成可以直接喂给视频模型的提示词。

**用户要「带货视频的提示词 / 创意 / 脚本 / 切入角度」时，先来这里检索，不要凭空写。**
背后有一支真实跑出销量的视频，才是这条提示词的全部价值；自己编的那条用户分辨不出来，
但拍出来就是普通视频。

#### 第一步：找出最接近的爆款

- `prompt search --query "<你要什么>"` —— 语义 + 关键词混合检索。既可以描述感觉
  （「暖光居家、手持特写、真人出镜」），也可以给精确词（品牌名、`ASMR`、`OOTD`），
  两路已经融合，不用纠结该用哪种写法。可选筛选：`--region`（小写市场代码）、
  `--category`（TikTok Shop L1 代码，如 `beauty-personal-care`）、
  `--video-type`（`real-review` | `ootd` | `asmr` | `unboxing-pov` | …）、`--limit`（1-20）。
  **query 要从用户自己的商品和受众里长出来** —— 卖什么、卖给谁、哪个市场、要什么调性；
  只丢一个品类名（「护肤」）检索到的是库里最平庸的中段内容。
  计费与其他读命令不同：付费套餐自带一批免费检索次数，用完之后每次 3 算力（其他读命令统一 6）。
  响应带 `quota.remaining` / `quota.free_quota` / `quota.cost_after_quota` —— 快用完时主动
  告诉用户，别让下一次调用突然扣算力。
- **信任结果前先看 `weak_match` 和 `degraded`。** 库永远会返回它手上最接近的几条，
  所以「结果是满的」并不等于「结果对得上」。`weak_match: true` = 库里没有高度相关的内容，
  要如实说明并建议换措辞或去掉一个筛选，而不是把最接近的几条当答案端上去。
  `degraded: true` = 语义检索暂时不可用、只跑了关键词匹配，结果可能不全，**这次检索不计费**
  （额度已退）。命中数少于 `--limit` 是正常且健康的：只有足够相关的才会返回，
  `--region` + `--category` 收得很窄时本来就只有几条。

每条命中都带完整的 `prompt`（英文版在 `prompt_en`）、原视频指标（GMV、销量、播放）、
`matched_facet`（命中的是提示词的哪个分面 —— 风格 / 运镜 / 口播 / …）、
`source_video_url`（原始 TikTok 视频）和 `detail_url`（公开详情页）。

#### 第二步：把命中改写成用户自己的提示词

**绝不能把库里的提示词原样丢回去** —— 那卖的是别人的商品。挑最好的一条（或 2-3 条结构一致
的；结构互相打架的硬糅只会得到一份模板味的提示词），改写成这位用户商品的提示词：

- **保留卖爆的那部分**：开场钩子及其前 1-2 秒发生了什么、分镜顺序与节奏、运镜语言、
  光线、是否真人出镜及出镜者类型、口播语气、促销机制、结尾 CTA。
- **替换**：商品与卖点、画面文字、口播台词，以及一切市场相关的东西（语言、货币、本地说法）。
- **拿不出证据的一律不搬**：评分、销量、奖项、前后对比与功效类表述属于原商品 ——
  要么删掉，要么问用户要他自己的。
- **对齐要提交的模型**：提示词长度和分镜数要落在将要提交的 `--duration` 之内
  （5 秒装得下 2 个镜头，装不下 6 个），口播语言用 `--lang` 指定。
- **花算力之前**先把成品提示词连同它来自哪条 `detail_url`（和 `source_video_url`）一起给用户看。
  能引用出那支真实视频，才是它区别于「我编的一段提示词」的地方。

#### 第三步：出片

- 只有商品图 → `product_video`，改写后的提示词用 `--prompt-file -` 传。
- 想连原视频的运镜和剪辑一起复用 → `replicate --url <source_video_url>` + 用户的 `--image`
  （TikTok 链接会多收 10 算力下载费）。
- 两条都是付费命令：先用同一套参数 `quote` → 与用户确认 → 带 `--expected-credits` 提交
  （见「付费视频命令的算力确认」）。

```bash
clipcat prompt search --query "手持特写精华液瓶身，暖色浴室光，真人口播" \
  --region us --category beauty-personal-care --limit 5
# 挑一条命中 → 按用户的商品改写它的 prompt → 报价并与用户确认：
clipcat quote --model seedance2 --resolution 480p --duration 8
clipcat product_video --image serum.jpg --model seedance2 --duration 8 \
  --resolution 480p --size 9:16 --expected-credits <totalCredits> --prompt-file - <<'EOF'
<改写后的提示词>
EOF
```

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
| `grok_imagine`       | 10s, 15s              | 480p, 720p        | **试用**，默认。xAI Grok Imagine 1.5，仅 9:16 宽高比               |
| `veo3.1fast`         | 8s, 16s, 24s          | 720p              | **试用**。Google Veo 3.1 Fast，质量与成本均衡                     |
| `omini_flash`        | 10s, 20s              | 720p, 1080p       | **试用**。Gemini Omni Flash，Google 最新模型                      |
| `seedance2_mini`     | 4-15s（任意整数）     | 480p, 720p        | **试用**。Seedance 2 Mini，高性价比档。免费用户仅 480p——**必须显式传 `--resolution 480p`** |
| `mmh3_promo`         | 10s, 15s              | 480p, 720p, 2K    | **试用**。MiniMax H3 补贴渠道，免费用户可用                       |
| `seedance2`          | 4-15s（任意整数）     | 480p, 720p, 1080p | 标准（付费）。字节 Seedance 2，顶级质量。**默认 480p**            |
| `seedance2_5`        | 4-30s（任意整数）     | 480p, 720p        | 标准（付费）。字节 Seedance 2.5，新一代模型，最长 30s。**默认 480p** |
| `seedance2_fast`     | 4-15s（任意整数）     | 480p, 720p        | 标准（付费）。字节 Seedance 2 Fast，快速版本。**默认 480p**       |
| `wan30`              | 5-30s（任意整数）     | 480p, 720p, 1080p | 标准（付费）。阿里 Wan 3.0，最长 30s                             |
| `minimax_h3`         | 10s, 15s              | 768p, 2K          | 标准（付费）。MiniMax H3                                          |
| `happyhorse10`       | 3-15s（任意整数）     | 720p, 1080p       | 标准（付费）。阿里 HappyHorse 1.1                                |

模型清单和实时「分辨率 × 时长」算力都以 `clipcat models` 为准 —— 它没返回的模型即已下架，提交必被拒，`-h` 和本表都不作数。`clipcat models` 列出 `mmh3_promo` 时优先用它而不是 `minimax_h3`：同一个模型走限时补贴渠道，算力只要一小部分，且免费用户可用。

**`seedance2` / `seedance2_5` / `seedance2_fast` 默认 `--resolution 480p`**（CLI 在 `quote`、`replicate`、`product_video` 省略 `--resolution` 时自动套用）。只有用户明确要求更高分辨率才传别的值，且 `quote` 与提交必须用同一个值。

`seedance2_mini` **不在**这个自动默认范围里：免费用户只能用它的 480p 档，省略 `--resolution` 会走服务端默认的 720p，提交会被拒。报价和提交都要显式传 `--resolution 480p`。

## 支持的语言（`--lang`）

`en` `zh` `fr` `de` `ms` `vi` `th` `ja` `ko` `id` `fil` `es`

## 地区（`--region`）

ISO 3166-1 alpha-2，大写：`US` `GB` `DE` `ES` `FR` `IT` `JP` `MX` `BR` `ID` `MY` `PH` `SG` `TH` `VN`。服务端强校验；超出范围的代码会返回当前允许的列表。

## 良好的 agent 行为

- 不确定用哪个命令时，先运行 `clipcat -h`。
- 用户要带货视频的提示词 / 创意 / 脚本时：先跑 `clipcat prompt search`，把最接近的那条
  已验证爆款改写成用户商品的提示词，并引用它的 `detail_url`。凭空写等于把「爆款」这两个字
  唯一的依据丢掉了。
- 付费视频命令（`replicate`、`product_video`）：先用 `clipcat quote`（同一套参数）报出精确算力，向用户展示模型/时长/分辨率/`totalCredits` 并获得明确确认，再带 `--expected-credits <totalCredits>` 提交。绝不自己算算力——让 `clipcat quote` 返回。
- 分辨率：报价和提交一律用模型默认档位（`seedance2` / `seedance2_5` / `seedance2_fast` 为 `480p`），除非用户明确要求更高分辨率。绝不静默升到 720p/1080p——更高分辨率会多扣算力。
- 记录任务 ID；跨轮次重复调用 `query_task` 来跟踪长耗时任务。
- 保持签名视频 URL 完整 —— 它们含有 `X-Amz-*` 参数，截断后会失效。
- agent 应优先使用默认的 JSON 输出。
