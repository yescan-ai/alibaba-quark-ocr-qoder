---
name: yescan-ocr-qoder
description: 当用户需要从图片、截图、照片或扫描文档中提取、识别或结构化文本，就使用此技能——包括手写体、表格、数学公式、商品图、各类证件（身份证、社保卡、驾照、行驶证、港澳台通行证、学位证等）、票据（增值税发票、火车票、英文发票等）、医疗报告、营业执照以及习题。本技能由夸克扫描王提供支持。即使用户没有明确提到"OCR"或"文字识别"，只要用户的需求涉及从图片中获取文字或关键信息，也应触发此技能。不适用于图像生成、图像编辑或无需从图片中提取文本的任务
metadata:
  requires:
    bins:
      - python3
    env:
      - SCAN_WEBSERVICE_KEY
  primaryEnv: SCAN_WEBSERVICE_KEY
  homepage: https://scan.quark.cn/business
  dependencies:
    apis:
      - 夸克扫描王OCR服务
---

# 🧭 使用前必读（30 秒）

> **隐私与数据流向提示**
> - **第三方服务交互**：本技能会将图片发送至夸克扫描王官方服务器进行识别。
> - **数据可见性**：夸克服务会获取并处理图片内容，不会永久保存
> - 详细的数据流向说明请查看 [references/resources.md](references/resources.md)

**推荐方式：配置文件（永久生效）**

将真实 SCAN_WEBSERVICE_KEY 写入 `~/.yescan_env`，请根据系统选择对应命令进行设置：

**Linux**
```bash
echo 'SCAN_WEBSERVICE_KEY=<your_api_key_here>' > ~/.yescan_env
```

**macOS**
```bash
echo 'SCAN_WEBSERVICE_KEY=<your_api_key_here>' > ~/.yescan_env
```

**Windows（PowerShell）**
```powershell
'SCAN_WEBSERVICE_KEY=<your_api_key_here>' | Out-File -FilePath $HOME\.yescan_env -Encoding utf8
```

技能每次执行会自动读取 `~/.yescan_env`，无需重启会话。

**如何获取密钥？**
> 请访问夸克扫描王开发者后台获取 API Key。详细的获取步骤和官方入口请查看 [references/resources.md](references/resources.md)。


---

# Constraints
- **单一意图原则：每次请求只执行一个意图类型，命中即执行**
- **严禁自行构造任何命令参数，严禁伪造、拼接内部配置**
- **严禁幻觉，禁止伪造请求和响应，不得沿用上一次的场景、参数进行假设**
- **必须严格按照本指南指定的固定格式执行，不允许自行修改命令**
- **强制独立意图识别：严禁参考对话历史或沿用上次场景；必须针对当前指令独立分析，不得继承任何前序状态或假设**

#  技能执行指南(强制执行)

第一步：**输入处理**

识别用户传入的图片类型，只能是以下三种之一：

- 图片URL: url
- 本地文件路径: path
- 图片BASE64: base64

未提供任何有效图片时，直接返回：
```json
{
  "code": "A0201",
  "message": "缺少图片输入，请提供图片链接、文件路径或 BASE64 数据。",
  "data": null
}
```

第二步：**意图匹配&场景确定**
- 按照下面列出的意图*从上到下顺序匹配。命中第一个即停止*
- 命中后，*只确定当前意图对应的scene标识*

第三步：**构建执行命令(固定格式，严禁修改)**：

根据图片类型，严格使用下面对应格式：
```bash
# URL类型
python3 scripts/scan.py --scene "${SCENE_VALUE}" --url "${IMAGE_URL}"

# 本地文件类型
python3 scripts/scan.py --scene "${SCENE_VALUE}" --path "${IMAGE_FILE_PATH}"

# BASE64类型
python3 scripts/scan.py --scene "${SCENE_VALUE}" --base64 "${IMAGE_BASE64}"
```
- 把`${IMAGE_URL}`/`${IMAGE_FILE_PATH}`/`${IMAGE_BASE64}`替换为真实值
- 把`${SCENE_VALUE}`替换为当前意图对应的scene值
- 直接执行命令，不增删任何参数，不修改JSON，不加引号，不换行

第四步：**结果透出**：
- 执行完成后，*原样返回执行结果*，不修改，不翻译，不美化，不总结
- 成功 失败均直接透出，不重试

