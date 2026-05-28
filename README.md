# 🖼️ 多模态识图 Skill 模板

> 一个通用的多模态视觉识别技能模板，适用于 **Reasonix Code**，开箱即用。

## 📖 简介

本项目提供了一个通用化的 **Reasonix Code Skill 模板**，让你的 AI 助手具备图片识别能力——支持本地图片路径和网络图片 URL，自动完成图片编码、API 调用、结果解析的全流程。

### 核心能力

- ✅ **本地图片识别** — 自动 base64 编码并发送
- ✅ **网络图片识别** — 直接传 URL，无需下载
- ✅ **多模型回退** — 按固定顺序依次重试，逻辑确定可调试
- ✅ **智能错误处理** — 区分超时/鉴权/格式错误/模型不可用
- ✅ **Token 用量统计** — 每次返回带字段说明的完整消耗报告
- ✅ **图片大小预检** — 超过 10MB 自动拦截
- ✅ **格式验证** — 支持 jpg/png/webp/bmp 等常见格式
- ✅ **路径自动规范化** — `C:\path` 自动处理反斜杠，避免 JSON 转义坑
- ✅ **前置环境检查** — 执行前确认 Python 环境可用
- ✅ **完全自定义** — 只需替换 API 地址和密钥即可接入任意 OpenAI 兼容的视觉模型

## 🚀 快速开始

### 1. 安装到 Reasonix

将 `vision-skill-template.md` 的内容保存到：

```bash
# 全局安装（所有项目可用）
~/.reasonix/skills/vision.md

# 或项目级安装（仅当前项目）
.reasonix/skills/vision.md
```

### 2. 配置 API

编辑文件顶部的配置参数表，将占位符替换为你的实际信息：

| 参数 | 替换为 |
| --- | --- |
| `https://<your-api-endpoint>/v1/chat/completions` | 你的 API 地址 |
| `<your-api-key>` | 你的 API 密钥 |
| `<model-name>` | 使用的模型名称 |

### 3. 调用识图

```bash
# 识别本地图片
run_skill({ name: "vision", arguments: "C:\\Users\\xxx\\photo.jpg | 这张图里有什么？" })

# 识别网络图片
run_skill({ name: "vision", arguments: "https://example.com/photo.png | 描述一下" })

# 仅传图片（使用默认问题）
run_skill({ name: "vision", arguments: "C:\\Users\\xxx\\photo.jpg" })
```

## 🔧 支持的模型

本模板兼容所有 OpenAI 接口协议的视觉模型，包括：

| 模型 | 提供商 | 备注 |
| --- | --- | --- |
| GPT-4o / GPT-4o-mini | OpenAI | 最流行的视觉模型 |
| Gemini Pro Vision | Google | 免费额度可观 |
| Claude 3.5 Sonnet | Anthropic | 需要 Anthropic 兼容端点 |
| Qwen-VL | 阿里通义 | 国内可用 |
| MiMo 系列 | 小米 | mimo-v2.5 / mimo-v2-omni |
| 其他 | 任意 | 只要是 OpenAI 兼容格式均可 |

## 📂 文件结构

```
vision-skill-template/
├── vision-skill-template.md   ← 技能模板文件（Reasonix Skill）
└── README.md                  ← 本文件
```

## ⚙️ 进阶配置

### 修改 Temperature

- **稳定输出（推荐）**：`temperature: 0.3`
- **创意输出**：`temperature: 0.8`

### 调整最大 Token

```json
"max_tokens": 3000  ← 描述长度限制，越大输出越完整
```

### 添加更多回退模型

在错误处理表中按优先级添加更多模型名即可。

## ❓ 常见问题

**Q: 为什么提示 "图片格式不支持"？**
A: 目前支持 jpg/png/webp/bmp 格式。TIFF/HEIC 等格式请先转换。

**Q: 为什么请求超时？**
A: 大图（>10MB）或网络图片源站慢可能导致超时。建议小图优先用本地路径。

**Q: 怎么知道消耗了多少 Token？**
A: 每次返回结果末尾会显示 Token 消耗统计。

## 📄 许可证

MIT License — 随意使用、修改、分发。
