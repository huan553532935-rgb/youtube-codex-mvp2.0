# AGENTS.md - Codex 项目核心配置

## Context
我是 Andy Wang，在 Los Angeles，正在学习把 AI Agent 真正用起来做自动化工作流。当前项目聚焦于 YouTube 评论分析自动化：让 Codex 每天定时抓取指定 YouTube 频道的新评论，清洗、分类、总结，并生成可分享的 Excel 报表和 Streamlit 仪表盘。

## Goal
搭建一个每天自动运行的 YouTube 评论分析 MVP：每天从 `https://www.youtube.com/@TheValley101` 抓取最新 50 条频道相关评论，按“工具请求”“技术问题”“正面反馈”“负面反馈”四类做免费关键词规则分类，覆盖生成 `reports/youtube_comments_latest.xlsx`，并用 Python + Streamlit 提供一个本地仪表盘预览。v2 再升级 OpenAI 语义分类、日期归档、GitHub Actions 或 Codex Automations。

## Constraints
- 必须使用 Python + Streamlit 做前端
- 所有 API Key 只能放在 `.env.local`，永远不要写进代码
- 不允许使用付费第三方库，优先使用免费、开源、稳定的方案
- 每一步必须先 Plan 再执行
- 失败经验必须写进 `AGENTS.md` 或 memory 文件
- 不要手动硬编码 YouTube 评论数据，必须保留后续接入 YouTube Data API 的结构
- Excel 输出至少包含原始评论、清洗结果、分类结果、每日摘要四类数据
- 仪表盘至少展示评论总量、四类分类分布、最新评论表格、重点负面反馈和工具请求列表

## Memory
- 2026-06-13: 创建本地 MVP 结构。官方 YouTube Data API 接入路径为 `channels.list` + `forHandle` 解析频道 ID，再用 `commentThreads.list` + `allThreadsRelatedToChannelId` + `order=time` 拉取最新频道相关评论。API Key 只从 `.env.local` 读取。
- 2026-06-13: 验证时发现 bundled Python 缺少 `python-dotenv` 和 `requests`。解决方案：标准环境仍通过 `requirements.txt` 安装依赖，代码层增加 `.env.local` 轻量解析和 `urllib` fallback，保证基础脚本不因缺少这两个包完全阻塞。
- 2026-06-13: 验证时发现 `8501` 已被旧 Streamlit 股票监测服务占用。解决方案：保留用户旧服务，不杀外部进程，本项目预览改用 `127.0.0.1:8502`。
- 2026-06-13: Streamlit 1.58 提示 `use_container_width` 已过期。解决方案：仪表盘统一改用 `width="stretch"`，并将 `requirements.txt` 中 Streamlit 最低版本设为 `1.58.0`。
- 2026-06-13: YouTube 评论分析 MVP 完成脚本级验证：`py_compile` 通过、空报表生成通过、Excel sheet 结构通过、四类关键词分类测试通过；内置浏览器已验证 `127.0.0.1:8502` 空状态页面和临时非空数据页面。
- 2026-06-13: 用户提供 YouTube Data API Key 后，已写入本地 `.env.local` 且权限设为 `600`，没有写入代码或 Memory。真实抓取成功，频道解析为“硅谷101”，频道 ID 为 `UCKV2yWPB3wn0RTZh3cTD8YA`。
- 2026-06-13: 真实抓取生成 `reports/youtube_comments_latest.xlsx` 和 `data/youtube_comments_latest.json`。本次结果：49 条评论，工具请求 0，技术问题 6，正面反馈 42，负面反馈 1。
- 2026-06-13: 真实数据抽查发现“怎么 + 夸奖”会误判为技术问题、“连个像样/造不出来”未命中负面。解决方案：优化 `scripts/classify_comments.py`，增加正面词优先修正、中文负面词、繁体问题词，并把英文关键词改为词边界匹配。
- 2026-06-13: 用户反馈网页无法打开。原因是 `127.0.0.1:8502` 上的 Streamlit 服务已经停止，报表文件仍正常存在。解决方案：重新运行 `streamlit run app.py --server.address 127.0.0.1 --server.port 8502`，并用 `/_stcore/health` 返回 `ok` 确认服务恢复。
- 2026-06-14: GitHub MCP 能读取新仓库 `huan553532935-rgb/youtube-codex-mvp`，但创建 `README.md` 时返回 `403 Resource not accessible by integration`。原因是新仓库尚未授权给 GitHub MCP/Codex GitHub App 的写入范围。解决方案：在 GitHub/Codex 连接器中为新仓库授予访问权限后重试 MCP 写入。
- 2026-06-14: 用户登录 GitHub installations 页面后再次重试 GitHub MCP 写入，仍返回 `403 Resource not accessible by integration`，且 `_list_installations` 返回空。说明当前只是 GitHub 网页登录成功，MCP 写权限仍未安装/授权到 `youtube-codex-mvp` 仓库。
- 2026-06-14: 用户创建并打开新仓库 `huan553532935-rgb/youtube-codex-mvp2.0`。GitHub MCP 能读取 repo metadata，显示当前账号对仓库有 `push/admin` 权限，但 `_create_file` 和 `_create_blob` 均返回 `403 Resource not accessible by integration`。结论：Chrome/GitHub 登录态正常，仓库正常，阻塞点仍是 Codex/GitHub MCP integration 没有写入安装权限。

## Conventions
- 每次都用 Plan 模式先列编号计划
- 失败后把错误、原因、解决方案记录在 `## Memory` 里
- 每次成功完成一个可复用流程后，整理成 Skill 或脚本模板
- 输出永远要带“可直接复制运行”的代码块
- 代码优先保持简单、清晰、可本地运行
- 文件命名使用英文小写和下划线，例如 `youtube_comments_daily.py`
- 数据文件默认放在 `data/`
- 报表文件默认放在 `reports/`
- Streamlit 页面默认入口为 `app.py`
