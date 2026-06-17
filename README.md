<div align="right">

[English](#english) | **[中文](#中文)**

</div>

---

# yescan-ocr-qoder

[![ClawHub Downloads](https://img.shields.io/badge/ClawHub-1.9k%20downloads-blue?logo=data:image/svg%2Bxml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0id2hpdGUiPjxwYXRoIGQ9Ik0xMiAyQzYuNDggMiAyIDYuNDggMiAxMnM0LjQ4IDEwIDEwIDEwIDEwLTQuNDggMTAtMTBTMTcuNTIgMiAxMiAyem0tMiAxNWwtNS01IDEuNDEtMS40MUwxMCAxNC4xN2w3LjU5LTcuNTlMMTkgOGwtOSA5eiIvPjwvc3ZnPg==)](https://clawhub.ai/mozhihuidage/yescan-ocr-universal)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![QoderWork](https://img.shields.io/badge/QoderWork-Agent%20Skill-orange)](https://qoder.com)

由**夸克扫描王**提供的通用 OCR 技能，支持从图片、截图、照片或扫描文档中提取、识别和结构化文本。

本技能是 [QoderWork](https://qoder.com) Agent Skill，可在 QoderWork 桌面端安装并通过自然语言调用。

> 📦 ClawHub 主页：[yescan-ocr-universal](https://clawhub.ai/yescan-ai/yescan-ocr-universal)

---

## 功能特性

- **通用文档识别** — 印刷体、手写体混排内容识别
- **表格提取** — 复杂表格、合并单元格、跨页表格还原为结构化数据
- **数学公式识别** — LaTeX 格式输出，支持行内/行间公式
- **证件识别** — 身份证、社保卡、驾照、行驶证、港澳台通行证、学位证等
- **票据识别** — 增值税发票、火车票、英文发票（商业发票）等
- **医疗报告** — 化验单、体检报告、医疗报告单
- **商品图识别** — 商品信息、产品包装、标签
- **题目识别** — 考题、习题、手写作文

## 支持的识别场景（scene）

共 17 个场景，Agent 按下表顺序匹配，命中第一个即停止。

| # | 场景名称 | Scene 标识 | 关键词 |
|---|---|---|---|
| 1 | 手写文档识别 | `handwritten-ocr` | 手写、笔迹、潦草字迹、手写作文 |
| 2 | 表格识别 | `table-ocr` | 表格、Excel、报表、单据 |
| 3 | 身份证识别 | `idcard-ocr` | 身份证、居民身份证 |
| 4 | 社保卡识别 | `social-security-card-ocr` | 社保卡、医保卡 |
| 5 | 港澳通行证识别 | `travel-permit-ocr` | 港澳通行证、港澳台通行证 |
| 6 | 学位证识别 | `degree-certificate-ocr` | 学位证、学历证书 |
| 7 | 增值税发票识别 | `vat-invoice-ocr` | 增值税发票、发票 |
| 8 | 火车票识别 | `train-ticket-ocr` | 火车票、车票 |
| 9 | 公式识别 | `formula-ocr` | 数学公式、LaTeX、化学方程式 |
| 10 | 题目识别 | `question-ocr` | 题目、考题、习题 |
| 11 | 驾驶证识别 | `driver-license-ocr` | 驾驶证、驾照 |
| 12 | 行驶证识别 | `vehicle-license-ocr` | 行驶证、车辆 |
| 13 | 英文发票识别 | `commercial-invoice-ocr` | 英文发票、商业发票 |
| 14 | 医疗报告单识别 | `medical-report-ocr` | 化验单、体检报告、医疗报告 |
| 15 | 营业执照识别 | `business-license-ocr` | 营业执照、工商执照 |
| 16 | 商品图片识别 | `product-image-ocr` | 商品、产品、包装、标签 |
| 17 | 通用文字提取 | `general-ocr` | 提取文字、识别文字（兜底） |

## 安装前提

### 1. 获取密钥

本技能需要**夸克扫描王开放平台**的 API Key（AI Agent 接入类型）。

前往 [scan.quark.cn](https://scan.quark.cn/business) 开发者后台：

1. 注册 / 登录（手机号）
2. 进入「业务中心」→「密钥管理」
3. 创建 Key，选择 **AI Agent 接入**（Key ID 前缀为 `AI_`）
4. 复制 Key 值

### 2. 配置密钥（推荐写入配置文件）

将 Key 写入 `~/.yescan_env`，技能每次执行会自动读取，无需重启会话：

**Linux / macOS**
```bash
echo 'SCAN_WEBSERVICE_KEY=<your_api_key_here>' > ~/.yescan_env
```

**Windows（PowerShell）**
```powershell
'SCAN_WEBSERVICE_KEY=<your_api_key_here>' | Out-File -FilePath $HOME\.yescan_env -Encoding utf8
```

> 也可以使用 QoderWork 内置的 `yescan-key-setup` 技能自动完成注册和配置。

## 使用方式

安装本技能后，在 QoderWork 中直接用自然语言调用即可：

```
帮我识别这张图片里的文字
把这张发票识别成结构化的 JSON
识别这张手写的数学公式
这张火车票的出发时间和车次是什么
```

Agent 会根据意图自动匹配对应的 scene 并调用底层脚本。

### 底层命令行接口

```bash
python3 scripts/scan.py --scene "${SCENE}" --url "https://example.com/image.jpg"
python3 scripts/scan.py --scene "${SCENE}" --path "/local/path/to/image.png"
python3 scripts/scan.py --scene "${SCENE}" --base64 "<base64 编码的图片数据>"
```

支持的输入方式：URL、本地文件路径、Base64 编码。

## 输出格式

脚本执行成功后，标准输出返回 JSON 对象，包含识别文本、位置坐标、置信度等结构化字段。Agent 会自动解析并呈现给用户。

## 约束与限制

- 单张图片大小不超过 **5 MB**
- 支持的图片格式：`jpg / jpeg / png / gif / bmp / webp / tiff / wbmp`
- 每次调用处理单张图片，不支持批量
- 图片数据会发送至 `scan-business.quark.cn` 进行处理，不会永久保存
- 临时文件保存至 `/tmp`，任务结束后自动清理

## 目录结构

```
yescan-ocr-qoder/
├── SKILL.md                          # 技能定义（QoderWork Agent 读取）
├── scripts/
│   ├── scan.py                       # 主执行脚本（Python 3.9+）
│   └── common/                       # 公共工具模块
├── references/
│   ├── scenarios-detail.md           # 场景详细说明
│   ├── error-codes.md                # 错误码对照表
│   └── resources.md                  # 官方链接和资源
├── evals/                            # 评测用例
└── LICENSE                           # MIT License
```

## 相关链接

- [夸克扫描王开放平台](https://scan.quark.cn/business) — 密钥申请、API 文档
- [ClawHub 技能页](https://clawhub.ai/mozhihuidage/yescan-ocr-universal) — 安装、下载量、评分
- [QoderWork](https://qoder.com) — Agent 运行环境
- [yescan-ai GitHub Org](https://github.com/yescan-ai) — 更多技能

## 许可证

[MIT License](LICENSE)

---

<a id="english"></a>

# yescan-ocr-qoder

[![ClawHub Downloads](https://img.shields.io/badge/ClawHub-1.9k%20downloads-blue?logo=data:image/svg%2Bxml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0id2hpdGUiPjxwYXRoIGQ9Ik0xMiAyQzYuNDggMiAyIDYuNDggMiAxMnM0LjQ4IDEwIDEwIDEwIDEwLTQuNDggMTAtMTBTMTcuNTIgMiAxMiAyem0tMiAxNWwtNS01IDEuNDEtMS40MUwxMCAxNC4xN2w3LjU5LTcuNTlMMTkgOGwtOSA5eiIvPjwvc3ZnPg==)](https://clawhub.ai/mozhihuidage/yescan-ocr-universal)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![QoderWork](https://img.shields.io/badge/QoderWork-Agent%20Skill-orange)](https://qoder.com)

A universal OCR skill powered by **Quark Scan King**. Extracts, recognizes, and structures text from images, screenshots, photos, and scanned documents.

This is a [QoderWork](https://qoder.com) Agent Skill — install it in the QoderWork desktop app and invoke it with natural language.

> 📦 ClawHub page: [mozhihuidage/yescan-ocr-universal](https://clawhub.ai/mozhihuidage/yescan-ocr-universal)

---

## Features

- **General Document Recognition** — Printed and handwritten text, including mixed layouts
- **Table Extraction** — Complex tables with merged cells and cross-page tables, output as structured data
- **Math Formula Recognition** — LaTeX output, supporting inline and display formulas
- **ID Card Recognition** — National ID cards, social security cards, driver's licenses, vehicle registration, travel permits, degree certificates, and more
- **Receipt & Invoice OCR** — VAT invoices, train tickets, commercial invoices, etc.
- **Medical Reports** — Lab reports, physical examination reports, medical records
- **Product Image OCR** — Product information, packaging, labels
- **Question Recognition** — Exam questions, exercises, handwritten essays

## Supported Scenes

17 scenes in total. The agent matches in the order listed below and stops at the first hit.

| # | Scene Name | Scene ID | Keywords |
|---|---|---|---|
| 1 | Handwritten OCR | `handwritten-ocr` | Handwriting, cursive, essays |
| 2 | Table OCR | `table-ocr` | Tables, Excel, reports, receipts |
| 3 | ID Card OCR | `idcard-ocr` | National ID card |
| 4 | Social Security Card OCR | `social-security-card-ocr` | Social security, medical insurance |
| 5 | Travel Permit OCR | `travel-permit-ocr` | HK/Macau/TW travel permit |
| 6 | Degree Certificate OCR | `degree-certificate-ocr` | Degree, diploma, academic certificate |
| 7 | VAT Invoice OCR | `vat-invoice-ocr` | VAT invoice, tax invoice |
| 8 | Train Ticket OCR | `train-ticket-ocr` | Train ticket, railway ticket |
| 9 | Formula OCR | `formula-ocr` | Math formula, LaTeX, chemical equation |
| 10 | Question OCR | `question-ocr` | Exam question, exercise, problem set |
| 11 | Driver's License OCR | `driver-license-ocr` | Driver's license |
| 12 | Vehicle License OCR | `vehicle-license-ocr` | Vehicle registration, license plate |
| 13 | Commercial Invoice OCR | `commercial-invoice-ocr` | Commercial invoice, English invoice |
| 14 | Medical Report OCR | `medical-report-ocr` | Lab report, health checkup, medical report |
| 15 | Business License OCR | `business-license-ocr` | Business license, registration certificate |
| 16 | Product Image OCR | `product-image-ocr` | Product, packaging, label |
| 17 | General Text Extraction | `general-ocr` | Extract text, recognize text (fallback) |

## Prerequisites

### 1. Get an API Key

This skill requires an API Key (AI Agent type) from the **Quark Scan King Open Platform**.

Visit the [scan.quark.cn](https://scan.quark.cn/business) developer console:

1. Register / log in (phone number)
2. Go to **Business Center** → **Key Management**
3. Create a key, select **AI Agent** (Key ID prefix: `AI_`)
4. Copy the key value

### 2. Configure the Key (Recommended: Config File)

Write the key to `~/.yescan_env`. The skill reads it automatically on each run — no session restart required.

**Linux / macOS**
```bash
echo 'SCAN_WEBSERVICE_KEY=<your_api_key_here>' > ~/.yescan_env
```

**Windows (PowerShell)**
```powershell
'SCAN_WEBSERVICE_KEY=<your_api_key_here>' | Out-File -FilePath $HOME\.yescan_env -Encoding utf8
```

> Alternatively, use the built-in `yescan-key-setup` skill in QoderWork for automated registration and configuration.

## Usage

After installing the skill, invoke it with natural language in QoderWork:

```
Extract the text from this image
Recognize this invoice as structured JSON
Identify the handwritten math formula in this photo
What is the departure time and train number on this ticket?
```

The agent will automatically match the appropriate scene and invoke the underlying script.

### CLI Interface

```bash
python3 scripts/scan.py --scene "${SCENE}" --url "https://example.com/image.jpg"
python3 scripts/scan.py --scene "${SCENE}" --path "/local/path/to/image.png"
python3 scripts/scan.py --scene "${SCENE}" --base64 "<base64-encoded image data>"
```

Supported input methods: URL, local file path, Base64 encoding.

## Output Format

On success, the script outputs a JSON object to stdout containing recognized text, bounding box coordinates, confidence scores, and other structured fields. The agent automatically parses and presents the result to the user.

## Constraints & Limitations

- Maximum image size: **5 MB**
- Supported formats: `jpg / jpeg / png / gif / bmp / webp / tiff / wbmp`
- Single image per call — batch processing is not supported
- Image data is sent to `scan-business.quark.cn` for processing and is not permanently stored
- Temporary files are saved to `/tmp` and cleaned up after the task completes

## Repository Structure

```
yescan-ocr-qoder/
├── SKILL.md                          # Skill definition (read by QoderWork Agent)
├── scripts/
│   ├── scan.py                       # Main execution script (Python 3.9+)
│   └── common/                       # Shared utility modules
├── references/
│   ├── scenarios-detail.md           # Detailed scene descriptions
│   ├── error-codes.md                # Error code reference
│   └── resources.md                  # Official links and resources
├── evals/                            # Evaluation test cases
└── LICENSE                           # MIT License
```

## Links

- [Quark Scan King Open Platform](https://scan.quark.cn/business) — API key & documentation
- [ClawHub Skill Page](https://clawhub.ai/mozhihuidage/yescan-ocr-universal) — Install, downloads & ratings
- [QoderWork](https://qoder.com) — Agent runtime environment
- [yescan-ai GitHub Org](https://github.com/yescan-ai) — More skills

## License

[MIT License](LICENSE)
