# Antigravity Subagent Governance (`antigravity-subagent-governance`)

Strict subagent restriction guidelines, decision matrix, and concurrency governance rules for the **Google Antigravity desktop client on Windows**.

---

## 🎯 Motivation

In AI agent frameworks like Google Antigravity, delegating tasks to subagents (`invoke_subagent`, `define_subagent`, `swarm`) often seems convenient. However, in practice, premature or unnecessary subagent invocation introduces significant problems:

- **Token Explosion**: Spawning a subagent duplicates system prompts and creates separate multi-turn reasoning loops.
- **Context Fragmentation**: Subagents do not inherit parent conversation history or nuanced requirements.
- **Latency Penalty**: Subagent cold starts and asynchronous message passing add unnecessary turn delays.
- **Quota Saturation**: Uncontrolled concurrency quickly exhausts API rate limits and host governor slots.

This project enforces a **Single-Agent First** policy: the main agent must handle tasks directly using its own workspace tools (`read_file`, `write_file`, `edit_file`, `run_command`, MCP) and only delegate when strict criteria are met.

---

## 🛡️ Core Rules & Prohibitions

### Default Direct-Execution Cases

The primary agent handles code search, single-file changes, bug fixes, tests, builds, Git commands, brainstorming, and sequential workflows directly. These are efficiency defaults rather than platform security boundaries.

### Controlled Delegation Conditions

Delegation is allowed when the user explicitly requests it, or when there is a large independent fan-out or an isolated high-risk branch. User direction removes the default efficiency restriction but does not bypass least privilege, disjoint write sets, credential isolation, cancellation, or external-write approval.

Before dispatch, record the task input, output, timeout, cancellation path, write set, and required permissions. If the host governor is unavailable, use a conservative single-active-worker fallback.

## 📂 Repository Structure

```text
antigravity-subagent-governance/
├── README.md
├── rules/
│   └── AGENTS.md                                # Always-on global/workspace rules
└── skills/
    └── subagent-governance/
        ├── SKILL.md                             # Main governance skill & decision matrix
        └── references/
            ├── decision-tree.md                 # Pre-flight checklist & anti-patterns
            └── quota-management.md              # Concurrency limits & host-governor sync
```

---

## Installation

Start in one workspace. Copy its skill folder to `.agents/skills/subagent-governance/`. Review the rule in [rules/AGENTS.md](rules/AGENTS.md) and merge its relevant section once into the workspace `AGENTS.md`, preserving existing rules. To upgrade, replace that skill folder and update that same section in place. To uninstall, remove the installed skill folder and the section you added.

For global use, follow Antigravity’s [rules](https://antigravity.google/docs/rules) and [skills](https://antigravity.google/docs/skills) documentation for your installed version. Avoid redundant global and workspace copies. Check that the client discovers the skill and rule in a fresh conversation.
