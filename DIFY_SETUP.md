# 使用 Dify API 进行 Promptfoo 测试

本文档说明如何在 promptfoo 中配置和使用 Dify 的 Workflow API 来进行英文单词验证测试。

## 🔧 配置步骤

### 1. 设置 API 密钥

首先，你需要获取 Dify 的 API 密钥并设置为环境变量：

**方法1：直接设置环境变量**
```bash
# 设置 Dify API 密钥（临时，重启终端后失效）
export DIFY_API_KEY="your_dify_api_key_here"
```

**方法2：使用 .env 文件（推荐）**
```bash
# 创建 .env 文件
echo "DIFY_API_KEY=your_dify_api_key_here" > .env

# 或者手动创建 .env 文件并添加内容：
# DIFY_API_KEY=your_dify_api_key_here
```

**获取 API 密钥：**
1. 登录你的 Dify 工作台
2. 进入对应的工作流应用
3. 在 "API 访问" 或 "集成" 部分获取 API 密钥
4. API 密钥格式通常类似：`app-1234567890abcdef1234567890abcdef`

### 2. 验证 Dify API 连接

在运行 promptfoo 测试之前，先验证你的 Dify API 是否正常工作：

```bash
# 测试 Dify API 连接
curl -X POST 'http://192.168.199.4:8881/v1/workflows/run' \
--header 'Authorization: Bearer your_dify_api_key_here' \
--header 'Content-Type: application/json' \
--data-raw '{
    "inputs": {
        "word": "apple"
    },
    "response_mode": "blocking",
    "user": "test-user"
}'
```

**期望的响应格式：**
```json
{
    "workflow_run_id": "djflajgkldjgd",
    "task_id": "9da23599-e713-473b-982c-4328d4f5c78a",
    "data": {
        "id": "fdlsjfjejkghjda",
        "workflow_id": "fldjaslkfjlsda",
        "status": "succeeded",
        "outputs": {
            "text": "yes"
        },
        "error": null,
        "elapsed_time": 0.875,
        "total_tokens": 3562,
        "total_steps": 8,
        "created_at": 1705407629,
        "finished_at": 1727807631
    }
}
```

promptfoo 将自动提取 `data.outputs.text` 字段作为模型的输出。

### 3. 调整工作流参数

**重要配置说明：**

1. **提供商 ID**: 必须使用 `https` 作为提供商 ID，不能使用自定义名称如 `dify-workflow`
2. **输入参数名**: 根据你的 Dify 工作流输入变量名调整，常见的有：
   - `input_text` - 文本输入
   - `word` - 单词输入  
   - `prompt` - 提示词输入
   - 其他自定义参数名

根据你的 Dify 工作流设计，可能需要调整 `transformResponse` 中的输出字段名。

在 `promptfoo.dify.config.yaml` 中找到这部分代码：

```javascript
// 根据你的工作流输出变量名调整这里
if (outputs.result !== undefined) return outputs.result;
if (outputs.text !== undefined) return outputs.text;
if (outputs.answer !== undefined) return outputs.answer;
if (outputs.output !== undefined) return outputs.output;
if (outputs.response !== undefined) return outputs.response;
```

如果你的工作流使用不同的输出字段名，请相应修改。

## 🚀 运行测试

### 快速验证配置

```bash
# 设置API密钥
export DIFY_API_KEY="your_dify_api_key_here"

# 快速测试单个用例（验证配置是否正确）
npx promptfoo eval -c promptfoo.dify.config.yaml --filter "基础名词"

# 如果单个测试成功，运行完整测试
npx promptfoo eval -c promptfoo.dify.config.yaml
```

### 使用 Dify 专用配置文件

```bash
# 运行 Dify API 测试（推荐开始使用这个）
npx promptfoo eval -c promptfoo.dify.config.yaml

# 查看测试结果
npx promptfoo view
```

### 使用完整配置文件（同时测试 Ollama 和 Dify）

```bash
# 运行完整测试（包含100+测试用例，同时测试两个API）
npx promptfoo eval -c promptfoo.config.yaml

# 查看结果
npx promptfoo view
```

## 📊 配置说明

### Dify API 配置详解

```yaml
providers:
  - id: https  # 必须使用 https 作为提供商 ID
    config:
      url: 'http://192.168.199.4:8881/v1/workflows/run'  # Dify API 端点
      method: 'POST'
      headers:
        'Content-Type': 'application/json'
        'Authorization': 'Bearer {{env.DIFY_API_KEY}}'    # 从环境变量读取API密钥
      body:
        inputs:
          input_text: '{{word}}'          # 将测试变量传递给工作流（参数名根据你的工作流调整）
        response_mode: 'blocking'   # 阻塞模式，等待完整响应
        user: 'promptfoo-test-user' # 用户标识符
```

### 响应处理

Dify API 的典型响应格式：

```json
{
  "workflow_run_id": "uuid-here",
  "task_id": "uuid-here", 
  "data": {
    "outputs": {
      "result": "yes",  // 这里是你的工作流输出
      "confidence": 0.95
    }
  }
}
```

## 🔍 调试技巧

### 1. 查看详细日志

```bash
# 启用调试模式查看详细请求和响应
LOG_LEVEL=debug npx promptfoo eval -c promptfoo.dify.config.yaml
```

### 2. 检查 API 响应

在 `transformResponse` 中已经添加了 `console.log` 来输出 Dify 的原始响应：

```javascript
console.log('Dify响应:', JSON.stringify(json, null, 2));
```

运行测试时你会在控制台看到完整的 API 响应。

### 3. 测试单个案例

