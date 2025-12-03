# BuildFlow Documentation

> Central documentation repository for BuildFlow - Enterprise-grade construction management system

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 📚 What's Here

This repository contains **all documentation** for the BuildFlow project:
- Business requirements and domain analysis
- Architecture Decision Records (ADRs)
- Implementation guides and roadmaps
- Testing strategies
- API contracts (OpenAPI specifications)

---

## 🎯 Project Overview

**BuildFlow** is a construction business management system built with enterprise-grade Laravel architecture, demonstrating:
- **Domain-Driven Design** (DDD)
- **Event-Driven Architecture**
- **CQRS** (Command Query Responsibility Segregation)
- **Multi-Tenancy**
- **Contract-First Development**

**Purpose:** Portfolio project showcasing **depth over breadth** - one framework (Laravel) implemented with production-ready, enterprise patterns.

---

## 📋 Documentation Structure

### 📊 Business & Domain

#### [BuildFlow_Business_Requirements_v1.0.md](BuildFlow_Business_Requirements_v1.0.md)
**45KB, 14 sections**

Complete business specification including:
- Problem statement (construction business pain points)
- User personas (Solo Contractor Sam, Growing Business Grace)
- Competitive analysis (Buildertrend, CoConstruct, Houzz Pro)
- Feature requirements
- Freemium pricing model (Starter/Pro/Business tiers)
- Success metrics

#### [DOMAIN_ANALYSIS_EVENT_STORMING.md](DOMAIN_ANALYSIS_EVENT_STORMING.md)
**~20KB**

Domain modeling using Event Storming technique:
- **8 Bounded Contexts** - QuoteManagement, ProjectManagement, InvoiceManagement, ClientManagement, DocumentManagement, ClientPortal, TeamManagement, Notifications
- **Domain Events** - QuoteAccepted, ProjectCreated, PaymentReceived, etc.
- **Aggregates** - Quote (with LineItems), Project (with Milestones), Invoice (with Payments)
- **Business Rules** - Invariants and constraints
- **User Journeys** - Quote to Project flow, Payment flow, etc.

#### [BuildFlow_GitHub_Roadmap.md](BuildFlow_GitHub_Roadmap.md)
**70 issues across 5 phases**

Development roadmap with:
- Phase 0: Foundation (9 issues)
- Phase 1: MVP (28 issues)
- Phase 2: Client Portal (12 issues)
- Phase 3: Team Collaboration (12 issues)
- Phase 4: Advanced Features (9 issues)

---

### 🏛️ Architecture Decisions

#### [docs/architecture/decisions/](docs/architecture/decisions/)

**13 Architecture Decision Records (ADRs)** documenting all major architectural choices:

**Foundation (ADR 001-010):**
- [ADR-001: Multi-Repository Strategy](docs/architecture/decisions/001-multi-repository-strategy.md)
- [ADR-002: API-First Development Approach](docs/architecture/decisions/002-api-first-approach.md)
- [ADR-003: JWT Token Authentication](docs/architecture/decisions/003-jwt-authentication.md)
- [ADR-004: Row-Level Multi-Tenancy](docs/architecture/decisions/004-multi-tenancy-row-level.md)
- [ADR-005: Cloud-Based File Storage](docs/architecture/decisions/005-cloud-file-storage.md)
- [ADR-006: Open Source MIT License](docs/architecture/decisions/006-open-source-mit-license.md) ⭐
- [ADR-007: PostgreSQL as Primary Database](docs/architecture/decisions/007-postgresql-primary-database.md)
- [ADR-008: OpenAPI Contract Testing Strategy](docs/architecture/decisions/008-contract-testing-strategy.md)
- [ADR-009: Feature Flags for Subscription Tiers](docs/architecture/decisions/009-feature-flags-for-tiers.md)
- [ADR-010: Frontend-Backend Separation](docs/architecture/decisions/010-frontend-backend-separation.md)

**Enterprise Patterns (ADR 011-013):** ⭐ **CORE ARCHITECTURE**
- [ADR-011: Domain-Driven Design](docs/architecture/decisions/011-domain-driven-design.md) - Aggregates, Value Objects, Domain Events
- [ADR-012: Event-Driven Architecture](docs/architecture/decisions/012-event-driven-architecture.md) - Cross-context communication
- [ADR-013: CQRS Basic](docs/architecture/decisions/013-cqrs-basic.md) - Read/Write separation

**Supporting Documents:**
- [README.md](docs/architecture/decisions/README.md) - How to write and use ADRs
- [SUMMARY.md](docs/architecture/decisions/SUMMARY.md) - Complete overview with dependencies
- [000-template.md](docs/architecture/decisions/000-template.md) - Template for new ADRs

