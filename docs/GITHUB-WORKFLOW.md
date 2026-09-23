# GitHub Workflow & Issue Management

**Status:** PLANNING  
**Last Updated:** 2026-09-22  
**Purpose:** Structured approach to development, issues, and collaboration

## Overview

This document establishes GitHub workflows for ArenaServ development, ensuring organized, trackable, and collaborative work aligned with the project roadmap.

---

## Repository Structure

### Branch Strategy

```
main (protected)
  ├── develop (integration branch)
  │   ├── feature/arena-queue-system
  │   ├── feature/match-lifecycle
  │   ├── bugfix/timer-cleanup-uaf
  │   └── research/virtual-windows
  └── hotfix/critical-crash-fix (emergency only)
```

**Branch Types:**

| Type | Naming | Purpose | Lifetime |
|------|--------|---------|----------|
| `main` | - | Production-ready | Permanent |
| `develop` | - | Integration | Permanent |
| `feature/*` | `feature/description` | New features | Until merged |
| `bugfix/*` | `bugfix/issue-description` | Bug fixes | Until merged |
| `research/*` | `research/question-id` | Proof-of-concepts | Until merged/closed |
| `hotfix/*` | `hotfix/critical-issue` | Emergency fixes | Until merged |
| `docs/*` | `docs/documentation-update` | Documentation | Until merged |

### Branch Protection Rules

**`main` branch:**
- ✅ Require pull request reviews (1+ approvers)
- ✅ Require status checks to pass (CI/CD)
- ✅ Require conversation resolution
- ✅ Do not allow force pushes
- ✅ Require linear history

**`develop` branch:**
- ✅ Require pull request reviews (1+ approvers)
- ✅ Require status checks to pass
- ⚠️ Allow force pushes (with care)

---

## Issue Management

### Issue Labels

#### Type Labels (Mutually Exclusive)
- `type: bug` 🐛 - Something isn't working
- `type: feature` ✨ - New feature request
- `type: research` 🔬 - Investigation needed
- `type: documentation` 📚 - Documentation improvement
- `type: refactor` ♻️ - Code restructuring
- `type: performance` ⚡ - Performance improvement

#### Priority Labels
- `priority: critical` 🔴 - Blocks development, crashes, data loss
- `priority: high` 🟠 - Important for upcoming release
- `priority: medium` 🟡 - Should be addressed soon
- `priority: low` 🟢 - Nice-to-have, not urgent

#### Phase Labels
- `phase: 0-research` - Phase 0 archaeology
- `phase: 1-foundation` - Phase 1 bugs & research
- `phase: 2-core` - Phase 2 core features
- `phase: 3-stats` - Phase 3 statistics
- `phase: 4-competitive` - Phase 4 ranked
- `phase: 5-advanced` - Phase 5 tournaments

#### Status Labels
- `status: blocked` 🚫 - Blocked by another issue
- `status: in-progress` 🔄 - Actively being worked on
- `status: needs-review` 👀 - Pull request needs review
- `status: needs-testing` 🧪 - Needs validation
- `status: wontfix` ⛔ - Will not be addressed

#### Component Labels
- `component: arena` - Arena system
- `component: database` - Database/persistence
- `component: networking` - Packets/protocol
- `component: ui` - User interface (windows)
- `component: combat` - Combat mechanics
- `component: matchmaking` - Queue/matching

#### Special Labels
- `good first issue` 👋 - Good for newcomers
- `help wanted` 🆘 - Extra attention needed
- `duplicate` 🔄 - Duplicate of another issue
- `question` ❓ - Further information requested

### Issue Templates

#### Bug Report Template

```markdown
## Bug Description
[Clear, concise description of the bug]

## Steps to Reproduce
1. [First step]
2. [Second step]
3. [...]

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happens]

## Environment
- **OS:** Windows/Linux
- **Compiler:** MSVC/GCC/Clang
- **Database:** SQL Server/MySQL/SQLite
- **ArenaServ Version:** [commit hash or branch]

## Evidence
- **Code Location:** `file.cpp:line`
- **Logs:** [paste relevant logs]
- **Screenshots:** [if applicable]

## Possible Fix
[Optional: Suggest a fix if known]

## Related Issues
[Link to related issues, if any]
```

