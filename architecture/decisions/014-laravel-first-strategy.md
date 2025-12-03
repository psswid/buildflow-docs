# ADR 014: Laravel-First Strategy (Supersedes Multi-Framework Approach)

**Status:** Accepted

**Date:** 2024-12-03

**Deciders:** Piotr Świderski

**Supersedes:** ADR-001 (Multi-Repository Strategy for multiple equal implementations)

**Technical Story:** After initial planning phase, re-evaluated portfolio strategy from "breadth" (3 frameworks at 30% depth each) to "depth" (1 framework at 100% with enterprise patterns).

---

## Context

### Original Strategy (ADR-001)
Initial plan was to build **three parallel backend implementations**:
- Laravel API
- Symfony API  
- Next.js API

All implementing the same OpenAPI contract, demonstrating ability to work with multiple frameworks.

### What Changed

After creating comprehensive documentation (13 ADRs, domain analysis, implementation guides), I realized:

**Problem with Original Approach:**
- ❌ **Shallow Knowledge**: 3 frameworks × 30% depth = surface-level implementations
- ❌ **No Enterprise Patterns**: Basic CRUD doesn't demonstrate senior-level thinking
- ❌ **Interview Weakness**: "I know Laravel, Symfony, Next.js" → "How deep?" → "Basic level"
- ❌ **Time Inefficiency**: Months on breadth, no production-ready depth
- ❌ **Recruiter Feedback**: "Nice variety, but I need deep Laravel expertise" (for Laravel roles)

**Portfolio Reality:**
- ✅ **Depth > Breadth** for senior roles
- ✅ **Production Patterns** matter more than framework count
- ✅ **Real Architecture** beats basic CRUD
- ✅ **One Deep Example** better than three shallow ones

