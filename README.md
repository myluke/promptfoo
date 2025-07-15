# 英文单词验证器 - Promptfoo 测试套件

这是一个使用 Promptfoo 框架的英文单词验证器测试项目。该项目通过 AI 模型来判断输入的字符串是否为有效的英文单词、缩写或常见术语。

## 📋 项目概述

本项目包含一个完整的测试套件，用于验证 AI 模型在英文单词识别任务上的表现。测试涵盖：

- ✅ **有效单词**：标准词典单词、专有名词、缩写、连字符单词等
- ❌ **无效输入**：拼写错误、随机字符串、乱码等

测试套件包含 **100+ 个测试用例**，确保模型能够准确区分有效和无效的英文输入。

## 🛠️ 安装步骤

### 1. 安装 Node.js
确保您的系统已安装 Node.js (版本 16 或更高)：
```bash
# 检查 Node.js 版本
node --version
npm --version
```

### 2. 安装 Promptfoo
```bash
# 全局安装 promptfoo
npm install -g promptfoo

# 或者使用 npx（推荐）
npx promptfoo --version
```

### 3. 安装和配置 Ollama
本项目使用 Ollama 作为模型提供商，需要安装并运行 gemma3:12b 模型。

#### 安装 Ollama：
```bash
# macOS
brew install ollama

# 或者访问 https://ollama.ai 下载安装包
```

#### 启动 Ollama 服务：
```bash
# 启动 Ollama 服务
ollama serve
```

#### 下载 gemma3:12b 模型：
```bash
# 下载并安装模型（可能需要一些时间）
ollama pull gemma3:12b
```

#### 验证模型安装：
```bash
# 列出已安装的模型
ollama list

# 测试模型
ollama run gemma3:12b "Hello, world!"
```

## 🚀 快速开始

### 1. 克隆或下载项目
确保您有 `promptfoo.config.yaml` 配置文件。

### 2. 验证配置
检查配置文件中的 Ollama 端点是否正确：
```yaml
providers:
  - id: ollama:chat:gemma3:12b
    config:
      apiBaseUrl: http://localhost:11434  # 确保端口正确
      temperature: 0
```

### 3. 运行测试
```bash
# 在项目目录中运行测试
promptfoo eval
promptfoo eval -c promptfoo.config.yaml

# 或者使用 npx
npx promptfoo eval
npx promptfoo@latest eval -c promptfoo.config.yaml

npx promptfoo eval -c promptfoo.dify.config.yaml --no-cache --max-concurrency 1
```

### 4. 查看结果
```bash
# 启动 Web 界面查看详细结果
promptfoo view

# 或者
npx promptfoo view
```

## 📊 测试用例说明

### 有效单词测试 (应返回 "yes")
- **基础词汇**：apple, running, beautiful, quickly
- **专有名词**：London, Google, iPhone, NASA
- **缩写术语**：PhD, HTML, FBI, DNA, H2O
- **连字符词**：state-of-the-art, mother-in-law
- **专业术语**：algorithm, medicine, architecture

### 无效输入测试 (应返回 "no")
- **拼写错误**：runnning, beautifull, definately
- **随机字符**：asdfghjkl, xyzqwerty, abc123def
- **乱码组合**：hello@#$, mnbvcx, zxcvbnm

## 🔧 配置自定义

### 修改模型设置
在 `promptfoo.config.yaml` 中调整模型参数：
```yaml
providers:
  - id: ollama:chat:gemma3:12b
    config:
      apiBaseUrl: http://localhost:11434
      temperature: 0  # 调整创造性（0-1）
      top_p: 1.0      # 添加其他参数
```

### 添加新测试用例
在配置文件的 `tests` 部分添加新用例：
```yaml
tests:
  - description: "您的测试描述"
    vars:
      word: "测试单词"
    options:
      transform: "output.trim()"
    assert:
      - type: equals
        value: "yes"  # 或 "no"
```

## 📈 结果解释

### 成功的测试
- ✅ **绿色**：模型输出与期望结果匹配
- 显示实际输出和期望输出

### 失败的测试
- ❌ **红色**：模型输出与期望不符
- 显示差异详情，便于调试

### 性能指标
- **准确率**：正确判断的百分比
- **响应时间**：每个测试的执行时间
- **成本**：API 调用成本（如果适用）

## 🔍 常见问题

### Q: Ollama 连接失败
**A:** 确保 Ollama 服务正在运行：
```bash
# 检查服务状态
curl http://localhost:11434/api/tags

# 重启服务
ollama serve
```

### Q: 模型响应格式不正确
**A:** 检查提示词是否清晰，模型是否按要求只输出 "yes" 或 "no"。

### Q: 测试运行缓慢
**A:** 考虑：
- 减少测试用例数量
- 使用更快的模型
- 调整并发设置

## 🎯 下一步

1. **扩展测试用例**：添加更多边缘情况和专业术语
2. **尝试其他模型**：比较不同模型的表现
3. **优化提示词**：改进指令以获得更好的准确率
4. **自动化测试**：集成到 CI/CD 流程中

## 📄 许可证

本项目仅用于学习和测试目的。

## 🤝 贡献

欢迎提交问题和改进建议！

---

**注意**：首次运行可能需要一些时间来下载和初始化模型。请确保网络连接稳定。 