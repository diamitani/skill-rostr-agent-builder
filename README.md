# skill-rostr-agent-builder

![Category: Product & Architecture](https://img.shields.io/badge/category-Product%20%26%20Architecture-blue) ![Status: Active](https://img.shields.io/badge/status-active-brightgreen)

Scaffold production-ready multi-agent Python systems from a simple JSON specification. This skill is part of the ROSTR platform — Patrick Diamitani's agentic infrastructure layer for the Artispreneur/ROSTR ecosystem. Give it a spec describing your agents, their roles, tool needs, and how they should interact, and it generates a complete, runnable Python package you can deploy immediately to AWS Lambda, GCP Cloud Functions, or run locally.

---

## What It Does

- Accepts a structured JSON spec defining agent roles, domains, tools needed, and LLM models
- Generates a complete Python package: `agents.py`, `tools.py`, `orchestrator.py`, `main.py`, `config.yaml`, `requirements.txt`, and a full `tests/` suite
- Supports three interaction patterns: **sequential** (chain outputs), **parallel** (concurrent execution), and **hierarchical** (router + specialist agents)
- Auto-wires 100+ external services via Composio (HubSpot, Gmail, Apollo, Slack, and more)
- Produces deploy-ready code with a CLI entry point and AWS Lambda / GCP Cloud Functions packaging

---

## How to Use

1. Write an `agents.json` spec describing your agent roles, tools, integrations, and LLM config (see `SKILL.md` for the full schema)
2. Invoke the skill in Claude Code:
   > "Build me a multi-agent system from this spec" — then paste or attach your `agents.json`
3. The skill generates the full package under `generated_agents_kit_{workspace_name}/`
4. Install dependencies and run:
   ```bash
   pip install -r requirements.txt
   python main.py run --config config.yaml --task "Your task here"
   ```
5. Deploy when ready:
   ```bash
   python main.py deploy --platform aws-lambda
   ```

---

## Trigger Phrases

Use any of the following to invoke this skill:

- "Build a multi-agent system from this spec"
- "Scaffold an agent kit"
- "Generate agents for [use case]"
- "Create a multi-agent Python system"
- "Build me an agent workflow for [sales/support/ops]"
- "ROSTR agent builder"
- "Generate agent boilerplate"

---

## Category

**Product & Architecture** — Part of the ROSTR platform's agent scaffolding layer.

---

## Author: Patrick Diamitani

GTM AI & Automation Manager at Atlas HXM. Founder of Artispreneur. Builder of the ROSTR agent platform.

- GitHub: [@diamitani](https://github.com/diamitani)
- LinkedIn: [linkedin.com/in/diamitani](https://linkedin.com/in/diamitani)

---

> Built with Claude Code · Part of [Patrick's Skills Library](https://github.com/diamitani)
