---
name: macos-ui-automation
category: macos
description: macOS 自动化三层决策框架 — CLI → MCP UI Tree → Computer Use 兜底。
---

# macOS UI 自动化三层策略

## 核心原则

**能脚本化的绝不碰 GUI，能拿结构化的绝不靠截图，实在没辙了再让视觉模型兜底。**

```
Tier 1: CLI 命令      (毫秒级，最可靠)
    ↓ 不行？
Tier 2: MCP UI Tree   (秒级，结构化)
    ↓ 不行？
Tier 3: Computer Use  (5-15秒，视觉兜底)
```

## Tier 1: CLI 命令（首选）

### 系统设置与控制
```bash
# 打开 App
open -a "AppName"

# 系统偏好设置（通过 AppleScript）
osascript -e 'tell application "System Preferences" to reveal pane id "com.apple.preference.display"'

# 音量控制
osascript -e "set volume output volume 50"

# 亮屏/休眠
osascript -e 'tell application "System Events" to tell process "Finder" to keystroke "q" using {command down}'

# 蓝牙、WiFi、飞行模式
networksetup -setairportpower en0 on/off
```

### 文件操作
```bash
# 查找文件
mdfind "kMDItemFSName == '*.xlsx'"        # Spotlight 搜索
find ~/Documents -name "*.xlsx"            # 传统 find

# 文件信息
mdls path/to/file                          # 元数据
stat path/to/file                          # 文件属性
```

### 应用控制
```bash
# 获取运行中的应用
osascript -e 'tell application "System Events" to get name of every application process whose background only is false'

# 强制退出
killall -9 "AppName"

# AppleScript 操作应用
osascript -e 'tell application "Safari" to open location "https://example.com"'
osascript -e 'tell application "Messages" to send "hello" to buddy "xxx"'
```

### 网络与 API
```bash
# 网络请求
curl -s https://api.example.com/data | jq '.results'

# 下载文件
curl -O https://example.com/file.zip
```

### 媒体控制
```bash
# 播放/暂停
osascript -e 'tell application "Spotify" to playpause'

# 截图
screencapture -x /tmp/screenshot.png       # 无声音截图
screencapture -R x,y,w,h /tmp/crop.png     # 区域截图
```

## Tier 2: MCP UI Tree 结构化搜索（次选）

当 CLI 无法精确操作时，使用 macOS Accessibility API 暴露的结构化 UI 树。

### 可用工具

| MCP 工具 | 功能 |
|---|---|
| `list_running_applications` | 列出所有运行中的应用 |
| `get_app_overview` | 快速获取所有应用及窗口浅层信息 |
| `find_elements_in_app` | 在指定 App 内深度搜索 UI 元素树 |
| `find_elements` | 用 JSONPath 在当前聚焦窗口搜索 UI 元素 |
| `get_element_details` | 获取某个元素的完整结构（含子节点） |
| `click_element_by_selector` | 通过 JSONPath 选择器精准点击 |
| `type_text_to_element_by_selector` | 通过 JSONPath 选择器输入文字 |

### JSONPath 常用选择器

```
# 按标识符查找
$..[?(@.ax_identifier=='loginButton')]

# 按角色查找
$..[?(@.ax_role=='AXButton')]

# 按标题/文字查找
$..[?(@.ax_title=='保存')]

# 组合条件
$..[?(@.ax_role=='AXTextField' && @.ax_identifier=='searchField')]

# 获取所有可交互元素
$..[?(@.ax_identifier)]
```

### 标准操作流程

```
1. get_app_overview              → 确认目标应用和窗口状态
2. find_elements_in_app(app="X")  → 搜索目标元素
3. get_element_details           → 确认元素结构正确
4. click_element_by_selector      → 执行操作
```

### 适用场景
- ✅ 原生 macOS App（Finder、系统设置、备忘录、日历等）
- ✅ 使用标准 AppKit 控件的应用
- ✅ 需要精准定位按钮/输入框/菜单项的场景

### 不适用场景
- ❌ 网页应用（Safari/Chrome 内页面）— Accessibility API 拿到的只是浏览器外壳
- ❌ 高度自定义 UI（游戏、Electron 应用的部分组件）
- ❌ 需要视觉判断的场景（验证码、图片识别）

## Tier 3: Computer Use 截图视觉兜底（最后手段）

当 CLI 和 MCP UI Tree 都无法解决时，使用截图 + 多模态 LLM 方案。

详见 `macos-computer-use` 技能。

### 适用场景
- 网页应用（Safari/Chrome 内页面）
- 自定义 UI 应用（如正新鸡排 OA 系统 `http://oa.zhengxinfood.com:8555/`）
- 需要视觉理解的内容（图片中的文字、验证码、图表）
- Electron 应用等 Accessibility API 不友好的场景

### 核心脚本
```bash
# 列出屏幕所有可交互元素
~/.hermes/scripts/screen_agent.py --list

# 查找并执行操作
~/.hermes/scripts/screen_agent.py "点击登录按钮"

# 安全预览（不实际操作）
~/.hermes/scripts/screen_agent.py --dry-run "点击设置"

# 纯截图解析（返回 JSON 坐标）
~/.hermes/scripts/screen_parser.py --query "找登录按钮" --output json
```

## 决策流程图

```
主人下达 UI 操作指令
        ↓
能用 CLI/AppleScript 直接完成？
    ├── Yes → Tier 1 执行 ✅
    └── No ↓
        ↓
目标是否是原生 macOS App？
    ├── Yes → Tier 2 MCP UI Tree
    │           → find_elements → click/type ✅
    └── No ↓
        ↓
        Tier 3 Computer Use 兜底
        → 截图 → 多模态解析 → 坐标操作 ✅
```

## 注意事项

1. **权限要求**
   - Tier 1: 通常不需要特殊权限
   - Tier 2: 需要辅助功能权限（系统设置 → 隐私与安全 → 辅助功能）
   - Tier 3: 需要屏幕录制权限 + 辅助功能权限

2. **安全原则**
   - 破坏性操作（删除、支付确认等）必须先 dry-run 或确认
   - Tier 3 的 pyautogui 已启用 FAILSAFE（鼠标移到左上角紧急中止）

3. **性能对比**
   - Tier 1: < 1 秒
   - Tier 2: 1-3 秒
   - Tier 3: 5-15 秒（含截图、LLM 调用、解析）
