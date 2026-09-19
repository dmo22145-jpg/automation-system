# Contributing to Instagram Automation Tool

Thank you for considering contributing to this project! We welcome contributions from everyone.

## How to Contribute

### Reporting Bugs

If you find a bug, please open an issue with:
- Clear description of the problem
- Steps to reproduce
- Expected vs actual behavior
- Screenshots if applicable
- Your environment (OS, Node version, etc.)

### Suggesting Features

We love new ideas! Please open an issue with:
- Clear description of the feature
- Use case and benefits
- Mockups or examples (if applicable)

### Code Contributions

1. **Fork the repository**
2. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Make your changes**
4. **Test thoroughly**
5. **Commit with clear messages**
   ```bash
   git commit -m "Add feature: your feature description"
   ```
6. **Push to your fork**
   ```bash
   git push origin feature/your-feature-name
   ```
7. **Open a Pull Request**

## Development Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/instagram-automation
   cd instagram-automation
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Set up environment variables**
   ```bash
   cp .env.example .env
   # Edit .env with your credentials
   ```

4. **Set up database**
   ```bash
   npx drizzle-kit push
   ```

5. **Run development server**
   ```bash
   npm run dev
   ```

## Code Style

- Use TypeScript for all new code
- Follow existing code formatting
- Use meaningful variable and function names
- Add comments for complex logic
- Keep functions small and focused

## Testing

Before submitting a PR:

1. **Run type checking**
   ```bash
   npm run typecheck
   ```

2. **Build the project**
   ```bash
   npm run build
   ```

3. **Test manually**
   - Test your changes in the browser
   - Test webhook integration
   - Test campaign creation and editing

## Pull Request Guidelines

- Keep PRs focused on a single feature/fix
- Update documentation if needed
- Add comments explaining complex code
- Ensure all tests pass
- Reference any related issues

## Code Review Process

1. Maintainers will review your PR
2. Feedback will be provided via PR comments
3. Make requested changes if needed
4. Once approved, your PR will be merged

## Areas for Contribution

We especially welcome contributions in:

- **UI/UX improvements** - Make the dashboard more user-friendly
- **Error handling** - Better error messages and recovery
- **Testing** - Add unit and integration tests
- **Documentation** - Improve guides and API docs
- **Performance** - Optimize database queries and API calls
- **Features** - Analytics dashboard, scheduling, multiple accounts, etc.

## Feature Ideas

Some ideas for new features:

- [ ] Analytics dashboard (response rates, engagement metrics)
- [ ] Schedule campaigns for specific times
- [ ] Support for multiple Instagram accounts
- [ ] Campaign templates library
- [ ] A/B testing for messages
- [ ] Rich media support in DMs
- [ ] Automated token refresh
- [ ] Export campaign performance data
- [ ] Role-based access control
- [ ] Comment sentiment analysis

## Questions?

Feel free to:
- Open an issue for discussion
- Reach out to maintainers
- Join our community discussions

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Happy Contributing! 🚀**
