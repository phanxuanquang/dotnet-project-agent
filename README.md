One prompt to bootstrap a full AI agent setup for any .NET repository. Supports both **GitHub Copilot** and **Claude Code**.

Send the appropriate prompt to the AI agent — it automatically downloads `explore-codebase.ps1`, scans the codebase, and generates a complete configuration: architecture docs, knowledge base, skills, agents, and conventions. No manual setup, no generic boilerplate.

## How it works

```mermaid
flowchart TD
    subgraph You
        A["Send Prompt.md\nto the AI agent"]
    end

    subgraph "AI Agent (Copilot / Claude Code)"
        B["Phase 0: Download & run\nexplore-codebase.ps1\n+ deep-read key files"]
        C["Phase 1: Generate knowledge base\n(concepts, glossary, conventions, architecture)"]
        D["Phase 2: Generate master instructions\n(.github/copilot-instructions.md or CLAUDE.md)"]
        E["Phase 3+: Generate skills, agents,\nrules, settings"]
        E2["MCP Server Discovery & Setup\n(Docs, NuGet, Database)"]
        F["Final: Cross-reference verification"]
    end

    A --> B --> C --> D --> E --> E2 --> F
```

1. **Send** `Prompt.md` (from `For GitHub Copilot/` or `For Claude Code/`) to the AI agent
2. The agent **downloads** `explore-codebase.ps1`, scans the repo, reads source files, then generates everything across multiple phases
3. **Review & commit** the generated output (`.github/` for Copilot, `CLAUDE.md` + `.claude/` for Claude Code)

## What `explore-codebase.ps1` detects

| Category | Details |
|---|---|
| **Projects** | `.sln` / `.csproj` files, target frameworks, NuGet packages, project references |
| **Database** | Engine (SQL Server, PostgreSQL, MySQL, SQLite, Oracle, Snowflake, Azure SQL) and connection strings |
| **Data access** | EF Core (DbContext, migrations, entities), Dapper, ADO.NET, RepoDB, Linq2db |
| **Frontend** | Framework, state management, UI library, build tool, test framework via `package.json` |
| **Tests** | Test projects and frameworks (xUnit, NUnit, MSTest) |
| **CI/CD** | Pipeline and DevOps configuration files |
| **Conventions** | File-scoped namespaces, nullable, records, primary constructors, naming patterns |
| **Structure** | Directory tree (depth 4), entry points (`Program.cs`, `Startup.cs`), config files |

Shared across both prompts — same script, same output.

## Usage Guidance

### GitHub Copilot

```mermaid
flowchart LR
    A["Open Copilot agent mode\nin VS Code"] --> B["Attach For GitHub Copilot/Prompt.md\nas context & send"]
    B --> C["Agent downloads & runs\nexplore-codebase.ps1"]
    C --> D["Review & commit\n.github/ directory"]
```

Generated output:
- `.github/copilot-instructions.md` — master instructions (single source of truth)
- `.github/knowledge-base/` — concepts, glossary, entity relationships, conventions, architecture
- `.github/prompts/*.prompt.md` — skill playbooks for recurring tasks
- `.github/agents/*.agent.md` — specialized sub-agents with scoped roles

### Claude Code

```mermaid
flowchart LR
    A["Start Claude Code\n(CLI or VS Code)"] --> B["Attach For Claude Code/Prompt.md\nas context & send"]
    B --> C["Agent downloads & runs\nexplore-codebase.ps1"]
    C --> D["Review & commit\nCLAUDE.md + .claude/"]
```

Generated output:
- `CLAUDE.md` — master instructions loaded automatically every session
- `.claude/knowledge-base/` — concepts, glossary, entity relationships, conventions, architecture
- `.claude/rules/*.md` — modular coding standards (auto-loaded or path-scoped)
- `.claude/skills/*/SKILL.md` — skill playbooks invokable via `/skill-name`
- `.claude/agents/*.md` — specialized subagents with isolated context and restricted tools
- `.claude/settings.json` — permission rules for allowed/denied tools and file access
