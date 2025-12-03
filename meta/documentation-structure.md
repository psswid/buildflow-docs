# BuildFlow - Documentation Structure

This document explains the organization of BuildFlow's documentation.

## 📁 Directory Structure

```
buildflow/
├── README.md                          # Main project README
├── CONTRIBUTING.md                    # Contributing guidelines
├── LICENSE                            # MIT License
├── CHANGELOG.md                       # Version history
│
├── docs/                              # All documentation
│   ├── overview.md                    # Project overview (START HERE)
│   │
│   ├── business/                      # Business documentation
│   │   ├── README.md                  # Business docs index
│   │   ├── requirements.md            # Complete business requirements
│   │   ├── monetization.md            # Business model and pricing
│   │   └── market-analysis.md         # Target market and competitors
│   │
│   ├── product/                       # Product management docs
│   │   ├── README.md                  # Product docs index
│   │   ├── user-stories.md            # All user stories and use cases
│   │   ├── personas.md                # User personas in detail
│   │   ├── metrics.md                 # Success metrics and KPIs
│   │   └── workflows.md               # Business workflows
│   │
│   ├── technical/                     # Technical documentation
│   │   ├── README.md                  # Technical docs index
│   │   ├── architecture.md            # System architecture
│   │   ├── data-model.md              # Database schema and ERD
│   │   ├── security.md                # Security considerations
│   │   ├── performance.md             # Performance requirements
│   │   └── infrastructure.md          # Deployment and scaling
│   │
│   ├── api/                           # API documentation
│   │   ├── README.md                  # API overview
│   │   ├── authentication.md          # Auth endpoints
│   │   ├── clients.md                 # Client endpoints
│   │   ├── quotes.md                  # Quote endpoints
│   │   ├── projects.md                # Project endpoints
│   │   ├── invoices.md                # Invoice endpoints
│   │   └── webhooks.md                # Webhook documentation
│   │
│   ├── roadmap/                       # Development planning
│   │   ├── README.md                  # Roadmap overview
│   │   ├── github-roadmap.md          # Complete GitHub issues/milestones
│   │   ├── phase-0-foundation.md      # Phase 0 details
│   │   ├── phase-1-mvp.md             # Phase 1 details
│   │   ├── phase-2-portal.md          # Phase 2 details
│   │   ├── phase-3-analytics.md       # Phase 3 details
│   │   └── phase-4-integrations.md    # Phase 4 details
│   │
│   ├── setup/                         # Setup and installation
│   │   ├── README.md                  # Setup overview
│   │   ├── nextjs.md                  # Next.js setup guide
│   │   ├── laravel.md                 # Laravel setup guide
│   │   ├── symfony.md                 # Symfony setup guide
│   │   ├── docker.md                  # Docker setup
│   │   └── deployment.md              # Deployment guides
│   │
│   ├── user-guide/                    # End-user documentation
│   │   ├── README.md                  # User guide index
│   │   ├── getting-started.md         # First steps
│   │   ├── clients.md                 # Managing clients
│   │   ├── quotes.md                  # Creating quotes
│   │   ├── projects.md                # Managing projects
│   │   ├── invoices.md                # Creating invoices
│   │   ├── documents.md               # Uploading files
│   │   ├── portal.md                  # Client portal guide
│   │   ├── tutorials.md               # Video tutorials
│   │   └── faq.md                     # Frequently asked questions
│   │
│   └── translations/                  # Translation management
│       ├── README.md                  # Translation guide
│       ├── TRANSLATORS.md             # Credits
│       ├── en.json                    # English (base)
│       ├── pl.json                    # Polish
│       └── es.json                    # Spanish
│
├── .github/                           # GitHub configuration
│   ├── workflows/                     # CI/CD pipelines
│   │   ├── test.yml                   # Run tests
│   │   ├── lint.yml                   # Code linting
│   │   └── deploy.yml                 # Deployment
│   │
│   ├── ISSUE_TEMPLATE/                # Issue templates
│   │   ├── bug_report.md              # Bug report template
│   │   ├── feature_request.md         # Feature request template
│   │   └── question.md                # Question template
│   │
│   ├── PULL_REQUEST_TEMPLATE.md       # PR template
│   └── CODEOWNERS                     # Code ownership
│
├── apps/                              # Application implementations
│   ├── nextjs/                        # Next.js app
│   │   ├── README.md                  # Next.js specific docs
│   │   ├── src/
│   │   ├── public/
│   │   ├── package.json
│   │   └── ...
│   │
│   ├── laravel/                       # Laravel app
│   │   ├── README.md                  # Laravel specific docs
│   │   ├── app/
│   │   ├── routes/
│   │   ├── composer.json
│   │   └── ...
│   │
│   └── symfony/                       # Symfony app
│       ├── README.md                  # Symfony specific docs
│       ├── src/
│       ├── config/
│       ├── composer.json
│       └── ...
│
├── packages/                          # Shared packages
│   ├── ui/                            # UI component library
│   │   ├── README.md                  # UI docs
│   │   └── src/
│   │
│   ├── types/                         # Shared TypeScript types
│   │   ├── README.md                  # Types docs
│   │   └── src/
│   │
│   └── utils/                         # Shared utilities
│       ├── README.md                  # Utils docs
│       └── src/
│
└── scripts/                           # Utility scripts
    ├── setup.sh                       # Initial setup
    ├── create-issues.sh               # Create GitHub issues
    └── generate-docs.sh               # Generate documentation
```

