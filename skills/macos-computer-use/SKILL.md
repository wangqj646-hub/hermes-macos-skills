---
name: macos-computer-use
category: macos
description: macOS 截图 → 多模态 LLM → UI 操作能力。三层自动化架构中的 Tier 3 视觉兜底方案。
---

# macOS Computer Use（Tier 3 视觉兜底）

> ⚠️ 本技能是 macOS 自动化的**第三层级兜底方案**。优先尝试 `macos-ui-automation` 技能中的 CLI 和 MCP UI Tree 方案，仅在那些方法不可行时使用本技能。

## 在三层架构中的位置

```
Tier 1: CLI 命令      ← macos-ui-automation 技能
Tier 2: MCP UI Tree   ← macos-ui-automation 技能
Tier 3: Computer Use  ← 本技能（视觉兜底）
```

**适用场景：** 网页应用（Safari/Chrome 内页面）、自定义 UI、Accessibility API 不友好的场景（如正新 OA 系统）、需要视觉判断的内容（验证码、图片识别）。

## 架构

```
screencapture → 截图(base64) → qwen3.6-plus 多模态接口 → 解析 UI 元素坐标 → pyautogui 执行点击/输入
```

## 文件位置

| 文件 | 功能 |
|------|------|
| `~/.hermes/scripts/screen_parser.py` | 截图 + 视觉解析，返回 UI 元素坐标 |
| `~/.hermes/scripts/screen_agent.py` | 智能体入口：接收指令 → 截图解析 → 执行操作 |
| `~/.hermes/computer-use/` | 独立 Python 3.12 venv，含 pyautogui 等依赖 |

## 使用方法

### 1. 列出屏幕所有可交互元素
```bash
~/.hermes/scripts/screen_agent.py --list
```

### 2. 查找并点击元素
```bash
~/.hermes/scripts/screen_agent.py "点击登录按钮"
```

### 3. 查找并在输入框输入文字
```bash
~/.hermes/scripts/screen_agent.py "在搜索框输入hello"
```

### 4. 只解析不执行（安全预览）
```bash
~/.hermes/scripts/screen_agent.py --dry-run "点击设置"
```

### 5. 纯截图解析（返回 JSON）
```bash
~/.hermes/scripts/screen_parser.py --query "找登录按钮" --output json
```

## API 配置

使用现有的 DashScope `coding-plan` provider 配置，**不需要额外 API Key**。脚本会自动读取已配置的凭证。

- 端点: `https://coding.dashscope.aliyuncs.com/v1`
- 模型: `qwen3.6-plus`（支持多模态）

## 关键参数

- `RETA_SCALE = 2.0` — MacBook Pro Retina 屏幕，原生像素 = 2x 逻辑点
- `pyautogui.FAILSAFE = True` — 鼠标移到左上角可紧急中止
- 截图路径: `/tmp/screen_hermes.png`

## 调用方式（Luna 内部）

Luna 可以直接调用脚本，获取 JSON 输出：

```bash
~/.hermes/scripts/screen_parser.py --query "找登录按钮" --output json
```

返回格式：
```json
[
  {"text": "登录", "box": [x1, y1, x2, y2], "action": "click", 
   "logical_box": [lx1, ly1, lx2, ly2], "center_logical": [cx, cy]}
]
```

然后 Luna 可以用 `mcp_macos_ui_click_at_position` 或 `pyautogui` 执行点击。

## 故障排除

### 截屏为空/黑屏
- MacBook 合盖且无外接显示器时屏幕输出关闭
- 需要打开盖子或接显示器才能正常截图

### JSON 解析失败
- 模型可能返回了 markdown 代码块包裹的 JSON
- `parse_response()` 函数已处理常见格式问题

### API 401 错误
- 检查现有 provider 配置中的 api_key 是否有效
- 或设置环境变量 `DASHSCOPE_API_KEY`

### pyautogui 操作无响应
- 需要授予 Python 辅助功能权限：系统设置 → 隐私与安全 → 辅助功能
- 目标路径: `~/.hermes/computer-use/bin/python3.12`

## 安全注意

1. **永远先 --dry-run 再执行** — 确认识别的元素正确后再实际操作
2. **FAILSAFE 已启用** — 鼠标移到屏幕左上角可紧急中止 pyautogui
3. 不要用于敏感操作（如删除、支付确认等），除非用户明确要求
