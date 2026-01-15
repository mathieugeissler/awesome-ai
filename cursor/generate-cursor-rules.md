# Generate Cursor Rules

Modern AI coding tools are powerful, but without clear guidance they can easily drift away from our standards, duplicate patterns, or ignore important constraints. To avoid that, we use **Cursor Rules** and a dedicated AI agent called **Rules Architect** to teach the AI how our projects are structured and how we want it to behave.

This document explains how we standardize AI usage across our repositories: what Cursor Rules are, how they're organized at user, project, and team level, and how the Rules Architect agent analyzes a codebase to generate and maintain project-specific rules. The goal is simple: any developer opening a project in Cursor should get an AI assistant that already speaks our language, understands our architecture, and follows our conventions out of the box.

---

## 1. Cursor & Cursor Rules: quick overview

### 1.1 Cursor in a nutshell

Cursor is an AI-powered IDE that:

- Understands your codebase,
- Lets you chat with an AI agent that can read and modify code,
- Adds features like Agent, Bugfixing, and slash commands (`/…`).

### 1.2 What is a Cursor Rule?

Cursor Rules are persistent instructions that tell the AI how to behave on a project:

- Which conventions to follow,
- Which libraries / patterns to use,
- What to avoid (security pitfalls, deprecated code, legacy areas, etc.).

They are stored in configuration files, are loaded automatically, and are injected into the AI's prompt whenever it works on your code.

---

## 2. The four levels of rules

We distinguish four complementary levels:

### 2.1 User Rules (personal)

- Configured in **Cursor → Settings → Rules for AI**.
- Apply to all projects on the user's machine.
- Good for personal preferences:
  - default language,
  - style of explanations,
  - small individual habits.

### 2.2 Project Rules (`.cursor/rules/*.mdc`)

- Files stored in the repository under `.cursor/rules/`.
- Versioned with Git → shared by everyone working on the project.
- Contain rules specific to that project / stack, for example:
  - frontend rules,
  - backend rules,
  - database rules,
  - testing rules,
  - infrastructure rules.
- Rules can be automatically attached based on file paths via globs.

> This is where our **Rules Architect** agent mainly operates.

### 2.3 Team Rules (organization-wide)

- Configured in the Cursor web dashboard for Team / Enterprise plans.
- Applied to all projects in the organization.
- Useful for:
  - global code quality standards,
  - security policies,
  - organization-wide architecture guidelines,
  - common AI behavior expectations.

### 2.4 Agent Rules (`AGENTS.md`)

- A simple `AGENTS.md` file at the root of the project.
- Regular Markdown describing how agents should behave:
  - roles,
  - responsibilities,
  - boundaries.

This is practical for giving high-level instructions to AI agents without going into detailed `.mdc` rules.

---

## 3. Structure of a Project Rule (`.mdc`)

Project Rules are written in MDC files: front-matter (YAML-like) + Markdown content.

### 3.1 Minimal example

```yaml
---
description: Frontend rules for the React app
globs:
  - apps/web/**/*.{ts,tsx}
alwaysApply: false
---
- Use React function components with hooks.
- Prefer strict TypeScript typing and avoid `any`.
- Use the existing design system components instead of raw HTML where possible.
- When modifying a component, preserve the file's existing style and patterns.
```

**Key fields:**

| Field         | Description                                                                                                      |
|---------------|------------------------------------------------------------------------------------------------------------------|
| `description` | Short explanation of what the rule file controls                                                                 |
| `globs`       | File path patterns where these rules apply                                                                       |
| `alwaysApply` | `true` → always active when working in this project, `false` → attached automatically when files match the globs |

### 3.2 Writing principles

When we write rules, we follow these principles:

- **Short and concrete:** "Use X for Y" instead of vague advice.
- **Prefer positive formulations:** "Use the shared validation helper for API inputs" instead of only "Don't write ad-hoc validation".
- **Clear scope:** one rule file per domain (frontend, backend, tests, DB, infra, etc.).
- **No secrets, no sensitive information** inside rules.
- Several focused files (`frontend.mdc`, `backend.mdc`, `tests.mdc`, …) are better than one giant file that tries to cover everything.

