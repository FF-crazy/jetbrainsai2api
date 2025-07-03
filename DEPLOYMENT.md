# 部署指南

## 快速部署（推荐）

### 1. 使用 .env 文件配置

这是最简单和推荐的部署方式：

```bash
# 1. 克隆项目
git clone https://github.com/oDaiSuno/jetbrainsai2api.git
cd jetbrainsai2api

# 2. 创建配置文件
cp .env.example .env

# 3. 编辑 .env 文件，填入您的配置
nano .env  # 或使用其他编辑器
```

**编辑 .env 文件示例：**
```env
# 服务端口配置
PORT=8000

# 客户端 API 密钥配置 (用逗号分隔多个密钥)
CLIENT_API_KEYS=sk-my-custom-key-1,sk-my-custom-key-2

# JetBrains AI JWT 令牌配置 (用逗号分隔多个JWT)
JETBRAINS_JWTS=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...,eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...

# 调试模式 (可选)
DEBUG_MODE=false

# 可用模型配置 (可选，默认使用内置模型列表)
# AVAILABLE_MODELS=anthropic-claude-3.5-sonnet,anthropic-claude-4-sonnet
```

```bash
# 4. 启动服务
docker-compose up -d

# 5. 验证服务
curl -H "Authorization: Bearer sk-my-custom-key-1" http://localhost:8000/v1/models
```

### 2. 自定义端口部署

如果需要使用不同的端口：

```bash
# 在 .env 文件中设置
PORT=9000

# 启动服务
docker-compose up -d

# 验证服务
curl -H "Authorization: Bearer sk-my-custom-key-1" http://localhost:9000/v1/models
```

## 传统配置文件部署

如果您更喜欢使用 JSON 配置文件：

```bash
# 1. 创建配置文件
echo '[{"jwt": "your-jwt-here"}]' > jetbrainsai.json
echo '["sk-your-key-here"]' > client_api_keys.json

# 2. 启动服务
docker-compose up -d
```

## 本地开发部署

```bash
# 1. 安装依赖
pip install -r requirements.txt

# 2. 使用 .env 文件配置并启动
python main.py

# 或者使用 uvicorn 启动
uvicorn main:app --host 0.0.0.0 --port 8000
```

## 生产环境部署建议

### 1. 使用专用的 docker-compose 文件

```bash
# 使用专门为环境变量优化的配置
docker-compose -f docker-compose.env.yml up -d
```

### 2. 安全配置

- 确保 `.env` 文件权限设置为 600
- 不要将 `.env` 文件提交到版本控制系统
- 定期轮换 JWT 令牌和 API 密钥

```bash
# 设置文件权限
chmod 600 .env

# 添加到 .gitignore
echo ".env" >> .gitignore
```

### 3. 监控和日志

```bash
# 查看服务日志
docker-compose logs -f jetbrainsai2api

# 查看服务状态
docker-compose ps
```

## 故障排除

### 常见问题

1. **端口冲突**
   ```bash
   # 修改 .env 文件中的 PORT 值
   PORT=9000
   ```

2. **JWT 令牌无效**
   - 检查 JWT 令牌是否正确
   - 确保 JWT 令牌没有过期
   - 重新获取 JWT 令牌

3. **配置文件未找到**
   - 确保 .env 文件存在且格式正确
   - 检查文件权限

### 调试模式

启用调试模式获取更多日志信息：

```env
DEBUG_MODE=true
```

## 更新部署

```bash
# 1. 停止服务
docker-compose down

# 2. 拉取最新代码
git pull

# 3. 重新构建并启动
docker-compose up -d --build
```