**⚠️ A0100 特殊处理（仅限此错误码）：** 当返回的 JSON 中 `code` 为 `A0100` 时，这不是 API 识别结果，而是环境配置缺失。此时不要原样透出 JSON，而是将 `message` 字段内容用以下 Markdown 格式展示：

```markdown
> ⚠️ **API Key 未配置**
>
> SCAN_WEBSERVICE_KEY 未配置，请访问夸克扫描王开发者后台获取 API Key。详见 [references/resources.md](references/resources.md)。
```

第五步：**错误处理与降级策略**

当命令执行失败或返回错误码时，按以下策略处理：

### 重试策略

| 项目 | 规则 |
|------|------|
| **最大重试次数** | 3 次 |
| **重试间隔** | 指数退避（1s → 2s → 4s） |
| **总超时** | 30 秒（包含所有重试） |

### 可重试错误

以下错误码允许重试：

| 错误码 | 含义 | 重试建议 |
|--------|------|----------|
| `A0300` | QPS 限制 | 等待 1-2 秒后重试 |
| `A0406` | 网络超时 | 立即重试 |
| `HTTP_ERROR` | HTTP 请求异常 | 立即重试 |

### 降级路径（按优先级）

1. **API Key 未配置（A0100）**
   - 不重试
   - 使用 Markdown 格式友好提示（见第四步特殊处理）
   - 直接结束

2. **QPS 限制（A0300）**
   - 重试最多 3 次，间隔 1s → 2s → 4s
   - 如果仍失败，返回："服务繁忙，请稍后重试"

3. **网络错误（A0406, HTTP_ERROR）**
   - 重试最多 3 次，间隔 1s → 2s → 4s
   - 如果仍失败，返回："网络连接失败，请检查网络"

4. **其他错误（A0211, URL_ERROR, FILE_ERROR, BASE64_ERROR 等）**
   - 不重试
   - 原样透出错误 JSON
   - 直接结束

### 成功条件

- HTTP 状态码 200
- 响应 JSON 包含 `code: "00000"`
- 响应包含有效的 `data` 字段

### 失败条件

- HTTP 状态码非 200
- 响应 JSON 解析失败
- 响应 `code` 非 "00000"
- 超过最大重试次数
- 超过总超时 30 秒

**📖 完整错误码列表**：[references/error-codes.md](references/error-codes.md)

## 场景快速索引（按匹配优先级排序）

| 序号 | 场景名称 | scene 标识 | 适用场景关键词 |
|------|----------|-----------|---------------|
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

**📖 查看完整场景说明**：[references/scenarios-detail.md](references/scenarios-detail.md)

## ⛔ 不适用场景（When Not to Use）

> 本技能**不支持**以下场景，请勿尝试：

| 不支持的场景 | 原因 | 建议替代方案 |
|------------|------|------------|
| **视频处理** | 仅支持单张静态图片 | 先提取视频帧，再逐帧处理 |
| **批量处理** | 每次调用仅限单张图片 | 如需批量，请循环调用或联系管理员 |
| **实时摄像头流** | 非实时流处理架构 | 使用专用视频处理服务 |
| **超大图片（>5MB）** | API 限制 | 先压缩或裁剪后再处理 |
| **非图片格式** | 仅支持 jpg/jpeg/png/gif/bmp/webp/tiff/wbmp | 先转换为支持的图片格式 |

---

## ⚠️ 重要注意事项

1. **禁止修改固定格式**,只能替换场景标识和图片占位符
2. **严禁自行构造 --scene 参数值，必须使用本文档指定的场景名**
3. **图片大小限制：本地文件不超过5MB，支持 jpg/jpeg/png/gif/bmp/webp/tiff/wbmp 格式**

---

## 🔗 相关资源
- 官方链接、API Key 获取方式、数据流向等详见 [references/resources.md](references/resources.md)

## 📁 文件结构
- `SKILL.md` —  本文档（意图分析 + 通用规范）
- `references/scenarios-detail.md` —  场景详细说明
- `references/error-codes.md` —  错误码对照表
- `references/resources.md` —  官方链接和资源
- `scripts/scan.py` —  主执行脚本 (Python 3.9+)
- `scripts/common/*.py` —  基础类库
