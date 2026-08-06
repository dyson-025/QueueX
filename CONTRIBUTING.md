# Contributing to QueueX

Thank you for your interest in contributing to QueueX.

## Development Setup

1. Fork the repository.
2. Clone your fork.
3. Install dependencies.
4. Copy `.env.example` to `.env`.
5. Start the development environment with Docker Compose.
6. Run database migrations.
7. Start the API and worker services.

---

## Branch Naming

Use descriptive branch names:

* `feature/authentication`
* `feature/worker-engine`
* `feature/dashboard`
* `fix/retry-logic`
* `docs/readme`

---

## Commit Message Convention

Follow Conventional Commits:

* `feat:` New feature
* `fix:` Bug fix
* `docs:` Documentation
* `refactor:` Code refactoring
* `test:` Tests
* `chore:` Maintenance

Examples:

```text
feat: add JWT authentication
fix: resolve worker heartbeat timeout
docs: update architecture diagrams
```

---

## Coding Standards

* Follow PEP 8.
* Use type hints.
* Keep functions focused and small.
* Write meaningful commit messages.
* Document public APIs.

---

## Pull Requests

Before opening a pull request:

* Ensure all tests pass.
* Run the formatter and linter.
* Update documentation if required.
* Add tests for new functionality.

---

## Reporting Issues

When reporting an issue, include:

* Environment
* Steps to reproduce
* Expected behavior
* Actual behavior
* Logs or screenshots (if applicable)

---

Thank you for helping improve QueueX!
