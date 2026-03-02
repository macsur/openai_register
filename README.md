# 无限调用GPT-5.3高级模型！ 使用脚本 - Token无忧

# 流程图
```mermaid
graph LR
    A[本地运行注册脚本] --> B[批量生成 GitHub Token 文件]
    B --> C[上传 Token 到 CLIProxyAPI]
    C --> D[配置 API 密钥]
    D --> E[客户端配置: BaseURL + API Key]
    E --> F[免费调用 o1/gpt-4o/o3 等高级模型]
```


---
## 教程：免费无限使用 GPT-4o、o1 等高级模型

> **核心方案：** 利用 GitHub Copilot 免费额度 + `CLIProxyAPI` 代理 + 自动化注册脚本
> **目标模型：** GPT-4o, o1-preview, o1, o3-mini 等 GitHub Copilot 内的高级模型
> **原理简述：** 通过脚本自动批量注册 GitHub 账号并生成 Token（每个账号有免费额度），将这些 Token 批量导入自建的 `CLIProxyAPI` 代理服务。该代理提供标准 OpenAI API 格式接口，通过轮询所有 Token 实现了额度的“无限”使用。

---

## 一、前置准备

在开始前，请确保准备好以下环境：

1.  **一台 VPS（云服务器）**
    *   要求：能正常访问 `github.com`（推荐海外 VPS，如 AWS、Vultr、DigitalOcean 等）。
    *   系统：Linux（推荐 Ubuntu/Debian）。
    *   已安装 `Docker` 和 `Docker Compose`。
2.  **本地电脑**
    *   已安装 `Python 3` 环境。
    *   可访问互联网，用于下载脚本和 Token 文件。
3.  **一个支持 OpenAI API 格式的客户端**
    *   例如：CherryStudio、OneAPI、LobeChat、ChatGPT Next Web、各类浏览器插件等。

---
[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/sLjTilvm2RQ/0.jpg)]
---


## 二、详细部署步骤

### 步骤 1：部署 `CLIProxyAPI` 代理服务（在 VPS 上）

1.  **登录 VPS**，进入工作目录（例如 `~/`），创建项目文件夹并进入：
    ```bash
    mkdir copilot-proxy && cd copilot-proxy
    ```
2.  **下载配置文件**（以下命令来自项目 README）：
    ```bash
    # 下载 Docker Compose 配置文件
    curl -L https://raw.githubusercontent.com/CLIProxyAPI/CLIProxyAPI/main/docker-compose.yaml -o docker-compose.yaml

    # 下载基础配置文件示例
    curl -L https://raw.githubusercontent.com/CLIProxyAPI/CLIProxyAPI/main/config.example.yaml -o config.yaml
    ```
    > **注意**：项目仓库名 `CLIProxyAPI` 为推断，请以视频作者提供或实际搜索到的项目地址为准。

3.  **修改 `config.yaml`**：
    使用 `vim` 或 `nano` 编辑文件，找到并修改以下两项：
    ```yaml
    # 允许远程访问管理面板（必须改为 true）
    allow_remote: true

    # 设置访问管理面板的密钥（请自定义一个强密码）
    screen_key: "你的自定义管理密码"
    ```
    *   其他配置（如端口）通常无需修改。默认管理端口为 `8317`。如需修改，请同时修改 `docker-compose.yaml` 中的 `ports` 映射（例如 `"8320:8317"`）。

4.  **启动服务**：
    ```bash
    docker-compose up -d
    ```
    此命令会自动拉取镜像并以后台模式运行。

5.  **访问管理面板**：
    在浏览器中打开：`http://你的VPS公网IP:8317/management.html`
    *   输入刚才设置的 `screen_key` 即可登录。

