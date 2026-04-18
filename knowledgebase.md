---
name: knowledge-base
description: Analyze codebases and create persistent knowledge bases documenting architecture, patterns, and conventions. Used by other project skills to provide better context.
---

# Knowledge Base Skill

Analyze a codebase to create and maintain a persistent knowledge base documenting architecture, patterns, feature areas, integrations, and monitoring posture. This knowledge base is intended to support other skills and workflows by providing reusable project context.

## Quick Start

```bash
# Initialize knowledge base for a project
/knowledge-base init my-project

# Refresh when the codebase changes
/knowledge-base refresh my-project
```

---

## Commands

### 1. `/knowledge-base init <project-name>`

Analyze a codebase and create a persistent knowledge base documenting architecture, patterns, and conventions.

**When to use**: First time with a project, or after major architectural changes.

---

### 2. `/knowledge-base refresh <project-name>`

Update the knowledge base when the codebase has evolved significantly.

**When to use**: After major refactors, major dependency changes, or significant architectural updates. Recommended periodically.

---

## Configuration

This skill reads from a shared project configuration file such as `.claude/projects.json`.

**Template + local override pattern:**
1. **Template (committed)**: `.claude/projects.template.json`
2. **Local config (gitignored)**: `.claude/projects.json`

**First-time setup:**
```bash
cp .claude/projects.template.json .claude/projects.json
# Edit projects.json with your actual paths
```

**Example configuration:**
```json
{
  "my-project": {
    "path": "/Users/you/projects/my-project",
    "excludeDirs": ["node_modules", "dist", "build", ".git"],
    "sourceControl": {
      "provider": "github",
      "repository": "your-repo"
    }
  }
}
```

### Path Conventions

Output paths are derived from the project's `path` field. Skills do not need a separate output path configuration.

| Skill | Output Location |
|-------|-----------------|
| knowledge-base | `{path}/docs/knowledge-base/` |
| spec-generator | `{path}/docs/specs/` |
| review-related skills | `{path}/docs/reviews/` |

---

## How It Works

### Command 1: Initialize Knowledge Base

```bash
/knowledge-base init my-project
```

#### Execution Flow

**1. Validate Configuration**

```bash
python3 scripts/shared/config.py --project {project-name}
```

If `"success": true`, the project exists and its path is valid. Use `project_config.path` for subsequent file operations.

If `"success": false`, check `error`:
- `"config_not_found"` — guide the user through setup
- `"project_not_found"` — show available project names
- `"path_not_found"` — show the invalid path and tell the user to fix the configuration

**2. Check for Existing Knowledge Base**

```bash
python3 scripts/shared/knowledge_base.py --project {project-name} --status-only
```

If `"success": true` and `data.found` is `true`, a knowledge base already exists. Prompt the user:
- "Knowledge base already exists for {project} (last updated: {data.last_updated}). What would you like to do?"
- Options: "Refresh it" or "Reinitialize from scratch"

If `data.found` is `false`, proceed with fresh analysis.

**3. Choose Analysis Depth**

Prompt the user:
- Question: "How thorough should the analysis be?"
- Options:
  - "Quick (Recommended)" — major patterns and architecture, representative file samples
  - "Comprehensive" — deeper analysis, more examples, broader coverage

**4. Launch Codebase Analysis**

Use an exploration-oriented agent or workflow:

