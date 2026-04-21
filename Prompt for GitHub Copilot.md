The mission for you now is to generating a complete `.github/` directory that enables GitHub Copilot to maintain this codebase without exploring it from scratch. The output must be tailored to *THIS specific repository*, not any other generic codebases.

**Tech stack assumption**: C#, .NET (detect the actual version from `.csproj` files). The database engine, ORM/data access approach, and frontend presence will be **auto-detected** — do not assume any specific database or framework.

**Supported database engines** (detect, don't assume): SQL Server, Azure SQL Database, PostgreSQL, MySQL, MariaDB, SQLite, Oracle Database, Snowflake, or none.

**Supported data access** (detect, don't assume): Entity Framework Core, Dapper, raw ADO.NET, RepoDB, Linq2db, or custom.

---

## Phase 0 — Deep Codebase Exploration (MANDATORY)

**Do NOT skip this phase. Do NOT generate anything until exploration is complete.**

Exploration has two stages: (1) run the automated PowerShell script to gather structured facts fast, (2) read specific files to fill gaps the script cannot cover.

### Stage 1 — Run the Exploration Script

Download the exploration script from GitHub and execute it from the repository root:

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/phanxuanquang/dotnet-project-agent/refs/heads/main/explore-codebase.ps1" -OutFile "explore-codebase.ps1"; .\explore-codebase.ps1
```

This script is designed to quickly gather a wide range of structured information about the codebase, which will inform all subsequent steps. It uses a combination of file system scanning, static code analysis, and heuristic patterns to detect key aspects of the project.

This script automatically detects and reports:
- Solution and project files (target framework, packages, project references)
- Database provider(s) (SQL Server, PostgreSQL, MySQL, MariaDB, SQLite, Oracle, Snowflake, Azure SQL)
- ORM / data access approach (EF Core, Dapper, ADO.NET, RepoDB, Linq2db)
- EF Core specifics if present (DbContext files, migration folders, entity/model directories)
- Non-EF data access patterns if EF Core is absent (files with raw SQL, Dapper calls, ADO.NET usage)
- SQL script files
- Connection string locations and provider hints
- Frontend signals (config files, directories, file extensions, package.json analysis including framework, state management, UI library, build tool, test framework)
- Test projects and frameworks
- CI/CD and DevOps files
- Entry points (Program.cs, Startup.cs)
- Directory tree (depth 4)
- Configuration files
- Code convention signals (file-scoped namespaces, nullable, records, primary constructors, etc.)

**Parse the script output carefully.** Record these key variables:
- `DB_ENGINE` = detected database engine(s), or `NONE`
- `DATA_ACCESS` = detected ORM/data access approach(es), or `NONE`
- `HAS_EFCORE` = true/false
- `HAS_FRONTEND` = true/false
- `FRONTEND_FRAMEWORK` = detected framework, or N/A

### Stage 2 — Deep-Read Specific Files

The script gives you the "what" and "where". Now read files to understand the "how".

### 0.1 — Solution & Project Structure
- The script already listed `.sln`, `.csproj`, target frameworks, packages, and references
- **Now read**: entry points (`Program.cs`, `Startup.cs`) to understand DI setup and hosting

### 0.2 — Backend Architecture
- Identify the application type: Web API, Worker Service, Console App, Blazor, MVC, gRPC, etc.
- Read the DI/hosting setup (`Program.cs`, `Startup.cs`)
- Identify major patterns: Repository pattern, CQRS, MediatR, clean architecture layers, vertical slices, etc.
- Find the main business logic files (controllers, workers, handlers, services)
- Map external integrations (HTTP clients, message queues, third-party APIs, Azure services, etc.)

### 0.3 — Data Layer

The approach here depends on what the script detected.

**IF `HAS_EFCORE = true`**:
- Read each DbContext file the script found — extract DbSet names, OnModelCreating relationships
- List all entity/model classes with their primary keys, relationships, and table/column mappings
- Check for `[Table]`, `[Column]`, `[PrimaryKey]` attributes and Fluent API configuration
- Find migrations or SQL schema scripts
- Identify the database name, schema(s), connection string config key
- Map lookup/enum tables and seed data (both EF HasData and SQL INSERT scripts)

**IF `HAS_EFCORE = false` (Dapper, ADO.NET, raw SQL, or other)**:
- Read the files the script flagged with data access patterns
- Identify the repository/data access layer pattern (if any)
- Find stored procedures, inline SQL, or query files
- Map the data models/DTOs used for query results
- Identify how connections are managed (connection string injection, connection factory, using statements)
- Find SQL schema scripts or migration tools (DbUp, FluentMigrator, custom scripts)
- Determine if there's a schema definition source of truth (SQL files, migration scripts, or code-first)

**IF `DB_ENGINE = NONE`**:
- Verify the project genuinely has no database (it might use file storage, in-memory, or external APIs only)
- Skip the Database Schema section in Phase 1

### 0.4 — Configuration
- Read `appsettings.json` and all variants (`appsettings.*.json`)
- Find all options/settings classes (bound via `Configure<T>` or `IOptions<T>`)
- Identify secrets management approach (user secrets, Key Vault, environment variables, plaintext)

### 0.5 — Frontend Detection
The script already detected frontend signals. If `HAS_FRONTEND = true`:
- Read the main frontend entry point and routing configuration
- Identify component structure patterns
- Check for API client layer (how frontend calls the backend)

### 0.6 — Testing
- The script already listed test projects and frameworks
- Skim a few test files to understand test patterns (arrange-act-assert, test fixtures, mocking)

### 0.7 — CI/CD & DevOps
- The script already listed CI/CD files
- Read them to understand build/deploy pipeline stages

### 0.8 — Code Conventions
- The script already detected convention signals (namespaces, nullable, records, etc.)
- Read 3-5 representative source files to confirm and extract:
  - Naming conventions (PascalCase, camelCase, prefixes)
  - File organization patterns
  - Error handling approach
  - Logging approach
  - DI registration patterns

---

## Phase 1 — Knowledge Base Generation

Create the `.github/knowledge-base/` directory with structured data files that the agents can query for quick facts. This includes:
- `concepts.json` — key business concepts, patterns, and architectural elements in THIS project.
- `glossary.json` — definitions of technical terms, especially those specific to THIS codebase.
- `dependency-graph.json` — a graph of project dependencies, including external integrations.
- `entity-relationship.json` — for EF Core projects, a structured representation of entities, relationships, and mappings.
- `docs-references.json` — links to official documentation about the technologies used, concepts, glossary terms, and patterns found in THIS codebase. Ensure the references are relevant to the actual tech stack and patterns in this project (e.g., if using Dapper, include links to Dapper documentation and best practices; if using React, include links to React docs and style guides). Ensure the reference hyperlinks exist and are not broken before including them.
- `code-conventions.md` — detailed patterns and conventions extracted from THIS codebase, organized by category (naming, error handling, async patterns, etc.) with examples.
- `architecture-overview.md` — a structured summary of the overall architecture, including application type, layers, data flow, and key components.

**Notes:** Additional files can be created if the exploration revealed other important structured information that would benefit the agents (e.g., `api-endpoints.json` for API projects, `message-queues.json` for projects using queues, etc.). Focus on information that is specific to THIS codebase and would be useful for the agents to reference.

---

## Phase 2 — Generate `.github/copilot-instructions.md`

Create the master system instructions file. This is the most important file — it's the single source of truth for all AI agents working on this codebase.

### Required Sections

```markdown
# {Project Name} — AI Agent System Instructions

## Overview
{Overview or intial introduction about this repository for the AI Agent to have a basic understanding}

## OODA Decision-Making Loop
{Copy this section exactly — it's methodology, not project-specific}

**Every task MUST follow the OODA loop.**

### Observe — Gather facts before acting
- Read before writing: Always read relevant source files before modifying them.
- Search before assuming: Use search tools to locate code — don't assume file paths or symbol names.
- Check current state: Before adding something, verify it doesn't already exist.
- Collect errors: After edits, check for compile/lint errors immediately.
- Verify the architecture: Understand how the change fits into the overall structure before proceeding.

### Orient — Analyze and contextualize
- Match conventions: New code must follow existing patterns.
- Map dependencies: Trace how the change flows through the project architecture.
- Identify doc impact: Check which docs need updating.
- Prioritize: Prefer the approach that changes fewer files and follows established patterns. Follow the current architectural style.

### Decide — Choose the approach explicitly
- Plan multi-step work: Create a todo list before writing code.
- Pick the right skill: Match the task to an existing agent skill and follow its playbook.
- Respect dependency order.

### Act — Execute with precision, then verify
- One concern at a time: Focused edits, but do not break the current architecture.
- Build after changes: Verify compilation.
- Update docs last: After all code changes are verified.
- Confirm completion.

### Loop — Re-observe if something unexpected happens

## Project Identity
{1-2 paragraphs: what this project IS, what it DOES, who it's FOR, what tech stack}

## Solution Structure
{ASCII tree of the ACTUAL directory layout — same style as the exploration output}
{Annotate each file/folder with a brief comment}

## Data Layer
{Include this section if any database is used. Adapt to the detected approach.}

### Database Engine
{State the detected engine: SQL Server, Azure SQL, PostgreSQL, MySQL, MariaDB, SQLite, Oracle, Snowflake}
{Note any engine-specific conventions: schema support, identity columns, sequences, JSONB, etc.}

### Data Access Approach
{State the detected approach: EF Core, Dapper, ADO.NET, RepoDB, Linq2db, mixed, etc.}

**IF EF Core**:
### Entity Relationship Map
{ASCII diagram showing all entities, PKs, FKs, relationships using arrows}
### Column Remappings
{Table of C# property → SQL column name where they differ, using [Column("...")] attributes}
### Lookup Tables
{Table of lookup/enum tables with their values}

**IF Dapper / ADO.NET / other**:
### Data Models
{List all data model/DTO classes with their corresponding table/query}
### Repository/Data Access Layer
{Describe the pattern: repository classes, query objects, stored procedures, inline SQL}
### Schema Source of Truth
{Where is the schema defined: SQL migration scripts, DbUp, FluentMigrator, manual DDL files, or none}

## External Integrations
{HTTP clients, APIs, message queues, Azure services, etc.}
{Auth method, rate limiting, pagination patterns}

## Worker/Pipeline/Controller Architecture
{Describe the main processing flow — ETL phases, request pipeline, controller structure, etc.}

## Code Conventions
{All patterns discovered in Phase 0.8 — be specific with examples from THIS codebase}

## Configuration
{JSON structure of appsettings.json with comments}
{List all options classes}

## Key Dependencies
{Table: Package | Version | Project — from actual .csproj files}

## Mandatory Documentation Maintenance Rule
{Impact matrix: what changed → which docs to update}
{Documentation file index: all doc files with their purpose}

## Sub-agent Index
{Table: Agent | File | Role | Primary Skills | Hands Off To}

## References
{Relative file paths to all generated knowledge base files (created in the `.github/knowledge-base/`), skill playbooks, and agent files for easy access}
```

**Rules for this file**:
- Every fact must come from the codebase exploration — no guessing
- File paths must be real, verified paths
- Entity names must match actual class names
- Config keys must match actual appsettings.json
- Keep it dense — tables over prose
- Concise, clear, specific - avoid verbose explanations or generic statements. 

**Notes:**
- Allow the AI Agent optionally create sub-agents and agent skills in case the task requires specialization that would benefit from a dedicated agent or skill playbook. Focus on tasks that are common and have non-trivial patterns in THIS codebase. The created sub-agents and skills must be *deleted* once the task is complete, to avoid clutter and confusion for future tasks.

---

## Phase 3 — Generate Agent Skills (`.github/prompts/`)

Create skill playbooks as `.prompt.md` files. Each skill is a step-by-step guide for a specific recurring task.

### Always-Generate Skills (for any .NET project)

| Skill File | Purpose | Key Content |
|---|---|---|
| `dotnet-cli.prompt.md` | .NET CLI mastery | Solution paths, build/run/test/publish commands, NuGet management, scaffolding |
| `database.prompt.md` | Database mastery (engine-specific) | Schema overview from THIS project, DDL patterns, diagnostic queries, type mappings. **Title and content must match the detected DB engine** (e.g., "SQL Server & SQL Mastery", "PostgreSQL Mastery", "SQLite Mastery"). If multiple engines: create one file covering all, with engine-specific sections. If no database: skip this skill. |
| `research.prompt.md` | Research & info gathering | Tool selection guide, search strategies, codebase/web/docs research patterns |
| `update-docs.prompt.md` | Post-change doc maintenance | Impact matrix, doc file index, verification checklist |
| `update-dependencies.prompt.md` | NuGet package updates | Package inventory from THIS project's .csproj files, breaking change warnings |
| `debug-{app-type}.prompt.md` | Debugging playbook | Common failure scenarios specific to THIS application type, diagnostic queries, log locations |
| `csharp-dotnet-best-practices.prompt.md` | C#/.NET best practices | Performance, memory efficiency, maintainability, scalability patterns **tailored to THIS codebase**. Must include: (1) collections & LINQ patterns actually used, (2) EF Core or data access patterns used, (3) async/await patterns, (4) string handling patterns, (5) numeric type choices with rationale, (6) allocation reduction techniques, (7) scope & lifetime management, (8) type design conventions (class vs record vs sealed record), (9) null handling patterns, (10) DI registration patterns, (11) error handling patterns, (12) batch processing & rate limiting if applicable, (13) C# language feature inventory with version. Every code example must come from the actual codebase. |

**Note:** Optionally create additional agent skills if the exploration revealed other recurring tasks that would benefit from a playbook (e.g., "add API integration", "modify processing pipeline", "add React component", etc.). Focus on tasks that are common and have non-trivial patterns in THIS codebase.

### Conditionally-Generate Skills (based on codebase features)

| Condition | Skill File | Purpose |
|---|---|---|
| Has EF Core entities | `add-entity.prompt.md` | Add new entity: model → DbContext → migration/DDL → business logic |
| Has EF Core entities | `add-entity-field.prompt.md` | Add field to existing entity: property → column → mapping → transformation |
| Has non-EF data access | `add-data-model.prompt.md` | Add new data model/DTO, repository method, SQL query/stored procedure, migration script |
| Has non-EF data access | `add-data-field.prompt.md` | Add field to existing model: property → SQL column → query update → migration |
| Has lookup/enum tables | `add-lookup-table.prompt.md` | Add lookup table: SQL → seed data → optional C# enum → FK |
| Has worker/pipeline | `modify-pipeline.prompt.md` | Modify the processing pipeline: phases, error handling, retry logic |
| Has external API client | `add-api-integration.prompt.md` | Add new external API endpoint: interface → implementation → caller |
| Has controllers/endpoints | `add-endpoint.prompt.md` | Add new API endpoint: controller → service → validation → tests |
| Has frontend (React) | `add-component.prompt.md` | Add React component: file structure, props, state, styling, tests |
| Has frontend (Angular) | `add-component.prompt.md` | Add Angular component: module, component, template, service, tests |
| Has frontend (Blazor) | `add-component.prompt.md` | Add Blazor component: razor, code-behind, DI, parameters |
| Has frontend (Vue) | `add-component.prompt.md` | Add Vue component: SFC structure, composables, props, emits |
| Has message queues | `add-message-handler.prompt.md` | Add queue message handler |
| Has CI/CD pipelines | `modify-pipeline-ci.prompt.md` | Modify CI/CD pipeline |

**Note:** Optionally create additional conditional skills based on specific patterns or technologies detected in the codebase (e.g., "add Blazor component", "add React component", "add message handler", "modify CI/CD pipeline", etc.). Focus on tasks that are common and have non-trivial patterns in THIS codebase.

### Skill File Format

Every skill must follow this structure:

```markdown
# {Skill Title}

{One-line description of when to use this skill}

## Context
{Brief architecture context relevant to this skill — file paths, patterns, dependencies}

## Steps

### Step 1 — {Action}
{What to do, which file to edit, what pattern to follow}
{Code example from THIS codebase showing the pattern}

### Step 2 — {Action}
...

## Checklist
- [ ] {Verification item}
- [ ] {Build passes}
- [ ] **Documentation updated** (run `update-docs` skill)
```

**Rules**:
- Every file path must be real
- Every code example must match actual codebase patterns
- Every checklist ends with the documentation update item

---

## Phase 4 — Generate Sub-Agents (`.github/agents/`)

### Agent File Format

```markdown
---
name: {AgentName}
description: {One-line role description specific to THIS project}
argument-hint: {What input this agent expects}
tools: [{tool list}]
---

# {AgentName} Agent

{One paragraph: role, expertise, project context}

## OODA Loop ({Focus} Focus)
> *ODDA Loop is known as the Observe-Orient-Decide-Act loop.*
{4 phases tailored to this agent's specialty}

## Primary Skills
{Table mapping task types to skill files}

## Key Files
{Table mapping concerns to actual file paths in THIS project}

## Boundaries
- **DO**: {what this agent does}
- **DON'T**: {what this agent must NOT do}
- **HAND OFF**: {when to delegate to other agents, with @mentions}
```

**Note:** Except from mentioned sub-agents below, you can optionally create additional sub-agents based on specific roles or responsibilities that would benefit from specialization in THIS codebase (e.g., "API Integration Agent", "Frontend Specialist Agent", "Message Queue Agent", etc.). Focus on roles that are distinct and have non-trivial patterns in THIS codebase.

### Always-Generate Agents

| Agent | Name | Tools | Role | Primary Skills |
|---|---|---|---|---|
| Debugger | `debugger.agent.md` | read, search, execute, edit, web, todo | Error diagnosis and fixing | `debug-*`, `database`, `research`, `dotnet-cli` |
| Researcher | `researcher.agent.md` | read, search, web, codebase exploration | READ-ONLY info gathering | `research` |
| Planner | `planner.agent.md` | read, search, todo | Task decomposition, coordination (PLANNING-ONLY, no edits) | All skills (reference) |
| Tester | `tester.agent.md` | read, search, execute, edit, todo | Test creation and validation | `dotnet-cli` |
| DatabaseAdmin | `database-admin.agent.md` | read, search, execute, edit, todo | DB schema, DDL, queries (engine-specific) | `database`, `add-lookup-table` |
| DocumentationWriter | `documentation-writer.agent.md` | read, search, edit, web, todo | Markdown docs, diagrams, guides | `update-docs`, `research` |

### Developer Agent(s) — Frontend-Dependent Logic

**IF `HAS_FRONTEND = false`**:
- Create ONE agent: `developer.agent.md` with name `Developer`
- Skills: all code-related skills (add-entity, modify-pipeline, update-dependencies, dotnet-cli, update-docs, etc.)
- Hands off to: @DatabaseAdmin (DDL), @Tester (tests), @Researcher, @Debugger

**IF `HAS_FRONTEND = true`**:
- Create TWO agents:
  1. `backend-developer.agent.md` with name `BackendDeveloper`
     - Description: "Specialist for C# backend development — {project-specific description}"
     - Skills: all BACKEND code skills (add-entity, modify-pipeline, update-dependencies, dotnet-cli, update-docs)
     - Boundaries: DO backend C#, DON'T touch frontend files or break the current backend architecture. HAND OFF frontend → @FrontendDeveloper
  2. `frontend-developer.agent.md` with name `FrontendDeveloper`
     - Description: "Specialist for {Framework} frontend development — {project-specific description}"
     - Tools: read, search, execute, edit, todo, web
     - Skills: `add-component` + any frontend-specific skills
     - OODA loop tailored to frontend: component design, state management, styling, accessibility
     - Key files: component directories, routing config, state stores, API client layer, style files
     - Boundaries: DO frontend code, DON'T touch C# backend/database or break the current frontent architecture. HAND OFF backend → @BackendDeveloper, DB → @DatabaseAdmin
- Update ALL cross-references:
  - @Planner must list both @BackendDeveloper and @FrontendDeveloper
  - @Debugger hands off to @BackendDeveloper OR @FrontendDeveloper based on error location
  - @Tester covers both backend and frontend tests
  - Sub-agent index table includes both developers

### DocumentationWriter Agent — Required Traits

The DocumentationWriter agent MUST include:
- **Writing principles**: concise > verbose, concrete > abstract, structure > prose, diagrams where they help, junior-friendly
- **Intent clarification step**: always confirm what/who/how-deep/format before writing
- **Multi-file guidance**: split large docs into focused files under `docs/`, keep README short with links
- **Mermaid diagram templates**: architecture, data flow, ER diagram, sequence diagram — using THIS project's actual components
- **Markdown style guide**: heading hierarchy, code block languages, table limits, list nesting limits
- **Document templates**: README template, architecture doc template — pre-filled with THIS project's structure

---

## Phase 5 — MCP Server Discovery & Setup

Before cross-referencing, search for and configure MCP (Model Context Protocol) servers that will give future AI Agent sessions access to authoritative documentation, package intelligence, and database tooling for THIS project's detected tech stack.

### 5.1 — Microsoft Docs / Learn MCP

Search the web for the latest **Microsoft Learn MCP server** (an MCP server that can search and fetch official Microsoft / .NET / Azure documentation). Configure it so the AI Agent can query `learn.microsoft.com` content on demand.

**Recommended candidates** (verify availability before configuring):
| MCP Server | Source | Capabilities |
|---|---|---|
| VS Code built-in Microsoft Docs MCP | Enabled via VS Code MCP settings | `microsoft_docs_search`, `microsoft_docs_fetch`, `microsoft_code_sample_search` — search and fetch official Microsoft Learn documentation |

If the built-in server is not available, search for a standalone MCP server package that wraps the Microsoft Learn API.

### 5.2 — NuGet Package Intelligence MCP

Search the web for an MCP server that provides **NuGet package version lookup, compatibility checking, or dependency resolution** (e.g., a server wrapping the NuGet API).

**Recommended candidates** (verify availability before configuring):
| MCP Server | Source | Capabilities |
|---|---|---|
| VS Code built-in NuGet MCP | Enabled via VS Code MCP settings | `get-latest-package-version`, `get-nuget-solver`, `get-package-context`, `update-package-to-version` |
| `artmann/package-registry-mcp` | `npx -y package-registry-mcp` | Search and get up-to-date info about NuGet (+ NPM, Cargo, PyPi) packages |
| `sammcj/mcp-package-version` | `npx -y mcp-package-version` | Suggest latest stable package versions when writing code |

Pick the one that is currently available, actively maintained, and covers NuGet. Add its configuration to `.vscode/mcp.json`.

### 5.3 — Database MCP (Engine-Specific)

**Only if `DB_ENGINE ≠ NONE`.** Search the web for the best MCP server matching the detected database engine(s). Prefer **official** or **top-rated / widely adopted** servers.

Use the table below as a starting point — **verify each candidate is still available and actively maintained** before configuring:

| Detected Engine | Recommended MCP Server | Source / Install | Notes |
|---|---|---|---|
| **SQL Server / Azure SQL** | VS Code built-in MSSQL MCP | Enabled via VS Code MCP settings | `mssql_connect`, `mssql_run_query`, `mssql_list_tables`, `mssql_list_databases`, etc. Official Microsoft. |
| **SQL Server / Azure SQL** | `mbentham/SqlAugur` | `dotnet tool install -g SqlAugur` | C#-based, AST query validation, read-only safety, schema exploration, ER diagrams. |
| **SQL Server / Azure SQL** | `wenerme/wener-mssql-mcp` | `npx -y wener-mssql-mcp` | TypeScript, schema inspection and query capabilities. |
| **PostgreSQL** | `crystaldba/postgres-mcp` | `uvx postgres-mcp` | All-in-one: performance analysis, tuning, health checks. Top-rated. |
| **PostgreSQL** | `modelcontextprotocol/server-postgres` | `npx -y @modelcontextprotocol/server-postgres` | Official MCP reference server. Schema inspection + queries. |
| **MySQL** | `designcomputer/mysql_mcp_server` | `uvx mysql_mcp_server` | Configurable access controls, schema inspection, security guidelines. |
| **MySQL** | `benborla29/mcp-server-mysql` | `npx -y @benborla29/mcp-server-mysql` | Node.js-based, configurable access controls. |
| **MariaDB** | `skysqlinc/skysql-mcp` | See repo | Official SkySQL/MariaDB cloud MCP. Text-to-SQL and DB-level AI agents. |
| **SQLite** | `modelcontextprotocol/server-sqlite` | `uvx mcp-server-sqlite` | Official MCP reference server. Built-in analysis features. |
| **SQLite** | `jparkerweb/mcp-sqlite` | `npx -y @jparkerweb/mcp-sqlite` | Comprehensive SQLite interaction. |
| **Oracle Database** | `runekaagaard/mcp-alchemy` | `uvx mcp-alchemy` | Universal SQLAlchemy-based; supports Oracle, SQL Server, PostgreSQL, MySQL, MariaDB, SQLite, and more. |
| **Snowflake** | `Snowflake-Labs/mcp` | See repo (`uvx snowflake-mcp-server`) | Official Snowflake-Labs server. Cortex Agents, structured & unstructured data, RBAC. |
| **Multiple engines** | `TheRaLabs/legion-mcp` | `uvx legion-mcp` | Universal: PostgreSQL, MySQL, SQL Server, BigQuery, Oracle, SQLite, Redshift, CockroachDB. |

**Selection rules**:
1. If a **VS Code built-in MCP** exists for the detected engine (e.g., MSSQL), prefer it — zero setup required.
2. If an **official** server exists for the detected engine (marked 🎖️ in MCP registries), prefer it next.
3. Otherwise, pick the **most starred / most maintained** community server.
4. If multiple engines are detected, prefer a **universal** server (e.g., `legion-mcp`, `mcp-alchemy`) or configure one server per engine.
5. **Always verify** the server is still actively maintained (check last commit date, open issues) before adding it.

### 5.4 — Configuration

For each selected MCP server:
1. Add the server configuration to `.vscode/mcp.json` (for VS Code-native MCPs) or document the setup in `copilot-instructions.md`.
2. Document the server in the `research.prompt.md` skill so future sessions know what external tools are available.
3. Add any required connection strings or environment variable references (never hardcode credentials — use environment variables or secrets management).
4. If a server requires authentication, document the setup steps in `copilot-instructions.md` under a new **"MCP Servers"** section.

**If no suitable MCP server is found** for a category, document this gap in the Output Summary and recommend manual setup or alternative approaches.

---

## Phase 6 — Cross-Reference Verification

Before finishing, verify consistency across ALL generated files:

### Checklist
- [ ] Every file path mentioned in any doc exists in the actual codebase
- [ ] Every entity/class name matches the actual source code
- [ ] Sub-agent index in `copilot-instructions.md` lists ALL generated agents
- [ ] Documentation file index in `copilot-instructions.md` lists ALL generated files
- [ ] Documentation file index in `update-docs.prompt.md` lists ALL generated files
- [ ] Every agent's "HAND OFF" section references agents that actually exist
- [ ] Every agent's "Primary Skills" references skill files that actually exist
- [ ] No two agents claim ownership of the same file type (clear boundaries)
- [ ] Dependency order is consistent: schema/DDL → C# models/DTOs → DbContext or repository → business logic → tests → docs
- [ ] Database engine name is consistent everywhere (no "SQL Server" if using PostgreSQL)
- [ ] `database.prompt.md` skill content matches the actual detected engine(s) and data access approach
- [ ] If non-EF data access: skills reference repository/query patterns instead of DbContext/migration patterns
- [ ] Solution structure tree matches actual directory layout
- [ ] Package versions in Key Dependencies match actual `.csproj` files
- [ ] If `HAS_FRONTEND = true`: Developer is split into BackendDeveloper + FrontendDeveloper everywhere (no stale "Developer" references)
- [ ] Build command in all agents uses the actual `.sln` path
- [ ] Delete the exploration script (a.k.a the `explore-codebase.ps1` file) to avoid confusion for future tasks.

---

## Output Summary

When complete, list:
1. All files created with their paths
2. Total agent count and their names
3. Total skill count and their names
4. Whether frontend was detected (and what framework)
5. Database engine(s) and data access approach detected
6. Needed MCP servers configured with their capabilities
7. Any gaps or limitations discovered (e.g., no MCP server for the detected database, no official documentation available for a key technology, etc.)
8. Suggested next actions (e.g., "Ask @DocumentationWriter to generate a README")