## 📚 Documentation Types

### 1. Getting Started Documentation
**Audience:** New users, new developers  
**Location:** `README.md`, `docs/overview.md`, `docs/setup/`

Start here to understand what BuildFlow is and how to get it running.

### 2. Business Documentation
**Audience:** Product managers, stakeholders, business analysts  
**Location:** `docs/business/`

Complete business requirements, market analysis, and monetization strategy.

### 3. Product Documentation
**Audience:** Product managers, designers, QA  
**Location:** `docs/product/`

User stories, personas, workflows, and success metrics.

### 4. Technical Documentation
**Audience:** Developers, architects, DevOps  
**Location:** `docs/technical/`, `docs/api/`

System architecture, data models, API references, and infrastructure.

### 5. User Documentation
**Audience:** End users (contractors, clients)  
**Location:** `docs/user-guide/`

How-to guides, tutorials, and FAQ for using the application.

### 6. Development Documentation
**Audience:** Contributors, developers  
**Location:** `CONTRIBUTING.md`, `docs/setup/`, `docs/roadmap/`

How to contribute, setup guides, and development roadmap.

## 🎯 Navigation Paths

### For New Users
1. Start with `README.md`
2. Read `docs/overview.md`
3. Check `docs/user-guide/getting-started.md`
4. Watch videos in `docs/user-guide/tutorials.md`

### For New Developers
1. Read `README.md`
2. Review `docs/overview.md`
3. Read `CONTRIBUTING.md`
4. Follow setup in `docs/setup/[your-stack].md`
5. Check `docs/roadmap/github-roadmap.md` for issues

### For Product Managers
1. Read `docs/overview.md`
2. Review `docs/business/requirements.md`
3. Check `docs/product/user-stories.md`
4. Review `docs/roadmap/github-roadmap.md`

### For Architects/Tech Leads
1. Review `docs/technical/architecture.md`
2. Study `docs/technical/data-model.md`
3. Check `docs/api/README.md`
4. Review `docs/technical/infrastructure.md`

## 📖 Key Documents

### Must-Read Documents

| Document | Audience | Purpose |
|----------|----------|---------|
| `README.md` | Everyone | Project introduction |
| `docs/overview.md` | Everyone | Detailed project overview |
| `docs/business/requirements.md` | PM, Dev | Complete specifications |
| `docs/roadmap/github-roadmap.md` | PM, Dev | Development plan |
| `CONTRIBUTING.md` | Developers | How to contribute |
| `docs/setup/[stack].md` | Developers | Development setup |

### Reference Documents

| Document | Audience | Purpose |
|----------|----------|---------|
| `docs/technical/architecture.md` | Architects | System design |
| `docs/technical/data-model.md` | Developers | Database schema |
| `docs/api/README.md` | Developers | API reference |
| `docs/product/personas.md` | PM, UX | User personas |
| `docs/user-guide/faq.md` | Users | Common questions |

## 🔄 Documentation Updates

### When to Update Documentation

- **Feature added** → Update user guide and technical docs
- **API changed** → Update API documentation
- **Bug fixed** → Update changelog
- **Process changed** → Update contributing guide
- **Setup changed** → Update setup guides

### Documentation Review Process

1. Changes to docs require PR like code
2. Technical docs reviewed by maintainers
3. User docs reviewed by community
4. All docs checked for broken links
5. Changelog updated for releases

## 🌍 Translations

Translation files are in `docs/translations/`:
- `en.json` - English (base language)
- `pl.json` - Polish
- `es.json` - Spanish (future)
- `de.json` - German (future)

See `docs/translations/README.md` for translation guidelines.

## 🔗 External Links

### Community
- Website: https://buildflow.dev (coming soon)
- Discord: https://discord.gg/buildflow (coming soon)
- Twitter: @buildflow_app (coming soon)

### Development
- GitHub: https://github.com/yourusername/buildflow
- Issues: https://github.com/yourusername/buildflow/issues
- Discussions: https://github.com/yourusername/buildflow/discussions

## 📝 Documentation Standards

### Writing Style

- **Clear and Concise** - Get to the point quickly
- **Examples** - Show don't just tell
- **Screenshots** - Visual aids where helpful
- **Links** - Link to related documentation
- **Up-to-date** - Keep docs current with code

### Formatting

- Use Markdown for all docs
- Use proper heading hierarchy (H1 → H2 → H3)
- Include table of contents for long docs
- Use code blocks with syntax highlighting
- Use tables for comparison data

### Templates

See `.github/ISSUE_TEMPLATE/` for standard templates:
- Bug reports
- Feature requests
- Questions
- Pull requests

## 🚀 Quick Links

- [Main README](../../README.md)
- [Project Overview](overview.md)
- [Business Requirements](business/requirements.md)
- [GitHub Roadmap](roadmap/github-roadmap.md)
- [Contributing Guide](../../CONTRIBUTING.md)
- [User Guide](user-guide/README.md)

---

**Need help?** Open an issue or join our Discord!

**Last Updated:** November 12, 2025