```text
Analyze the {project-name} codebase and document:

1. Tech Stack & Architecture
   - Languages, frameworks, libraries
   - Architecture style
   - Build tools and package managers

2. Project Structure
   - Directory organization
   - Feature or module boundaries
   - Naming conventions

3. Common Code Patterns
   - Component or module structure
   - State management approach
   - Integration patterns
   - Error handling patterns
   - Testing approaches

4. Feature Areas
   - Major features and their locations
   - Key files for each feature area
   - Domain boundaries

5. External Integrations
   - Service integrations and clients
   - Authentication methods
   - Request and response patterns
   - Existing endpoints or interfaces

6. Navigation / Routing
   - How routing or navigation works
   - Flow between pages, screens, or services

7. Development Gotchas
   - Quirks or limitations
   - Common mistakes to avoid
   - Important constraints

8. Monitoring & Observability
   - Logging framework and configuration
   - Log destinations
   - Structured logging patterns
   - Tracing and metrics setup
   - Error reporting tools
   - Health checks
   - Background processes or consumers
   - Existing alerting or monitoring definitions
   - Existing diagnostic queries
   - Data entities commonly used for verification

9. Development Environment & Platform
   - Platform-specific scripts
   - CI/CD pipeline target environments
   - Container usage
   - Line ending configuration
   - Setup instructions in docs
   - Shell assumptions in build scripts
   - Cross-platform concerns

Provide specific examples from the codebase where useful.
Focus on patterns that help future contributors implement new features safely and consistently.
```

**5. Generate Knowledge Base Files**

Create these files in `{project}/docs/knowledge-base/`:

- **architecture-overview.md**: tech stack, architecture, structure, platform profile
- **common-patterns.md**: reusable code patterns and conventions
- **feature-areas.md**: major features and their locations
- **external-integrations.md**: integration patterns, interfaces, authentication
- **observability.md**: monitoring tools, logging patterns, tracing, alerts, diagnostic queries

After writing the markdown files, generate `.metadata.json` using your project’s metadata script or a similar process.

Example:
```bash
python3 scripts/shared/knowledge_base.py --project {project-name} --write-metadata \
    --analysis-depth quick \
    --files-analyzed 247 \
    --tech-stack "TypeScript,React,Node.js" \
    --feature-areas 5
```

The metadata writer should handle timestamps, preserve `createdAt` on refreshes, and write consistent JSON output.

**6. Validate Output**

```bash
python3 scripts/shared/knowledge_base.py --project {project-name} --validate
```

If `data.validation.valid` is `true`, continue to summary.

If `data.validation.valid` is `false`, inspect `data.validation.errors`, fix the issues, and validate again before finishing.

**7. Display Summary**

```text
Knowledge base initialized for {project-name}

Analysis Summary:
- Files analyzed: 247
- Tech stack: TypeScript + React + Node.js
- Architecture: Feature-based modular application
- Feature areas: 5 documented
- Integration pattern: service clients with structured interfaces
- Observability: logging, tracing, and alerting documented
- Platform: cross-platform local development, CI pipeline, containerized deployment

Knowledge base saved to:
   /path/to/my-project/docs/knowledge-base/

Ready to use with downstream skills and workflows.
```

#### Error Handling

Errors from configuration or knowledge base scripts should be returned as structured JSON and mapped to clear next steps.

---

### Command 2: Refresh Knowledge Base

```bash
/knowledge-base refresh my-project
```

#### Execution Flow

**1. Validate Configuration**

```bash
python3 scripts/shared/config.py --project {project-name}
```

Handle configuration errors the same way as in the init flow.

**2. Check Existing Knowledge Base**

```bash
python3 scripts/shared/knowledge_base.py --project {project-name} --status-only
```

If `data.found` is `false`, tell the user:
`No knowledge base found. Run /knowledge-base init {project-name} first.`

If `data.found` is `true`, display the current state using fields such as `last_updated`, `days_old`, and `stale`.

**3. Load Existing Knowledge Base Files**

```bash
python3 scripts/shared/knowledge_base.py --project {project-name}
```

Load the existing files so the analysis can compare them against the current state of the codebase.

**4. Ask What to Refresh**

Prompt the user:
- Question: "How should I refresh the knowledge base?"
- Options:
  - "Quick scan (Recommended)" — check for new files, features, and major pattern changes
  - "Full refresh" — complete re-analysis and rewrite
  - "Targeted refresh" — update only selected sections

**5. Run Refresh Analysis**

