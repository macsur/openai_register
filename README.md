# 🚀 openai_register 部署指南（优化版）

> ✨ 面向：希望快速部署 OpenAI 兼容代理，并接入 Chatbox / 代码客户端的同学  
> ✅ 特性：目录跳转、折叠块 `<details>`、可直接复制命令块、FAQ 锚点**

![截图](https://raw.githubusercontent.com/macsur/openai_register/main/2026-03-04_135204.png)

![示例图片](https://github.com/macsur/openai_register/blob/main/2026-03-04_153139.png)

![示例图片](https://github.com/macsur/openai_register/blob/main/2026-03-04_153357.png)
---
# 🚀 openai_register 部署指南（GitHub README 优化版）

> ✨ 面向：希望快速部署 OpenAI 兼容代理，并接入 Chatbox / 代码客户端的同学  
> ✅ 特性：目录跳转、折叠块 `<details>`、可直接复制命令块、FAQ 锚点

---

## 📚 目录

- [✨ 项目说明](#project-intro)
- [⚡ 快速开始（3 分钟）](#quick-start)
- [🧰 环境要求](#requirements)
- [📦 部署步骤（Docker Compose）](#deploy-docker)
- [🔧 配置说明（.env）](#env-config)
- [🧪 接口连通性验证](#health-check)
- [💬 Chatbox 接入方式](#chatbox)
- [🔐 生产环境建议（HTTPS/反代/监控）](#production)
- [🔄 升级与回滚](#upgrade-rollback)
- [❓ FAQ（常见问题）](#faq)
- [📎 参考链接](#references)

---

<a id="project-intro"></a>

## ✨ 项目说明

本教程用于部署 `openai_register` 相关方案，核心目标是提供 **OpenAI 兼容 API**，让你用统一接口接入不同上游模型。🧩  
如果本文与上游仓库存在差异，请以仓库最新 `README` / `docker-compose.yml` / `.env.example` 为准。📌

---

<a id="quick-start"></a>

## ⚡ 快速开始（3 分钟）

```bash
# 1) 拉代码
git clone https://github.com/macsur/openai_register.git
cd openai_register

# 2) 创建 .env（按下方模板）
cp .env.example .env 2>/dev/null || touch .env

# 3) 启动容器
docker compose up -d

# 4) 看日志
docker compose logs -f --tail=200
```

---

<a id="requirements"></a>

## 🧰 环境要求

- Linux 服务器（推荐 `1C1G` 起步）🖥️  
- Docker + Docker Compose 🐳  
- 可用上游 API Key（OpenAI 或其他 provider）🔑  
- 开放端口（如 `3000`）🌐  

---

<a id="deploy-docker"></a>

## 📦 部署步骤（Docker Compose）

### 1) 创建 `.env`

```env
# 服务监听
PORT=3000

# 对外鉴权（务必改为随机强密码）
AUTH_TOKEN=replace_with_long_random_token

# 上游配置（至少一组有效）
OPENAI_API_KEY=sk-xxxx
OPENAI_BASE_URL=https://api.openai.com/v1

# 日志
LOG_LEVEL=info
ERROR_LOGS_MAX_FILES=10
```

### 2) 创建 `docker-compose.yml`

```yaml
version: "3.9"
services:
  cliproxyapiplus:
    image: ghcr.io/cliproxyapi/cliproxyapiplus:latest
    container_name: cliproxyapiplus
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "3000:3000"
```

### 3) 启动服务

```bash
docker compose up -d
docker compose ps
docker compose logs -f --tail=200
```

<details>
<summary>🧯 启动失败排查（点击展开）</summary>

```bash
# 检查端口占用
ss -lntp | grep 3000

# 检查 compose 语法
docker compose config

# 强制重建
docker compose down
docker compose up -d --force-recreate
```

</details>

---

<a id="env-config"></a>

## 🔧 配置说明（.env）

| 变量名 | 必填 | 说明 |
|---|---:|---|
| `PORT` | ✅ | 服务监听端口 |
| `AUTH_TOKEN` | ✅ | 你的代理访问令牌（客户端填写这个） |
| `OPENAI_API_KEY` | ✅* | 上游 API Key（至少一组上游可用） |
| `OPENAI_BASE_URL` | ✅* | 上游 API 地址 |
| `LOG_LEVEL` | ⛔ | 日志级别（`info`/`debug`） |
| `ERROR_LOGS_MAX_FILES` | ⛔ | 错误日志保留文件数上限 |

> `✅*` 表示：按你的 provider 方案至少保证一组可用配置。🔁

---

<a id="health-check"></a>

## 🧪 接口连通性验证

### 1) 模型列表

```bash
curl http://127.0.0.1:3000/v1/models \
  -H "Authorization: Bearer replace_with_long_random_token"
```

### 2) 聊天接口

```bash
curl http://127.0.0.1:3000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer replace_with_long_random_token" \
  -d '{
    "model":"gpt-4o-mini",
    "messages":[{"role":"user","content":"你好"}]
  }'
```

返回 JSON 即表示链路基本正常。✅

---

<a id="chatbox"></a>

## 💬 Chatbox 接入方式

在 Chatbox 中填写：🧩

- `Base URL`：`http://<你的服务器IP>:3000/v1`
- `API Key`：`AUTH_TOKEN`（不是上游 key）
- `Model`：从 `/v1/models` 返回值中选择

---

<a id="production"></a>

## 🔐 生产环境建议（HTTPS/反代/监控）

- 使用 Nginx/Caddy 做 HTTPS 反向代理 🔒  
- 限制来源 IP，避免接口裸奔 🌐  
- 定期轮换 `AUTH_TOKEN` 和上游 Key ♻️  
- 开启日志轮转、限制日志体积 📚  
- 监控状态码（4xx/5xx）和响应耗时 📈

<details>
<summary>🌍 Nginx 反代示例（点击展开）</summary>

```nginx
server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate     /path/fullchain.pem;
    ssl_certificate_key /path/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

</details>

---

<a id="upgrade-rollback"></a>

## 🔄 升级与回滚

### 升级

```bash
docker compose pull
docker compose up -d
docker compose logs -f --tail=200
```

### 回滚（示例思路）

```bash
# 1) 先记录当前镜像 tag（建议上线前就固定 tag）
docker images | grep cliproxyapiplus

# 2) 在 docker-compose.yml 改回旧 tag
# image: ghcr.io/cliproxyapi/cliproxyapiplus:<old-tag>

# 3) 重新启动
docker compose up -d
```

---

<a id="faq"></a>

## ❓ FAQ（常见问题）

### Q1: 为什么报 `401 Unauthorized`？  
通常是 `AUTH_TOKEN` 不匹配，或请求头没带：  
`Authorization: Bearer <AUTH_TOKEN>`。🔐

### Q2: 为什么报 `429 Too Many Requests`？  
上游限流或额度不足，先检查上游账户配额和并发策略。⏱️

### Q3: 为什么报 `500/502`？  
优先检查上游连通性、`OPENAI_BASE_URL` 是否正确、DNS/网络是否可达。🧭

### Q4: 为什么提示模型不存在？  
请求模型名不在 `/v1/models` 返回列表里，请先查询可用模型。📋

### Q5: Chatbox 配好了还是不出字？  
确认三项：`Base URL`、`API Key=AUTH_TOKEN`、`Model` 三者一致且有效。🛠️

<details>
<summary>🧪 一键自检命令（点击展开）</summary>

```bash
echo "== compose ps ==" && docker compose ps
echo "== last logs ==" && docker compose logs --tail=100
echo "== health models ==" && curl -s http://127.0.0.1:3000/v1/models \
  -H "Authorization: Bearer $AUTH_TOKEN" | head
```

</details>

---

<a id="references"></a>

## 📎 参考链接

- https://github.com/macsur/openai_register  
- https://github.com/Ant-Intelligence/CLIProxyAPIPlus/blob/main/README.md  
- https://github.com/CliProxyApi/CLIProxyAPIPlus  

---

## 🏷️ Tags

`#OpenAI兼容接口` `#Docker` `#Chatbox` `#CLIProxyAPIPlus` `#部署教程`
