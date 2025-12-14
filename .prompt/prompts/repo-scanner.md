# Repository Scanner Prompt

You are a repository analysis tool specializing in static code structure extraction.

## Task

Scan the current project directory and generate two structured metadata files that will be stored in `.prompt/` for downstream analysis.

## Scan Rules

1. **Scope**: Analyze the entire codebase recursively.
2. **Exclusions**: Respect standard `.gitignore` rules to skip irrelevant files and directories.
3. **CRITICAL EXCEPTIONS**: ALWAYS include the following directories even if git-ignored:
   - `.prompt/`
   - `.copilot/`
   - `.claude/`
4. **Read Strategy**: Perform a shallow read for file tree and deep read for key files only.

## Output Requirements

Generate exactly two files:

### File 1: .prompt/repo-summary.md

A concise, structured summary containing:

#### 1. Project Metadata

- Project name (infer from root directory or package.json/setup.py/Cargo.toml)
- Primary language(s)
- Repository type (library / application / monorepo / tool / other)

#### 2. Technology Stack

- Core languages and versions (from lockfiles or config files)
- Main frameworks (React, Django, Express, Spring, etc.)
- Key libraries (top 5–8 most important dependencies)
- Build tools (Webpack, Vite, Cargo, Maven, etc.)
- Package manager (npm, yarn, pnpm, pip, cargo, etc.)

#### 3. Project Structure (File Tree)

- Show top 2 levels of directory hierarchy.
- For each top-level directory, provide a one-sentence purpose.
- List key entry points (main files, config files, scripts).

Example:

```text

src/          - Main source code (application logic)
tests/        - Test suites (unit + integration)
docs/         - Documentation and design specs
scripts/      - Build and deployment automation
.github/      - CI/CD workflows and Copilot instructions
.prompt/      - AI assistant prompt templates (THIS DIRECTORY)

```

#### 4. Key Commands

- Install/setup: `tool install` or equivalent
- Build: `tool build` or equivalent
- Run (dev): `tool run dev` or equivalent
- Run (prod): `tool run start` or equivalent
- Test: `tool test` or equivalent
- Lint/format: if present

#### 5. Known Constraints & Non-Goals

- Infer from README, CONTRIBUTING, or code comments.
- Examples: "No external API dependencies", "Must support Node 16+", "Browser-only, no SSR"

#### 6. Entry Points & Main Workflows

- Identify the main entry file(s) (e.g., `src/index.ts`, `main.py`, `cmd/server/main.go`).
- Describe the initialization flow (if obvious from code structure).

---

### File 2: .prompt/key-files-snapshot.md

A collection of full or partial content from the most important files for understanding the project.

#### Selection Criteria

Include (if they exist):

- readme file `README.md` (full content, max 500 lines)
- instructive file `CONTRIBUTING.md` or `DEVELOPMENT.md` (full content, max 300 lines)
- project metadata file `package.json` / `pyproject.toml` / `Cargo.toml` / `pom.xml` (full content)
- lint config file `tsconfig.json` / `jsconfig.json` / `.eslintrc` / `setup.cfg` (full content)
- product config file `docker-compose.yml` / `Dockerfile` (full content if < 100 lines)
- workflow file `.github/workflows/*.yml` (first workflow file only, max 100 lines)
- Main entry file (e.g., `src/index.ts`, `main.py`, first 50 lines)
- Root config files (e.g., `vite.config.ts`, `next.config.js`, first 50 lines)

#### Format

For each file, use this structure:

```text

---FILE: <relative-path>---
<content>

```

If a file is too long, include only the first N lines and add a marker:

```text

... (truncated, total 500 lines)

```

---

## Output Format (**STRICT**)

Return exactly two sections:

---FILE: .prompt/repo-summary.md---

```text

(content as specified above, in Markdown)

```

---FILE: .prompt/key-files-snapshot.md---

```text

(content as specified above, in Markdown)

```

## Constraints

- Use professional, technical language.
- Be concise: repo-summary.md should be 80–150 lines.
- Be selective: key-files-snapshot.md should be 300–800 lines total.
- Do NOT include any meta-commentary or explanation.
- Do NOT output chain-of-thought reasoning.
- Output ONLY the two file contents.