Run the same analysis as init, but compare against the existing documentation and focus on identifying meaningful changes.

**6. Show Diff of Changes**

```text
Analysis complete. Comparing with existing knowledge base...

Changes Detected:

architecture-overview.md:
+ Added: updated framework and dependency notes
~ Modified: architecture summary

common-patterns.md:
+ Added: new reusable patterns
~ Modified: integration and testing examples

feature-areas.md:
+ Added: new feature area
+ Added: newly discovered files and folders

external-integrations.md:
~ Modified: authentication or client usage patterns
- Removed: outdated integration references

observability.md:
+ Added: new monitoring coverage
~ Modified: alerts, logging, or diagnostic references

Summary:
Files analyzed: 262
Significant changes: 5 documentation files
New patterns: 2
New features: 1
```

**7. Ask for Confirmation**

Prompt the user:
- Question: "Update knowledge base with these changes?"
- Options:
  - "Yes, save all updates"
  - "No, keep existing knowledge base"
  - "Review changes in detail first"

**8. Save Updates**

- Overwrite the markdown files with updated content
- Update metadata while preserving the original `createdAt`
- Validate the output again
- Show a completion message

```text
Knowledge base updated

Location: {project}/docs/knowledge-base/
Updated: {current-date}
Files: 262 analyzed

Ready for use with downstream skills and workflows.
```

---

## Output Structure

### Knowledge Base Files

```text
{project}/docs/knowledge-base/
├── architecture-overview.md
├── common-patterns.md
├── feature-areas.md
├── external-integrations.md
├── observability.md
└── .metadata.json
```

### Observability File Structure

The `observability.md` file captures the project’s monitoring posture and should be useful to any downstream workflow that needs operational context.

```markdown
# Observability Knowledge Base

**Project:** {project-name}
**Generated:** {date}
**Last Updated:** {date}

## Tools & Configuration

| Tool | Purpose | Package/SDK | Configuration |
|------|---------|-------------|---------------|
| {tool} | {purpose} | {package} | {config location} |

## Services / Components

| Service or Component | Type | Identifier | Deployment |
|----------------------|------|------------|------------|
| {name} | {type} | {identifier} | {deployment} |

## Logging Patterns

### Framework & Destinations
- Framework: {logging framework}
- Destinations: {console, file, telemetry backend, etc.}
- Structured logging: {yes/no, with examples}

### Log Source Identifiers
List identifiers used to filter logs by feature, service, or module.

| Identifier | Feature or Area | File |
|------------|------------------|------|
| {identifier} | {feature} | {path} |

### Success / Failure Logging Patterns
Document how the codebase records successful operations and failures.

## Tracing

### Tracing Configuration
Describe how tracing is configured, what exporters are used, and whether custom trace sources exist.

### Distributed Tracing
Describe whether correlation is propagated across services and how.

## Background Jobs / Workers

| Job or Process | Purpose | Monitoring | Health Check |
|----------------|---------|------------|--------------|
| {name} | {purpose} | {details} | {yes/no} |

## Queue / Event Monitoring

| Queue or Stream | Consumer | Current Monitoring | Alert Threshold |
|-----------------|----------|--------------------|-----------------|
| {name} | {consumer} | {details} | {threshold} |

## Existing Alerts

| Alert | Signal | Condition | Action | Severity |
|-------|--------|-----------|--------|----------|
| {name} | {signal} | {condition} | {action} | {severity} |

## Existing Diagnostic Queries

List any existing queries or investigation patterns found in code, docs, or comments.

## Data Entities for Verification Queries

Key tables, collections, or entities that can be used to verify whether important operations completed successfully.

| Entity | Key Fields | Verification Use |
|--------|------------|------------------|
| {name} | {fields} | {purpose} |

## Monitoring Coverage Summary

| Category | Coverage | Tools Used | Gaps |
|----------|----------|------------|------|
| Infrastructure | {Partial/Full} | {tools} | {gaps} |
| Interfaces | {Partial/Full} | {tools} | {gaps} |
| Business Operations | {Partial/Full} | {tools} | {gaps} |
| Background Processes | {Partial/Full} | {tools} | {gaps} |
| Error Rates | {Partial/Full} | {tools} | {gaps} |
| Success / Throughput | {Partial/Full} | {tools} | {gaps} |
```

