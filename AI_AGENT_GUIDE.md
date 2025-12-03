# AI Agent Navigation Guide

> **For:** OpenCode, Claude Code, Cursor, and other AI coding assistants  
> **Purpose:** Efficiently navigate BuildFlow documentation for context-aware development  
> **Last Updated:** 2024-12-03

---

## 🎯 Quick Start for AI Agents

### Essential Context (Load First)

**For ANY BuildFlow query, always load these 2 files:**

1. **[PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md)** (~10KB)
   - Complete project summary
   - Current status and phase
   - Technology stack
   - Architecture principles

2. **This file** (AI_AGENT_GUIDE.md)
   - Navigation instructions
   - Query-to-document mapping

**Then load task-specific documents based on query type** ↓

---

## 📚 Context Loading by Query Type

### "What is BuildFlow?" / "Project Overview"

**Load Order:**
1. `PROJECT_OVERVIEW.md` - Complete context
2. `business/requirements.md` - Full requirements
3. `domain/bounded-contexts.md` - Domain model overview

**Keywords:** BuildFlow, construction management, portfolio project, Laravel-first

---

### "Show me the architecture" / "How is it structured?"

**Load Order:**
1. `architecture/ARCHITECTURE.md` - Complete architecture
2. `architecture/decisions/SUMMARY.md` - All ADRs indexed
3. `architecture/decisions/014-laravel-first-strategy.md` - Current strategy
4. `architecture/decisions/011-domain-driven-design.md` - DDD patterns
5. `architecture/decisions/012-event-driven-architecture.md` - Event patterns
6. `architecture/decisions/013-cqrs-basic.md` - CQRS patterns

**Keywords:** architecture, DDD, event-driven, CQRS, bounded context, aggregate

---

### "How do I implement [X]?" / "Start coding [X]"

**Load Order:**
1. `implementation/getting-started-laravel.md` - Step-by-step guide with TDD
2. `domain/bounded-contexts.md` - Find which context [X] belongs to
3. `domain/aggregates.md` - Aggregate design for [X]
4. `architecture/decisions/011-domain-driven-design.md` - DDD patterns reference
5. `implementation/testing-strategy.md` - Testing requirements

**Keywords:** implementation, Laravel, TDD, aggregate, value object, repository

**Examples:**
- "Implement Quote aggregate" → Focus on QuoteManagement context
- "Add domain event" → Load ADR-012 + event-storming.md
- "Create read model" → Load ADR-013 + CQRS examples

---

### "What are the domain events?" / "How does [business flow] work?"

**Load Order:**
1. `domain/event-storming.md` - Complete event storming results
2. `domain/domain-events.md` - All domain events catalog
3. `domain/user-journeys.md` - Business flows (Quote→Project, etc.)
4. `architecture/decisions/012-event-driven-architecture.md` - Technical implementation

**Keywords:** domain event, aggregate, business rule, workflow, user journey

**Examples:**
- "QuoteAccepted event" → Find in domain/domain-events.md
- "Quote to Project flow" → Load domain/user-journeys.md

---

### "Why was [decision] made?" / "What's the rationale for [X]?"

**Load Order:**
1. `architecture/decisions/SUMMARY.md` - Find relevant ADR
2. Specific ADR (e.g., `014-laravel-first-strategy.md`)
3. Related ADRs listed in original ADR

**Keywords:** decision, rationale, trade-off, alternative, ADR

**Examples:**
- "Why Laravel-first?" → ADR-014
- "Why not microservices?" → ADR-001, ADR-014
- "Why PostgreSQL?" → ADR-007
- "Why DDD?" → ADR-011

---

### "How do I test [X]?" / "Testing strategy?"

**Load Order:**
1. `implementation/testing-strategy.md` - Complete testing approach
2. `architecture/decisions/008-contract-testing-strategy.md` - API contract tests
3. `implementation/getting-started-laravel.md` - TDD examples

**Keywords:** test, TDD, coverage, unit test, integration test, architecture test

**Coverage Requirements:**
- Domain Layer: 90%+
- Application Layer: 80%+
- Overall: 80%+

---

### "What's the current status?" / "What should I work on?"

**Load Order:**
1. `PROJECT_OVERVIEW.md` - Current phase
2. `implementation/roadmap.md` - 10-week plan
3. `roadmap/github-issues.md` - Detailed issue list

**Keywords:** status, roadmap, phase, milestone, next steps

**Current Phase (2024-12-03):**
- Phase 0: ✅ Complete (Foundation)
- Phase 1: 🚧 In Progress (Quote Management aggregate)
- Phases 2-5: ⏳ Planned