---

## 4. The Rules Architect agent

### 4.1 Role

**Rules Architect** is a specialized AI agent (a slash command in Cursor) whose only mission is to:

- Analyze the project (structure, stack, conventions),
- Propose a structured set of Cursor Project Rules,
- Generate `.cursor/rules/*.mdc` files,
- Optionally suggest documentation (architecture, conventions, etc.).

> It is **not** for building features.  
> Its purpose is to improve how AI interacts with the project.

### 4.2 What the agent does

When you invoke Rules Architect, it:

#### 1. Discovers the project

- Reads the directory structure (`apps/`, `packages/`, `src/`, etc.),
- Looks at config files (`package.json`, `tsconfig`, language configs, linters, CI),
- Reads existing documentation (`README`, `docs/`, internal guidelines) if provided.

#### 2. Proposes a plan

- Suggests which rule files to create in `.cursor/rules/` (e.g. `frontend.mdc`, `backend.mdc`, `tests.mdc`),
- Defines the globs for each rule file,
- Explains what each file will control.

#### 3. Generates the rules

- Produces the actual `.mdc` content:
  - descriptions,
  - globs,
  - alwaysApply,
  - concrete bullet-point rules.
- Explains exactly where to save each file (path and filename).

#### 4. Iterates over time

- Updates rules when the architecture changes,
- Refines rules based on team feedback,
- Suggests new rules when new domains / services appear.

---

## 5. How to use Rules Architect in Cursor

### 5.1 Creating the `/generate-rules` command

We expose Rules Architect as a **Cursor Command** so that anyone can call it with `/generate-rules`.

In your repository:

1. **Create the commands folder** if it doesn't exist:

```bash
mkdir -p .cursor/commands
```

2. **Create the file:**

[./commands/generate-rules](.cursor/commands/generate-rules.md)

Cursor automatically loads commands from `.cursor/commands/*.md` and makes them available as slash commands in the chat input.

### 5.2 Running the agent

To use Rules Architect in a project:

1. Open the project in Cursor.
2. Open the **Agent chat**.
3. Type `/generate-rules` and press Enter.
4. Add a request, for example:

> Analyze this repository and propose a `.cursor/rules` layout adapted to our current architecture.

Rules Architect will reply with:

- A **project overview** (what it sees: stacks, domains, structure),
- A **proposed set of rule files and globs**,
- Then — once you confirm — the **full content of the `.mdc` files** to create.

---

## 6. Recommended workflow in a project

### Initialization

1. Add `/generate-rules` to the repo.
2. Run Rules Architect once to generate:
   - `.cursor/rules/*.mdc`,
   - optionally `AGENTS.md`.

### Review

- Treat rule files like code:
  - open a pull request,
  - review rules for clarity, strictness, and alignment with team practices.

### Commit & share

- Commit `.cursor/rules/` and `.cursor/commands/generate-rules.md`.
- From then on, every team member using Cursor benefits from the same AI behavior.

### Evolve over time

When architecture or conventions change:

- run `/generate-rules` again,
- ask it to update existing rules or propose new ones.

> Keep rules in sync with reality (they're part of the codebase, not a one-shot setup).

---

## 7. Internal FAQ

### Q: Do Cursor Rules apply to other tools (Claude in VS Code, GitHub Copilot, etc.)?

**A:** Technically no — only Cursor reads `.cursor/rules/*.mdc`.

However, since these files live in the repo, we can re-use their content as prompts or guidelines in other tools.

### Q: Do Team Rules replace Project Rules?

**A:** No.

- **Team Rules** = organization-wide standards (security, quality, common AI behavior).
- **Project Rules** = project-specific architecture, stack, and conventions.

They complement each other.

### Q: Who owns the rules?

**A:** The project team does.

Rules Architect assists with discovery and generation, but humans:

- validate,
- review,
- and maintain the rules like any other important piece of configuration.
