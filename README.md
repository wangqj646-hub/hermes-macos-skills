# macOS Skills for Hermes Agent

Two skills for macOS automation with Hermes Agent:

## macos-ui-automation
Three-tier automation decision framework: **CLI → MCP UI Tree → Computer Use fallback**.
- Tier 1: CLI commands (fastest, most reliable)
- Tier 2: MCP Accessibility API structured UI tree search
- Tier 3: Screenshot + multimodal LLM visual fallback

## macos-computer-use
Tier 3 visual fallback — screenshot + multimodal LLM for UI element detection and control.
Use when CLI and MCP UI Tree approaches are not applicable (web apps, custom UIs, etc.).

## Install

```bash
hermes skills tap add wangqj646-hub/hermes-macos-skills
hermes skills install macos-ui-automation
hermes skills install macos-computer-use
```
