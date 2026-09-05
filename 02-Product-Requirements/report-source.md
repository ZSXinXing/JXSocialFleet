# 五平台任务运行参数采集记录

采集日期：2026-09-03

## 研究范围

目标是为极星AI社媒的 Android 手机任务定义可配置的运行参数，覆盖登录/接入、推荐页养号、关键词养号、发布作品和运营数据采集。平台范围为 Facebook、YouTube、Instagram、TikTok、Twitter/X。

## 一手来源与证据

### YouTube Data API

- [Videos: insert](https://developers.google.com/youtube/v3/docs/videos/insert)（Google for Developers）：发布请求需要 `snippet` 与 `status`；文档列出 `snippet.title`、`snippet.description`、`snippet.tags[]`、`snippet.categoryId`、`status.privacyStatus`、`status.publishAt`、`status.selfDeclaredMadeForKids`，并说明 `notifySubscribers` 可控制订阅通知。
- 由此映射：视频文件、标题、描述、标签、分类、隐私级别（public/unlisted/private）、预约时间、儿童内容声明、是否通知订阅者。

### TikTok Content Posting API

- [Get Started - Direct Post](https://developers.tiktok.com/doc/content-posting-api-get-started/)（TikTok for Developers）：发布前需要 `video.publish` 授权；创作者信息接口返回 `privacy_level_options` 与 `max_video_post_duration_sec`；视频支持 `FILE_UPLOAD` 或 `PULL_FROM_URL`；示例包含 `privacy_level`、`disable_duet`、`disable_comment`、`disable_stitch`、`video_cover_timestamp_ms` 和 `source_info`。
- 由此映射：视频/图片素材、标题、隐私级别、是否允许评论/合拍/拼接、封面时间点、素材来源、授权状态和时长校验。

### Twitter/X API

- [Create Posts](https://docs.x.com/x-api/posts/create-post)（X Developer Platform）：创建帖子结构包含 `text`、`media`、`poll`、`reply`、`reply_settings`、`quote_tweet_id`；文档注明引用帖子能力受套餐限制。
- 由此映射：正文、媒体列表、投票选项与时长、回复目标、回复权限、引用帖子 ID；平台能力不可用时必须在任务预检中提示。

### Meta/Facebook/Instagram

- [Facebook Page Feed](https://developers.facebook.com/docs/graph-api/reference/page/feed/)（Meta for Developers）：页面发布通常围绕 message/link/attached media、发布状态与预约时间构造请求，实际可用字段取决于 Page 权限和 Graph API 版本。
- [Instagram Content Publishing](https://developers.facebook.com/docs/instagram-api/guides/content-publishing)（Meta for Developers）：Instagram 发布采用容器创建后发布的两阶段流程，按图片、视频、轮播区分媒体类型，并需要媒体 URL、caption、children 等上下文；实际能力取决于专业账号、权限和 API 版本。
- 由于 Meta 文档在当前网络环境下无法稳定读取，以上两项保留官方链接并将字段标为“需按 Graph API 版本校验”，不把推断字段当作已验证事实。

## 运行参数归纳

### 登录/注册/接入

- 平台、账号标识、目标 Android 手机/设备组、代理配置、登录方式、人工验证策略、超时、重试次数、成功判定、会话检查频率。
- 凭据不在原型中保存；验证码、二次验证、身份审核和注册安全步骤必须人工接管。

### 推荐页养号

- 平台、推荐入口、总时长、单条停留区间、浏览数量上限、随机动作开关（关注/点赞/转发/收藏/评论）、动作概率、评论模板/AI 生成策略、每日上限、设备并发、人工接管阈值。

### 关键词养号

- 平台、关键词/标签组、结果排序、结果页深度、每词浏览数量、停留区间、随机动作开关与概率、地区/语言过滤、每日上限、异常处理。

### 发布作品

- 通用：素材类型（视频/图文）、素材、标题/文案、标签、发布方式（立即/预约）、时区、失败重试、发布后校验、目标手机组。
- 平台差异：YouTube 的分类/隐私/儿童声明/订阅通知；TikTok 的隐私/评论/合拍/拼接/封面/上传来源；X 的正文/媒体/投票/回复/引用；Instagram 的媒体类型/轮播/封面/定位/用户标记；Facebook 的消息/链接/媒体/预约时间/受众。

### 运营数据采集

- 作品指标：播放/观看、点赞、评论、转发/分享、收藏、完播率（平台提供时）、发布时间、最近采集时间。
- 账号指标：粉丝/订阅数、关注数、作品数、互动率、账号状态、最近登录、最近同步时间。
- 采集参数：平台、账号/作品范围、指标集合、时间范围、采集频率、失败重试、数据去重和异常阈值。

## 设计决策

- 原型使用“通用参数 + 平台专属参数”两层结构；平台专属字段只在对应平台任务中显示。
- AI 调度可以根据自然语言填充参数，但创建任务前必须展示参数预览；固定脚本必须声明所需参数和默认值。
- 任何字段是否可执行，最终由平台适配器根据授权、账号类型、API 版本和手机端能力校验。