---

### "How does multi-tenancy work?" / "Specific technical question"

**Load Order:**
1. Search `architecture/decisions/SUMMARY.md` for relevant ADR
2. Load specific ADR (e.g., `004-multi-tenancy-row-level.md`)
3. Load `architecture/ARCHITECTURE.md` for implementation details

**Common Technical Topics:**
- **Authentication:** ADR-003
- **Multi-tenancy:** ADR-004
- **File Storage:** ADR-005
- **Feature Flags:** ADR-009
- **JWT:** ADR-003
- **PostgreSQL:** ADR-007

---

## 🗺️ Repository Structure Map

```
buildflow-docs/
│
├── 📄 README.md                    # Entry point
├── 📄 PROJECT_OVERVIEW.md          # Complete context ⭐
├── 📄 AI_AGENT_GUIDE.md            # This file ⭐
│
├── 📂 business/                    # What we're building
│   ├── requirements.md             # Full requirements (45KB)
│   ├── user-personas.md            # Solo Sam, Growing Grace
│   ├── competitive-analysis.md
│   └── pricing-model.md
│
├── 📂 domain/                      # Domain modeling (DDD)
│   ├── event-storming.md           # Event storming results ⭐
│   ├── bounded-contexts.md         # 8 contexts ⭐
│   ├── aggregates.md               # Quote, Project, Invoice
│   ├── domain-events.md            # All events catalog
│   ├── business-rules.md
│   └── user-journeys.md
│
├── 📂 architecture/                # Technical architecture
│   ├── ARCHITECTURE.md             # Complete architecture ⭐
│   └── decisions/                  # ADRs
│       ├── SUMMARY.md              # ADR index ⭐
│       ├── 001-014...md            # 14 ADRs
│       └── README.md
│
├── 📂 implementation/              # How to build
│   ├── roadmap.md                  # 10-week plan ⭐
│   ├── getting-started-laravel.md  # Step-by-step TDD ⭐
│   ├── testing-strategy.md         # Testing approach ⭐
│   └── examples/                   # Code examples
│
├── 📂 api/                         # API contract
│   ├── openapi.yaml                # OpenAPI 3.0 spec
│   └── README.md
│
├── 📂 roadmap/                     # Project planning
│   └── github-issues.md            # 70 issues, 5 phases
│
└── 📂 guides/                      # How-to guides
    ├── contributing.md
    └── for-developers.md
```

---

## 🔍 Search Keywords by Topic

### Domain-Driven Design
**Keywords:** `DDD`, `aggregate`, `value object`, `bounded context`, `domain event`, `entity`, `repository interface`

**Key Files:**
- `architecture/decisions/011-domain-driven-design.md`
- `domain/aggregates.md`
- `implementation/getting-started-laravel.md`

---

### Event-Driven Architecture
**Keywords:** `event`, `listener`, `event bus`, `saga`, `process manager`, `QuoteAccepted`, `ProjectCreated`

**Key Files:**
- `architecture/decisions/012-event-driven-architecture.md`
- `domain/event-storming.md`
- `domain/domain-events.md`

---

### CQRS (Command Query Responsibility Segregation)
**Keywords:** `CQRS`, `read model`, `write model`, `projection`, `projector`, `command`, `query`, `denormalized`

**Key Files:**
- `architecture/decisions/013-cqrs-basic.md`
- `architecture/ARCHITECTURE.md`

---

### Testing
**Keywords:** `test`, `TDD`, `unit test`, `integration test`, `feature test`, `architecture test`, `Pest`, `coverage`

**Key Files:**
- `implementation/testing-strategy.md`
- `architecture/decisions/008-contract-testing-strategy.md`
- `implementation/getting-started-laravel.md`

---

### Specific Domains/Contexts
**Keywords:** `Quote`, `Project`, `Invoice`, `Client`, `Document`, `Portal`

**Key Files:**
- `domain/bounded-contexts.md` (overview of all 8 contexts)
- `domain/aggregates.md` (detailed aggregate design)

---

## 🎯 Common Query Patterns

### Pattern: "I need to implement [Feature]"

**Steps:**
1. Load `domain/bounded-contexts.md` → Find which context [Feature] belongs to
2. Load `domain/aggregates.md` → Find aggregate design
3. Load `implementation/getting-started-laravel.md` → Follow TDD approach
4. Load `architecture/decisions/011-domain-driven-design.md` → Pattern reference

**Example:** "Implement Quote acceptance"
1. Bounded Context: QuoteManagement
2. Aggregate: Quote (with accept() method)
3. Domain Event: QuoteAccepted
4. Cross-context: Triggers ProjectCreated