#### Feature Request Template

```markdown
## Feature Description
[Clear, concise description of the feature]

## Use Case
[Why is this feature needed? Who benefits?]

## Proposed Solution
[Describe how this could be implemented]

## Client Constraints
- [ ] Works with vanilla client
- [ ] Requires client modification
- [ ] Unknown/needs research

## Implementation Complexity
- [ ] Trivial (< 1 day)
- [ ] Simple (1-3 days)
- [ ] Moderate (1 week)
- [ ] Complex (2+ weeks)
- [ ] Very Complex (1+ month)

## Dependencies
[Other features or research questions this depends on]

## Alternatives Considered
[Other approaches, and why this is preferred]

## Roadmap Phase
[Which phase does this belong to?]
```

#### Research Question Template

```markdown
## Question
[Clear research question]

## Category
- [ ] Client Capabilities
- [ ] Protocol/Packets
- [ ] Data Files
- [ ] Game Mechanics
- [ ] Performance
- [ ] Other: [specify]

## Current Evidence
[What we know from code analysis]

## Unknown Factors
[What we don't know]

## Theory/Hypothesis
[What we think might work]

## Testing Plan
1. [Test step 1]
2. [Test step 2]
3. [...]

## Expected Outcomes
- **If confirmed:** [Impact]
- **If refuted:** [Impact]

## Priority
- [ ] P0 - Critical (blocks development)
- [ ] P1 - High (major feature enabler)
- [ ] P2 - Medium (important for polish)
- [ ] P3 - Low (nice-to-have)

## Related Documentation
[Link to docs/research/UNKNOWN-BEHAVIOR.md entry]
```

---

## Pull Request Workflow

### PR Template

```markdown
## Description
[Describe what this PR does]

## Type of Change
- [ ] Bug fix (non-breaking)
- [ ] New feature (non-breaking)
- [ ] Breaking change
- [ ] Documentation update
- [ ] Research findings

## Closes Issues
Closes #[issue number]

## Changes Made
- [Change 1]
- [Change 2]
- [...]

## Testing
- [ ] Unit tests added
- [ ] Integration tests added
- [ ] Manual testing completed
- [ ] No regressions

### Test Summary
[Describe testing performed]

## Documentation
- [ ] Code comments added
- [ ] Documentation updated
- [ ] README updated (if needed)

## Screenshots
[If applicable]

## Checklist
- [ ] Code follows project style
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No new warnings
- [ ] Tests passing
- [ ] Conflicts resolved
```

### PR Review Process

1. **Author submits PR**
   - Complete PR template
   - Assign reviewers
   - Link related issues
   - Add appropriate labels

2. **Automated Checks**
   - Build passes (CI)
   - Tests pass
   - Linter passes
   - No conflicts

3. **Peer Review**
   - At least 1 approval required
   - Address all comments
   - Resolve conversations
   - Re-request review if needed

4. **Merge**
   - Squash and merge (keep history clean)
   - Delete source branch
   - Close related issues

---

## Commit Message Convention

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation only
- `style` - Code style (formatting, no logic change)
- `refactor` - Code refactoring
- `test` - Adding or updating tests
- `chore` - Build, dependencies, etc.

### Scopes
- `arena` - Arena system
- `combat` - Combat mechanics
- `queue` - Matchmaking/queue
- `stats` - Statistics/leaderboards
- `db` - Database
- `config` - Configuration
- `ui` - User interface

### Examples

```
feat(arena): add ELO-based matchmaking queue

Implements queue system with ELO range filtering (±200).
Players wait up to 5 minutes before timeout.

Closes #42
```

```
fix(arena): fix reentrancy issue in simultaneous attacks

Updates victim state immediately before checking match end
condition to prevent double-elimination bug.

Fixes #8 (Critical Bug #1 from LEGACY-ARENA-BUGS.md)
```

