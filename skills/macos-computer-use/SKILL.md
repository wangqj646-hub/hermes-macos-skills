     1|---
     2|name: macos-computer-use
     3|category: macos
     4|description: "Screenshot → multimodal LLM → UI control. Tier 3 visual fallback in the three-tier macOS automation architecture."
     5|---
     6|
     7|# macOS Computer Use (Tier 3 Visual Fallback)
     8|
     9|> ⚠️ This skill is the **third-tier fallback** for macOS automation. Always try the CLI and MCP UI Tree approaches from the `macos-ui-automation` skill first. Only use this skill when those methods are not applicable.
    10|
    11|## Position in the Three-Tier Architecture
    12|
    13|```
    14|Tier 1: CLI Commands      ← macos-ui-automation skill
    15|Tier 2: MCP UI Tree       ← macos-ui-automation skill
    16|Tier 3: Computer Use      ← this skill (visual fallback)
    17|```
    18|
    19|**Use cases:** Web apps (pages inside Safari/Chrome), custom UIs, Accessibility API unfriendly environments, content requiring visual judgment (CAPTCHAs, image recognition).
    20|
    21|## Architecture
    22|
    23|```
    24|screencapture → Screenshot (base64) → qwen3.6-plus multimodal API → Parse UI element coordinates → pyautogui executes click/type
    25|```
    26|
    27|## File Locations
    28|
    29|| File | Purpose |
    30||------|---------|
    31|| `~/.hermes/scripts/screen_parser.py` | Screenshot + visual parsing, returns UI element coordinates |
    32|| `~/.hermes/scripts/screen_agent.py` | Agent entry point: receives instruction → parses screenshot → executes action |
    33|| `~/.hermes/computer-use/` | Isolated Python 3.12 venv with pyautogui and dependencies |
    34|
    35|## Usage
    36|
    37|### 1. List all interactive elements on screen
    38|```bash
    39|~/.hermes/scripts/screen_agent.py --list
    40|```
    41|
    42|### 2. Find and click an element
    43|```bash
    44|~/.hermes/scripts/screen_agent.py "click the login button"
    45|```
    46|
    47|### 3. Find and type into an input field
    48|```bash
    49|~/.hermes/scripts/screen_agent.py "type hello in the search box"
    50|```
    51|
    52|### 4. Safe preview only (no actual operation)
    53|```bash
    54|~/.hermes/scripts/screen_agent.py --dry-run "click settings"
    55|```
    56|
    57|### 5. Pure screenshot parsing (returns JSON)
    58|```bash
    59|~/.hermes/scripts/screen_parser.py --query "find the login button" --output json
    60|```
    61|
    62|## API Configuration
    63|
    64|Uses the existing DashScope `coding-plan` provider configuration — **no additional API key required**. The script automatically reads credentials from the configured provider.
    65|
    66|- Endpoint: `https://coding.dashscope.aliyuncs.com/v1`
    67|- Model: `qwen3.6-plus` (supports multimodal)
    68|
    69|## Key Parameters
    70|
    71|- `RETA_SCALE = 2.0` — MacBook Pro Retina display, native pixels = 2x logical points
    72|- `pyautogui.FAILSAFE = True` — Move mouse to the top-left corner to abort
    73|- Screenshot path: `/tmp/screen_hermes.png`
    74|
    75|## Internal Usage (Agent Invocation)
    76|
    77|The agent can call the script directly and parse the JSON output:
    78|
    79|```bash
    80|~/.hermes/scripts/screen_parser.py --query "find the login button" --output json
    81|```
    82|
    83|Response format:
    84|```json
    85|[
    86|  {"text": "Login", "box": [x1, y1, x2, y2], "action": "click",
    87|   "logical_box": [lx1, ly1, lx2, ly2], "center_logical": [cx, cy]}
    88|]
    89|```
    90|
    91|The agent can then use `mcp_macos_ui_click_at_position` or `pyautogui` to perform the click.
    92|
    93|## Troubleshooting
    94|
    95|### Blank/black screenshot
    96|- When the MacBook lid is closed with no external display, the screen output is disabled
    97|- Open the lid or connect a display for screenshots to work
    98|
    99|### JSON parsing failure
   100|- The model may return JSON wrapped in a markdown code block
   101|- The `parse_response()` function already handles common format issues
   102|
   103|### API 401 error
   104|- Check that the configured provider's api_key is valid
   105|- Or set the `DASHSCOPE_API_KEY` environment variable
   106|
   107|### pyautogui not responding
   108|- Grant Accessibility permission to Python: System Settings → Privacy & Security → Accessibility
   109|- Target path: `~/.hermes/computer-use/bin/python3.12`
   110|
   111|## Safety Notes
   112|
   113|1. **Always `--dry-run` first** — Verify the identified elements are correct before actual execution
   114|2. **FAILSAFE is enabled** — Move the mouse to the top-left corner to abort pyautogui
   115|3. **Never use for sensitive operations** (deletion, payment confirmation, etc.) unless explicitly requested by the user
   116|