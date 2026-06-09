# Contributing to DirectEdu

## Code of Conduct

Please be respectful and constructive in all interactions.

## Development Setup

1. Fork and clone the repository
2. Install dependencies: `npm install`
3. Copy `.env.example` to `.env`
4. Start development: `npm run docker:up`
5. Run migrations: `npm run migrate`

## Git Workflow

1. Create a feature branch: `git checkout -b feature/your-feature`
2. Commit changes: `git commit -m "feat: your feature"`
3. Push to branch: `git push origin feature/your-feature`
4. Create Pull Request

## Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:** feat, fix, docs, style, refactor, test, chore, perf

## Code Style

- Run `npm run lint` before committing
- Run `npm run format` to auto-format code
- Follow existing code patterns

## Testing

- Write tests for new features
- Run `npm test` to verify
- Maintain coverage above 80%

## Pull Request Process

1. Update documentation
2. Add tests for changes
3. Pass all CI checks
4. Get code review approval
5. Squash and merge

---

Thank you for contributing to DirectEdu!