## ✅ Pre-Submission Checklist

Before you open a Pull Request, confirm that you have completed all items below.

**Tip:** Copy this checklist into your PR description and tick each item.

### 🔹 Functional Requirements

- [x] FastAPI app with `/signup` and `/login` routes implemented.
- [x] At least 2 unit tests written and passing.
- [x] Dockerfile builds the app image successfully.
- [x] docker-compose runs app + Postgres together.
- [x] GitHub Actions workflow runs tests and builds the image.
- [ ] Secrets stored via environment variables (not hardcoded).

### 🔹 Code Quality & Structure

- [x] Repo structure matches provided scaffolding.
- [x] Code is modular and documented.
- [x] Tests are clear and reproducible.
- [x] No unused files or dependencies.

### 🔹 Documentation

- [x] `README.md` includes setup and execution instructions.
- [x] `docs/decision-log.md` explains tool choices and design decisions.
- [x] Environment variables and ports are documented.

### 🔹 Git & Collaboration

- [x] Commits are small and descriptive.
- [x] Branch name is descriptive (e.g., `feature/fastapi-auth`).
- [x] I have reviewed at least one existing PR in the repository.
- [x] I am ready to respond to reviewer feedback in the PR discussion.
