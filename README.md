# dmint-skills

> **Official AI agent skills and prompt assets for Dmint security policy management.**

`dmint-skills` provides specialized agent skill packages and system prompts that enable AI coding assistants and agent frameworks to create, understand, verify, modify, and manage Dmint security policies through natural conversation.

---

## Architecture: Skill vs CLI vs Engine

Dmint enforces a strict separation of concerns between conversational guidance, policy compilation, and runtime enforcement:

```text
┌───────────────────────────────────────────────────────────────┐
│                    Non-Technical User                         │
│       "Protect my GitHub MCP server with read-only access"     │
└──────────────────────────────┬────────────────────────────────┘
                               │ Plain Human English
                               ▼
┌───────────────────────────────────────────────────────────────┐
│           dmint-policy-manager (Agent Skill)                  │
│  - Guides conversation & clarifies requirements               │
│  - Writes human requirements to access.md                     │
│  - NEVER writes or modifies policy JSON directly              │
└──────────────────────────────┬────────────────────────────────┘
                               │ Invokes CLI commands
                               ▼
┌───────────────────────────────────────────────────────────────┐
│             dmint-cli (Authoritative Tooling)                 │
│  - dmint create-mcp-policy --discover                         │
│  - dmint create-policy                                        │
│  - dmint verify-policy (Mandatory validation gate)            │
└──────────────────────────────┬────────────────────────────────┘
                               │ Produces validated artifacts
                               ▼
┌───────────────────────────────────────────────────────────────┐
│               dmint / dmint-mcp (Runtime Engine)              │
│  - Deterministic PEP/PDP enforcement gate                     │
│  - Cryptographic approval verification (Ed25519)              │
│  - Transparent MCP stdio / HTTP proxy                         │
└───────────────────────────────────────────────────────────────┘
```

### Core Responsibilities
- **`dmint-policy-manager` (Skill)**: An AI assistant instruction set. Discovers capabilities, writes `access.md`, calls `dmint-cli`, and explains results in simple human terms.
- **`dmint-cli` (Compiler & Verifier)**: The authoritative compiler that discovers live capabilities, generates valid policy files, and validates them via `verify-policy`.
- **`dmint` / `dmint-mcp` (Runtime Engine)**: The trusted execution gate that intercepts capability invocations and enforces rules.

---

## Fundamental Security Invariants

1. **No Manual JSON Surgery**: The assistant must **never** construct, patch, or hand-edit `policy.json` directly. All policy generation and modification goes through `dmint create-policy` or `dmint create-mcp-policy`.
2. **Mandatory Verification**: A policy is **never** declared ready until `dmint verify-policy --policy <file>` passes with exit code `0`.
3. **Execution Boundary Protection**:
   > **Important Enforcement Invariant**:
   > *Dmint does not secure an MCP server if the agent can bypass the protected execution path and call the original capability directly.*
   > The assistant must always guide the user to point their agent's configuration to the Dmint-protected proxy rather than the upstream MCP server.
4. **Least Privilege by Default**: Deny by default. Destructive operations (delete, drop, force push, terminate) always require explicit confirmation and appropriate approval controls.
5. **Plain-Language First**: Avoid confusing jargon (AST, tuples, JSON schemas, regex tokens). Translate permissions into everyday concepts: "Allowed", "Needs your approval", and "Blocked".

---

## Primary Skill: `dmint-policy-manager`

The primary skill definition is located at:
- Repository path: `.agents/skills/dmint-policy-manager/SKILL.md`
- Installed package asset: `dmint_skills/skills/dmint-policy-manager/SKILL.md`

### Conversational Workflows

#### 1. Protecting an MCP Server
When the user asks to protect an MCP server (e.g. GitHub, Figma, PostgreSQL, Filesystem):
1. **Check CLI**: Verify `dmint-cli` is installed (`dmint --help`).
2. **Discover Capabilities**: Run `dmint create-mcp-policy --discover` with server command or URL to inspect actual available tools.
3. **Draft Requirements**: Write plain-language rules to `access.md` based on discovered tools and user intent.
4. **Compile Policy**: Run `dmint create-mcp-policy` to generate `policy.json` and client configuration.
5. **Verify**: Run `dmint verify-policy --policy policy.json`.
6. **Report**: Present a simple safety summary and explain how to configure the agent to use the protected proxy.

#### 2. Protecting Local Code & APIs
When protecting Python functions or local APIs:
1. Identify capability identifiers (`module.function_name`).
2. Write rules into `access.md`.
3. Compile with `dmint create-policy --requirements access.md --output policy.json`.
4. Verify with `dmint verify-policy --policy policy.json`.
5. Guide the user on applying `@dmint.protect()` decorators in their code.

#### 3. Modifying Existing Permissions
When the user wants to adjust access (e.g. "Allow the agent to push code now"):
1. **Never edit `policy.json` directly.**
2. Update the requirements in `access.md`.
3. Re-run `dmint create-policy` or `dmint create-mcp-policy` to recompile.
4. Re-run `dmint verify-policy`.
5. Confirm the updated permissions with the user.

#### 4. Diagnosing Blocked Actions
When the user asks "Why did my agent get blocked?":
1. Run `dmint verify-policy --policy policy.json` to inspect rules.
2. Check if the capability was explicitly denied or defaulted to deny.
3. Explain clearly what action was attempted and why it was blocked.
4. Offer safe options to allow or require approval if appropriate.

#### 5. Multi-MCP Configurations
When protecting multiple MCP servers:
- Keep separate policies and proxy entry points per server (e.g., `policies/github-policy.json`, `policies/slack-policy.json`).
- Verify each policy individually.
- Ensure the agent configuration routes each server through its respective Dmint proxy.

---

## Installation

```bash
pip install dmint-skills
```

To install test dependencies:
```bash
pip install "dmint-skills[test]"
```

Ensure `dmint-cli` is also installed in your environment:
```bash
pip install dmint-cli
```

---

## Usage

### Loading the Policy Manager Skill in Python

```python
from dmint_skills import (
    load_policy_manager_skill,
    get_skill_path,
    DMINT_POLICY_MANAGER_SKILL,
)

# 1. Load skill content as a string
skill_content = load_policy_manager_skill()
print(f"Skill loaded ({len(skill_content)} characters)")

# 2. Get absolute filesystem path to SKILL.md
skill_path = get_skill_path("dmint-policy-manager")
print(f"Skill path: {skill_path}")

# 3. Access prompt string constant directly
print(DMINT_POLICY_MANAGER_SKILL[:120])
```

### Legacy Policy System Prompt

For backward compatibility with existing agent configurations:

```python
from dmint_skills import DMINT_POLICY_SYSTEM_PROMPT

print(DMINT_POLICY_SYSTEM_PROMPT[:100])
```

### Available Skills in `.agents/skills/`

- **`dmint-policy-manager`** *(Primary)*: Official end-to-end policy manager for conversational authoring, modification, explanation, and verification via `dmint-cli`.
- `dmint-policy-authoring`: Core policy drafting guidelines.
- `dmint-security`: Security invariants and least-privilege principles.
- `dmint-authorization`: Authorization engine design and evaluation semantics.
- `dmint-approval-security`: Cryptographic approval signing and lifecycle rules.
- `dmint-adversarial-testing`: Testing and fuzzing guidelines.
- `dmint-agent-execution`: Gate execution and decorator usage.

---

## Testing

Run the full test suite (including all 20 scenario tests for `dmint-policy-manager`):

```bash
pytest -v
```

---

## License

Licensed under the [Apache License, Version 2.0](LICENSE).
See the [`LICENSE`](LICENSE) file for details.