---

### 🛠️ Implementation Guides

#### [IMPLEMENTATION_ROADMAP.md](IMPLEMENTATION_ROADMAP.md)
**~15KB, 10-week plan**

Detailed implementation plan:
- **Phase 0** - Foundation (Week 1): Project setup, Shared Kernel, Auth & Multi-tenancy
- **Phase 1** - Core Aggregate (Weeks 2-3): Quote Management with full DDD patterns
- **Phase 2** - Event-Driven (Week 4): Event Bus, cross-context communication
- **Phase 3** - CQRS (Week 5): Read models, projectors, optimized queries
- **Phase 4** - Remaining Contexts (Weeks 6-8): Project, Invoice, Documents, Portal
- **Phase 5** - Production Polish (Weeks 9-10): Testing, security, CI/CD, deployment

Each phase includes:
- Daily task breakdown
- Code examples
- Testing requirements
- Deliverables

#### [LARAVEL_DDD_STARTER_GUIDE.md](LARAVEL_DDD_STARTER_GUIDE.md)
**~20KB, step-by-step tutorial**

Practical guide for implementing first aggregate (Quote):
- **Step 1** - Planning (Aggregate design, business rules, events)
- **Step 2** - Folder structure
- **Step 3** - Value Objects with TDD (QuoteId, Money, QuoteStatus)
- **Step 4** - Domain Events
- **Step 5** - Aggregate implementation (Quote with business logic)
- **Step 6** - Repository interface
- **Step 7** - Application layer (Commands & Handlers)
- **Step 8** - Infrastructure layer (Eloquent, HTTP)
- **Step 9** - Testing strategy

Each step includes:
- Test-first examples
- Implementation code
- Common pitfalls
- Commit messages

#### [TESTING_STRATEGY.md](TESTING_STRATEGY.md)
**~15KB, comprehensive testing**

Complete testing approach:
- **Testing Pyramid** - 50% unit, 25% integration, 20% feature, 5% E2E
- **Unit Tests** - Domain logic (Aggregates, Value Objects)
- **Integration Tests** - Application layer (Handlers, Events)
- **Feature Tests** - HTTP/API endpoints
- **Architecture Tests** - Enforce structural rules
- **Contract Tests** - OpenAPI compliance
- **Test Doubles** - When to use Fakes, Mocks, Spies
- **Coverage Goals** - 80% overall, 90% domain
- **CI/CD Integration** - GitHub Actions setup

---

### 📖 Complete Overview

#### [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md)
**~10KB**

Single document with everything:
- What is BuildFlow?
- Complete documentation index
- Architecture overview
- Why this architecture?
- What you have now
- Next steps
- All file references

**Perfect starting point for new contributors!**

---

## 🏗️ Repository Structure

```
buildflow-docs/
│
├── README.md (this file)
├── LICENSE (MIT)
│
├── Business & Domain/
│   ├── BuildFlow_Business_Requirements_v1.0.md
│   ├── DOMAIN_ANALYSIS_EVENT_STORMING.md
│   └── BuildFlow_GitHub_Roadmap.md
│
├── Architecture/
│   ├── ARCHITECTURE.md
│   └── docs/
│       └── architecture/
│           └── decisions/
│               ├── README.md
│               ├── SUMMARY.md
│               ├── 000-template.md
│               ├── 001-multi-repository-strategy.md
│               ├── 002-api-first-approach.md
│               ├── 003-jwt-authentication.md
│               ├── 004-multi-tenancy-row-level.md
│               ├── 005-cloud-file-storage.md
│               ├── 006-open-source-mit-license.md
│               ├── 007-postgresql-primary-database.md
│               ├── 008-contract-testing-strategy.md
│               ├── 009-feature-flags-for-tiers.md
│               ├── 010-frontend-backend-separation.md
│               ├── 011-domain-driven-design.md
│               ├── 012-event-driven-architecture.md
│               └── 013-cqrs-basic.md
│
├── Implementation/
│   ├── IMPLEMENTATION_ROADMAP.md
│   ├── LARAVEL_DDD_STARTER_GUIDE.md
│   └── TESTING_STRATEGY.md
│
├── API/
│   └── openapi.yaml (Coming Soon)
│
└── PROJECT_OVERVIEW.md
```

---

## 🔗 Related Repositories

