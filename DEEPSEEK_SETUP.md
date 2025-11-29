# 方舟 DeepSeek v3.2 配置指南

## 概述
项目已修改为支持火山引擎（方舟）的 DeepSeek v3.2 模型。以下是配置步骤。

## 1. 获取 API 凭证

### 步骤 1：在方舟控制台获取信息
1. 访问 https://console.volcengine.com/
2. 进入 **在线推理** → **接入点列表**
3. 获取以下信息：
   - **API_KEY**: 你的 API Key
   - **接入点 ID**: 格式如 `ep-m-20251122165113-n8vmc`（注意：不能使用模型名称）

### 步骤 2：配置 credentials.json

修改项目根目录下的 `credentials.json` 文件：

```json
{
    "API_KEY": "your-volc-api-key-here",
    "BASE_URL": "https://ark.cn-beijing.volces.com/api/v3",
    "MODEL": "ep-m-20251122165113-n8vmc"
}
```

**参数说明：**
- `API_KEY`: 从方舟控制台获取的 API Key
- `BASE_URL`: 火山引擎方舟 API 的固定地址（区域：北京）
- `MODEL`: 你的接入点 ID（从控制台获取）

> ⚠️ **重要**：MODEL 字段必须填写接入点 ID，不能填写 "deepseek-v3.2" 这样的模型名称。

## 2. 代码变更说明

### 自动识别逻辑

修改后的 `app.py` 会自动识别使用的 LLM：

```python
if "volces.com" in BASE_URL.lower() or API_KEY.startswith("sk-"):
    # 使用 OpenAI 兼容客户端（支持方舟、OpenRouter 等）
    client = AsyncOpenAI(...)
    USE_GEMINI = False
else:
    # 使用 Google Gemini
    gemini_client = genai.Client()
    USE_GEMINI = True
```

### 方舟 API 特殊参数

为方舟 API 添加了特殊参数支持：

```python
if "volces.com" in BASE_URL.lower():
    request_params["top_p"] = 0.9
```

## 3. 运行项目

```bash
# 1. 安装依赖
pip install -r requirements.txt

# 2. 启动应用
python start_fogsight.py

# 应用将自动在浏览器打开 http://127.0.0.1:8000
```

## 4. 测试

在页面中输入一个主题（如"冒泡排序"），点击生成按钮，应该能看到 DeepSeek 模型生成的动画代码。

## 5. 常见问题

### Q: 接入点 ID 应该填在哪里？
A: 填在 `credentials.json` 的 `MODEL` 字段，格式为 `ep-xxxxxx-xxxxx`。

### Q: 如何切换到其他模型？

**切换到 OpenAI GPT-4o：**
```json
{
    "API_KEY": "sk-your-openai-key",
    "BASE_URL": "https://api.openai.com/v1",
    "MODEL": "gpt-4o"
}
```

**切换到 Claude（通过 OpenRouter）：**
```json
{
    "API_KEY": "sk-or-v1-your-openrouter-key",
    "BASE_URL": "https://openrouter.ai/api/v1",
    "MODEL": "anthropic/claude-3.5-sonnet"
}
```

### Q: 报错 "API_KEY not configured"？
A: 检查 `credentials.json` 文件是否存在，且 API_KEY 不能是 "sk-REPLACE_ME"。

### Q: 生成速度很慢？
A: 
1. 检查网络连接
2. 确认 API Key 和接入点 ID 正确
3. 可尝试调整 `top_p` 和 `temperature` 参数

## 6. 支持的模型

修改后的项目支持以下所有 OpenAI 兼容的 API：

- ✅ 火山引擎方舟（DeepSeek v3.2 等）
- ✅ OpenAI（GPT-4o, GPT-4-turbo 等）
- ✅ OpenRouter（Claude, Llama, Mistral 等）
- ✅ 其他 OpenAI 兼容的 API
- ✅ Google Gemini（配置 GEMINI_API_KEY）

## 7. 更多信息

- [火山引擎方舟文档](https://www.volcengine.com/docs/82379)
- [OpenAI 兼容 API](https://platform.openai.com/docs/api-reference/chat/create)
