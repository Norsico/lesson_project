# FindFish

FindFish（找舆）是一个面向舆情研究的多引擎分析系统。输入关键词、事件或问题后，系统检索公开网页资料，按需分析图文内容，整理多源证据，并生成可预览、可下载的综合报告。

## 功能概览

- **网页检索**：Query Engine 收集新闻、公告、网页正文和来源信息。
- **图文内容分析**：Media Engine 处理搜索结果中的图片、视频、图文和结构化内容。
- **本地数据分析**：Insight Engine 从本地 PostgreSQL 读取舆情记录，仅在高强度模式中按需启用。
- **综合报告**：Report Engine 生成结构化 HTML，并支持 PDF、Markdown 下载。
- **实时工作区**：左侧显示当前引擎页面，右侧显示任务日志和阶段状态。
- **任务控制**：报告必须手动点击开始；生成期间可以取消，模型请求不会无限等待。

## 工作流程

```
输入关键词或问题
        │
        ├─ Query Engine：检索网页和新闻
        ├─ Media Engine：分析图文和多媒体结果（中、高强度）
        └─ Insight Engine：查询本地舆情数据库（高强度）
        │
        ▼
Report Engine：模板选择 → 文档布局 → 章节生成 → HTML/PDF/MD
```

### 报告强度

| 模式 | 启用引擎 | 适用场景 | 特点 |
| --- | --- | --- | --- |
| 低（快速报告） | Query | 先看基本事实和网页来源 | 处理内容较少 |
| 中（默认报告质量） | Query、Media | 常规舆情分析 | 同时参考网页与图文资料 |
| 高（高质量报告） | Query、Media、Insight | 需要多角度和本地数据对照 | 启用本地数据库，章节上限更高 |

Forum Engine 保留在系统中，但默认关闭，不参与普通任务。

## 系统结构

```
FindFish/
├── app.py                         # Flask 主应用和任务控制接口
├── templates/index.html           # 找舆主工作区
├── SingleEngineApp/               # 各引擎页面入口
├── QueryEngine/                   # 网页检索与资料整理
├── MediaEngine/                   # 图文与多媒体资料分析
├── InsightEngine/                 # 本地 PostgreSQL 数据分析
├── ReportEngine/                  # 报告编排、渲染与导出
├── ForumEngine/                   # 可选的多引擎讨论模块
├── MindSpider/                    # 本地舆情数据采集与数据库结构
├── final_reports/                 # 最终报告文件
├── logs/                          # 引擎运行日志
├── docker-compose.yml             # FindFish 与 PostgreSQL 服务
├── .env.example                   # 配置示例
└── QUICKSTART.md                  # 首次启动手册
```

## 运行要求

- Docker Desktop，包含 Docker Compose
- 可用的 OpenAI 兼容模型接口
- 至少一个网页搜索服务：Tavily、Anspire 或 Bocha
- 建议为 Docker Desktop 分配 4 GB 以上内存

