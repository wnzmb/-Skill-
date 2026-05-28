---
name: vision
description: 使用 [你的多模态模型] 识别图片内容 — 支持本地图片路径和图片 URL
---
# 多模态识图 Skill 模板

> 这是一个通用模板，将 `<...>` 替换为你自己的配置即可使用。

## 功能
使用多模态大模型 API 识别图片内容，返回图片描述。

## 调用格式
```
run_skill({ name: "vision", arguments: "<图片路径或URL> | [可选问题]" })
```

- `arguments` 中的 ` | `（空格竖线空格）之前的值为图片来源，之后为可选的问题
- 图片来源可以是**本地路径**或**网络 URL**
- 问题默认为 "请详细描述这张图片中的内容"
- **备用格式**：如果问题中天然含 ` | `，可用 JSON 格式传入：
  ```json
  {"image": "D:\\pic.jpg", "question": "A | B 的含义"}
  ```

## 配置参数（按需修改）

| 参数 | 值 | 说明 |
| --- | --- | --- |
| API 地址 | `https://<your-api-endpoint>/v1/chat/completions` | OpenAI 兼容接口 |
| API Key | `<your-api-key>` | 替换为你的密钥 |
| 首选模型 | `<model-name>` | 如 `mimo-v2.5`、`gpt-4o` |
| 备选模型（按顺序） | `<fallback-1>` → `<fallback-2>` | 首选失败后依次尝试 |
| Temperature | `0.3` | 稳定输出用 0.3，创意输出用 0.8 |
| 最大 Token | `3000` | 根据描述长度需求调整 |
| 支持图片格式 | `.jpg`, `.jpeg`, `.png`, `.webp`, `.bmp` | 多数 API 支持这些格式 |

## 执行步骤

### 第〇步：前置环境检查
在执行前确保 Python 环境可用：
```bash
python --version
```
如果失败则尝试：
```bash
python3 --version
```
两者都不可用时，提示用户安装 Python 并终止。这是所有后续步骤的基础依赖。

### 第一步：解析参数
从 `arguments` 中分离图片来源和问题。解析优先顺序：
1. **JSON 格式**：如果参数以 `{` 开头，尝试 JSON 解析，读取 `image` 字段和 `question` 字段
2. **竖线分隔**：如果包含 ` | `，`|` 前为图片来源，后为问题
3. **兜底**：整个字符串为图片来源，问题使用默认值

**路径规范化**：提取到图片来源路径后，自动处理路径中的反斜杠 —— 如果路径中包含单反斜杠 `\`，将其转换为双反斜杠 `\\`，确保后续拼接 JSON 字符串时不会因转义问题导致解析失败。例如 `C:\Users\pic.jpg` → `C:\\Users\\pic.jpg`。

### 第二步：检查图片格式（仅本地路径）
如果是本地路径，先检查文件扩展名。支持的格式：`.jpg`, `.jpeg`, `.png`, `.webp`, `.bmp`。
如果文件扩展名不在支持列表中，提示用户转换格式并终止处理。

### 第三步：预检图片大小（仅本地路径）
使用 Python 检查文件大小（比 PowerShell 更健壮，路径含单引号/方括号也不受影响）：
```bash
python -c "import os; size=os.path.getsize(r'<图片路径>')/1048576; print(f'{size:.1f}')"
```
超过 **10MB** 则提示用户压缩并终止处理。

### 第四步：获取图片内容
- **如果是 URL**：直接记录 URL，跳过 base64 编码
- **如果是本地路径**：使用 Python 获取纯净 base64：
  ```bash
  python -c "import base64,sys; print(base64.b64encode(open(sys.argv[1],'rb').read()).decode())" <图片路径>
  ```
  如果 `python` 不可用，回退到 `python3`。

### 第五步：调用 API
按固定顺序依次重试：
1. **首选模型**（如 `mimo-v2.5`）
2. **备选模型 1**（如 `mimo-v2-omni`）
3. **备选模型 2**（如需扩展）

请求体格式（OpenAI 兼容）：
```json
{
  "model": "<当前尝试的模型名>",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "<用户问题>"},
        {
          "type": "image_url",
          "image_url": {
            "url": "<图片URL或data:image/xxx;base64,xxx>"
          }
        }
      ]
    }
  ],
  "max_tokens": 3000,
  "temperature": 0.3
}
```

curl 命令示例（带超时）：
```bash
curl -s --max-time 115 <API地址> \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <API Key>" \
  -d '<JSON 请求体>'
```

### 第六步：解析响应
从返回的 JSON 中提取：

| 字段 | 含义 | 说明 |
| --- | --- | --- |
| `choices[0].message.content` | 图片描述文本 | 模型生成的描述结果 |
| `usage.prompt_tokens` | 提示词 Token 消耗 | 你输入的图片编码 + 问题文本消耗的 Token 数，是**成本大头** |
| `usage.completion_tokens` | 生成 Token 消耗 | 模型输出描述消耗的 Token 数 |
| `usage.total_tokens` | 总计 Token 消耗 | `prompt_tokens` + `completion_tokens` 的总和 |

> 💡 **理解成本**：`prompt_tokens` 中图片部分占绝对大头（一张图片 ≈ 数百到数千 Token），问题文本的消耗可忽略。如果你的 API 按 Token 计费，尽量用压缩后的小图。

### 第七步：返回结果
```
<描述内容>

（Token 消耗：提示 <N> / 生成 <N> / 总计 <N>）

💡 提示词 Token 是成本大头（图片编码），生成 Token 是输出长度。
```

## 错误处理与回退逻辑

| 错误类型 | 判断方式 | 处理方式 |
| --- | --- | --- |
| 超时 | HTTP 408 或 `TimeoutError` | 提示"请求超时，图片可能过大"，重试当前模型一次 |
| 认证失败 | HTTP 401 | 提示"API Key 无效"，**不重试** |
| 模型不可用 | HTTP 404 含 "not found"/"endpoints" | 切换到下一个备选模型 |
| 图片格式错误 | HTTP 400 含 "image"/"format" | 提示"图片格式不支持"，**不重试** |
| 其他错误 | 其他 4xx/5xx 或 JSON 解析失败 | 返回原始错误信息以便调试 |

回退顺序（严格按列表依次执行，不跳过）：
1. **首选模型** — 第一尝试
2. **备选模型 1** — 首选失败时尝试
3. **备选模型 2**（可选） — 备选1也失败时尝试
4. **全部失败** → 返回格式化错误信息

## 注意事项
- 设置 `timeoutSec: 120`，curl 加 `--max-time 115`
- API Key 不要在日志/回显中暴露
- 网络大图（>5MB）建议优先使用本地路径
- **Python 环境**：执行前先确认 `python` 或 `python3` 可用
- **路径反斜杠**：`C:\path` 格式的路径在 JSON 中会自动标准化为 `C:\\path`
- 不支持批量多张图片
- 如果问题中天然含 ` | `，改用 JSON 格式传入
