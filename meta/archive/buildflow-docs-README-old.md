# BuildFlow Documentation

> Central documentation hub for the BuildFlow ecosystem

BuildFlow is an open-source business management system for construction and renovation businesses, with multiple backend implementations sharing a common API contract.

## 📚 Documentation Structure

### [Business Requirements](business/requirements.md)
Complete product specifications, feature requirements, user stories, and business logic.
- 14 comprehensive sections
- Technology-agnostic design
- MVP to Phase 4 requirements

### [API Contract](api-contract/openapi.yaml)
OpenAPI 3.0 specification that ALL backend implementations must follow.
- RESTful endpoints
- JSON request/response schemas
- Authentication flow
- Error handling

### [Architecture](architecture/overview.md)
System architecture, data models, and technical decisions.
- Entity Relationship Diagrams
- Multi-repo architecture
- Security considerations
- Performance requirements

### [Development Roadmap](roadmap/github-roadmap.md)
Complete development plan with 70 issues across 5 phases.
- Phase 0: Foundation
- Phase 1: MVP Core
- Phase 2: Client Portal
- Phase 3: Analytics & Team
- Phase 4: Integrations

### [User Guide](user-guide/README.md)
End-user documentation for contractors and clients.
- Getting started
- Feature guides
- Video tutorials
- FAQ

## 🏗️ Implementations

BuildFlow follows a **contract-first, multi-implementation** approach. One API contract, multiple backend implementations, one frontend.

### Backend Implementations