```
docs(research): document virtual window test results

Q1.1 (virtual boards): CONFIRMED - works without tile
Q1.2 (virtual quests): FAILED - requires NPC in range

Updates docs/research/UNKNOWN-BEHAVIOR.md
```

---

## Issue Lifecycle

### 1. Issue Created
- User/maintainer creates issue
- Applies appropriate labels
- Assigns to milestone (optional)

### 2. Triage
- Validate issue (reproducible, clear)
- Assign priority
- Assign to appropriate phase
- Add to project board

### 3. Planning
- Break down into subtasks (if complex)
- Estimate effort
- Identify dependencies
- Assign to developer (or leave unassigned)

### 4. Development
- Create feature/bugfix branch
- Mark issue as `status: in-progress`
- Regular commits
- Push to remote regularly

### 5. Pull Request
- Create PR
- Link to issue (`Closes #N`)
- Mark issue as `status: needs-review`
- Address review comments

### 6. Review & Merge
- Peer review
- Automated checks pass
- Merge to `develop`
- Issue automatically closed

### 7. Testing & Release
- Test in `develop` branch
- Merge `develop` → `main` for release
- Tag release version
- Update changelog

---

## Project Boards

### Board: Arena Development

**Columns:**
1. **Backlog** - Planned but not started
2. **To Do (Current Phase)** - Ready to work on
3. **In Progress** - Actively being developed
4. **Review** - Pull request open
5. **Testing** - Needs validation
6. **Done** - Completed and merged

**Automation:**
- Issues opened → Backlog
- Issues assigned → To Do
- PR opened → Review
- PR merged → Done
- Issues closed → Done

### Board: Research Questions

**Columns:**
1. **Unknown** - Not yet investigated
2. **In Progress** - Currently researching
3. **Confirmed** - Research complete, answer found
4. **Blocked** - Requires external resource

---

## Milestones

### Phase Milestones

| Milestone | Description | Target Date |
|-----------|-------------|-------------|
| Phase 0 Complete | Documentation foundation | ✅ 2026-09-22 |
| Phase 1 Complete | Bugs fixed, research done | TBD |
| Phase 2 Complete | 1v1 arena functional | TBD |
| Phase 3 Complete | Stats & leaderboards | TBD |
| Phase 4 Complete | Ranked seasons | TBD |
| Phase 5 Complete | Team modes & tournaments | TBD |

### Feature Milestones

- **v0.1.0 - Foundation** - Phase 1 complete
- **v0.2.0 - Core Arena** - Phase 2 complete
- **v0.3.0 - Statistics** - Phase 3 complete
- **v1.0.0 - Competitive** - Phase 4 complete
- **v2.0.0 - Advanced** - Phase 5 complete

---

## CI/CD Pipeline

### GitHub Actions Workflows

#### Build & Test (`.github/workflows/ci.yaml`)

```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build-linux:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install dependencies
        run: sudo apt-get install -y cmake g++ libsqlite3-dev
      - name: Build
        run: |
          mkdir build && cd build
          cmake ..
          cmake --build .
      - name: Test
        run: cd build && ctest

  build-windows:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup MSVC
        uses: microsoft/setup-msbuild@v1
      - name: Build
        run: |
          mkdir build
          cd build
          cmake -G "Visual Studio 17 2022" -A Win32 ..
          cmake --build . --config Release
      - name: Test
        run: cd build && ctest -C Release
```

#### Lint (`.github/workflows/lint.yaml`)

```yaml
name: Lint

on:
  pull_request:
    branches: [ main, develop ]

jobs:
  clang-format:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run clang-format
        run: |
          find src -name '*.cpp' -o -name '*.hpp' | xargs clang-format --dry-run --Werror
```

---

## Release Process

### Versioning

**Semantic Versioning:** `MAJOR.MINOR.PATCH`

- **MAJOR:** Breaking changes, incompatible API
- **MINOR:** New features, backwards-compatible
- **PATCH:** Bug fixes, backwards-compatible

### Release Checklist

