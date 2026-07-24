# ROSTR-Agent-Builder.skill

**Purpose:** Scaffold production-ready multi-agent Python systems from agent specifications.

**Version:** 1.0.0  
**Status:** Ready for implementation

---

## What It Does

Takes a JSON specification of agent roles, interaction patterns, and tool requirements, then generates:

- **agents.py** — Core agent classes + orchestration logic
- **tools.py** — Composio tool wrappers + unified interface
- **main.py** — CLI entry point + workflow runner
- **config.yaml** — LLM settings + integrations
- **requirements.txt** — Python dependencies
- **tests/** — Unit tests + fixtures

Output is a fully self-contained Python package ready to run or deploy to AWS Lambda, GCP Cloud Functions, etc.

---

## Input Schema

```json
{
  "agent_roles": [
    {
      "name": "string",
      "description": "string",
      "domain": "sales|marketing|ops|support|custom",
      "tools_needed": ["web_search", "email", "crm", "custom_tool"],
      "llm_model": "claude-opus|claude-sonnet|gpt-4",
      "system_prompt": "optional override"
    }
  ],
  "interaction_patterns": {
    "type": "sequential|parallel|hierarchical",
    "primary_agent": "agent_name",
    "dependencies": {
      "agent_name": ["depends_on_agent_1", "depends_on_agent_2"]
    }
  },
  "integrations": [
    {
      "type": "composio",
      "services": ["HubSpot", "Gmail", "Slack", "Apollo"]
    },
    {
      "type": "custom_api",
      "name": "internal_api",
      "base_url": "https://api.internal.com",
      "auth": "bearer_token"
    }
  ],
  "llm_config": {
    "default_model": "claude-opus-4-1-20250805",
    "temperature": 0.7,
    "max_tokens": 2000,
    "top_p": 1.0
  },
  "workspace_name": "team_name"
}
```

---

## Output Structure

```
generated_agents_kit_{workspace_name}/
├── README.md                   # Quick start + architecture
├── requirements.txt            # pip install -r requirements.txt
├── setup.py                    # For pip install .
├── config.yaml                 # LLM + Composio config
├── src/
│   ├── __init__.py
│   ├── agents.py               # Agent classes + orchestrator
│   ├── tools.py                # Composio + custom tool wrappers
│   ├── orchestrator.py         # Workflow execution logic
│   ├── models.py               # Pydantic models
│   └── utils.py                # Helper functions
├── main.py                     # CLI entry point
├── run_workflow.py             # Example usage
├── tests/
│   ├── __init__.py
│   ├── test_agents.py
│   ├── test_tools.py
│   ├── test_orchestrator.py
│   └── fixtures/
│       ├── sample_task.json
│       └── mock_responses.json
└── .env.example                # Credentials template
```

---

## Example Use Cases

### Use Case 1: Sales Development

**Input:**
```json
{
  "agent_roles": [
    {
      "name": "Prospect Researcher",
      "description": "Researches companies and contacts",
      "tools_needed": ["web_search", "crm_lookup"]
    },
    {
      "name": "Email Drafter",
      "description": "Writes personalized cold emails",
      "tools_needed": ["email_drafting", "storage"]
    }
  ],
  "interaction_patterns": {
    "type": "sequential",
    "primary_agent": "Prospect Researcher",
    "dependencies": {
      "Email Drafter": ["Prospect Researcher"]
    }
  },
  "integrations": [
    {
      "type": "composio",
      "services": ["HubSpot", "Apollo", "Gmail"]
    }
  ]
}
```

**Generated Usage:**
```python
from src.orchestrator import MultiAgentOrchestrator

orchestrator = MultiAgentOrchestrator.from_config('config.yaml')

task = {
  "company": "Acme Corp",
  "contact": "john@acme.com",
  "campaign": "enterprise_software"
}

result = orchestrator.execute(task)
# Returns: { email_draft: "...", research_findings: {...} }
```

### Use Case 2: Support Ticket Routing (Hierarchical)

**Input:**
```json
{
  "agent_roles": [
    {
      "name": "Intake Agent",
      "description": "Classifies incoming tickets"
    },
    {
      "name": "Technical Expert",
      "description": "Resolves technical issues"
    },
    {
      "name": "Billing Specialist",
      "description": "Handles payment & account issues"
    }
  ],
  "interaction_patterns": {
    "type": "hierarchical",
    "primary_agent": "Intake Agent",
    "routing": {
      "technical": "Technical Expert",
      "billing": "Billing Specialist"
    }
  }
}
```

---

## Implementation Details

### Generated agents.py Structure

```python
class Agent:
    """Base agent class"""
    def __init__(self, config: AgentConfig):
        self.config = config
        self.client = anthropic.Anthropic(api_key=os.getenv('ANTHROPIC_API_KEY'))
        self.tools = self._load_tools()
    
    async def execute(self, task: str, context: dict) -> AgentResponse:
        """Execute task with agentic loop"""
        messages = [{"role": "user", "content": task}]
        
        while True:
            response = await self.client.messages.create(
                model=self.config.model,
                max_tokens=self.config.max_tokens,
                system=self.config.system_prompt,
                tools=self.tools,
                messages=messages
            )
            
            if response.stop_reason == "end_turn":
                return AgentResponse(text=response.content)
            
            # Handle tool calls
            if response.stop_reason == "tool_use":
                for block in response.content:
                    if block.type == "tool_use":
                        result = await self._execute_tool(block.name, block.input)
                        messages.append({
                            "role": "user",
                            "content": json.dumps(result)
                        })
```

### Generated orchestrator.py Structure

```python
class MultiAgentOrchestrator:
    def __init__(self, agents: dict[str, Agent], pattern: str):
        self.agents = agents
        self.pattern = pattern  # sequential|parallel|hierarchical
    
    async def execute(self, initial_task: str) -> dict:
        if self.pattern == "sequential":
            return await self._execute_sequential(initial_task)
        elif self.pattern == "parallel":
            return await self._execute_parallel(initial_task)
        elif self.pattern == "hierarchical":
            return await self._execute_hierarchical(initial_task)
    
    async def _execute_sequential(self, task: str) -> dict:
        """Run agents in sequence, passing outputs"""
        results = {}
        current_input = task
        
        for agent_name in self.execution_order:
            agent = self.agents[agent_name]
            result = await agent.execute(current_input, results)
            results[agent_name] = result
            current_input = result.text  # Pass to next agent
        
        return results
```

---

## Composio Integration

The skill auto-wraps 100+ external services via Composio:

```python
# tools.py (auto-generated)
from composio import Composio, Tool

COMPOSIO_SERVICES = {
    "HubSpot": ["create_contact", "update_deal", "list_deals"],
    "Gmail": ["send_email", "search_emails", "create_draft"],
    "Apollo": ["lookup_email", "get_company_data"],
    "Slack": ["send_message", "upload_file"],
    # ... 100+ services
}

async def execute_composio_tool(service: str, action: str, params: dict):
    composio = Composio(api_key=os.getenv('COMPOSIO_API_KEY'))
    result = await composio.execute_action(
        entity_id=service,
        action=action,
        params=params
    )
    return result
```

---

## CLI Usage

```bash
# Generate agent kit
python main.py generate \
  --spec agents.json \
  --output ./my_agents_kit

# Run workflow
python main.py run \
  --config config.yaml \
  --task "Research Acme Corp and draft outreach email" \
  --input task.json

# Test agents
python -m pytest tests/

# Deploy to AWS Lambda
python main.py deploy --platform aws-lambda
```

---

## Testing & Validation

Generated `tests/` includes:

- **test_agents.py** — Unit tests for each agent
- **test_tools.py** — Mock Composio tool calls
- **test_orchestrator.py** — Integration tests for workflow
- **fixtures/** — Sample inputs, expected outputs

---

## Next Steps (Phase 2)

- Support for custom tools (not just Composio)
- Agent memory & context persistence
- Cost tracking per agent execution
- A/B testing framework for prompts