### Architecture Overview — Development Environment & Platform Section

The `architecture-overview.md` file should include a **Development Environment & Platform** section that captures the project’s platform story and helps downstream skills make better suggestions.

```markdown
## Development Environment & Platform

### Project Platform Profile
- **Primary development OS(es):** {windows, macOS, linux, cross-platform}
- **CI/CD platform:** {platform}
- **Deployment target:** {container, VM, serverless, desktop, mobile, etc.}
- **Container-based development:** {yes/no}

### Platform-Specific Scripts Found

| Script | Platform | Purpose |
|--------|----------|---------|
| {script} | {platform} | {purpose} |

### Cross-Platform Considerations
- **Line endings:** {configured/not configured}
- **Path handling:** {style}
- **Docker:** {yes/no}
- **Dev containers:** {yes/no}
- **Build scripts:** {shell assumptions or cross-platform tooling}

### For Downstream Skills

When generating commands, scripts, or file paths for this project:

1. Use the user's runtime OS for command syntax.
2. Follow the project's existing script patterns where practical.
3. Respect the path conventions already in use.
4. Prefer tools already present in the project.
5. Use containerized alternatives when platform differences are significant.
6. Consider CI/CD target environments when suggesting scripts or automation.
```

### `.metadata.json` Format

```json
{
  "projectName": "my-project",
  "createdAt": "2026-01-15T10:30:00Z",
  "updatedAt": "2026-02-20T14:00:00Z",
  "analysisDepth": "quick",
  "filesAnalyzed": 247,
  "featureAreas": 5,
  "techStack": ["TypeScript", "React", "Node.js"],
  "observabilityAnalyzed": true,
  "deploymentType": "containerized",
  "observabilityTools": ["logging", "tracing", "error reporting"],
  "platformProfile": {
    "primaryDevOS": ["windows", "macos"],
    "ciPlatform": "ubuntu",
    "deploymentTarget": "linux-container",
    "hasDevContainer": false,
    "hasDocker": true,
    "scriptTypes": ["ps1", "sh"],
    "crossPlatform": true
  },
  "version": "1.0"
}
```

---

## Token Budget

| Depth | Tokens | Best For |
|-------|--------|----------|
| Quick init | ~50k | Most projects, first-time setup |
| Comprehensive init | ~100k | Complex or large codebases |
| Quick refresh | ~30k | Regular updates |
| Full refresh | ~80k | Major refactors |

---

## Error Handling Reference

| Error | Source | Action |
|-------|--------|--------|
| `config_not_found` | config loader | Guide setup of local config |
| `project_not_found` | config loader | Show available project names |
| `path_not_found` | config loader | Ask user to fix the configured path |
| KB not found (refresh) | knowledge base status check | Show init command |
| KB stale | knowledge base status check | Inform user and suggest refresh |
| Analysis fails | analysis agent or script | Offer retry or reduced depth |
| No write permissions | file write error | Show output path and suggest permission fix |

---

## Best Practices

### When to Initialize
- First time with a project
- After major architectural changes
- When patterns or conventions have changed significantly

### When to Refresh
- Periodically
- After major feature additions
- When the knowledge base feels outdated

### Choosing Depth

**Quick**:
- Standard projects
- Regular updates
- Time-sensitive analysis

**Comprehensive**:
- Large or complex codebases
- First-time analysis of unfamiliar systems
- Projects with many patterns or integrations

### Tips
- Commit knowledge base files to version control
- Review generated documentation and improve it with project context
- Downstream skills work best with a fresh knowledge base
- Refresh before major planning or implementation work