---
[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/sLjTilvm2RQ/0.jpg)](https://www.youtube.com/watch?v=sLjTilvm2RQ&t=634s)
### 步骤 2：批量生成 GitHub Token（在本地电脑上）

1.  **获取注册脚本**：
    从视频描述区或作者的 **Telegram 频道** 下载自动化注册脚本（例如名为 `OpenAI-regest.py` 或 `github_token_generator.py` 的文件）。
2.  **安装依赖**（通常只需 `requests`）：
    ```bash
    pip install requests
    ```
3.  **运行脚本**：
    ```bash
    python OpenAI-regest.py
    ```
    脚本会自动使用临时邮箱，每约 26 秒注册一个 GitHub 账号并生成对应的 Token。成功后会保存在脚本同目录下的文件中（例如 `tokens.txt` 或 `github_tokens.json`，每行一个 Token）。

4.  **下载 Token 文件**：
    脚本运行一段时间后，将生成的 Token 文件下载到你的本地电脑。

---

### 步骤 3：在 `CLIProxyAPI` 中配置 Token

1.  回到 **CLIProxyAPI 管理面板** (`http://IP:8317/management.html`)。
2.  找到 **“认证文件”** 或 **“Tokens”** 管理页面。
3.  点击 **“上传文件”**，选择你下载的 `tokens.txt` 文件并上传。
4.  上传成功后，页面会显示已导入的 Token 数量增加。
5.  **获取 API 密钥**：
    *   在管理面板找到 **“API 密钥”** 列表。
    *   点击“新增”或复制一个已有的密钥，**保存此密钥**。后续调用 API 时需使用它。

---
[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/JVcUJoWbsLs/0.jpg)](https://www.youtube.com/watch?v=JVcUJoWbsLs)
### 步骤 4：在客户端配置并使用

1.  打开你支持的客户端（如 **CherryStudio**）。
2.  添加一个新的模型提供商/接入点，填写以下信息：
    *   **API 地址 (Base URL)**：`http://你的VPS公网IP:8317/v1`
        > **注意**：路径必须包含 `/v1`。
    *   **API 密钥 (API Key)**：填写上一步复制的 `CLIProxyAPI` 的密钥。
    *   **模型名称**：在模型列表中，选择 `o1-preview`、`o1`、`gpt-4o`、`o3-mini` 等（具体可用模型取决于你上传的 Token 权限）。
3.  保存配置，开始对话即可免费使用高级模型。

---

## 三、重要注意事项与风险

1.  **不稳定性**：此方法依赖 GitHub 免费额度和自动化脚本，**随时可能因 GitHub 政策调整或反爬虫机制而失效**。
2.  **账号与额度风险**：
    *   大量注册的 GitHub 账号若被检测到异常行为，**Token 可能被批量吊销**，导致服务中断。
    *   每个免费 Token 有速率限制（RPM/TPM），当所有 Token 的总额度用尽时，响应会变慢或报错 `429 Too Many Requests`。
3.  **安全建议**：
    *   务必设置强 `screen_key`。
    *   **切勿**将管理面板（`/management.html`）暴露在公网无保护环境下。建议通过防火墙（如 `ufw`）限制访问 IP 仅限你的个人电脑。
4.  **用途限制**：仅供**个人学习与研究**。请遵守 GitHub 和 OpenAI 的服务条款，**严禁用于商业或高负载生产环境**。

---
=======================================================================
@"
# 🚀 免费无限使用 GPT-4o / o1 / o3 高级模型

### 完整部署教程：CLIProxyAPI + 自动注册脚本实战

**适用对象：** 零基础用户 ｜ **预计耗时：** 30-60 分钟 ｜ **难度：** ⭐⭐☆☆☆

---

## 🎬 视频教程

> 点击缩略图将在 **新窗口** 中打开 YouTube 播放。

### 📺 视频一：免费无限使用 GPT！ChatGPT 白嫖方案全教程

<a href="https://www.youtube.com/watch?v=sLjTilvm2RQ&t=634s" target="_blank">
  <img src="https://img.youtube.com/vi/sLjTilvm2RQ/0.jpg" alt="免费无限使用GPT ChatGPT白嫖方案全教程" width="560">
</a>

### 📺 视频二：补充教程与进阶配置

<a href="https://www.youtube.com/watch?v=JVcUJoWbsLs" target="_blank">
  <img src="https://img.youtube.com/vi/JVcUJoWbsLs/0.jpg" alt="补充教程与进阶配置" width="560">
</a>

---

## 📖 目录

1. [原理概述](#1-原理概述)
2. [前置准备](#2-前置准备)
3. [第一阶段：购买与连接云服务器](#3-第一阶段购买与连接云服务器)
4. [第二阶段：部署 CLIProxyAPI 代理服务](#4-第二阶段部署-cliproxyapi-代理服务)
5. [第三阶段：批量注册 GitHub 账号并生成 Token](#5-第三阶段批量注册-github-账号并生成-token)
6. [第四阶段：上传 Token 并配置 API](#6-第四阶段上传-token-并配置-api)
7. [第五阶段：在客户端中接入使用](#7-第五阶段在客户端中接入使用)
8. [常见问题排查 FAQ](#8-常见问题排查-faq)
9. [安全加固建议](#9-安全加固建议)
10. [免责声明](#10-免责声明)

---

## 1. 原理概述

### 🔑 核心逻辑

| 步骤 | 说明 |
|:---:|:---|
| **① 免费额度** | GitHub 为所有注册账号提供 Copilot 免费 API 调用额度（含高级模型） |
| **② 批量注册** | 通过 Python 脚本 + 临时邮箱，自动批量注册 GitHub 账号，生成大量 Token |
| **③ API 代理** | 部署 CLIProxyAPI 服务，批量导入 Token，自动轮询调用 |
| **④ 标准接口** | CLIProxyAPI 对外提供标准 OpenAI API 格式，兼容所有主流 AI 客户端 |

### 整体架构
注册脚本(Python) --批量注册(临时邮箱)--> GitHub账号池(每个有免费额度)
|
生成 Token
|
你的电脑(客户端) --上传Token--> CLIProxyAPI(VPS上运行) <--标准OpenAI格式-- 各种AI客户端
<--返回结果--

支持模型：GPT-4o | o1-preview | o1 | o3-mini | CodeX-mini

---

## 2. 前置准备

### 🛠️ 需要准备的东西

| 项目 | 要求 | 说明 |
|:---|:---|:---|
| **云服务器 VPS** | 1核1G 起步，能访问 GitHub | 推荐海外 VPS（Vultr、AWS Lightsail、DigitalOcean 等） |
| **本地电脑** | Windows / Mac / Linux 均可 | 用于运行注册脚本和访问管理面板 |
| **SSH 客户端** | Windows 自带 PowerShell 即可 | 也可使用 PuTTY、Xshell、Termius 等 |
| **Python 3** | 3.8 及以上版本 | 本地电脑安装，用于运行注册脚本 |
| **浏览器** | Chrome / Edge / Firefox | 用于访问 CLIProxyAPI 管理面板 |

### 📥 需要下载的资源

| 资源 | 来源 | 用途 |
|:---|:---|:---|
| CLIProxyAPI 项目 | GitHub 仓库（视频描述区链接） | API 代理服务 |
| 自动注册脚本 OpenAI-regest.py | 作者 Telegram 频道 | 批量注册 GitHub 账号 |

---

## 3. 第一阶段：购买与连接云服务器

### 3.1 购买 VPS

> 选择任意一家能访问 GitHub 的云服务商。以下以 **Vultr** 为例：

1. 注册并登录 [Vultr](https://www.vultr.com/)
2. 点击 **Deploy New Server**
3. 选择配置：
   - **Server Type**：Cloud Compute
   - **Location**：Tokyo / Singapore / Los Angeles（任选）
   - **OS**：Ubuntu 22.04 LTS
   - **Plan**：最低配 5美元/月 1核1G 足够
4. 点击 **Deploy Now**，等待部署完成
5. 记录下服务器的 **公网 IP 地址** 和 **root 密码**

### 3.2 配置安全组 / 防火墙（⚠️ 极其重要）

> **90% 的连接失败都是因为这一步没做！**

登录云服务商控制台，找到实例的 **安全组 Security Group** 或 **防火墙 Firewall** 设置，添加以下 **入站规则**：

| 协议 | 端口 | 来源 IP | 策略 | 用途 |
|:---:|:---:|:---:|:---:|:---|
| TCP | **22** | 0.0.0.0/0 | ✅ **允许** | SSH 远程连接 |
| TCP | **8317** | 0.0.0.0/0 | ✅ **允许** | CLIProxyAPI 管理面板 |
| TCP | 80 | 0.0.0.0/0 | ✅ **允许** | HTTP（可选） |
| TCP | 443 | 0.0.0.0/0 | ✅ **允许** | HTTPS（可选） |

> ⚠️ **注意**：策略必须是 **允许 Accept**，不能是 禁用 或 拒绝。

> 🔒 **安全提示**：部署成功后，建议将 SSH 22端口的来源 IP 改为你自己的公网 IP。

### 3.3 连接服务器

打开本地电脑的 **PowerShell** Windows 或 **Terminal** Mac/Linux，输入：

``````bash
ssh root@你的服务器公网IP

注册脚本(Python) --批量注册(临时邮箱)--> GitHub账号池(每个有免费额度)
|
生成 Token
|
你的电脑(客户端) --上传Token--> CLIProxyAPI(VPS上运行) <--标准OpenAI格式-- 各种AI客户端
<--返回结果--

支持模型：GPT-4o | o1-preview | o1 | o3-mini | CodeX-mini


---

## 2. 前置准备

### 🛠️ 需要准备的东西

| 项目 | 要求 | 说明 |
|:---|:---|:---|
| **云服务器 VPS** | 1核1G 起步，能访问 GitHub | 推荐海外 VPS（Vultr、AWS Lightsail、DigitalOcean 等） |
| **本地电脑** | Windows / Mac / Linux 均可 | 用于运行注册脚本和访问管理面板 |
| **SSH 客户端** | Windows 自带 PowerShell 即可 | 也可使用 PuTTY、Xshell、Termius 等 |
| **Python 3** | 3.8 及以上版本 | 本地电脑安装，用于运行注册脚本 |
| **浏览器** | Chrome / Edge / Firefox | 用于访问 CLIProxyAPI 管理面板 |

### 📥 需要下载的资源

| 资源 | 来源 | 用途 |
|:---|:---|:---|
| CLIProxyAPI 项目 | GitHub 仓库（视频描述区链接） | API 代理服务 |
| 自动注册脚本 OpenAI-regest.py | 作者 Telegram 频道 | 批量注册 GitHub 账号 |

---

## 3. 第一阶段：购买与连接云服务器

### 3.1 购买 VPS

> 选择任意一家能访问 GitHub 的云服务商。以下以 **Vultr** 为例：

1. 注册并登录 [Vultr](https://www.vultr.com/)
2. 点击 **Deploy New Server**
3. 选择配置：
   - **Server Type**：Cloud Compute
   - **Location**：Tokyo / Singapore / Los Angeles（任选）
   - **OS**：Ubuntu 22.04 LTS
   - **Plan**：最低配 5美元/月 1核1G 足够
4. 点击 **Deploy Now**，等待部署完成
5. 记录下服务器的 **公网 IP 地址** 和 **root 密码**

### 3.2 配置安全组 / 防火墙（⚠️ 极其重要）

> **90% 的连接失败都是因为这一步没做！**

登录云服务商控制台，找到实例的 **安全组 Security Group** 或 **防火墙 Firewall** 设置，添加以下 **入站规则**：

| 协议 | 端口 | 来源 IP | 策略 | 用途 |
|:---:|:---:|:---:|:---:|:---|
| TCP | **22** | 0.0.0.0/0 | ✅ **允许** | SSH 远程连接 |
| TCP | **8317** | 0.0.0.0/0 | ✅ **允许** | CLIProxyAPI 管理面板 |
| TCP | 80 | 0.0.0.0/0 | ✅ **允许** | HTTP（可选） |
| TCP | 443 | 0.0.0.0/0 | ✅ **允许** | HTTPS（可选） |

> ⚠️ **注意**：策略必须是 **允许 Accept**，不能是 禁用 或 拒绝。

> 🔒 **安全提示**：部署成功后，建议将 SSH 22端口的来源 IP 改为你自己的公网 IP。

### 3.3 连接服务器

打开本地电脑的 **PowerShell** Windows 或 **Terminal** Mac/Linux，输入：

``````bash
ssh root@你的服务器公网IP
首次连接会提示是否信任主机指纹，输入 yes 回车
然后输入 root 密码（输入时不会显示字符，这是正常的）
出现 root@xxxx:~# 提示符即表示登录成功 ✅
3.4 安装 Docker 和 Docker Compose
登录服务器后，依次执行以下命令：
# 更新系统包
apt update && apt upgrade -y

# 安装 Docker
curl -fsSL https://get.docker.com | sh

# 验证 Docker 安装
docker --version

# 安装 Docker Compose
apt install docker-compose-plugin -y

# 验证 Docker Compose 安装
docker compose version

4. 第二阶段：部署 CLIProxyAPI 代理服务
4.1 创建项目目录
mkdir -p /opt/cliproxy && cd /opt/cliproxy
4.2 下载配置文件
# 下载 docker-compose.yaml
curl -L https://raw.githubusercontent.com/CLIProxyAPI/CLIProxyAPI/main/docker-compose.yaml -o docker-compose.yaml

# 下载 config.yaml
curl -L https://raw.githubusercontent.com/CLIProxyAPI/CLIProxyAPI/main/config.example.yaml -o config.yaml
📌 以上 URL 为推断路径，请以视频描述区或项目 README 中的实际地址为准。
4.3 修改配置文件（⚠️ 关键步骤）
nano config.yaml
找到并修改以下两项：
# 允许远程访问管理面板（默认是 false，必须改为 true）
allow_remote: true

# 设置管理面板的访问密钥（请自定义一个强密码）
screen_key: "MySecretKey123!"
保存并退出：按 Ctrl+O 回车保存，按 Ctrl+X 退出。
4.4 检查端口映射（可选）
默认端口映射为：

YAML
ports:
  - "8317:8317"
如果 8317 端口被占用，可修改冒号左边的数字，但后续访问地址也要相应更改。

4.5 启动服务
BASH
docker compose up -d
检查运行状态：

BASH
docker compose ps
看到 STATUS 为 Up 即表示启动成功 ✅

4.6 访问管理面板
在本地浏览器中打开：

TEXT
http://你的服务器IP:8317/management.html
⚠️ 必须加上 /management.html 路径！

输入你在 config.yaml 中设置的 screen_key 密码，即可进入管理面板。

管理面板主要功能：

菜单	功能
📊 仪表盘	查看调用统计、Token 状态
🔑 认证文件	上传/管理 GitHub Token
📋 API 密钥	创建/管理 API Key
🤖 管理模型	添加/选择可用模型
⚙️ 系统设置	修改系统配置
5. 第三阶段：批量注册 GitHub 账号并生成 Token
5.1 获取注册脚本
从视频作者的 Telegram 频道 或视频描述区获取自动注册脚本文件 OpenAI-regest.py，保存到本地电脑。

5.2 安装 Python 环境
Windows 用户：

访问 python.org 下载最新版 Python
安装时 务必勾选 Add Python to PATH
打开 PowerShell 验证：python --version
Mac 用户：

BASH
brew install python3
5.3 安装脚本依赖
BASH
pip install requests
5.4 运行注册脚本
BASH
cd C:\AI-Tools\
python OpenAI-regest.py
脚本每约 26 秒自动注册一个 GitHub 账号，并将生成的 Token 保存到同目录下的文件中。

5.5 收集 Token 文件
建议至少运行 10-20 分钟，积累 20-40 个 Token 后按 Ctrl+C 停止。在脚本同目录下找到生成的 Token 文件。

6. 第四阶段：上传 Token 并配置 API
6.1 上传 Token 文件
打开 CLIProxyAPI 管理面板
点击 认证文件 或 Tokens
点击 上传文件，选择 Token 文件
上传成功后页面会显示已导入的 Token 数量
💡 可以多次上传，新 Token 会累加到已有列表中。

6.2 配置 API 密钥
在管理面板点击 配置面板 或 API 密钥
复制或新增一个 API 密钥
📋 记录下这个密钥，后续客户端配置需要使用
6.3 添加可用模型
进入 管理模型 页面
点击 添加新模型
选择 GitHub Copilot 来源
点击 拉取当前模型
可用模型列表：

模型名称	类型	支持图片
gpt-4o	聊天模型	✅ 是
gpt-4o-mini	聊天模型	✅ 是
o1-preview	推理模型	❌ 否
o1	推理模型	❌ 否
o3-mini	推理模型	❌ 否
codex-mini	代码模型	❌ 否
7. 第五阶段：在客户端中接入使用
7.1 通用配置参数
参数	值
API 地址 Base URL	http://你的服务器IP:8317/v1
API 密钥 API Key	在 CLIProxyAPI 管理面板的 API密钥 中复制
模型名称 Model	gpt-4o / o1-preview / o1 / o3-mini 等
7.2 CherryStudio 配置
打开 CherryStudio → 设置 → 模型服务
点击添加
服务名称：CLIProxyAPI
API 地址：http://你的服务器IP:8317/v1
API 密钥：你的密钥
选择模型，保存即可使用
7.3 ChatGPT Next Web 配置
打开设置 → 自定义接口
接口地址：http://你的服务器IP:8317
API Key：你的密钥
模型：gpt-4o
保存设置
7.4 OpenClaw 配置
打开 OpenClaw → 设置
Base URL：http://你的服务器IP:8317/v1
API Key：你的密钥
Model：gpt-4o
⚠️ OpenClaw 特别注意：纯文本模型（o1-preview、o1、o3-mini）不要发送图片，否则会报错 400 is not a multimodal model。只有 gpt-4o 和 gpt-4o-mini 支持图片输入。

8. 常见问题排查 FAQ
❌ SSH 连接被拒绝 Connection refused
检查项	解决方案
安全组未开放 22 端口	添加入站规则：TCP / 22 / 0.0.0.0/0 / 允许
安全组规则状态为禁用	删除禁用规则，重新创建允许规则
SSH 服务未启动	通过 VNC 登录执行 sudo systemctl start sshd
服务器 IP 已变更	回到云控制台确认当前公网 IP
本地网络封锁 22 端口	切换至手机热点重试
❌ 管理面板无法访问
检查项	解决方案
安全组未开放 8317 端口	添加入站规则：TCP / 8317 / 0.0.0.0/0 / 允许
Docker 容器未运行	执行 docker compose ps 检查状态
URL 路径不对	必须访问 http://IP:8317/management.html
allow_remote 为 false	修改为 true 后重启：docker compose restart
❌ 400 is not a multimodal model
原因	解决方案
向纯文本模型发送了图片	切换到 gpt-4o 或 gpt-4o-mini
模型映射配置错误	在管理面板检查模型名称映射
❌ 429 Too Many Requests
解决方案
再次运行注册脚本生成更多 Token
将新 Token 文件上传到管理面板
等待已有 Token 额度重置（通常每月）
9. 安全加固建议
🔒 限制 SSH 访问来源
将安全组中 22 端口的来源 IP 从 0.0.0.0/0 改为你的个人公网 IP。

🔒 修改 SSH 默认端口
BASH
nano /etc/ssh/sshd_config
# 将 Port 22 改为 Port 2222
systemctl restart sshd
修改端口后安全组也要同步放行新端口。

🔒 设置强密码
确保 config.yaml 中的 screen_key 包含大小写字母、数字、特殊符号，长度 12 位以上。

🔒 限制管理面板访问
在安全组中将 8317 端口的来源 IP 也限制为你的个人 IP。

10. 免责声明
⚠️ 重要提示

合规风险：本教程涉及批量注册 GitHub 账号和利用免费额度的操作，可能违反 GitHub 和 OpenAI 的服务条款。请自行评估风险。
不稳定性：此方法依赖 GitHub 的免费政策，随时可能因政策调整而失效。
用途限制：仅供 个人学习与研究 使用，严禁用于商业用途或高负载生产环境。
安全责任：请务必做好服务器安全防护，避免 API 密钥泄露或服务器被入侵。
自行承担：使用本教程产生的任何后果，由使用者自行承担。
🎉 教程完成！
如果一切顺利，你现在已经可以通过自建的 API 代理，免费使用 GPT-4o、o1、o3-mini 等高级模型 了。

📺 回顾视频教程
<a href="https://www.youtube.com/watch?v=sLjTilvm2RQ&t=634s" target="_blank"> <img src="https://img.youtube.com/vi/sLjTilvm2RQ/0.jpg" alt="视频教程一" width="280"> </a> <a href="https://www.youtube.com/watch?v=JVcUJoWbsLs" target="_blank"> <img src="https://img.youtube.com/vi/JVcUJoWbsLs/0.jpg" alt="视频教程二" width="280"> </a>
教程整理于 2025 年 | 基于视频：免费无限使用GPT ChatGPT白嫖方案全教程
"@ | Out-File -FilePath "$env:USERPROFILE\Desktop\免费无限使用GPT高级模型完整教程.md" -Encoding UTF8

TEXT

---

执行完成后，文件保存在：

C:\Users\Administrator\Desktop\免费无限使用GPT高级模型完整教程.md

TEXT

也就是您的 **桌面** 上，文件名为 `免费无限使用GPT高级模型完整教程.md`。

您可以用 **VS Code**、**Typora**、**Obsidian** 或任何 Markdown 编辑器打开查看。








