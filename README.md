# BuildFlow Documentation

> **Central documentation repository for BuildFlow** - A construction business management system built with Laravel, DDD, Event-Driven Architecture, and CQRS patterns.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Laravel](https://img.shields.io/badge/Laravel-11.x-red.svg)](https://laravel.com)
[![Status](https://img.shields.io/badge/Status-Phase%201-yellow.svg)](roadmap/github-issues.md)

---

## 🚀 Quick Start

**New here?** Start with these documents:

1. **[PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md)** - Complete project context
2. **[AI_AGENT_GUIDE.md](AI_AGENT_GUIDE.md)** - AI agent navigation guide
3. **[Architecture Overview](architecture/ARCHITECTURE.md)** - Technical architecture
4. **[Getting Started Guide](implementation/getting-started-laravel.md)** - Start coding

---

## 📁 Repository Structure

```
buildflow-docs/
├── 📄 PROJECT_OVERVIEW.md          # Complete project context ⭐
├── 📄 AI_AGENT_GUIDE.md            # AI agent navigation ⭐
├── 📄 ARCHITECTURE.md              # Complete architecture
├── 📄 CONTRIBUTING.md              # How to contribute
│
├── 📂 business/                    # Business requirements & product vision
├── 📂 domain/                      # Domain modeling (DDD)
├── 📂 architecture/                # Architecture & ADRs
│   └── decisions/                  # 14 Architecture Decision Records
├── 📂 implementation/              # Implementation guides
├── 📂 api/                         # API contract (OpenAPI)
├── 📂 roadmap/                     # Project planning
├── 📂 guides/                      # How-to guides
└── 📂 meta/                        # Meta documentation
```

**For detailed structure, see:** [Repository Structure](meta/repository-structure-proposal.md)

---

## 🎯 What is BuildFlow?

BuildFlow is an **open-source construction business management system** designed for small-to-medium construction and renovation businesses. It provides:

- **Quote Management** - Professional quotes with versioning
- **Project Tracking** - Active project management
- **Invoice Management** - Automated invoicing
- **Client Portal** - Client self-service interface
- **Document Management** - Centralized file storage

**Target Users:** Solo contractors and small construction businesses (1-10 employees)

**Business Model:** Freemium SaaS with generous free tier

---

## 🏗️ Architecture

BuildFlow uses **enterprise-grade patterns** for a Laravel-based system:

### Core Patterns

- **Domain-Driven Design (DDD)** - [ADR-011](architecture/decisions/011-domain-driven-design.md)
  - Pure domain layer (no framework dependencies)
  - Aggregates as consistency boundaries
  - Value Objects for immutability

- **Event-Driven Architecture** - [ADR-012](architecture/decisions/012-event-driven-architecture.md)
  - Domain events for cross-context communication
  - Event sourcing for audit trail
  - Sagas for complex workflows

- **CQRS (Basic)** - [ADR-013](architecture/decisions/013-cqrs-basic.md)
  - Separate read and write models
  - Projectors for read model updates
  - Optimized queries

### 8 Bounded Contexts

1. QuoteManagement
2. ProjectManagement
3. InvoiceManagement
4. ClientManagement
5. DocumentManagement
6. UserManagement
7. ClientPortal
8. NotificationManagement

**See:** [Domain Documentation](domain/) | [Architecture Details](architecture/ARCHITECTURE.md)

---

## 🗺️ Current Status

**Phase:** 1 of 5 (Quote Management - Showcase) 🚧

**Focus:** Building Quote aggregate with full DDD patterns as showcase implementation

**Progress:**
- ✅ Phase 0: Foundation (Documentation, ADRs, Planning)
- 🚧 Phase 1: Quote Management (Current)
- ⏳ Phase 2: Event-Driven Communication
- ⏳ Phase 3: CQRS Implementation
- ⏳ Phase 4-5: Complete & Polish

**See:** [Implementation Roadmap](implementation/roadmap.md) | [GitHub Issues](roadmap/github-issues.md)

---

## 🚀 For Developers

### Getting Started

1. **Read:** [Project Overview](PROJECT_OVERVIEW.md)
2. **Understand:** [Architecture](architecture/ARCHITECTURE.md)
3. **Build:** [Getting Started Guide](implementation/getting-started-laravel.md)
4. **Test:** [Testing Strategy](implementation/testing-strategy.md)

### Development Approach

- **TDD First** - Write tests before implementation
- **Domain Pure** - No Laravel in domain layer
- **Event-Driven** - Use domain events for communication
- **80%+ Coverage** - High test coverage required

### Tech Stack

- **Backend:** Laravel 11.x (PHP 8.3+)
- **Database:** PostgreSQL 16
- **Testing:** Pest PHP
- **API:** OpenAPI 3.0
- **Frontend:** React (separate repo)

---

## 🤖 For AI Agents

This repository is optimized for AI coding assistants (OpenCode, Claude Code, Cursor).

**Essential Files to Load:**
1. [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md) - Complete context
2. [AI_AGENT_GUIDE.md](AI_AGENT_GUIDE.md) - Navigation guide
3. Task-specific documents from guide

**See:** [AI Agent Guide](AI_AGENT_GUIDE.md) for detailed navigation instructions.

---

## 📚 Key Documentation

### Business & Product
- [Business Requirements](business/requirements.md) - Complete requirements (~45KB)
- [User Personas](business/requirements.md#user-personas) - Solo Sam, Growing Grace

### Domain Modeling
- [Event Storming](domain/event-storming.md) - Domain events and aggregates
- [Bounded Contexts](domain/event-storming.md#bounded-contexts) - 8 contexts detailed

### Architecture Decisions
- [ADR Summary](architecture/decisions/SUMMARY.md) - All 14 ADRs indexed
- [Laravel-First Strategy](architecture/decisions/014-laravel-first-strategy.md) - Why Laravel
- [Domain-Driven Design](architecture/decisions/011-domain-driven-design.md) - DDD patterns
- [Event-Driven Architecture](architecture/decisions/012-event-driven-architecture.md) - Events
- [CQRS Basic](architecture/decisions/013-cqrs-basic.md) - Read/write separation

### Implementation
- [10-Week Roadmap](implementation/roadmap.md) - Detailed plan
- [Getting Started with Laravel](implementation/getting-started-laravel.md) - Step-by-step TDD
- [Testing Strategy](implementation/testing-strategy.md) - Test approach

### API
- [OpenAPI Specification](api-contract/openapi.yaml) - Complete API contract

---

## 🤝 Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for:
- Setup instructions
- Code standards
- Testing requirements
- PR process

---

## 📊 Project Stats

- **Documentation:** ~260KB across 50+ files
- **ADRs:** 14 architecture decisions documented
- **Issues:** 70 issues planned across 5 phases
- **Test Coverage Goal:** 80%+ overall, 90%+ domain layer

---

## 🔗 Related Repositories

- **buildflow-docs** - This repository (documentation)
- **buildflow-laravel-api** - Laravel backend implementation (planned)
- **buildflow-react-web** - React frontend (planned)

---

## 📄 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) for details.

**Why Open Source?** See [ADR-006: Open Source MIT License](architecture/decisions/006-open-source-mit-license.md)

---

## 📮 Contact & Support

- **Author:** Piotr Świderski
- **GitHub:** [@psswid](https://github.com/psswid)
- **Issues:** [GitHub Issues](https://github.com/psswid/buildflow-docs/issues)

---

## 🎯 Project Goals

1. **Real Business Value** - Solve brother's construction business needs
2. **Portfolio Showcase** - Demonstrate enterprise Laravel patterns
3. **Learning Vehicle** - Master DDD, Events, CQRS in production context

**Current Focus:** Building Quote Management as exemplar implementation

---

**Ready to start?** → [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md) | [Getting Started](implementation/getting-started-laravel.md)
