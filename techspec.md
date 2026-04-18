---
name: spec-generator
description: Generate comprehensive technical specifications from work items by leveraging a project's knowledge base. Transforms high-level requests into detailed implementation specs.
---

# Spec Generator Skill

Transform high-level work items into comprehensive technical specifications by leveraging a project knowledge base, identifying similar patterns, and providing detailed implementation guidance.

## Quick Start

```bash
# Prerequisite: Initialize knowledge base first
/knowledge-base init my-project

# Generate spec from a work item
/spec-generator spec ITEM-123 my-project

# With explicit depth
/spec-generator spec ITEM-123 my-project --depth targeted
```

---

## Commands

### `/spec-generator spec <item-id> <project-name> [--depth quick|targeted|comprehensive]`

Generates a technical specification from a work item by analyzing the codebase and identifying implementation patterns.

**Example**: `/spec-generator spec ITEM-123 my-project --depth targeted`

---

## Deprecated Commands

The `init` and `refresh` commands belong to the **knowledge-base** skill:

```bash
/knowledge-base init my-project
/knowledge-base refresh my-project
```

---

## Configuration

This skill reads from the shared project configuration at `.claude/projects.json`.

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
    "workTracking": {
      "provider": "generic"
    },
    "sourceControl": {
      "provider": "github",
      "repository": "your-repo"
    }
  }
}
```

### Path Conventions

Output paths are derived from the project's `path` field:
- Knowledge base: `{path}/docs/knowledge-base/`
- Specs: `{path}/docs/specs/`

### Environment

- **Shared scripts**: This skill may use shared scripts for config validation, work item retrieval, and knowledge base loading.
- **Knowledge Base** (required): Run `/knowledge-base init <project>` before generating specs.
- **File system access**: Reads knowledge base, writes specs to `{project}/docs/specs/`

---

## How It Works

### Generate Specification

```bash
/spec-generator spec ITEM-123 my-project --depth targeted
```

#### Execution Flow

##### Phase 0: Setup & Validation

**1. Validate project config**
```bash
python3 scripts/shared/config.py --project {project-name}
```

If the project exists and its path is valid, use `project_config.path` for subsequent file operations.

If validation fails:
- `config_not_found` — guide the user through setup
- `project_not_found` — show available project names

**2. Load knowledge base**
```bash
python3 scripts/shared/knowledge_base.py --project {project-name}
```

If the knowledge base exists, use these files for context:
- `architecture-overview.md`
- `common-patterns.md`
- `feature-areas.md`
- `api-integration.md`
- `observability.md`

If the knowledge base is stale, suggest a refresh but continue.

If no knowledge base exists, stop and tell the user to run:
```bash
/knowledge-base init {project-name}
```

##### Phase 1: Fetch Work Item

**3. Fetch work item details**

Retrieve the work item from the configured tracking system if available. If automatic retrieval fails, fall back to manual input.

Prompt the user to provide:
```text
Title:

Description:

Acceptance Criteria:

Estimate (optional):
```

**4. Display Work Item Summary**

```text
Work Item: ITEM-123 - Add Summary Screen

Description: As a user, I want to view a summary before submitting...

Acceptance Criteria:
- Display relevant details
- Provide navigation to edit and submit
- Handle loading and error states

Estimate: 5
```

##### Phase 2: Pre-Analysis & Classification

**5. Classify work item type**

Analyze the request using knowledge base context.

Possible classifications:
- UI-Only
- Backend/API
- Full-Stack
- Background Process
- Integration
- Documentation or Non-Code

Also identify:
- likely feature area
- similar features or modules
- existing integrations or endpoints that may be reused
- whether new backend work appears necessary

**6. Ask about depth**

If depth is not provided via flag, ask the user:
- **Quick** — Knowledge base only. Best for small changes.
- **Targeted** — Knowledge base plus focused exploration. Best for most work.
- **Comprehensive** — Broader exploration. Best for complex or unfamiliar areas.

##### Phase 3: Targeted Codebase Exploration

**7. Build exploration mission**

Generate an exploration plan based on the work item type.

**For UI-focused work:**
- Read similar screens or components
- Identify navigation patterns
- Identify API usage patterns
- Identify reusable components and styling patterns
- Identify gaps between similar work and requested work

**For backend-focused work:**
- Read similar endpoints or services
- Identify request and response patterns
- Identify data model and validation patterns
- Identify error handling and authorization patterns

**For full-stack work:**
- Combine frontend and backend exploration
- Identify handoff points and shared models

**8. Launch exploration**

Use an exploration step or subagent to gather:
1. Similar implementations
2. Existing integrations already in use
3. Reusable components or utilities
4. Patterns to follow
5. Gaps that still need to be solved
6. What can be adapted versus built new

**9. Display exploration results inline**

Show the actual findings in a structured way. Do not compress them into a vague summary.

Example structure:

```text
Exploration Complete

