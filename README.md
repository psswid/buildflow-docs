# BuildFlow

> Open-source business management system for construction and renovation businesses

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

---

## 🎯 Vision

BuildFlow empowers small construction businesses to operate with the professionalism of large enterprises. We replace scattered spreadsheets, emails, and paper notes with a unified platform that's accessible, affordable, and actually built for contractors.

---

## ✨ Key Features

### 📊 Core Functionality
- **Client Management** - Centralized contact database with history
- **Professional Quotes** - Beautiful PDFs in minutes, not hours
- **Project Tracking** - Visual timelines and status updates
- **Invoice Management** - Automatic calculations and payment tracking
- **Document Storage** - Photos and files organized by project
- **Client Portal** - Modern transparency that clients love (Pro tier)

### 🎁 What Makes BuildFlow Different
- **Forever Free Tier** - 3 projects, 10 clients, full features
- **Mobile First** - Built for contractors who work on-site
- **Technology Agnostic** - Multiple implementations (Next.js, Laravel, Symfony)
- **Open Source** - Transparent, customizable, community-driven

---

## 🚀 Quick Start

```bash
# Clone the repository
git clone https://github.com/yourusername/buildflow.git

# Choose your stack
cd buildflow/apps/nextjs  # or /apps/laravel or /apps/symfony

# Follow stack-specific setup instructions
# See docs/setup/[nextjs|laravel|symfony].md
```

---

## 📚 Documentation

### Getting Started
- [Project Overview](docs/overview.md) - Understanding BuildFlow
- [Business Requirements](docs/business/requirements.md) - Complete feature specifications
- [Setup Guide](docs/setup/README.md) - Installation for each stack

### For Developers
- [Architecture](docs/technical/architecture.md) - System design and patterns
- [Data Model](docs/technical/data-model.md) - Database schema and relationships
- [API Documentation](docs/api/README.md) - RESTful API reference
- [Contributing Guide](CONTRIBUTING.md) - How to contribute

### For Product Managers
- [User Stories](docs/product/user-stories.md) - All user stories and use cases
- [Roadmap](docs/roadmap/github-roadmap.md) - Development phases and milestones
- [Success Metrics](docs/product/metrics.md) - KPIs and success criteria

### For Users
- [User Guide](docs/user-guide/README.md) - How to use BuildFlow
- [FAQ](docs/user-guide/faq.md) - Frequently asked questions
- [Video Tutorials](docs/user-guide/tutorials.md) - Step-by-step guides

---

## 🏗️ Project Structure

```
buildflow/
├── apps/                    # Multiple stack implementations
│   ├── nextjs/             # Next.js implementation
│   ├── laravel/            # Laravel implementation
│   └── symfony/            # Symfony implementation
│
├── packages/               # Shared packages
│   ├── ui/                # UI components library
│   ├── types/             # TypeScript types
│   └── utils/             # Shared utilities
│
├── docs/                  # Documentation
│   ├── business/          # Business requirements
│   ├── technical/         # Technical documentation
│   ├── product/           # Product documentation
│   ├── roadmap/           # Development roadmap
│   ├── setup/             # Setup guides
│   └── user-guide/        # User documentation
│
├── .github/               # GitHub specific files
│   ├── workflows/         # CI/CD workflows
│   ├── ISSUE_TEMPLATE/    # Issue templates
│   └── PULL_REQUEST_TEMPLATE.md
│
└── scripts/               # Utility scripts
```

---

## 💰 Pricing

### Starter (Free Forever)
- 3 active projects
- 10 clients
- 5 quotes/month
- 3 invoices/month
- 100 MB storage
- All core features

### Professional ($19-29/month)
- 20 active projects
- 100 clients
- 50 quotes/month
- 30 invoices/month
- 10 GB storage
- **Client portal access**
- Branded PDFs
- Advanced analytics

### Business ($49-79/month)
- Unlimited projects & clients
- Unlimited quotes & invoices
- 100 GB storage
- 10 team members
- **Team collaboration**
- **API access**
- SMS notifications
- Priority support

---

## 🎯 Target Users

### Solo Contractor Sam
35-50 years old, works alone or with 1-2 helpers, does 20-50 projects/year. Needs to look professional without spending hours on admin work.

### Growing Business Grace
30-45 years old, team of 3-8 employees, 50-200 projects/year. Needs team collaboration and client transparency to scale efficiently.

---

## 📅 Development Roadmap

### Phase 1: MVP (Months 1-3) ✅ Current Phase
- Core features: Clients, Quotes, Projects, Invoices, Documents
- Professional PDF generation
- Basic dashboard
- Email notifications

### Phase 2: Client Portal (Months 4-6)
- Client authentication and portal
- Project status visibility for clients
- Photo sharing and messaging
- Quote/invoice templates

### Phase 3: Analytics & Team (Months 7-9)
- Advanced reporting
- Multi-user support with RBAC
- Team collaboration features
- Calendar and scheduling

### Phase 4: Integrations (Months 10-12)
- Payment processing (Stripe)
- Accounting software integration
- Public API
- White-label options

See [detailed roadmap](docs/roadmap/github-roadmap.md) for complete issue breakdown.

---

## 🤝 Contributing

We welcome contributions! BuildFlow is built by contractors, for contractors.

### Ways to Contribute
- 🐛 Report bugs
- 💡 Suggest features
- 📝 Improve documentation
- 💻 Submit pull requests
- 🌍 Translate to other languages
- ⭐ Star the project

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Good First Issues
Look for issues tagged with `good first issue` - perfect for newcomers!

---

## 🛠️ Tech Stacks

BuildFlow supports multiple technology implementations:

### Next.js (TypeScript)
- Next.js 14+ with App Router
- React Server Components
- Prisma ORM + PostgreSQL
- Tailwind CSS + Shadcn/ui
- NextAuth.js

### Laravel (PHP)
- Laravel 11+
- Eloquent ORM + MySQL/PostgreSQL
- Livewire + Alpine.js
- Tailwind CSS
- Laravel Sanctum

### Symfony (PHP)
- Symfony 7+
- Doctrine ORM + PostgreSQL
- Twig + Stimulus
- Tailwind CSS
- Symfony Security

---

## 📊 Project Stats

- **Issues:** 70 planned features
- **Milestones:** 5 phases over 10 months
- **Target Users:** 10,000+ in year 1
- **Contributors:** Open to all!

---

## 📄 License

BuildFlow is open source software licensed under the [MIT License](LICENSE).

---

## 🌟 Support the Project

- ⭐ Star this repository
- 🐛 Report bugs and request features
- 💬 Join discussions
- 🔗 Share with other contractors
- 💰 Sponsor development (coming soon)

---

## 📞 Contact & Community

- **Website:** [buildflow.dev](https://buildflow.dev) (coming soon)
- **Twitter:** [@buildflow_app](https://twitter.com/buildflow_app) (coming soon)
- **Discord:** [Join our community](https://discord.gg/buildflow) (coming soon)
- **Email:** hello@buildflow.dev

---

## 🙏 Acknowledgments

Built with ❤️ by [Piotr Świderski](https://github.com/psswid)

Inspired by the real needs of construction professionals, starting with [Prime Build Develop](https://www.primebuilddevelop.com/).

---

**Status:** 🚧 In Development - Phase 1 (MVP)  
**Version:** 0.1.0  
**Last Updated:** November 12, 2025