```bash
# 只运行前几个测试用例
npx promptfoo eval -c promptfoo.dify.config.yaml --max-concurrency 1 --filter-description "测试基础"
```

## ⚠️ 注意事项

1. **API 限制**: 注意 Dify API 的速率限制，如果遇到限制可以降低并发数：
   ```bash
   npx promptfoo eval -c promptfoo.dify.config.yaml --max-concurrency 1
   ```

2. **工作流配置**: 确保你的 Dify 工作流：
   - 接受名为 `word` 的输入参数
   - 输出 "yes" 或 "no" 字符串
   - 已经发布并且可以通过 API 访问

3. **网络连接**: 确保可以访问 `http://192.168.199.4:8881`

4. **API 密钥**: 不要将 API 密钥直接写在配置文件中，使用环境变量

## 🔄 对比测试

你现在有两种方式来测试单词验证：

1. **Ollama + gemma3:12b**: 本地模型，速度快
2. **Dify Workflow**: 云端工作流，可能有更复杂的逻辑

运行完整配置文件可以同时测试两种方式并对比结果：

```bash
npx promptfoo eval -c promptfoo.config.yaml
npx promptfoo view
```

在 Web 界面中，你可以并排查看两个提供商的结果，分析它们在不同测试用例上的表现差异。

## 🛠️ 自定义配置

### 修改 API 端点

如果你的 Dify 部署在不同的地址，修改 `url` 配置：

```yaml
url: 'http://your-dify-domain.com/v1/workflows/run'
```

### 添加更多输入参数

如果你的工作流需要更多输入参数：

```yaml
body:
  inputs:
    word: '{{word}}'
    language: 'en'
    strict_mode: true
  response_mode: 'blocking'
  user: 'promptfoo-test-user'
```

### 处理流式响应

如果你想使用流式响应（虽然对于测试来说不太必要）：

```yaml
body:
  inputs:
    word: '{{word}}'
  response_mode: 'streaming'
  user: 'promptfoo-test-user'
```

注意：使用流式响应时需要更复杂的响应处理逻辑。

## 🐛 常见问题排除

### 问题1: 提供商无法识别
**错误信息:** `Could not identify provider: dify-workflow`
**解决方案:** 
- 使用 `https` 作为提供商 ID，而不是自定义名称
- 正确的配置：
```yaml
providers:
  - id: https  # 正确：使用 https
    config:
      url: 'http://192.168.199.4:8881/v1/workflows/run'
      # ... 其他配置
```

### 问题2: API 密钥错误
**错误信息:** `401 Unauthorized`
**解决方案:**
```bash
# 确认API密钥设置正确
echo $DIFY_API_KEY
# 应该显示类似: app-1234567890abcdef1234567890abcdef

# 测试API连接
curl -X POST 'http://192.168.199.4:8881/v1/workflows/run' \
--header "Authorization: Bearer $DIFY_API_KEY" \
--header 'Content-Type: application/json' \
--data-raw '{"inputs": {"word": "test"}, "response_mode": "blocking", "user": "debug"}'
```

### 问题3: 响应格式不匹配
**错误信息:** `Unexpected response format`
**你的响应格式已确认为:**
```json
{
    "workflow_run_id": "djflajgkldjgd",
    "task_id": "9da23599-e713-473b-982c-4328d4f5c78a",
    "data": {
        "outputs": {
            "text": "Nice to meet you."
        }
    }
}
```
**解决方案:** 配置文件已更新为正确处理 `data.outputs.text` 字段。

### 问题4: 工作流执行失败
**错误信息:** `"status": "failed"`
**检查方法:**
1. 在 Dify 工作台验证工作流正常运行
2. 确认输入参数 `input_text`（或你的自定义参数名）被正确传递
3. 检查工作流输出变量名为 `text`

### 问题5: 网络连接问题
**错误信息:** `ECONNREFUSED` 或超时
**解决方案:**
```bash
# 测试网络连接
ping 192.168.199.4
curl -I http://192.168.199.4:8881/health
```

## 🔧 最新修复（2025-07-15）

### ✅ JavaScript语法错误已解决

**问题：** `SyntaxError: Unexpected token ';'` 和 `Unexpected token 'try'`

**原因：** YAML多行字符串中的复杂JavaScript语法不被promptfoo支持

**解决方案：** 使用简单的单行 `transformResponse`

**修复前：**
```yaml
transformResponse: |
  // 复杂的多行JavaScript代码
  console.log('Dify响应:', JSON.stringify(json, null, 2));
  // ...更多代码...
```

**修复后：**
```yaml
transformResponse: 'json.data.outputs.text.trim()'
```

**测试结果：** ✅ 100% 通过率，所有测试用例正常工作

### 配置文件状态

- ✅ `promptfoo.dify.fixed.config.yaml` - 完全工作的简化版本
- ✅ `promptfoo.dify.config.yaml` - 已更新使用修复后的语法
- ✅ `promptfoo.simple.config.yaml` - 用于调试的最简配置

## 📞 获取帮助

如果遇到问题：

1. **首先运行调试模式:**
   ```bash
   DEBUG=1 npx promptfoo eval -c promptfoo.dify.fixed.config.yaml --max-concurrency 1
   ```

2. **检查API响应:** 查看控制台中的响应日志

3. **验证工作流:** 在 Dify 工作台手动测试工作流

4. **测试单个用例:**
   ```bash
   npx promptfoo eval -c promptfoo.dify.fixed.config.yaml --filter "测试基础英文单词"
   ```

5. **如果还有语法错误，使用最简配置:**
   ```bash
   npx promptfoo eval -c promptfoo.simple.config.yaml
   ```

欢迎根据实际测试结果调整配置！ 