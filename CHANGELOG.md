# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2024-01-15

### Added
- Initial release of Instagram Comment-to-DM Automation Tool
- Webhook receiver for Instagram comment events
- Automatic comment reply functionality
- Automatic DM sending to commenters
- Campaign management system
- Post preview with thumbnail and caption
- Keyword-based trigger system (case-insensitive, partial match)
- Comment deduplication to prevent double-processing
- Web dashboard with Settings and Campaigns tabs
- Instagram API integration (Graph API v21.0)
- HMAC signature verification for webhook security
- Exponential backoff retry logic for rate-limited requests
- PostgreSQL database with Drizzle ORM
- Docker support with standalone build
- Deployment configurations for Railway and Render
- Comprehensive documentation (README, SETUP_GUIDE, QUICK_START, API_DOCUMENTATION)
- Health check endpoint for deployment monitoring
- Environment variable configuration
- Campaign activation/deactivation toggle
- Campaign editing and deletion
- Post ID auto-fetch and validation

### Security
- Webhook signature verification using HMAC-SHA256
- Environment-based credential management
- Masked credentials in API responses
- HTTPS-only webhook handling

### Documentation
- Complete setup guide for Instagram API configuration
- Quick start guide for rapid deployment
- API documentation with endpoint reference
- Troubleshooting guide
- Contributing guidelines
- MIT License

---

## Future Releases

### [1.1.0] - Planned
- [ ] Analytics dashboard
- [ ] Campaign performance metrics
- [ ] Export functionality
- [ ] Campaign templates
- [ ] Rich text editor for messages

### [1.2.0] - Planned
- [ ] Multiple Instagram account support
- [ ] Team collaboration features
- [ ] Role-based access control
- [ ] Webhook event logs viewer

### [2.0.0] - Planned
- [ ] Advanced automation rules
- [ ] Conditional logic for campaigns
- [ ] A/B testing for messages
- [ ] Scheduled campaigns
- [ ] AI-powered response generation

---

## Versioning

This project uses [Semantic Versioning](https://semver.org/):
- **MAJOR** version for incompatible API changes
- **MINOR** version for new functionality (backward-compatible)
- **PATCH** version for bug fixes (backward-compatible)

## Release Process

1. Update CHANGELOG.md
2. Update version in package.json
3. Create git tag: `git tag -a v1.0.0 -m "Release 1.0.0"`
4. Push tags: `git push origin --tags`
5. Create GitHub release with changelog
