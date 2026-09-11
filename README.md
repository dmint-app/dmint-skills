# dmint-skills

> **AI agent policy-authoring skills and system prompt guidance for Dmint.**

`dmint-skills` contains development-time skill definitions and prompt assets designed to guide AI coding assistants when creating Dmint security policies from human developer requirements.

```text
Human Security Intent (access.md)
       ↓
Policy Authoring Assistant (Skill Guidance)
       ↓
Candidate Policy JSON
       ↓
Dmint Core Validation (Policy.from_mapping)
       ↓
Authoritative Runtime policy.json
```

## Security Invariants

- **Development-Time Asset Only:** The authoring skill guides LLMs during development to draft policies.
- **Never the Runtime Authority:** An LLM or skill is **never** part of the trusted runtime authorization engine.
- **Least Privilege:** Policies default to DENY for unspecified capabilities.
- **No Permission Expansion:** The skill rules explicitly forbid converting DENY rules to ALLOW or turning specific resources into wildcards (`*`).

## Installation

```bash
pip install dmint-skills
```

## Usage

Import the system prompt constant for LLM policy authoring workflows:

```python
from dmint_skills import DMINT_POLICY_SYSTEM_PROMPT

print(DMINT_POLICY_SYSTEM_PROMPT[:100])
```

Available agent skills in `.agents/skills/`:
- `dmint-policy-authoring`: Policy drafting and validation guidance.
- `dmint-security`: Threat modeling and security invariants.
- `dmint-authorization`: Authorization engine design principles.
- `dmint-approval-security`: Approval lifecycle & Ed25519 signing rules.
- `dmint-adversarial-testing`: Concurrency, replay, and security testing guidelines.
- `dmint-agent-execution`: Enforcement gate and decorator execution rules.

## Testing

Run the skill prompt validation tests:

```bash
pytest -v
```

## License

Apache-2.0