### Business Context
Project has **real business use case** (brother's construction company in UK). Better to deliver:
- One production-ready system they can actually use
- Than three experimental systems with limited functionality

---

## Decision

**We will focus on Laravel as primary implementation with enterprise-grade patterns. Other frameworks (Symfony, Next.js) become low-priority optional learning experiments.**

### New Strategy: "Laravel-First, Enterprise-Grade"

**Primary Focus (100% effort):**
```
buildflow-laravel-api/
  ├── Domain-Driven Design (DDD)
  ├── Event-Driven Architecture  
  ├── CQRS with Read Models
  ├── Modular Monolith (Bounded Contexts)
  ├── 80%+ Test Coverage
  ├── Architecture Tests
  └── Production-Ready Deployment
```

**Secondary (Optional, Future):**
```
buildflow-symfony-api/     # LOW PRIORITY - Learning experiment only
buildflow-nextjs-api/      # LOW PRIORITY - Maybe never
```

### Repository Structure (Updated)

**High Priority:**
- `buildflow-docs` - Central documentation ✅ (Complete)
- `buildflow-laravel-api` - **PRIMARY IMPLEMENTATION** 🚧 (Phase 1)
- `buildflow-react-web` - Frontend ⏳ (Phase 2)

**Low Priority (Optional):**
- `buildflow-symfony-api` - ⏳ Maybe after Laravel is 100% complete
- `buildflow-nextjs-api` - ⏳ Probably never

### Implementation Phases

**Phase 0-5 (Weeks 1-10): Laravel Only**
- Week 1: Foundation (Shared Kernel, Auth, Multi-tenancy)
- Weeks 2-3: **Quote Management** with full DDD showcase
- Week 4: Event-Driven Architecture
- Week 5: CQRS with Read Models
- Weeks 6-8: Remaining Contexts (Project, Invoice, Documents)
- Weeks 9-10: Production Polish (CI/CD, monitoring, deployment)

**After Laravel MVP (Optional):**
- Symfony implementation as **learning exercise**
- Explicit positioning: "Comparing approaches" not "equal alternatives"

---

## Rationale

### Why Laravel First?

**1. Market Demand**
- Most job openings require deep Laravel expertise
- "Senior Laravel Developer" roles need production patterns
- Better to be expert in one than beginner in three

**2. Portfolio Impact**

**Before (Multi-Framework):**
> "I built the same API in Laravel, Symfony, and Next.js"
> 
> Interviewer: "Show me your DDD implementation"
> 
> Me: "Well, it's mostly CRUD with repositories..."

**After (Laravel-First):**
> "I built enterprise-grade Laravel API with DDD, Event-Driven Architecture, and CQRS"
> 
> Interviewer: "Show me your DDD implementation"
> 
> Me: "Here's Quote aggregate with business rules, domain events, and cross-context communication via Event Bus. Here's CQRS projectors updating read models. Here are architecture tests enforcing layer separation."

**3. Learning Depth**

| Approach | Result |
|----------|--------|
| 3 frameworks shallow | Understand syntax, basic patterns |
| 1 framework deep | Understand architecture, design patterns, production concerns |

**4. Time to Value**

- Multi-Framework: 6 months → 3 shallow implementations → Not production-ready
- Laravel-First: 2.5 months → 1 deep implementation → Production-ready MVP

**5. Real Business Value**

Brother's company needs **working system**, not experiments. Laravel-first delivers:
- ✅ Production-ready system in 10 weeks
- ✅ Real-world testing and feedback
- ✅ Portfolio with actual usage metrics
- ✅ Case study: "Built system used by real business"

---

## Consequences

### Positive

✅ **Deep Expertise**: Master Laravel with production patterns (DDD, Events, CQRS)

✅ **Portfolio Strength**: One enterprise example > three basic examples

✅ **Interview Advantage**: Can discuss trade-offs, architecture decisions, production concerns

✅ **Documentation Value**: 13 ADRs demonstrate architectural thinking

✅ **Production Ready**: Actually deployable system with monitoring, tests, CI/CD

✅ **Real Usage**: Brother's business provides real-world validation

✅ **Learning Depth**: Understanding *when* to use patterns, not just *how*

✅ **Time Efficiency**: 10 weeks to production-ready MVP vs 6+ months for three basics

### Negative

❌ **Less Framework Variety**: Only Laravel, not Symfony/Next.js

❌ **Perception Risk**: "One framework only?" → Mitigated by depth + ADRs showing architectural understanding

❌ **Learning Scope**: Won't learn Symfony/Next.js patterns (yet)

### Neutral

⚪ **Optional Future**: Symfony can still be added later as explicit "learning comparison"

⚪ **Documentation Reuse**: All domain analysis, ADRs still valid for future implementations

⚪ **OpenAPI Contract**: Still framework-agnostic, still valid

---

## Migration from ADR-001

### What Remains Valid from ADR-001

✅ **Multi-repository structure** - Still using separate repos
- `buildflow-docs` (central documentation)
- `buildflow-laravel-api` (primary backend)  
- `buildflow-react-web` (frontend)

✅ **Contract-First Development** - OpenAPI still source of truth

✅ **Frontend Backend Separation** - React still backend-agnostic

✅ **Independent CI/CD** - Each repo still has own pipeline

### What Changes from ADR-001

❌ **Multiple Equal Implementations** → **One Primary Implementation**

❌ **Framework Comparison Focus** → **Enterprise Patterns Focus**

❌ **Breadth Strategy** → **Depth Strategy**

---

## Related Decisions

- **ADR-001**: Multi-Repository Strategy → **SUPERSEDED** by this ADR for implementation priority
- **ADR-002**: API-First Approach → Still valid
- **ADR-011**: Domain-Driven Design → **NEW**, core pattern for Laravel
- **ADR-012**: Event-Driven Architecture → **NEW**, core pattern for Laravel
- **ADR-013**: CQRS Basic → **NEW**, core pattern for Laravel

---

## Interview Talking Points

### When They Ask: "Why Only Laravel?"

> "I chose depth over breadth. Rather than building three shallow CRUD APIs, I focused on one enterprise-grade implementation with DDD, Event-Driven Architecture, and CQRS. This demonstrates production-ready thinking that's framework-agnostic - the patterns apply to any backend."

### When They Ask: "Do You Know Other Frameworks?"

> "Yes, I'm familiar with Symfony and Node.js. But for this portfolio project, I wanted to showcase mastery of Laravel with patterns you'd see in production: bounded contexts, domain events, CQRS projectors, architecture tests. The documentation shows I understand these patterns conceptually - I just haven't duplicated the implementation across frameworks yet."

### When They Ask: "Why Changed Strategy?"

> "After completing architecture documentation, I realized recruiters value deep expertise over framework variety. I documented this decision in ADR-014 - a meta-decision about prioritizing production patterns over breadth. The 13 ADRs themselves demonstrate architectural thinking."

---

## Verification

### Success Metrics

**After 10 Weeks:**
- [ ] Laravel API implements full DDD patterns (Aggregates, Value Objects, Domain Events)
- [ ] Event-Driven communication between contexts working
- [ ] CQRS read models with projectors functional  
- [ ] 80%+ test coverage (90%+ domain layer)
- [ ] Architecture tests enforcing layer separation
- [ ] CI/CD pipeline with automated testing
- [ ] Deployed demo environment
- [ ] Brother's business using system (real-world validation)

**Interview Readiness:**
- [ ] Can explain DDD with concrete Quote aggregate example
- [ ] Can show event-driven flow (QuoteAccepted → ProjectCreated)
- [ ] Can demonstrate CQRS optimization (list view vs aggregate)
- [ ] Can discuss trade-offs documented in ADRs
- [ ] Can point to production-ready code (not just diagrams)

---

## Notes

### Why Document This Decision?

This ADR itself demonstrates:
- **Strategic thinking**: Recognizing when to pivot
- **Self-awareness**: Understanding portfolio needs vs learning desires
- **Pragmatism**: Prioritizing value delivery over completeness
- **Transparency**: Openly documenting decision rationale

### Future Considerations

If Symfony implementation is added later:
- Must be explicitly positioned as "learning comparison"
- Must not dilute primary Laravel implementation
- Must include "Comparison Analysis" documentation showing:
  - What's easier/harder in Symfony
  - Performance differences
  - Developer experience differences
  - When to choose each framework

---

## References

- Original Strategy Discussion: buildflow-docs planning sessions (Nov 2024)
- Recruiter Perspective Feedback: Internal project review (Dec 2024)
- Enterprise Patterns Research: Eric Evans (DDD), Martin Fowler (EDA, CQRS)
- Portfolio Strategy: Various "how to stand out" articles for senior dev roles

---

**Decision Date:** 2024-12-03

**Review Date:** After Laravel MVP completion (estimated Feb 2025)

**Status:** ✅ Accepted - All new development follows this strategy
