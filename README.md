# JetBrains AI OpenAI API 适配器

<div align="center">

![版本](https://img.shields.io/badge/版本-3.0.0-blue.svg)
![许可证](https://img.shields.io/badge/许可证-MIT-green.svg)
![Python](https://img.shields.io/badge/Python-3.11+-brightgreen.svg)

</div>

> 高性能异步 AI 代理服务，将 JetBrains AI 的大语言模型转换为 OpenAI API 格式，支持真正的流式响应和高并发处理。

## ✨ 核心特性

- **⚡ 高并发异步架构**：基于 httpx + FastAPI，支持数千并发连接
- **🔧 OpenAI 完全兼容**：零修改集成现有 OpenAI 客户端和工具
- **🔐 JWT 轮询认证**：支持配置多个 JetBrains JWT 令牌，实现自动轮询和负载均衡
- **📦 开箱即用**：Docker 一键部署，配置简单

## ⚡ 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/oDaiSuno/jetbrainsai2api.git
cd jetbrainsai2api
```

### 2. 配置密钥

#### 方式一：使用 .env 文件配置（推荐）

1. **获取 JetBrains AI JWT**
   通过IDE(这里以Pycharm为例)和Reqable(小黄鸟)获取JWT：
   - 打开Pycharm中的`设置`，搜索`代理`，选择`自动检测代理设置`并应用
     <img src="images/image-20250703175459818.png" alt="image-20250703175459818" style="zoom:33%;" />
   - 打开小黄鸟并启动`代理设置`，在pycharm中与AI聊下天，在小黄鸟中找到类似于`v5/llm/chat/stream/v7`的接口，把请求头中`grazie-authenticate-jwt`的内容复制下来即为你的`JWT`。
     <img src="images/image-20250703175648995.png" alt="image-20250703175648995" style="zoom:33%;" />
     <img src="images/image-20250703175928552.png" alt="image-20250703175928552" style="zoom: 33%;" />

2. **创建 .env 文件**
   复制 `.env.example` 为 `.env` 并填入您的配置：

   ```bash
   cp .env.example .env
   ```

   编辑 `.env` 文件：

   ```env
   # 服务端口配置
   PORT=8000

   # 客户端 API 密钥配置 (用逗号分隔多个密钥)
   CLIENT_API_KEYS=sk-your-custom-key-1,sk-your-custom-key-2

   # JetBrains AI JWT 令牌配置 (用逗号分隔多个JWT)
   JETBRAINS_JWTS=your-jwt-token-1,your-jwt-token-2

   # 调试模式 (可选)
   DEBUG_MODE=false

   # 可用模型配置 (可选，默认使用内置模型列表)
   # AVAILABLE_MODELS=anthropic-claude-3.5-sonnet,anthropic-claude-4-sonnet
   ```

#### 方式二：使用配置文件（传统方式）

如果您不使用 .env 文件，可以继续使用传统的 JSON 配置文件：

**创建 `jetbrainsai.json` 文件：**

```json
[
    {
        "jwt": "your-jwt-here-1"
    },
    {
        "jwt": "your-jwt-here-2"
    }
]
```

**创建 `client_api_keys.json`：**

```json
[
  "sk-client-key-1",
  "sk-client-key-2"
]
```

**创建 `models.json`（可选）：**

```json
[
    "anthropic-claude-3.7-sonnet",
    "anthropic-claude-4-sonnet",
    "google-chat-gemini-pro-2.5",
    "openai-o4-mini",
    "openai-o3-mini",
    "openai-o3",
    "openai-o1",
    "openai-gpt-4o",
    "anthropic-claude-3.5-sonnet",
    "openai-gpt4.1"
]
```

### 3. 启动服务

#### 方式一：Docker 部署（推荐）

使用 .env 文件配置：

```bash
# 确保已创建并配置 .env 文件
docker-compose up -d
```

使用传统配置文件：

```bash
# 确保已创建 jetbrainsai.json 和 client_api_keys.json 文件
docker-compose up -d
```

#### 方式二：本地运行

使用 .env 文件配置：

```bash
pip install -r requirements.txt
# 确保已创建并配置 .env 文件
python main.py
```

使用传统配置文件：

```bash
pip install -r requirements.txt
# 确保已创建配置文件
uvicorn main:app --host 0.0.0.0 --port 8000
```

### 4. 验证服务

默认端口 8000：

```bash
curl -H "Authorization: Bearer sk-client-key-1" http://localhost:8000/v1/models
```

自定义端口（如果在 .env 中设置了 PORT=9000）：

```bash
curl -H "Authorization: Bearer sk-client-key-1" http://localhost:9000/v1/models
```

## 🔌 API 接口

### 聊天完成

```http
POST /v1/chat/completions
Authorization: Bearer <client-api-key>
Content-Type: application/json
```

**请求示例：**

```json
{
  "model": "anthropic-claude-3.5-sonnet",
  "messages": [
    {"role": "user", "content": "你好"}
  ],
  "stream": true
}
```

### 模型列表

```http
GET /v1/models
Authorization: Bearer <client-api-key>
```

## 💻 使用示例

### Python + OpenAI SDK

```python
import openai

client = openai.OpenAI(
    api_key="sk-client-key-1",
    base_url="http://localhost:8000/v1"
)

# 流式对话
response = client.chat.completions.create(
    model="anthropic-claude-3.5-sonnet",
    messages=[{"role": "user", "content": "写一首关于春天的诗"}],
    stream=True
)

for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### cURL

```bash
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Authorization: Bearer sk-client-key-1" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "anthropic-claude-3.5-sonnet",
    "messages": [{"role": "user", "content": "你好"}],
    "stream": true
  }'
```

## 📁 项目结构

```text
jetbrainsai2api/
├── main.py              # 主程序（异步服务器 + API 适配器）
├── requirements.txt     # Python 依赖
├── Dockerfile          # Docker 构建文件
├── docker-compose.yml  # Docker Compose 配置
├── .env.example        # 环境变量配置模板
├── .env                # 环境变量配置文件（需要创建）
├── jetbrainsai.json     # JetBrains AI JWT 配置（可选，传统方式）
├── client_api_keys.json # 客户端 API 密钥配置（可选，传统方式）
└── models.json         # 可用模型配置（可选）
```

## 🆕 新特性说明

### 环境变量配置支持

- **推荐使用 .env 文件**：更安全、更方便的配置方式
- **向后兼容**：仍支持传统的 JSON 配置文件
- **灵活端口配置**：可通过 PORT 环境变量自定义服务端口
- **多密钥支持**：支持逗号分隔的多个 API 密钥和 JWT 令牌

---

<div align="center">

**如果这个项目对您有帮助，请考虑给个 ⭐ Star！**

[![Star History Chart](https://api.star-history.com/svg?repos=oDaiSuno/jetbrainsai2api&type=Date)](https://www.star-history.com/#oDaiSuno/jetbrainsai2api&Date)
</div>
