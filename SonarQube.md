# SonarQube

*An open-source platform for continuous inspection of code quality. It runs static analysis to find bugs, security vulnerabilities, and maintainability issues ("code smells") across 25+ languages, and gates your CI/CD pipeline on objective quality criteria.*

## 1. The Problem It Solves

Code quality erodes silently. Every rushed change adds a little **technical debt**; manual code review is inconsistent and misses whole classes of issues (subtle null dereferences, injection flaws, copy-pasted blocks); and different developers apply different standards. By the time quality problems surface — a production bug, a security incident — they are expensive to fix and spread across the codebase.

SonarQube makes quality **objective, automated, and continuous**: every commit or pull request is analyzed against the same rules, and a pass/fail **Quality Gate** stops substandard code before it merges. Quality stops being a matter of opinion in review and becomes a measured, enforced property of the build.

## 2. What SonarQube Is

SonarQube is a **self-hosted server** that performs **static analysis** — examining source code *without running it* — and stores the results as a queryable history. You run it as a service in your own infrastructure (a server process plus a database), point a **scanner** at your code (usually from CI), and view findings and trends in its web dashboard.

It analyzes code along four dimensions:
- **Bugs** — code that is likely to behave incorrectly at runtime.
- **Vulnerabilities** — security flaws that are exploitable.
- **Security hotspots** — security-sensitive code that needs human review (not necessarily a flaw).
- **Code smells** — maintainability issues that make code harder to change.

## 3. How It Works — Architecture and Analysis Flow

SonarQube has four moving parts:

```
[ Scanner ]  ──analysis report──►  [ SonarQube Server ]
 (runs in CI,                          ├─ Compute Engine (processes the report)
  reads source)                        ├─ Web Server (dashboard, API)
                                       └─ Database (results, history, config)
```

- **Scanner** — runs where your code is (a CI agent, a developer machine). It collects source files plus metadata (and imported test-coverage reports) and sends an analysis report to the server. It does **not** decide pass/fail itself.
- **Server / Compute Engine** — receives the report, applies the active rule set, computes metrics, generates issues, and stores everything.
- **Database** — PostgreSQL (recommended), SQL Server, or Oracle; holds results, configuration, and history.
- **Web interface** — dashboards, issue triage, configuration, and the REST API.

**Analysis flow:** collect source → parse → execute rules → compute metrics (complexity, coverage, duplication) → generate issues with severities → store → evaluate the Quality Gate → report status back to CI.

**Analysis scope** can be a full project scan, an **incremental** analysis of changed code, a **branch** analysis, or a **pull request** analysis (branch and PR analysis require Developer Edition or higher — see §7).

## 4. Metrics and Ratings

SonarQube quantifies quality so it can be gated:

| Metric | Meaning |
|---|---|
| Lines of code | Total and new code volume |
| Cyclomatic complexity | How branchy / hard to test the code is |
| Coverage | Test coverage % (imported from your test runner, not computed by SonarQube) |
| Duplication | % of duplicated lines |
| Technical debt | Estimated time to fix all maintainability issues |
| Reliability / Security / Maintainability | Letter ratings (A–E) derived from issue severity |

The **A–E maintainability rating** maps to the technical-debt ratio: **A** = 0–5%, **B** = 6–10%, **C** = 11–20%, **D** = 21–50%, **E** = >50%.

## 5. Quality Gates and "Clean as You Code"

A **Quality Gate** is a set of pass/fail conditions a project must meet to be considered releasable. When the gate fails, the CI step fails and the merge is blocked. The default ("Sonar way") gate evaluates **new code** and requires:

- Coverage on new code ≥ 80%
- Duplicated lines on new code ≤ 3%
- Maintainability / Reliability / Security rating on new code = A
- Security hotspots reviewed = 100%

The philosophy behind this is **Clean as You Code** (also called *fix the leak*): instead of trying to fix an entire legacy codebase at once, you hold a strict standard on **new and changed code only**. Quality then improves steadily as the code is touched, without an impossible up-front cleanup. This is the single most important concept for adopting SonarQube successfully.

Custom gates let you tailor conditions per project type or roll out stricter standards gradually.

## 6. Continuous Inspection — Where It Plugs In

Analysis happens at several points in the workflow, the earlier the better:

- **IDE — SonarLint** gives real-time feedback as you type (Eclipse, IntelliJ, VS Code, Visual Studio), and can connect to the server to share the same rules.
- **Pull request analysis** — the scanner runs on the PR; **PR decoration** posts the new-code issues and gate status directly onto the PR in GitHub / Azure DevOps / GitLab, and **branch protection** blocks merging a failing PR.
- **CI/CD pipeline** — analysis runs on every build via the scanner. Integrations exist for **Azure DevOps**, GitHub Actions, Jenkins, GitLab CI, Bamboo, and TeamCity.
- **Scheduled full scans** — periodic complete analysis of the main branch.

## 7. Going Deeper

### Editions (what gates which feature)

| Edition | Adds |
|---|---|
| **Community** (free) | Core analysis, main branch only — **no branch or PR analysis** |
| **Developer** | **Branch + pull-request analysis**, more languages, taint/security analysis |
| **Enterprise** | Portfolio management, governance/compliance reporting, parallel processing |
| **Data Center** | High availability, horizontal scaling |

Branch and PR analysis being a **Developer Edition** feature (not Community) is the most common surprise — the free edition analyzes only the main branch.

### Security analysis
Detects **OWASP Top 10** and **CWE/SANS Top 25** categories, classifies **security hotspots** by area (authentication, authorization, input validation, cryptography, session management), and (in paid editions) performs **taint analysis** to track tainted input to a sink. SonarQube focuses on first-party code; dependency vulnerability scanning is better covered by dedicated SCA tools.

### Centralized governance
Global quality profiles (rule sets) and quality gates apply organization-wide; portfolio views aggregate quality across many projects; RBAC and LDAP/AD integration manage access; compliance reports roll up for stakeholders.

## 8. Setup

**Prerequisites:** Java 17+, a supported database (PostgreSQL recommended — do **not** use the bundled H2 in production), and adequate resources (the Compute Engine and Elasticsearch index are memory-hungry; start with several GB of RAM).

1. Download and extract SonarQube; configure the database connection in `sonar.properties`.
2. Set JVM/heap parameters for the web and compute processes.
3. Start the server; open `http://localhost:9000` and complete the setup wizard.
4. Configure authentication (LDAP/AD/SSO) and install any language/plugin analyzers.
5. Generate a token and wire the **scanner** into your CI pipeline.

## ⚠️ Gotchas

- **Community Edition has no branch/PR analysis.** It scans only the main branch; branch and PR analysis need Developer Edition or higher.
- **SonarQube does not run your tests.** Coverage is *imported* from a report your test runner produces (e.g. `lcov`, `cobertura`, `jacoco`). No report in, no coverage shown — gates then fail on "0% coverage."
- **Don't gate on overall code.** Pointing the gate at total coverage/issues on a legacy codebase makes it permanently red. Use the default new-code gate (Clean as You Code).
- **Memory and the embedded search index.** Under-provisioned heap or disk causes the Compute Engine or Elasticsearch to fail; large projects need tuning.
- **Never run on the bundled H2 database in production.** It is for evaluation only and loses data; use PostgreSQL.
- **Some languages need the build.** Compiled languages (C#, Java, C/C++) require analysis to run as part of (or alongside) the build so the scanner sees compiled output.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Quality Gate shows 0% coverage | No coverage report imported | Generate a coverage report in CI and point `sonar.coverage` paths at it |
| Server won't start | Heap/disk too small, or Elasticsearch bootstrap checks failing | Increase RAM; raise `vm.max_map_count`; check `sonar.log` |
| Branch/PR analysis "not available" | Community Edition | Upgrade to Developer+ |
| Scanner can't reach server | Network/token/URL misconfig | Verify `sonar.host.url` and token; check firewall |
| Analysis very slow on large repos | Single-threaded compute, big project | Tune heap; Enterprise adds parallel processing; scope analysis |
| C#/Java issues missing | Scanner didn't see compiled output | Run the begin/end scanner steps around the build |

## Quick Reference

SonarQube is a self-hosted **static analysis** platform: a **scanner** (run from CI) sends an analysis report to the **server** (compute engine + web UI + database), which finds **bugs, vulnerabilities, security hotspots, and code smells** and computes **metrics and A–E ratings**. A **Quality Gate** turns those into pass/fail and blocks the build when new code falls short — applied under the **Clean as You Code** principle (hold the standard on *new* code, not the whole legacy base). It plugs in at the **IDE (SonarLint)**, on **pull requests (PR decoration)**, and in **CI/CD (Azure DevOps, GitHub Actions, Jenkins, etc.)**. **Branch and PR analysis require Developer Edition**; coverage is **imported** from your test runner, not computed; and production needs **PostgreSQL** and **Java 17+**.
