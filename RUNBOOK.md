# YouTube Comments MVP Runbook

## 1. 本地初始化

```bash
cd /Users/macbookpro/Desktop/codex2.0
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.local.example .env.local
```

打开 `.env.local`，填入：

```bash
YOUTUBE_API_KEY=你的 YouTube Data API v3 Key
YOUTUBE_CHANNEL_HANDLE=@TheValley101
YOUTUBE_CHANNEL_ID=
MAX_COMMENTS=50
OUTPUT_EXCEL_PATH=reports/youtube_comments_latest.xlsx
```

`YOUTUBE_CHANNEL_ID` 可以留空，脚本会用 `YOUTUBE_CHANNEL_HANDLE` 自动解析频道 ID。

## 2. 生成 Excel

```bash
source .venv/bin/activate
python scripts/run_daily.py
```

输出文件会覆盖：

```bash
reports/youtube_comments_latest.xlsx
```

如果还没有 API Key，只想先看 Excel 结构：

```bash
python scripts/run_daily.py --init-empty-report
```

## 3. 启动 Streamlit 仪表盘

```bash
source .venv/bin/activate
streamlit run app.py
```

浏览器打开 Streamlit 提供的本地地址后，即可预览分类分布、最新评论、工具请求和负面反馈。

如果 `8501` 已经被其他 Streamlit 服务占用，可以指定 `8502`：

```bash
streamlit run app.py --server.address 127.0.0.1 --server.port 8502
```

## 4. macOS cron 每日运行

先确认 `.venv` 已安装依赖，然后运行：

```bash
crontab -e
```

添加一行，每天早上 9 点运行：

```bash
0 9 * * * cd /Users/macbookpro/Desktop/codex2.0 && /Users/macbookpro/Desktop/codex2.0/.venv/bin/python scripts/run_daily.py >> logs/youtube_comments_cron.log 2>&1
```

## 5. Windows 任务计划程序

程序填写：

```text
C:\path\to\codex2.0\.venv\Scripts\python.exe
```

参数填写：

```text
scripts\run_daily.py
```

起始位置填写：

```text
C:\path\to\codex2.0
```

## 6. v2 计划

- 增加 OpenAI 语义分类，替代关键词规则
- 增加每日归档文件，例如 `youtube_comments_YYYY-MM-DD.xlsx`
- 增加 GitHub Actions 定时运行
- 增加 Codex Automations 定时唤醒
- 增加按视频、主题、情绪趋势的更完整仪表盘