| Implementation | Repository | Status | Stack |
|---------------|------------|--------|-------|
| **Laravel** | [buildflow-laravel-api](https://github.com/yourusername/buildflow-laravel-api) | 🚧 In Progress | Laravel 11 + MySQL |
| **Symfony** | [buildflow-symfony-api](https://github.com/yourusername/buildflow-symfony-api) | ⏳ Planned | Symfony 7 + PostgreSQL |
| **Next.js** | [buildflow-nextjs-api](https://github.com/yourusername/buildflow-nextjs-api) | ⏳ Planned | Next.js 14 + Prisma |

All implementations:
- ✅ Follow the same OpenAPI specification
- ✅ Use the same data model
- ✅ Implement identical business logic
- ✅ Pass the same contract tests
- ✅ Work with the same React frontend

### Frontend

| Implementation | Repository | Status | Stack |
|---------------|------------|--------|-------|
| **React Web** | [buildflow-react-web](https://github.com/yourusername/buildflow-react-web) | ⏳ Planned | React 18 + Vite + TypeScript |

### Mobile (Future)

| Implementation | Repository | Status |
|---------------|------------|--------|
| **Android** | [buildflow-android](https://github.com/yourusername/buildflow-android) | 💡 Future |
| **iOS** | [buildflow-ios](https://github.com/yourusername/buildflow-ios) | 💡 Future |

## 🚀 Quick Start

### For Users
1. Choose any backend implementation (Laravel, Symfony, or Next.js)
2. Follow the setup guide in that repository
3. Access via React web frontend

### For Developers

**Contributing to existing implementation:**
```bash
# Clone the implementation you want to work on
git clone https://github.com/yourusername/buildflow-laravel-api.git

# Follow setup instructions in that repo
cd buildflow-laravel-api
```

**Creating a new implementation:**
1. Read [API Contract](api-contract/openapi.yaml)
2. Review [Business Requirements](business/requirements.md)
3. Follow the contract specifications exactly
4. Implement contract tests
5. Submit PR to add your implementation to this list

### For Product Managers
1. Review [Business Requirements](business/requirements.md)
2. Check [Development Roadmap](roadmap/github-roadmap.md)
3. Track progress across all implementations

## 📋 API Contract Compliance

All backend implementations MUST pass contract tests to ensure compatibility with the React frontend.

**Contract Testing:**
```bash
# Each backend repo has contract tests
npm run test:contract  # or
php artisan test --testsuite=Contract  # or
bin/phpunit --testsuite=Contract
```

**Frontend compatibility check:**
```bash
# In buildflow-react-web
npm run test:api-compatibility
```

## 🎯 Development Philosophy

### Why Multiple Implementations?

1. **Portfolio Diversity** - Showcase expertise across stacks
2. **Developer Choice** - Use the framework you love
3. **Learning** - Compare approaches in different ecosystems
4. **Risk Mitigation** - Not locked into one technology
5. **Community** - More developers can contribute

### Why One Frontend?

1. **Consistency** - Same user experience regardless of backend
2. **Contract Enforcement** - Forces API standardization
3. **Efficiency** - Don't rebuild UI for each backend
4. **Focus** - Backend teams focus on backend, frontend on UX

## 📊 Project Status

### Overall Progress

```
Documentation:     [████████████████████] 100% Complete
API Contract:      [██████░░░░░░░░░░░░░░]  30% In Progress
Laravel Backend:   [████░░░░░░░░░░░░░░░░]  20% Phase 0
Symfony Backend:   [░░░░░░░░░░░░░░░░░░░░]   0% Not Started
Next.js Backend:   [░░░░░░░░░░░░░░░░░░░░]   0% Not Started
React Frontend:    [░░░░░░░░░░░░░░░░░░░░]   0% Not Started
```

### Current Focus
🎯 **Phase 0: Foundation** - Laravel implementation  
📅 **Timeline:** Weeks 1-2  
👤 **Lead:** [@psswid](https://github.com/psswid)

## 🤝 Contributing

We welcome contributions to any part of the BuildFlow ecosystem!

### Ways to Contribute

**Documentation:**
- Improve API documentation
- Add examples and tutorials
- Translate user guides
- Report documentation issues

**Implementation:**
- Build features in any backend
- Fix bugs
- Improve tests
- Add new implementations

**Frontend:**
- Build React components
- Improve UX
- Add features
- Mobile apps (future)

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## 📖 Key Documents

### Must Read (Before Starting)
- [Business Requirements](business/requirements.md) - What we're building and why
- [API Contract](api-contract/openapi.yaml) - The contract all backends follow
- [Architecture Overview](architecture/overview.md) - How it all fits together
- [Development Roadmap](roadmap/github-roadmap.md) - 70 issues, 5 phases

### Reference
- [Data Model](architecture/data-model.md) - Database schema
- [Security](architecture/security.md) - Security considerations
- [API Changelog](api-contract/changelog.md) - API version history

## 🎓 Learning Resources

### Building Your First Implementation

1. **Study the contract** - [OpenAPI Spec](api-contract/openapi.yaml)
2. **Review data model** - [ERD](architecture/data-model.md)
3. **Check existing implementation** - [Laravel](https://github.com/yourusername/buildflow-laravel-api)
4. **Follow the roadmap** - [Phase 1 Issues](roadmap/github-roadmap.md)
5. **Write contract tests** - Ensure API compliance

### Recommended Reading
- [RESTful API Design Best Practices](https://restfulapi.net/)
- [OpenAPI Specification](https://swagger.io/specification/)
- [Multi-Tenant Architecture](https://docs.microsoft.com/en-us/azure/architecture/guide/multitenant/overview)

## 📞 Community

- **GitHub Discussions:** [Ask questions](https://github.com/yourusername/buildflow-docs/discussions)
- **Discord:** [Join community](https://discord.gg/buildflow) (coming soon)
- **Twitter:** [@buildflow_app](https://twitter.com/buildflow_app) (coming soon)
- **Email:** hello@buildflow.dev

## 📜 License

All BuildFlow projects are open source under the [MIT License](LICENSE).

## 🙏 Acknowledgments

**Created by:** [Piotr Świderski](https://github.com/psswid)  
**Inspired by:** Real needs of construction professionals, starting with [Prime Build Develop](https://www.primebuilddevelop.com/)

---

**⭐ Star this repo** if you find it useful!  
**🔔 Watch** for updates on all implementations  
**🍴 Fork** to create your own implementation

---

**Last Updated:** November 12, 2025  
**Version:** 1.0.0  
**Status:** 🚧 Active Development
