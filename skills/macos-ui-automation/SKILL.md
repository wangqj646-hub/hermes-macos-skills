     1|---
     2|name: macos-ui-automation
     3|category: macos
     4|description: "Three-tier macOS automation decision framework: CLI → MCP UI Tree → Computer Use fallback."
     5|---
     6|
     7|# macOS UI Automation — Three-Tier Strategy
     8|
     9|## Core Principle
    10|
    11|**Never touch the GUI if it can be scripted. Never use a screenshot if structured data is available. Only fall back to the vision model when all else fails.**
    12|
    13|```
    14|Tier 1: CLI Commands      (milliseconds, most reliable)
    15|    ↓ Doesn't work?
    16|Tier 2: MCP UI Tree       (seconds, structured)
    17|    ↓ Doesn't work?
    18|Tier 3: Computer Use      (5-15 seconds, visual fallback)
    19|```
    20|
    21|## Tier 1: CLI Commands (First Choice)
    22|
    23|### System Settings & Control
    24|```bash
    25|# Open an app
    26|open -a "AppName"
    27|
    28|# System Preferences (via AppleScript)
    29|osascript -e 'tell application "System Settings" to reveal pane id "com.apple.preference.displays"'
    30|
    31|# Volume control
    32|osascript -e "set volume output volume 50"
    33|
    34|# WiFi on/off
    35|networksetup -setairportpower en0 on/off
    36|```
    37|
    38|### File Operations
    39|```bash
    40|# Find files
    41|mdfind "kMDItemFSName == '*.xlsx'"        # Spotlight search
    42|find ~/Documents -name "*.xlsx"            # Traditional find
    43|
    44|# File metadata
    45|mdls path/to/file                          # Metadata
    46|stat path/to/file                          # File attributes
    47|```
    48|
    49|### Application Control
    50|```bash
    51|# List running apps
    52|osascript -e 'tell application "System Events" to get name of every application process whose background only is false'
    53|
    54|# Force quit
    55|killall -9 "AppName"
    56|
    57|# AppleScript app control
    58|osascript -e 'tell application "Safari" to open location "https://example.com"'
    59|osascript -e 'tell application "Messages" to send "hello" to buddy "xxx"'
    60|```
    61|
    62|### Network & APIs
    63|```bash
    64|# API calls
    65|curl -s https://api.example.com/data | jq '.results'
    66|
    67|# Download files
    68|curl -O https://example.com/file.zip
    69|```
    70|
    71|### Media Control
    72|```bash
    73|# Play/Pause
    74|osascript -e 'tell application "Spotify" to playpause'
    75|
    76|# Screenshot
    77|screencapture -x /tmp/screenshot.png       # Silent screenshot
    78|screencapture -R x,y,w,h /tmp/crop.png     # Region capture
    79|```
    80|
    81|## Tier 2: MCP UI Tree Structured Search (Second Choice)
    82|
    83|When CLI cannot precisely operate, use the structured UI tree exposed by the macOS Accessibility API.
    84|
    85|### Available Tools
    86|
    87|| MCP Tool | Purpose |
    88||---|---|
    89|| `list_running_applications` | List all running applications |
    90|| `get_app_overview` | Quick overview of all apps and their windows |
    91|| `find_elements_in_app` | Deep-search UI element tree within a specific app |
    92|| `find_elements` | Search UI elements in the current focused window using JSONPath |
    93|| `get_element_details` | Get full structure of an element (including children) |
    94|| `click_element_by_selector` | Precise click via JSONPath selector |
    95|| `type_text_to_element_by_selector` | Type text via JSONPath selector |
    96|
    97|### Common JSONPath Selectors
    98|
    99|```
   100|# Find by identifier
   101|$..[?(@.ax_identifier=='loginButton')]
   102|
   103|# Find by role
   104|$..[?(@.ax_role=='AXButton')]
   105|
   106|# Find by title/text
   107|$..[?(@.ax_title=='Save')]
   108|
   109|# Combined conditions
   110|$..[?(@.ax_role=='AXTextField' && @.ax_identifier=='searchField')]
   111|
   112|# Get all interactive elements
   113|$..[?(@.ax_identifier)]
   114|```
   115|
   116|### Standard Workflow
   117|
   118|```
   119|1. get_app_overview              → Confirm target app and window state
   120|2. find_elements_in_app(app="X")  → Search for target elements
   121|3. get_element_details           → Verify element structure is correct
   122|4. click_element_by_selector      → Execute the operation
   123|```
   124|
   125|### When to Use
   126|- ✅ Native macOS apps (Finder, System Settings, Notes, Calendar, etc.)
   127|- ✅ Apps using standard AppKit controls
   128|- ✅ Scenarios requiring precise button/input/menu targeting
   129|
   130|### When NOT to Use
   131|- ❌ Web apps (pages inside Safari/Chrome) — Accessibility API only sees the browser shell
   132|- ❌ Highly custom UIs (games, some Electron app components)
   133|- ❌ Scenarios requiring visual judgment (CAPTCHAs, image recognition)
   134|
   135|## Tier 3: Computer Use Visual Fallback (Last Resort)
   136|
   137|When both CLI and MCP UI Tree fail, use the screenshot + multimodal LLM approach.
   138|
   139|See the `macos-computer-use` skill for details.
   140|
   141|### When to Use
   142|- Web apps (pages inside Safari/Chrome)
   143|- Custom UI apps
   144|- Content requiring visual understanding (text in images, CAPTCHAs, charts)
   145|- Accessibility API unfriendly environments (e.g., Electron apps)
   146|
   147|### Core Scripts
   148|```bash
   149|# List all interactive elements on screen
   150|~/.hermes/scripts/screen_agent.py --list
   151|
   152|# Find and execute an action
   153|~/.hermes/scripts/screen_agent.py "click the login button"
   154|
   155|# Safe preview (no actual operation)
   156|~/.hermes/scripts/screen_agent.py --dry-run "click settings"
   157|
   158|# Pure screenshot parsing (returns JSON coordinates)
   159|~/.hermes/scripts/screen_parser.py --query "find login button" --output json
   160|```
   161|
   162|## Decision Flow
   163|
   164|```
   165|User requests a UI operation
   166|        ↓
   167|Can it be done with CLI/AppleScript directly?
   168|    ├── Yes → Tier 1 execute ✅
   169|    └── No ↓
   170|        ↓
   171|Is the target a native macOS app?
   172|    ├── Yes → Tier 2 MCP UI Tree
   173|    │           → find_elements → click/type ✅
   174|    └── No ↓
   175|        ↓
   176|        Tier 3 Computer Use fallback
   177|        → Screenshot → Multimodal parsing → Coordinate operation ✅
   178|```
   179|
   180|## Important Notes
   181|
   182|1. **Permissions**
   183|   - Tier 1: Usually no special permissions needed
   184|   - Tier 2: Requires Accessibility permission (System Settings → Privacy & Security → Accessibility)
   185|   - Tier 3: Requires Screen Recording + Accessibility permissions
   186|
   187|2. **Safety**
   188|   - Always dry-run destructive operations (deletion, payment confirmation, etc.) first
   189|   - Tier 3 pyautogui has FAILSAFE enabled (move mouse to top-left corner to abort)
   190|
   191|3. **Performance**
   192|   - Tier 1: < 1 second
   193|   - Tier 2: 1-3 seconds
   194|   - Tier 3: 5-15 seconds (including screenshot, LLM call, parsing)
   195|