- [ ] All tests passing
- [ ] Documentation updated
- [ ] CHANGELOG.md updated
- [ ] Version number bumped
- [ ] Tag created (`git tag v1.0.0`)
- [ ] Release notes written
- [ ] Binary artifacts built
- [ ] GitHub release published

### Release Notes Template

```markdown
## ArenaServ v1.0.0 - Competitive Arena Release

**Release Date:** 2026-XX-XX

### 🎉 New Features
- Ranked matchmaking with ELO system
- Seasonal leaderboards
- Reward shop with arena tokens
- Achievement system

### 🐛 Bug Fixes
- Fixed reentrancy issue in simultaneous attacks (#8)
- Fixed occupant count desync (#12)
- Fixed timer cleanup use-after-free (#15)

### 📚 Documentation
- Added comprehensive arena documentation
- Updated API reference
- Added development guide

### ⚡ Performance
- Optimized database queries (50% faster leaderboard)
- Reduced packet overhead

### 🔧 Technical
- Refactored match lifecycle manager
- Improved error handling
- Added unit tests (80% coverage)

### 📊 Statistics
- 1000+ matches tested
- 100+ active players
- 99.9% uptime

### 🙏 Contributors
Thanks to all contributors! [list]

### 📥 Downloads
- [Windows Binary](link)
- [Linux Binary](link)
- [Source Code](link)

### 🔗 Links
- [Documentation](link)
- [Changelog](link)
- [Issue Tracker](link)
```

---

## Code Review Guidelines

### What to Look For

#### Functionality
- ✅ Does it work as intended?
- ✅ Are edge cases handled?
- ✅ Are error cases handled?

#### Code Quality
- ✅ Is code readable?
- ✅ Are complex sections commented?
- ✅ Consistent with project style?
- ✅ No unnecessary complexity?

#### Testing
- ✅ Are tests included?
- ✅ Do tests cover edge cases?
- ✅ Are tests passing?

#### Documentation
- ✅ Is documentation updated?
- ✅ Are comments accurate?
- ✅ Is API documented?

#### Security
- ✅ No SQL injection risks?
- ✅ Input validation present?
- ✅ No buffer overflows?

### Review Comments

**Types:**
- 💡 **Suggestion** - Optional improvement
- ❓ **Question** - Clarification needed
- 🐛 **Issue** - Problem that must be fixed
- 🎨 **Style** - Code style issue
- ⚡ **Performance** - Performance concern
- 🔒 **Security** - Security issue

**Examples:**
```
💡 Suggestion: Consider extracting this into a separate function

❓ Question: Why is this timeout set to 60 seconds?

🐛 Issue: This could cause a null pointer dereference

🎨 Style: Line exceeds 120 character limit

⚡ Performance: This loop could be optimized with early exit

🔒 Security: User input not validated
```

---

## Collaboration Guidelines

### Communication

#### Where to Discuss
- **GitHub Issues** - Feature requests, bugs, specific problems
- **Pull Requests** - Code review, implementation details
- **Discussions** - General ideas, questions, help

#### Response Times
- **Critical issues:** < 24 hours
- **Pull requests:** < 3 days
- **Questions:** < 1 week
- **Feature requests:** Best effort

### Code Ownership

**Maintainers:**
- Review and merge pull requests
- Manage releases
- Triage issues
- Guide project direction

**Contributors:**
- Submit pull requests
- Report bugs
- Suggest features
- Help with documentation

### Getting Help

1. **Check documentation** - `docs/` folder
2. **Search issues** - May already be answered
3. **Ask in discussions** - Community help
4. **Open issue** - If problem persists

---

## Summary

This workflow provides structure for organized, trackable ArenaServ development:

✅ **Branching** - Clear branch strategy  
✅ **Issues** - Comprehensive labeling and templates  
✅ **Pull Requests** - Structured review process  
✅ **Commits** - Conventional commit messages  
✅ **CI/CD** - Automated testing  
✅ **Releases** - Versioned releases  
✅ **Reviews** - Quality code reviews  
✅ **Collaboration** - Clear communication  

**Next Step:** Set up GitHub repository with these workflows

---

**Last Review:** 2026-09-22  
**Next Review:** After repository setup

