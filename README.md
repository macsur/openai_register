# 使用无限注册脚本:openai_register.py 即可Token无忧。


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

## 四、总结流程图

```mermaid
graph LR
    A[本地运行注册脚本] --> B[批量生成 GitHub Token 文件]
    B --> C[上传 Token 到 CLIProxyAPI]
    C --> D[配置 API 密钥]
    D --> E[客户端配置: BaseURL + API Key]
    E --> F[免费调用 o1/gpt-4o/o3 等高级模型]

