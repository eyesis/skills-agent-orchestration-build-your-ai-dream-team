# Agent team

To build Mona's Project Pulse dashboard, I orchestrate four custom agents defined under `.github/agents/`, driven from **GitHub Copilot CLI running in a Codespace**.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Coordinates the Planner, Coder, and Designer. Breaks the request into phases, assigns non-overlapping file scopes, decides what can run in parallel vs. sequentially, and reports the integrated result. Does not implement anything itself.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the codebase, docs, dependencies, and edge cases, then produces an implementation plan (ordered steps, file assignments, dependencies, parallelizable work, validation expectations, open questions). Writes no code.
- **Definition:** `.github/agents/planner.agent.md`

## Coder

- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements the dashboard logic and any support configuration (e.g., `.vscode/launch.json` pointed at `app` as the working directory, opening `index.html`) within the file scope the Orchestrator assigns. Keeps code explicit, deterministic, and testable.
- **Definition:** `.github/agents/coder.agent.md`

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Owns the UI/UX — usability, accessibility, information hierarchy, and visual design — delivering a polished dashboard with project cards, status badges, priority treatment, and deterministic CSS hooks like `.dashboard` and `.project-card`.
- **Definition:** `.github/agents/designer.agent.md`

All agents avoid git operations (staging, committing, pushing); those remain under my control via Copilot CLI prompts in the Codespace.
