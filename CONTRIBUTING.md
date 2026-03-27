# Contributing

Contributions are welcome — features, bug fixes, and documentation
clarifications.

## Getting Started

1. Fork the repository
2. Clone your fork
3. Install [mise](https://mise.jdx.dev/) — all dev dependencies are managed
   through mise
4. Run `mise install` to set up your environment

## Development Workflow

1. Make your changes
2. Commit using [Conventional Commits](#conventional-commits)
3. Push to your fork and open a pull request

### Making `go` changes

1. Run `go fmt ./...` to format your code (ideally use an editor that formats on
   save)
2. Run `go test -cover ./...` and verify all tests pass with 100% coverage

## Conventional Commits

All repositories enforce
[Conventional Commits](https://www.conventionalcommits.org/). Commits that do
not follow the convention will be rejected.

[`cog`](https://docs.cocogitto.io/guide/init.html) should be available through
the `mise` install to ease conventional commit management.

Common prefixes:

- `feat:` — new functionality
- `fix:` — bug fix
- `docs:` — documentation only
- `refactor:` — code change that neither fixes a bug nor adds a feature
- `test:` — adding or updating tests
- `chore:` — maintenance tasks

## Testing

### `go` testing

Tests are expected to achieve 100% coverage. Run tests with:

```
go test -cover ./...
```

## Pull Requests

- PRs come from forks
- Keep PRs focused — one logical change per PR
- Ensure CI passes before requesting review
- Reference any related issues in the PR description

## License

By contributing, you agree that your contributions will be licensed under the
[Apache License 2.0](LICENSE).
