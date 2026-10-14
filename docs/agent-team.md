# Agent team

To build Mona's Project Pulse dashboard, I will use this custom agent team:

| Agent | Target model | Responsibility | Definition |
|---|---|---|---|
| **Planner** | Claude Opus 4.7 (copilot) | Researches the repository, documentation, dependencies, and edge cases, then produces a phased implementation plan without writing code. | `.github/agents/planner.agent.md` |
| **Designer** | Gemini 3.1 Pro (copilot) | Shapes the dashboard's user experience, accessibility, information hierarchy, interactions, visual design, and responsive behavior. | `.github/agents/designer.agent.md` |
| **Coder** | GPT-5.5 (copilot) | Implements assigned code and runnable-app support, including Project Pulse launch configuration when assigned, and validates the changes. | `.github/agents/coder.agent.md` |
| **Orchestrator** | Claude Opus 4.7 (copilot) | Coordinates the specialists, assigns file scopes and phases, manages dependencies and parallel work, and verifies the integrated result without implementing it. | `.github/agents/orchestrator.agent.md` |

I will use GitHub Copilot CLI in a Codespace to orchestrate the work, delegating planning, design, and implementation to the appropriate agents.