Similar Implementations:
1. ExistingSummaryPage
   Pattern: Displays summary with edit/submit actions
   Components used:
   - SummaryHeader
   - DetailsCard
   - PricingBreakdown
   - ActionButtons
   Reusability: High

Existing Integration:
- Summary endpoint already exists
- Data hook already exists
- Error handling pattern already established

Reusable Components:
- SummaryHeader
- DetailsCard
- ActionButtons

Gaps to Address:
1. Feature-specific fields
2. New route registration
3. Additional validation rules

Implementation Approach:
Recommended: Adapt existing summary flow as the starting point
Complexity: Low-Medium
```

##### Phase 4: Specification Generation

**10. Generate comprehensive spec**

Using the work item, knowledge base, and exploration results, generate a spec with this structure:

```markdown
# ITEM-123: Work Item Title - Technical Specification

**Generated:** {date}
**Project:** {project-name}
**Estimate:** {estimate if available}

---

## 1. Refined Request

### Original Request
{original description}

### Original Acceptance Criteria
{original acceptance criteria}

### Enhanced Acceptance Criteria
{expanded acceptance criteria with technical clarity}

**Edge Cases to Consider:**
- Loading states
- Empty or missing data
- Error handling
- Partial completion states
- Retry behavior where relevant

**Non-Functional Requirements:**
- Performance
- Accessibility where relevant
- Error clarity
- Reliability

---

## 2. Existing Integrations and Dependencies

### Existing Integrations Used
- APIs, services, utilities, modules, or workflows already available

### Reuse Assessment
- What already exists
- What can be adapted
- What needs to be created

### Integration Checklist
- [ ] Confirm existing dependencies cover the use case
- [ ] Confirm data shape matches requirements
- [ ] Confirm error handling path
- [ ] Confirm any feature-specific gaps

---

## 3. Implementation Checklist

### Similar Implementation Reference
- Primary reference files or modules
- Reusability estimate
- Recommended starting point

### Files to Create
- [ ] List new files

### Files to Modify
- [ ] List files to update

### Components or Utilities to Reuse
- [ ] List reusable pieces

### Dependencies
- Note whether new packages or services are needed

### Testing Requirements
- [ ] Unit tests
- [ ] Integration tests
- [ ] End-to-end or workflow tests where applicable

---

## 4. Implementation Details

### Recommended Approach
{step-by-step implementation guidance}

### Code Structure
{component, module, service, or workflow skeleton}

### Important Patterns to Follow
{patterns from the knowledge base and exploration}

---

## 5. Questions and Gaps

### Clarifications Needed
{questions for product, stakeholder, or team review}

### Technical Assumptions
{assumptions that affect implementation}

---

## 6. Test Scenarios

### Happy Path Tests
{given/when/then style scenarios}

### Edge Case Tests
{specific edge cases}

### Error Condition Tests
{error scenarios and recovery expectations}

---

## 7. Timeline Estimate

**Complexity Assessment:** Low / Medium / High
**Estimated Effort:** {estimate}
**Breakdown:** {task-level estimates}
**Dependencies:** {blocking items}
**Risks:** {what could go wrong}
```

**11. Save specification file**

Write to:
```text
{project}/docs/specs/{ITEM-ID}-spec.md
```

##### Phase 5: Observability Definition

This phase runs automatically for feature and bug work unless the work item is clearly non-runtime work, such as documentation or simple housekeeping.

The goal is to answer two questions:
1. Are there errors?
2. Is the feature or process working?

Use `observability.md` from the knowledge base when available.

**12. Load observability context**

Extract:
- tools in use
- logging conventions
- tracing conventions
- alerting patterns
- verification query patterns
- database entities or events relevant to verification

If `observability.md` is missing, generate generic defaults and recommend creating or refreshing the knowledge base.

**13. Classify feature type for observability**

Possible types:
- Background Job or Worker
- Queue Consumer or Message Processor
- New API or Service Endpoint
- Existing Endpoint Reuse
- Business Workflow or Multi-Step Operation
- UI-Only

For UI-only work, do not require new server-side monitoring unless project conventions require client-side tracking.

**14. Generate observability requirements**

Generate three categories:

**A. Code-level instrumentation**
- success logs
- failure logs
- scoped instrumentation or trace names

**B. Verification and investigation queries**
- data-store verification queries where relevant
- telemetry or log queries where relevant

**C. Operational handoff**
- alert recommendations
- dashboard suggestions
- thresholds or conditions to monitor

Example structure:

```markdown
## Log Statements to Add
- Success log for completed operation
- Failure log with structured context
- Feature-scoped instrumentation name if applicable

## Verification Queries
- Throughput check
- Failure check

## Investigation Queries
- Error rate over time
- Success throughput over time
- Failure detail lookup

