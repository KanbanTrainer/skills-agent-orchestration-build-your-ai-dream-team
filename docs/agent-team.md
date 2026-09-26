# Agent team

To build Mona's Project Pulse dashboard, I am using a team of four custom agents defined under `.github/agents/` in this repository. I orchestrate them using GitHub Copilot CLI running in a Codespace.

| Agent | Model | Responsibility | Definition |
|---|---|---|---|
| Orchestrator | Claude Opus 4.7 (copilot) | Coordinates the Planner, Coder, and Designer agents. Breaks requests into phases, assigns non-overlapping file scopes, runs work in parallel or sequentially as needed, and reports the final outcome. Does not implement work itself. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 (copilot) | Researches the codebase, documentation, dependencies, and edge cases to produce implementation plans (steps, file assignments, dependencies, parallelizable work, validation expectations). Does not write code. | `.github/agents/planner.agent.md` |
| Designer | Gemini 3.1 Pro (copilot) | Handles UI/UX, accessibility, information architecture, and visual design for the dashboard, including project cards, status badges, and responsive layout, within the file scope assigned by the Orchestrator. | `.github/agents/designer.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code and support configuration (e.g. `.vscode/launch.json`) within the assigned file scope, following repository patterns and validating changes before reporting completion. | `.github/agents/coder.agent.md` |

All four agents are prohibited from staging, committing, or pushing changes — I control all git operations through Copilot CLI prompts.
