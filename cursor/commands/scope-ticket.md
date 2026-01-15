# scope-ticket

## Goal

This command is used to **scope a Jira ticket** (e.g. `SPY-14526`) and produce an actionable **handoff**: target stack (Legacy vs New stack), likely intervention areas, risks, and TODOs.

## Usage

- **With argument**: `/scope-ticket SPY-14526`
- **Without argument**: `/scope-ticket`

## Expected behavior (the agent must follow these steps)

### 0) Resolve the ticket key

- **If an argument is provided**:

  - **Use the argument** as the ticket key.
  - Quickly validate the expected format (e.g. `ABC-12345`).

- **Otherwise (no argument)**:
  - Retrieve the current branch name.
  - **If the branch is `master` or matches `devc-*`**:
    - Stop here and answer:
      - that the ticket cannot be determined automatically,
      - and ask the user to:
        - pass an argument (`/scope-ticket SPY-14526`), **or**
        - create/checkout a dedicated branch for the ticket (e.g. `SPY-14526-...`).
  - **Otherwise**:
    - Try extracting a ticket key from the branch name (e.g. `feature/SPY-14526-remove-delay` → `SPY-14526`).
    - If no ticket key is found, apply the same fallback as above (argument or dedicated branch).

> Once the ticket key is determined, proceed to the next step.

### 1) Fetch the ticket contents (Jira)

- **Call** the Jira MCP tool: `jira_get_issue` with the ticket key.

- **If the tool does not exist**:

  - Explain that Jira is not accessible from the agent.
  - Ask the user to install/configure the Atlassian MCP (`mcp-atlassian`) to expose `jira_get_issue`.

- **If the tool exists but the call fails**:
  - **If it failed because of MCP misconfiguration**:
    - Ask the user to verify their environment variables:
      - `JIRA_PERSONAL_TOKEN`
      - `USER_EMAIL` (or the equivalent depending on MCP config: sometimes `JIRA_USERNAME`)
    - Point out where to create the Jira token: [Jira “Personal Access Tokens” page](https://jira.ogury.io/secure/ViewProfile.jspa?selectedTab=com.atlassian.pats.pats-plugin:jira-user-personal-access-tokens)
  - **If it failed because the ticket was not found or doesn't exist**:
    - Ask the user to double check the branch name or the name of the ticket in the provided argument
  - Propose an immediate **bypass**: the user can **paste the ticket contents** (title, description, AC, useful comments) directly in the chat.

> Once the content is fetched (via MCP or pasted), proceed to the next step.

### 2) Analysis & scoping (stack + intervention areas)

- **Identify the stack** where changes should be made:

  - New stack (TypeScript) vs Legacy (JS/Flow).
  - Refer to `docs/architecture.md` to understand stack boundaries.

- **Analyze the architecture** to target intervention areas:
  - Where the ticket-related logic lives (format-specific vs `common/`).
  - Compatibility constraints (ES5/older devices on legacy).

### 3) Expected output (handoff for a relay)

The agent must produce a concise, actionable output:

- **Ticket summary** (problem, measurable objective, definition of done).
- **Target stack** (Legacy/New) + justification.
- **Assumptions / unknowns** to confirm (instrumentation, repro conditions, etc.).
- **Likely code areas** (directories/files to inspect) and why.
- **Work plan** as steps + confirmed TODO list.
- **Risks & tests**:
  - how to validate (unit/integration/visual/e2e depending on the stack),
  - which signals to observe (events, timings, etc.).
