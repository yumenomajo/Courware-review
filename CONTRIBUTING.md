# Contributing

Thank you for your interest in contributing to Courseware Review! This document provides guidelines for contributing to this project.

---

## What We're Looking For

- **Bug fixes** in the skill definition (`SKILL.md`)
- **Rule improvements** that make the review process more rigorous or reliable
- **Fallback strategy enhancements** for better cross-platform support
- **Documentation improvements** for clarity and accessibility
- **Example outputs** demonstrating the skill in action

---

## How to Contribute

### 1. Fork and Clone

```bash
git fork https://github.com/yumenomajo/Courware-review.git
cd Courware-review
```

### 2. Create a Branch

```bash
git checkout -b feat/your-feature-name
# or
git checkout -b fix/your-bug-fix
```

### 3. Make Your Changes

- Edit `SKILL.md` for skill logic changes
- Edit `README.md` or `ARCHITECTURE.md` for documentation changes
- Add example outputs to the `examples/` directory (if applicable)

### 4. Test

Test your changes with a real PDF in Claude Code:

1. Install the modified `SKILL.md` into your Claude Code skills directory
2. Run the skill with a test PDF
3. Verify the output matches the expected format and quality

### 5. Commit and Push

```bash
git add SKILL.md          # or other changed files
git commit -m "feat: describe your change"
git push origin feat/your-feature-name
```

### 6. Open a Pull Request

Create a PR against the `main` branch. Include:
- A clear description of what you changed and why
- Before/after examples if applicable
- Any testing you performed

---

## Style Guidelines

- Use **markdown** for all documentation files
- Use **tables** for structured data (rules, phases, schemas)
- Use **code blocks** for commands and output formats
- Keep descriptions concise but precise
- Chinese content in the main skill (`SKILL.md`) should remain Chinese — English translations belong in `README.md`

---

## Code of Conduct

- Be respectful in all interactions
- Provide constructive feedback on PRs and issues
- Test your changes before submitting

---

## Reporting Issues

Please include:
- Claude Code version
- Operating system
- PDF type (text-based / scanned / slides / textbook)
- The specific rule or phase that had the issue
- Expected vs actual behavior

---

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