---

### Pattern: "How does [Business Flow] work?"

**Steps:**
1. Load `domain/user-journeys.md` → Find business flow
2. Load `domain/event-storming.md` → See event sequence
3. Load `domain/domain-events.md` → Event details

**Example:** "How does quote to project flow work?"
1. User Journey: Quote → Accept → Project Created
2. Events: QuoteDraftCreated → QuoteSent → QuoteAccepted → ProjectCreated
3. Contexts: QuoteManagement → ProjectManagement

---

### Pattern: "Why did you choose [Technology/Pattern]?"

**Steps:**
1. Load `architecture/decisions/SUMMARY.md` → Find relevant ADR number
2. Load specific ADR → Read rationale, alternatives, consequences

**Example:** "Why Laravel instead of multiple frameworks?"
- ADR-014: Laravel-First Strategy
- Rationale: Depth over breadth for portfolio
- Supersedes: ADR-001 (multi-framework approach)

---

## ⚙️ Configuration for AI Tools

### For Claude Code / OpenCode

**Recommended Context Loading:**

```yaml
# Always load first (baseline context)
baseline_files:
  - PROJECT_OVERVIEW.md
  - AI_AGENT_GUIDE.md

# Load based on query type
query_mappings:
  implementation:
    - implementation/getting-started-laravel.md
    - domain/bounded-contexts.md
    - architecture/decisions/011-domain-driven-design.md
  
  architecture:
    - architecture/ARCHITECTURE.md
    - architecture/decisions/SUMMARY.md
    - architecture/decisions/014-laravel-first-strategy.md
  
  domain:
    - domain/event-storming.md
    - domain/bounded-contexts.md
    - domain/aggregates.md
```

---

### For Cursor

**Cursor Rules (.cursorrules):**

```
# BuildFlow Project Rules

## Context Priority
1. Always read PROJECT_OVERVIEW.md for project context
2. For architecture questions, read architecture/ARCHITECTURE.md
3. For implementation, follow implementation/getting-started-laravel.md
4. All decisions are documented in architecture/decisions/

## Code Standards
- Follow DDD patterns from ADR-011
- Domain layer must be pure PHP (no Laravel imports)
- Test coverage: 80%+ overall, 90%+ domain layer
- Use TDD approach from getting-started guide

## When Asked "Why?"
- Check architecture/decisions/SUMMARY.md for relevant ADR
- All major decisions have ADRs with rationale
```

---

## 📊 File Size Reference

**Small (<5KB) - Quick to load:**
- Most ADRs
- README files
- Extracted sections

**Medium (5-15KB) - Moderate:**
- PROJECT_OVERVIEW.md (~10KB)
- Most implementation guides
- Domain sections

**Large (15KB+) - Load selectively:**
- business/requirements.md (~45KB)
- domain/event-storming.md (~20KB)
- implementation/getting-started-laravel.md (~20KB)

**Strategy:** Load large files only when specifically needed, prefer extracted sections.

---

## 🚨 Important Reminders for AI Agents

### 1. **Laravel-First Strategy**
- Primary implementation is Laravel (not Symfony, not Next.js)
- Symfony/Next.js are low-priority optional future experiments
- See ADR-014 for full rationale

### 2. **DDD is Non-Negotiable**
- Domain layer must be pure PHP
- Business logic in Aggregates, not Controllers
- Value Objects for immutable concepts
- See ADR-011 for patterns

### 3. **Event-Driven Communication**
- Contexts communicate via Domain Events
- No direct dependencies between contexts
- See ADR-012 for patterns

### 4. **Testing is Required**
- TDD approach (test first)
- 80%+ overall coverage
- 90%+ domain layer coverage
- Architecture tests enforce structure
- See implementation/testing-strategy.md

### 5. **Current Phase**
- Phase 0: ✅ Complete
- Phase 1: 🚧 Quote Management (showcase implementation)
- Focus: Building Quote aggregate with full DDD patterns

---

## 🔗 Related Resources

- **GitHub Repository:** [buildflow-docs](https://github.com/psswid/buildflow-docs)
- **Implementation Repo:** [buildflow-laravel-api](https://github.com/psswid/buildflow-laravel-api) (future)
- **ADR Format:** [ADR Template](architecture/decisions/000-template.md)

---

## 📝 Feedback & Updates

This guide evolves as the project grows. If you're an AI agent and found this helpful (or confusing), humans can update this file to improve navigation.

**Last Updated:** 2024-12-03  
**Version:** 1.0  
**Status:** Active

---

**Happy coding! 🤖🚀**