系统默认通过 Compose 启动 PostgreSQL。数据库目录为 **db_data/**，容器重启后数据仍会保留。

## Docker 启动

首次使用时，在项目根目录创建配置文件：

```
cp .env.example .env
```

编辑 **.env**，至少填写 Query、Report 和网页搜索所需的配置。完整步骤见 [QUICKSTART.md](./QUICKSTART.md)。

启动服务：

```
docker compose up -d
```

查看服务状态：

```
docker compose ps
curl http://localhost:5050/api/status
```

浏览器打开：

```
http://localhost:5050
```

停止服务但保留数据库数据：

```
docker compose down
```

查看主应用日志：

```
docker compose logs -f findfish
```

## 配置说明

### 文本和报告模型

FindFish 的文本分析和报告生成使用 OpenAI 兼容接口。推荐将 Query、Report 配置为 DeepSeek，模型名称以当前服务商实际提供的名称为准。

```
QUERY_ENGINE_API_KEY=你的模型密钥
QUERY_ENGINE_BASE_URL=https://api.deepseek.com
QUERY_ENGINE_MODEL_NAME=deepseek-v4-flash

REPORT_ENGINE_API_KEY=你的模型密钥
REPORT_ENGINE_BASE_URL=https://api.deepseek.com
REPORT_ENGINE_MODEL_NAME=deepseek-v4-flash
```

Media Engine 处理纯文本搜索结果时可以使用同类文本模型。确实需要理解图片或视频时，再为 Media 配置支持多模态的国内模型，例如 Kimi 或 MiniMax M3。

### 搜索服务

Query Engine 默认使用 Tavily。Media Engine 可选择 Anspire 或 Bocha：

```
TAVILY_API_KEY=你的Tavily密钥
SEARCH_TOOL_TYPE=AnspireAPI
ANSPIRE_API_KEY=你的Anspire密钥
BOCHA_WEB_SEARCH_API_KEY=
```

### PostgreSQL

Compose 内的数据库服务名是 **db**，容器内部端口是 **5432**：

```
DB_DIALECT=postgresql
DB_HOST=db
DB_PORT=5432
DB_USER=fish
DB_PASSWORD=fish
DB_NAME=fish
```

高强度模式需要 Insight Engine 访问数据库。低、中强度不需要启动 Insight。

### 运行时配置窗口

页面顶部的 **LLM 配置** 按钮可以查看和修改配置。配置窗口不会在页面启动时自动弹出。保存配置后，按页面提示重新加载服务。

## 使用流程

1. 启动 Docker 服务并打开 http://localhost:5050。
2. 选择报告强度。
3. 在输入框填写舆情关键词或具体问题。
4. 点击 **生成报告**，按当前模式收集资料。
5. 进入 **Report Engine** 页面，检查输入文件和任务状态。
6. 点击 **生成最终报告** 才会开始报告编排。
7. 报告完成后，预览 HTML，或下载 HTML、PDF、Markdown。

生成期间右侧日志会显示当前阶段。若模型请求长时间没有响应，页面提供 **取消生成** 按钮。

## 输出目录

| 目录 | 内容 |
| --- | --- |
| **query_engine_streamlit_reports/** | Query Engine 检索报告 |
| **media_engine_streamlit_reports/** | Media Engine 分析报告 |
| **insight_engine_streamlit_reports/** | Insight Engine 本地数据报告 |
| **final_reports/** | 最终 HTML、IR、Markdown 和 PDF |
| **logs/** | 各引擎和 Report Engine 日志 |
| **db_data/** | PostgreSQL 数据文件 |

## PDF 导出

Report Engine 完成后，点击 **下载 PDF**。若导出失败，可查看：

```
docker compose logs --tail=200 findfish
```

HTML 和 Markdown 不依赖 PDF 渲染组件，可以作为替代输出格式。

## 常见情况

### 页面提示模型未配置

检查 **.env** 中对应引擎的 API Key、Base URL 和模型名称，修改后执行：

```
docker compose restart findfish
```

### Report Engine 提示没有新文件

先选择低、中或高强度并点击 **生成报告**，等待所需引擎产生新的 Markdown 文件，再进入 Report Engine。容器重启不会自动把已有输入报告当成新任务。

### 页面能打开但引擎没有内容

```
curl http://localhost:5050/api/status
docker compose logs --tail=200 findfish
```

Query、Media、Insight 分别使用 8503、8502、8501 端口，主工作区使用 5050 端口。

### 需要取消正在生成的报告

在 Report Engine 页面点击 **取消生成**。任务会标记为已取消，正在等待的模型请求会在超时或收到取消标记后结束。任务结束前不要重复提交新的报告任务。

## 开发检查

```
python -m py_compile app.py ReportEngine/flask_interface.py ReportEngine/agent.py
python tests/run_tests.py
```

页面文件是 **templates/index.html**。修改前端文件后，浏览器使用 Command + R（Reload）重新加载页面。

## 许可证

项目许可证见 [LICENSE](./LICENSE)。
