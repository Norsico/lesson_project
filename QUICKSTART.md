# FindFish 快速启动

这份手册用于第一次在本机启动找舆 FindFish。默认使用 Docker，不需要在宿主机安装 Python 依赖。

## 1. 检查 Docker

打开 Docker Desktop，确认 Docker Engine 已启动：

```
docker --version
docker compose version
```

两个命令都能返回版本号后，进入项目目录：

```
cd /path/to/project
```

## 2. 创建配置文件

项目根目录已有 **.env.example** 时，复制为 **.env**：

```
cp .env.example .env
```

编辑 **.env**，先填写下面这些字段：

```
DB_DIALECT=postgresql
DB_HOST=db
DB_PORT=5432
DB_USER=fish
DB_PASSWORD=fish
DB_NAME=fish

QUERY_ENGINE_API_KEY=你的DeepSeek密钥
QUERY_ENGINE_BASE_URL=https://api.deepseek.com
QUERY_ENGINE_MODEL_NAME=deepseek-v4-flash

REPORT_ENGINE_API_KEY=你的DeepSeek密钥
REPORT_ENGINE_BASE_URL=https://api.deepseek.com
REPORT_ENGINE_MODEL_NAME=deepseek-v4-flash

TAVILY_API_KEY=你的Tavily密钥
SEARCH_TOOL_TYPE=AnspireAPI
ANSPIRE_API_KEY=你的Anspire密钥
```

中强度模式还会使用 Media Engine。纯文本任务可以让它使用兼容的文本模型；图片或视频任务需要支持多模态的模型：

```
MEDIA_ENGINE_API_KEY=你的模型密钥
MEDIA_ENGINE_BASE_URL=模型服务地址
MEDIA_ENGINE_MODEL_NAME=模型名称
```

高强度模式还需要 Insight Engine 的模型配置：

```
INSIGHT_ENGINE_API_KEY=你的模型密钥
INSIGHT_ENGINE_BASE_URL=模型服务地址
INSIGHT_ENGINE_MODEL_NAME=模型名称
```

不要把真实密钥写入 README、截图或版本库。

## 3. 启动服务

```
docker compose up -d
```

查看容器状态：

```
docker compose ps
```

需要看到：

- **findfish**：主应用，端口 5050
- **findfish-db**：PostgreSQL，状态为 healthy

## 4. 检查接口

```
curl http://localhost:5050/api/status
```

返回 JSON 后，打开 http://localhost:5050。

## 5. 生成第一份报告

1. 顶部选择 **中（默认报告质量）**。
2. 输入关键词或问题，例如：**小鹏机器人风波**。
3. 点击 **生成报告**，等待 Query 和 Media 产生新的结果。
4. 点击右侧 **Report Engine**。
5. 检查左侧状态框显示输入文件已准备就绪。
6. 点击 **生成最终报告**。
7. 任务完成后，点击 **下载 HTML**、**下载 PDF** 或 **下载 MD**。

点击 Report Engine 只会打开报告页面，不会自动生成。生成期间可以点击 **取消生成**。

## 6. 查看日志

```
docker compose logs -f findfish
```

也可以查看宿主机日志：

```
tail -f logs/query.log
tail -f logs/media.log
tail -f logs/report.log
```

停止查看日志使用 Control + C，不会停止容器。

## 7. 停止和重新启动

停止服务并保留 PostgreSQL 数据：

```
docker compose down
```

再次启动：

```
docker compose up -d
```

只重启主应用：

```
docker compose restart findfish
```

## 8. 常用检查

```
curl http://localhost:5050/api/status
curl 'http://localhost:5050/api/report/status?quality=medium'
docker compose ps
```

## 9. 常见处理

### docker compose up -d 报命令错误

确认使用的是带空格的 Compose 子命令：

```
docker compose version
```

不要使用系统中不存在的旧版 docker-compose 命令。

### 页面提示模型连接失败

检查 **.env** 中的密钥、Base URL 和模型名称，保存后执行：

```
docker compose restart findfish
```

### Report Engine 没有输入文件

先回到顶部选择报告强度并点击 **生成报告**。Report Engine 使用本次任务产生的新 Markdown 文件，不能只打开 Report Engine 页面等待输入。

### PDF 下载失败

先下载 HTML 或 MD，确认报告正文已经生成，再查看：

```
docker compose logs --tail=200 findfish
```

## 10. 关闭环境

```
docker compose down
```

不要使用 docker compose down -v，否则会删除 Compose 管理的数据卷。项目中的 **db_data/** 目录也应保留。
