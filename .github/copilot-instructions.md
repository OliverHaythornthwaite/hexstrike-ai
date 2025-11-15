<!--
Purpose: concise, project-specific guidance for AI coding agents (Copilot/agents).
Update: created by repository helper based on README and source files.
-->

# Copilot / AI Agent Instructions — HexStrike AI

This file gives the minimal, concrete knowledge an AI coding agent needs to be immediately productive in this repository.

**Big Picture**
- **Primary components:** `hexstrike_server.py` (HTTP API server & decision engine) and `hexstrike_mcp.py` (MCP client that exposes `mcp.tool()` wrappers). The README contains an architecture diagram and integration examples.
- **Communication:** AI agents connect using the FastMCP protocol (see `hexstrike_mcp.py`) and/or via REST API endpoints exposed by the server (e.g. `/api/tools/<tool>` and `/api/intelligence/*`).
- **Purpose:** orchestrate 150+ security tools, provide intelligent tool selection via `IntelligentDecisionEngine`, and present results with `ModernVisualEngine`.

**Key files to reference (examples)**
- `hexstrike_server.py` — decision engine, REST endpoints, `decision_engine.select_optimal_tools()` and `IntelligentDecisionEngine.create_attack_chain()`.
- `hexstrike_mcp.py` — many `@mcp.tool()` functions (e.g. `nmap_scan`, `gobuster_scan`, `nuclei_scan`) that call server endpoints like `api/tools/nmap`.
- `README.md` — startup commands, client integration snippets, and API examples (use for user-facing instructions).
- `requirements.txt` — Python dependencies; environment requires Python 3.8+.

**How to run (developer workflows)**
- Create and activate a venv: `python -m venv .venv` then on Windows: `.venv\Scripts\Activate.ps1` (PowerShell) or `.venv\Scripts\activate` (cmd).
- Install deps: `pip install -r requirements.txt`.
- Start the server: `python hexstrike_server.py` (add `--debug` for verbose logs). Default host/port from env: `HEXSTRIKE_HOST` and `HEXSTRIKE_PORT` (default 127.0.0.1:8888).
- Start MCP client for local AI integration/testing: `python hexstrike_mcp.py` (client uses FastMCP to register tools). Example MCP config snippets are in `README.md`.
- Health check: `curl http://localhost:8888/health`.

**Patterns & conventions agents should follow**
- Tool wrappers: functions in `hexstrike_mcp.py` named like `<tool>_scan` or `<tool>_run` are thin clients that post to `api/tools/<tool>`; prefer reusing these names/parameters when adding new tools.
- Endpoints: server exposes `/api/tools/<tool>` and `/api/intelligence/*`. When implementing new endpoints, mirror existing request/response shapes (JSON `{ "target": ..., "additional_args": ... }`).
- Logging: both server and MCP client use colorized logging and write to `hexstrike.log`. Avoid breaking existing logging formats.
- Decision logic: use `IntelligentDecisionEngine` and `decision_engine.optimize_parameters()` for parameter decisions rather than hardcoding flags in tool wrappers.

**Integration points & external dependencies**
- Many external native tools (nmap, masscan, ghidra, etc.) are assumed to be installed on the host. The code expects CLI binaries to be available — do not assume Python-only installs.
- Browser automation relies on Chrome/Chromedriver and `selenium` / `mitmproxy` for browser agent flows. See README browser agent section.
- MCP integration: `FastMCP` is used in `hexstrike_mcp.py` — agents register `@mcp.tool()` functions there.

**Concrete examples agents can use when editing or adding code**
- Add a new tool wrapper: copy pattern from `nmap_scan` in `hexstrike_mcp.py` and post to `hexstrike_client.safe_post("api/tools/<toolname>", data)`.
- Call decision engine from server code: `selected = decision_engine.select_optimal_tools(profile, objective="comprehensive")` and then build `AttackStep` objects as in `create_attack_chain()`.
- Health-check & debug: run `python hexstrike_server.py --debug` and inspect `hexstrike.log`.

**Safety & permissions**
- The codebase executes many potentially destructive tools. Always run in isolated test VMs or containers and ensure authorized targets. The README contains legal/ethical guidance — do not remove or contradict it.

If any section is unclear or you want more examples (unit tests, a new tool wrapper, or an integration example for a specific agent), tell me which part to expand.