## Operational Recommendations
- Error rate alert
- Throughput drop alert where applicable
- Dashboard panel suggestion
```

**15. Review and save observability definition**

Write to:
```text
{project}/docs/specs/{ITEM-ID}-observability.md
```

If it already exists, compare the previous version against the new spec and show only what changed before updating.

##### Phase 6: Review & External Attachment

**16. Ask what to do next**

Offer:
- Attach or post summary to the work tracking system
- Review and edit first
- Save locally only
- Regenerate with a different approach

If no work tracking integration exists, save locally and provide the file path.

**17. Display final summary**

Always display a structured summary:

```text
Specification Generation Complete

Work Item: {ITEM-ID} - {title}
Spec: docs/specs/{ITEM-ID}-spec.md

Key Findings:
- Work Type: {classification}
- Similar Reference: {best matching feature or module}
- Existing Integration: {reused integration or “new integration needed”}
- Complexity: {Low/Medium/High}
- Estimate: {estimate}

Questions Flagged for Review:
1. {question}
2. {question}
3. {question}

Implementation Recommendation:
1. {step}
2. {step}
3. {step}
4. {step}
```

---

## Token Budget Estimates

| Depth Level | Load KB | Pre-Analysis | Exploration | Spec Gen | Total |
|-------------|---------|--------------|-------------|----------|-------|
| Quick | 10k | 5k | 0k | 20k | ~35k |
| Targeted | 10k | 5k | 30-50k | 30k | ~75-95k |
| Comprehensive | 10k | 5k | 80-120k | 40k | ~135-175k |

---

## Error Handling Reference

| Error | Action |
|-------|--------|
| Config not found | Guide through config setup |
| Project not in config | Show available projects |
| Knowledge base not found | Point to `/knowledge-base init` |
| Knowledge base outdated | Suggest refresh, allow continue |
| Work item fetch fails | Fall back to manual copy/paste mode |
| Work item not found | Ask user to verify the ID or paste details |
| Exploration fails | Offer fallback to KB-only mode |
| Write permissions fail | Show file path and suggest permission fix |

---

## Best Practices

### Choosing Depth Levels

**Quick**
- Small changes
- Well-understood patterns
- Low-risk work
- Time-sensitive specs

**Targeted**
- Most feature work
- New work in existing areas
- Moderate complexity
- Standard implementation planning

**Comprehensive**
- Large or ambiguous features
- New feature areas
- Architectural decisions
- Cross-cutting work

### Reviewing Generated Specs

Always review:
- Questions and gaps
- File paths and module names
- Integration assumptions
- Test scenarios
- Timeline estimate
- Observability expectations

### Team Collaboration

- Use specs before implementation begins
- Review specs in planning or refinement discussions
- Update specs as understanding changes
- Keep specs in version control
- Reference specs in implementation reviews

---

## Example Workflows

### Workflow 1: Simple UI Work

```bash
$ /spec-generator spec ITEM-123 my-project

> Work type: UI-Only
> Similar feature found
> Existing integration found

> Depth? [Quick]

> Spec generated: docs/specs/ITEM-123-spec.md
```

### Workflow 2: Complex New Feature

```bash
$ /knowledge-base refresh my-project
$ /spec-generator spec ITEM-456 my-project

> Work type: Full-Stack
> No strong existing match found
> Recommend: Comprehensive exploration

> Depth? [Comprehensive]

> Exploring patterns...
> Generating spec...
> Questions flagged for review
```

### Workflow 3: Manual Copy/Paste Mode

```bash
$ /spec-generator spec ITEM-789 my-project

> Automatic work item retrieval unavailable.
> Please paste title, description, and acceptance criteria.

> Spec generated
```

---

## Success Metrics

A successful spec generation means:

- The work type is classified correctly
- Relevant patterns were identified
- The checklist is actionable
- Open questions are meaningful
- The implementation can begin without major re-research
- Existing patterns are reused where appropriate

---

## Troubleshooting

### "No knowledge base found"

Run `/knowledge-base init <project-name>` first.

### "Knowledge base is outdated"

Run `/knowledge-base refresh <project-name>` or continue with the current version.

### "No similar features found"

This is normal for genuinely new work:
- Choose comprehensive depth
- Explore adjacent modules manually if needed
- Confirm that the work is actually new rather than just named differently

### "Exploration taking too long"

- Start with quick depth
- Narrow excluded directories in config
- Break the work item into smaller parts

### "Spec is too generic"

- Use targeted or comprehensive depth
- Refresh the knowledge base
- Ensure the work item includes enough acceptance criteria

### "Automatic item retrieval is not working"

Use manual copy/paste mode. Provide title, description, and acceptance criteria, then continue normally.

---

## Summary

The spec-generator skill transforms high-level work items into comprehensive technical specifications by:

1. Loading the knowledge base created by `/knowledge-base`
2. Retrieving or collecting the work item details
3. Exploring similar implementations and related code
4. Generating detailed specs with checklists, test scenarios, and open questions
5. Producing observability guidance alongside implementation guidance
6. Saving the result locally and optionally posting it to the team's tracking system

**Prerequisite**: Run `/knowledge-base init <project>` before first use.

**Get started**:
```bash
/spec-generator spec ITEM-123 <project>
```