| Repository | Purpose | Status |
|------------|---------|--------|
| [buildflow-docs](https://github.com/psswid/buildflow-docs) | Documentation (this repo) | ✅ Complete |
| [buildflow-laravel-api](https://github.com/psswid/buildflow-laravel-api) | Laravel API implementation | 🚧 Phase 1 |
| [buildflow-react-web](https://github.com/psswid/buildflow-react-web) | React frontend | ⏳ Planned |

---

## 🎯 Quick Start

### For Developers Starting Implementation

1. **Read** [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md) - 10 min overview
2. **Study** [ADR-011](docs/architecture/decisions/011-domain-driven-design.md), [ADR-012](docs/architecture/decisions/012-event-driven-architecture.md), [ADR-013](docs/architecture/decisions/013-cqrs-basic.md) - Understand patterns
3. **Follow** [IMPLEMENTATION_ROADMAP.md](IMPLEMENTATION_ROADMAP.md) - Week-by-week plan
4. **Use** [LARAVEL_DDD_STARTER_GUIDE.md](LARAVEL_DDD_STARTER_GUIDE.md) - Step-by-step coding

### For Architects Reviewing Design

1. **Read** [DOMAIN_ANALYSIS_EVENT_STORMING.md](DOMAIN_ANALYSIS_EVENT_STORMING.md) - Understand domain
2. **Review** [ADR Index](docs/architecture/decisions/SUMMARY.md) - See all decisions
3. **Check** [ARCHITECTURE.md](ARCHITECTURE.md) - System overview

### For Product Managers

1. **Read** [BuildFlow_Business_Requirements_v1.0.md](BuildFlow_Business_Requirements_v1.0.md) - Full specs
2. **Check** [BuildFlow_GitHub_Roadmap.md](BuildFlow_GitHub_Roadmap.md) - Development plan

---

## 💡 Philosophy

### Why This Approach?

**Traditional Portfolio Projects:**
- ❌ Multiple frameworks (Laravel, Symfony, Next.js) at 30% each
- ❌ Shallow CRUD implementations
- ❌ Business logic in controllers
- ❌ No architectural patterns
- ❌ "Jack of all trades, master of none"

**BuildFlow:**
- ✅ **One framework (Laravel) at 100%**
- ✅ **Enterprise patterns** (DDD, Events, CQRS)
- ✅ **Production-ready** code
- ✅ **Deep expertise** demonstration
- ✅ **"Master of one"**

### Documentation-First Approach

We document **before** we code:
1. **Why** - Business requirements
2. **What** - Domain analysis (Event Storming)
3. **How** - Architecture decisions (ADRs)
4. **When** - Implementation roadmap
5. **Test** - Testing strategy

This ensures:
- ✅ Thoughtful design
- ✅ Documented trade-offs
- ✅ Clear implementation path
- ✅ Easy onboarding

---

## 🤝 Contributing

### To Documentation

1. For business changes → Update `BuildFlow_Business_Requirements_v1.0.md`
2. For domain changes → Update `DOMAIN_ANALYSIS_EVENT_STORMING.md`
3. For architectural decisions → Create new ADR in `docs/architecture/decisions/`
4. For implementation details → Update relevant guide

### Creating New ADR

```bash
# 1. Copy template
cp docs/architecture/decisions/000-template.md docs/architecture/decisions/014-your-decision.md

# 2. Fill in all sections
# 3. Update SUMMARY.md
# 4. Submit PR
```

See [ADR README](docs/architecture/decisions/README.md) for details.

---

## 📊 Documentation Stats

- **Total Documentation:** ~150KB
- **ADRs:** 13 (+ 1 template)
- **Implementation Guides:** 3 major guides
- **Business Docs:** 3 documents
- **GitHub Issues:** 70 planned

---

## 📜 License

MIT License - see [LICENSE](LICENSE) file.

All documentation is freely available for learning and reference.

---

## 📧 Contact

**Project Maintainer:** Piotr Świderski

**Questions?**
- Open a [Discussion](https://github.com/psswid/buildflow-docs/discussions)
- Check existing [Issues](https://github.com/psswid/buildflow-docs/issues)

---

## 🙏 Acknowledgments

**Inspiration & Learning:**
- Eric Evans - Domain-Driven Design
- Vaughn Vernon - DDD Implementation Patterns
- Martin Fowler - Enterprise Architecture Patterns
- Laravel Community - Framework excellence

---

**Documentation is code.** Without proper docs, even the best architecture is just spaghetti. 🍝

---

**Last Updated:** 2024-11-12  
**Version:** 2.0 (Laravel-First with Enterprise Patterns)  
**Status:** Complete - Ready for Implementation
