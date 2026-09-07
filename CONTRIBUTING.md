# Contributing to Omega

Thank you for your interest in contributing to Omega\! This document provides guidelines and instructions for contributing to this project. Omega is a neural-symbolic agent framework built on the Hyperon AGI stack, and we welcome contributions from the community.

## Table of Contents

- [Code of Conduct](#code-of-conduct)  
- [How to Contribute](#how-to-contribute)  
- [Development Setup](#development-setup)  
- [Pull Request Process](#pull-request-process)  
- [Coding Standards](#coding-standards)  
- [Writing Documentation](#writing-documentation)  
- [Reporting Bugs](#reporting-bugs)  
- [Suggesting Enhancements](#suggesting-enhancements)  
- [Community](#community)  
- [License](#license)

---

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](https://docs.google.com/document/d/1GqFxlwpSRnyytYxiTi_eN987rPeMet9DJnYyazsT-Zc/edit?tab=t.0). By participating, you are expected to uphold this code. Please report unacceptable behavior to the maintainers.

---

## How to Contribute

There are many ways to contribute to Omega:

- **Bug fixes**: Fix issues reported by the community in the [issue tracker](https://github.com/asi-alliance/OmegaClaw-Core/issues) or found during your own testing.  
- **New features**: Propose and implement new capabilities for the agent framework.  
- **New skills:** Add new MeTTa skills following the skill dispatch architecture (see [tutorial-03](http://docs/tutorial-03-writing-a-custom-skill.md)).  
- **New channels:** Build communication channel adapters (see [tutorial-04](http://docs/tutorial-04-adding-a-channel.md)).  
- **New plugins**: Develop new plugins, or extensions that enhance Omega's functionality.  
- **Documentation**: Improve or expand documentation, tutorials, or inline code comments.  
- **Tests**: Add unit tests, integration tests, or autotest scenarios to improve code coverage and reliability.  
- **Code review**: Review open pull requests and provide constructive feedback.  
- **Provider integrations**: Add support for new LLM providers or communication channels.  
- **Docker:** Improve containerization, CI workflows, or deployment scripts.  
- **Reasoning engines:** Improve or extend NAL, PLN, or ONA reasoning integrations.

---

## Development Setup

### Fork and Clone

1. Fork the [Omega](https://github.com/singnet/Omega) repository.  
2. Clone your fork locally:

```shell
git clone https://github.com/<your-username>/Omega.git
cd Omega
```

3. Add the upstream remote to keep your fork synchronized:

```shell
git remote add upstream https://github.com/singnet/Omega.git
```

### Installation

Installation process described at [Readme](https://github.com/singnet/Omega#installation)

### Configuration and Usage

For usage and configuration instructions please refer to project [Readme](https://github.com/singnet/Omega#installation)

### Running Tests

Tests live in two places.

`tests/` holds the unit tests. The Python ones run on a plain checkout:

```shell
./tests/pytest.sh
```

The MeTTa ones need a PeTTa tree, so they run inside the container:

```shell
docker exec -e PETTA_PATH=/PeTTa omega /PeTTa/repos/Omega/tests/mettatest.sh
```

`Autotests/` holds the scenario tests, which drive a running agent, plus a unit tier in `Autotests/unit/`. Build the image, start an agent, then run the suite from that directory:

```shell
docker build -t omega:dev .
export TEST_SERVER_IP=host.docker.internal
./scripts/omega start -p Test -t test -d omega:dev -g http://host.docker.internal:18789
cd Autotests
pytest -s -v @run_mandatory
```

`run_mandatory` has to pass: CI treats it as blocking. `run_optional` is the non-blocking list, and one case in it needs `OMEGA_GIT_TOKEN`. The rest of `Autotests/` runs against real providers and needs their API keys, see `Autotests/README_live.md`.

CI runs all of this on every pull request to `main`: `build.yml` calls `common.yml`, which calls `autotests.yml`.

---

## Pull Request Process

1. **Create a branch** from `main` with a descriptive name:

```shell
git checkout main
git pull upstream main
git checkout -b fix/your-bug-fix    # or feature/your-feature, docs/your-docs
git push origin fix/your-bug-fix
```

2. **Make your changes** following the coding standards below.  
     
3. **Write tests** for your changes if applicable. Ensure all [existing tests](#running-tests) pass.

4. **Update documentation** if you are adding features, changing behavior, or modifying configuration options.  
     
5. **Commit** with clear, conventional commit messages:

```shell
git commit -m "feat(skills): add web-search skill with result caching"
git commit -m "fix(loop): handle empty LLM response in main loop"
git commit -m "docs(channels): update Telegram adapter setup guide"
```

We follow [Conventional Commits](https://www.conventionalcommits.org/) prefixes:

- `feat:` — new features  
- `fix:` — bug fixes  
- `docs:` — documentation only  
- `refactor:` — code changes that neither fix bugs nor add features  
- `test:` — adding or updating tests  
- `chore:` — build process, tooling, CI/CD  
- `perf:` — performance improvements  
    
6. **Open a Pull Request** against the `singnet/Omega` `main` branch. Use the PR framework provided at .github/pull\_request\_template.md, which includes a checklist including a "PR contains autogenerated code" disclosure. Provide a clear description of the change, its motivation, and any relevant issue numbers in the form.  
     
7. **Address review feedback** promptly. Maintainers may request changes, additional tests, or documentation updates.

---

## Coding Standards

### General

- No dead code: Remove unused imports, variables, and functions before submitting.  
- Consistent formatting: Use the same indentation and line length conventions as the surrounding code.  
- Backward compatibility: Avoid breaking changes to existing APIs. If a breaking change is necessary, discuss it in an issue first and document the migration path.

### Python Code

- Follow [PEP 8](https://peps.python.org/pep-0008/) style guidelines.  
- Use type hints for function signatures and return values where practical.  
- Write docstrings for all public functions, classes, and modules following the [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html#384-functions-and-methods).  
- Keep functions focused and concise. If a function exceeds \~50 lines, consider breaking it up.  
- Import order: standard library, third-party packages, local modules (separated by blank lines).

### MeTTa Code

- Follow the existing skill dispatch pattern documented in [reference-internals-skill-dispatch.md](http://docs/reference-internals-skill-dispatch.md).  
- New skills must conform to the **Signature → Purpose → Parameters → Returns → Examples → Notes/Limits** template (see existing reference docs for examples).  
- Keep the MeTTa core (`src/*.metta`) minimal — the design goal is simplicity and transparency.  
- Add inline comments explaining non-obvious symbolic reasoning patterns.

### Prolog (SWI-Prolog) Code

- Follow existing conventions in `src/skills.pl`.  
- Document predicate arity, expected input formats, and failure modes.

### Shell Scripts

- Use `#!/usr/bin/env bash` shebang.  
- Follow [ShellCheck](https://www.shellcheck.net/) recommendations — ensure scripts pass `shellcheck` with no errors.  
- Quote all variable expansions unless you explicitly require word splitting.

### Docker

- Base images should be pinned by digest or specific tag (avoid `latest` in Dockerfiles).  
- Follow multi-stage build patterns to minimize image size where possible.  
- Document any port mappings, volume mounts, and environment variables in comments.

---

## Writing Documentation

Omega uses flat Markdown files in the [`docs/`](http://docs/) directory. Documentation is organized by prefix:

| Prefix | Type | Example |
| :---- | :---- | :---- |
| `intro-*` | Conceptual introduction | `introduction.md` |
| `tutorial-NN-*` | Numbered, task-oriented walkthrough | `tutorial-03-writing-a-custom-skill.md` |
| `reference-*` | API, engines, internals | `reference-skills-memory.md` |

### Guidelines

- Tutorials must be self-contained and walk the reader through a concrete task step by step.  
- Reference pages must follow the standard template: Signature → Purpose → Parameters → Returns → Examples → Notes/Limits.  
- Use relative links for intra-repo references.  
- Include runnable code examples wherever possible.  
- If adding a new skill or channel, you must add or update the corresponding reference documentation.

---

## Reporting Bugs

If you find a bug, please open an issue on the [issue tracker](https://github.com/singnet/Omega/issues) by choosing one of the issue forms, per the screenshot below:  
![][image1]

In your report, be sure to include all relevant details such as:

1. Clear title and description — What happened and what you expected to happen.  
2. Steps to reproduce — Minimal, specific steps that trigger the bug.  
3. Environment details —  
   - OS and version  
   - Python version  
   - SWI-Prolog version  
   - LLM provider and model used  
   - Omega version or commit hash  
4. Logs or error output — Paste relevant terminal output or log snippets.  
5. Configuration — Any non-default configuration settings (redact API keys).

---

## Suggesting Enhancements

We welcome feature ideas\! Please open an issue with:

- A clear description of the proposed feature and its use case.  
- Motivation — Why is this feature useful to Omega users?  
- Possible implementation approach — If you have ideas on how it could be built (optional but helpful).

---

## Community

- **Issues**: [github.com/singnet/Omega/issues](https://github.com/singnet/Omega/issues)  
- **Telegram**: Chat with the Omega Developers community at [BGI Commons Telegram Channel](https://t.me/+AWxn8CUgdyE3Y2Jh)  
- **Discord**: Chat with the Omega Developers at [SingularityNET Discord](https://discord.gg/snet)

---

## License

By contributing to Omega, you agree that your contributions will be licensed under the [Apache License 2.0](http://LICENSE).  


[image1]: <data:image/png;base64,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>