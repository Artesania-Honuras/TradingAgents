<!-- .github/copilot-instructions.md - guidance for AI coding agents -->
# Quick guide for AI coding agents working on TradingAgents

This file collects concise, actionable knowledge to help an AI coder become productive quickly in this repository.

- Project entry points
  - `main.py` (project root) — simple programmatic example that instantiates `TradingAgentsGraph` and calls `.propagate()`.
  - `cli/main.py` — interactive rich-CLI frontend (uses `rich`, `typer`). Use `python -m cli.main` to reproduce CLI behavior.

- Big-picture architecture (high level)
  - `tradingagents/graph` contains the orchestration layer. The central class is `TradingAgentsGraph` (see `trading_graph.py`) which composes agents, triggers LLM calls, and exposes `.propagate()` and reflection methods.
  - `tradingagents/agents` contains role-based agent implementations grouped by team: `analysts/`, `researchers/`, `trader/`, `managers/`, and `risk_mgmt/`.
  - `tradingagents/dataflows` encapsulates data ingestion adapters (Alpha Vantage, yfinance, Google, local caches). Files like `alpha_vantage.py`, `y_finance.py`, and `local.py` define vendor-specific shapes.
  - `tradingagents/utils` contains helper tooling (agent state, memory, core stock tools, technical/fundamental helpers) used throughout the graph and agents.

- Key configuration and conventions
  - Central defaults live in `tradingagents/default_config.py`. Typical pattern: copy the dict, then mutate (e.g. `config = DEFAULT_CONFIG.copy()`), then pass into `TradingAgentsGraph(...)`.
  - LLM and vendor selection are strings in the config: e.g. `deep_think_llm`, `quick_think_llm`, and `data_vendors` (`yfinance`, `alpha_vantage`, `openai`, `google`, `local`). Prefer editing config at runtime rather than hardcoding values.
  - Environment variables and `.env` are used (the code calls `dotenv.load_dotenv()` in `main.py` and `cli/main.py`). Keys expected: `OPENAI_API_KEY`, `ALPHA_VANTAGE_API_KEY`. Results dir can be overridden by `TRADINGAGENTS_RESULTS_DIR`.

- Developer workflow & quick commands (discoverable in README)
  - Create a venv/conda env (README recommends Python 3.13). Install deps: `pip install -r requirements.txt`.
  - Run a quick programmatic experiment: import `TradingAgentsGraph` and run `.propagate()` (see `main.py` and README usage example).
  - Run interactive demo (CLI): `python -m cli.main` — this exercise shows the typical agent flow and live updates.
  - There is a `test.py` in repo root; run directly with `python test.py` for existing simple checks. (No formal pytest harness checked in.)

- Integration points & external dependencies
  - OpenAI (LLM backend): many modules assume `openai` API key and the provider named `openai` in config.
  - Alpha Vantage and yfinance: data flows may call either vendor. Check `tradingagents/dataflows/*` to see how vendor-specific responses are normalized.
  - LangGraph / langchain variants: the project uses LangGraph and several langchain adapters — be cautious when upgrading versions.

- Patterns to follow when editing code
  - Preserve configuration-by-dict pattern; tests and examples expect `DEFAULT_CONFIG.copy()` semantics.
  - Agents are role-specific and communicate by structured strings/reports. When modifying agent output formats, update any downstream parsers in `tradingagents/utils` and the `MessageBuffer` formatting logic in `cli/main.py`.
  - CLI/UI uses `rich` and truncates long messages; for debugging, prefer writing to `results_dir` or `message_buffer` rather than relying on the live CLI for full traces.

- Files to inspect first when debugging
  - `tradingagents/graph/trading_graph.py` — orchestration and LLM call placement.
  - `tradingagents/default_config.py` — default behavior and vendor selection.
  - `tradingagents/dataflows/*` — vendor adapters and caching behavior.
  - `tradingagents/utils/memory.py` and `tradingagents/utils/agent_utils.py` — state and message shaping.
  - `cli/main.py` — the live display and message truncation logic; useful for reproducing interactive issues.

- Small, concrete examples to cite
  - Change LLMs at runtime: `config = DEFAULT_CONFIG.copy(); config['deep_think_llm'] = 'gpt-4o-mini'` then init `TradingAgentsGraph(config=config)` (see `main.py`).
  - Switch data vendor: edit `config['data_vendors']['core_stock_apis']` to `'alpha_vantage'` or `'yfinance'` to change where price/indicator data come from.

- What not to assume
  - There is no enforced typing across configs — values are looked up by string keys. Always follow existing key names in `DEFAULT_CONFIG`.
  - No heavy test harness is present. Changes that affect orchestration should be validated via quick runs (`python main.py` or CLI) and by inspecting output written to `results_dir`.

- Next steps / requests
  - If you want me to include LLM prompt templates or list where each LLM call is constructed, tell me which agent or file (for example: `tradingagents/agents/analysts/fundamentals_analyst.py`).

If any section is unclear or you want further expansion (examples, tests, or a checklist for PR reviewers), tell me which area to expand.
