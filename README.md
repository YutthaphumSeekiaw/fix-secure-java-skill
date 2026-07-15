# 🛡️ Security Fixer — GitHub Copilot Custom Agent & Skills

Automated security vulnerability triage, remediation, and verification for **Java 17+ / Spring Boot 3.x** backend and **jQuery** frontend projects using GitHub Copilot Agent Mode in VS Code.

---

## 📁 Project Structure

```
.
├── .github/
│   └── agents/
│       └── security-fixer.md          # Custom Agent — main orchestrator
├── .copilot/
│   └── skills/
│       ├── plan-dependabot-fix.md     # Skill — vulnerability planning & risk assessment
│       └── code-review-verify.md      # Skill — post-fix code review & test recommendation
└── README.md
```

---

## 📄 File Summaries

### 🤖 Agent: `security-fixer.md`

> **Path:** `.github/agents/security-fixer.md`

The **main orchestrator agent** that acts as a Senior DevSecOps Engineer.

| Aspect        | Detail                                                                                   |
| ------------- | ---------------------------------------------------------------------------------------- |
| **Role**      | Triage, plan, fix, and verify security vulnerabilities (backend + frontend)              |
| **Data Source**| GitHub Dependabot, CodeQL Code Scanning, Secret Scanning (via `gh` CLI)                 |
| **Actions**   | Pulls open alerts → delegates planning → applies fixes to `pom.xml` / source code / JS / JSP → delegates review → produces final report |
| **Output**    | Markdown **Security Fix Report** with tables of fixed, deferred, and test results        |

---

### 📋 Skill: `plan-dependabot-fix.md`

> **Path:** `.copilot/skills/plan-dependabot-fix.md`

A **triage and planning** skill that transforms raw JSON alert data into a structured upgrade plan.

| Aspect        | Detail                                                                                   |
| ------------- | ---------------------------------------------------------------------------------------- |
| **Input**     | Raw JSON arrays from Dependabot, Code Scanning, and Secret Scanning APIs                 |
| **Process**   | Normalize alerts → classify ecosystem (Maven / npm / CDN) → assign risk level (🔴🟠🟡🟢) → sort by severity |
| **Output**    | Markdown **Upgrade Plan Table** (backend + frontend separated) + prioritised **Action List** + breaking-change warnings |

**Risk Assessment Matrix:**

| Severity | Has Safe Version? | Risk Level        |
| -------- | ----------------- | ----------------- |
| CRITICAL | Yes               | 🔴 CRITICAL       |
| CRITICAL | No                | 🟠 HIGH-DEFERRED  |
| HIGH     | Yes               | 🟠 HIGH           |
| HIGH     | No                | 🟡 MEDIUM-DEFERRED|
| MEDIUM   | Yes               | 🟡 MEDIUM         |
| LOW      | *                 | 🟢 LOW            |

---

### 🔍 Skill: `code-review-verify.md`

> **Path:** `.copilot/skills/code-review-verify.md`

A **post-fix review** skill that validates all changes for correctness, security, and quality.

| Aspect        | Detail                                                                                   |
| ------------- | ---------------------------------------------------------------------------------------- |
| **Input**     | List of changed file paths (`.java`, `.js`, `.html`, `.jsp`, `pom.xml`) + fix summary    |
| **Checks**    | Logic errors, security regressions, API compatibility, config validity, **jQuery DOM-XSS**, deprecated jQuery APIs, CDN SRI integrity, code style |
| **Output**    | Markdown **Code Review Report** with per-file findings + `mvn` test commands + frontend verification steps |

**Verdict System:**

| Verdict             | Meaning                                                    |
| ------------------- | ---------------------------------------------------------- |
| ✅ **APPROVED**      | All changes are correct, secure, and test-ready            |
| ⚠️ **NEEDS REVISION**| Minor issues found — listed with suggested fixes           |
| ❌ **REJECTED**      | Critical issues — new vulnerability introduced or build broken |

---

## 🔄 Workflow

```
┌─────────────────────────────────────────────────────────┐
│                  security-fixer (Agent)                  │
└─────────────────────────┬───────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
   ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
   │ Dependabot  │ │   Code      │ │   Secret    │
   │   Alerts    │ │  Scanning   │ │  Scanning   │
   └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
          │               │               │
          └───────────────┼───────────────┘
                          │  JSON
                          ▼
         ┌────────────────────────────────┐
         │   plan-dependabot-fix (Skill)  │
         │                                │
         │  • Normalize alerts            │
         │  • Risk assessment             │
         │  • Upgrade plan table          │
         │  • Prioritised action list     │
         └───────────────┬────────────────┘
                         │  Plan
                         ▼
         ┌────────────────────────────────┐
         │    security-fixer (Agent)      │
         │                                │
         │  • Update pom.xml / build.gradle│
         │  • Refactor vulnerable Java code│
         │  • Upgrade jQuery (WebJars/CDN)│
         │  • Fix DOM-XSS in .js / .jsp  │
         │  • Rotate / remove secrets     │
         └───────────────┬────────────────┘
                         │  Changed files
                         ▼
         ┌────────────────────────────────┐
         │  code-review-verify (Skill)    │
         │                                │
         │  • Static analysis review      │
         │  • jQuery DOM-XSS review      │
         │  • Dependency tree check       │
         │  • Test recommendations        │
         │  • Frontend verification       │
         │  • Verdict (✅ / ⚠️ / ❌)       │
         └───────────────┬────────────────┘
                         │  Report
                         ▼
         ┌────────────────────────────────┐
         │   Security Fix Report (Final)  │
         │                                │
         │  • Fixed vulnerabilities table │
         │  • Deferred items              │
         │  • Test results                │
         └────────────────────────────────┘
```

---

## 🚀 Usage

### Prerequisites

| Requirement                  | Details                                                        |
| ---------------------------- | -------------------------------------------------------------- |
| **VS Code**                  | Latest version with GitHub Copilot extension installed          |
| **GitHub Copilot Chat**      | Active subscription (Individual, Business, or Enterprise)       |
| **GitHub CLI (`gh`)**        | Installed and authenticated — [install guide](https://cli.github.com/) |
| **Java 17+**                 | Installed and available on `PATH`                               |
| **Maven or Gradle**          | Project must have `pom.xml` or `build.gradle`                   |
| **jQuery**                   | Loaded via WebJars (`pom.xml`), npm (`package.json`), or CDN `<script>` tag |
| **Repository permissions**   | Push access + Security alerts enabled on the GitHub repository  |

### Step 1 — Enable GitHub Security Features

Make sure the following are enabled in your GitHub repository under **Settings → Code security**:

- [x] Dependabot alerts
- [x] Dependabot security updates
- [x] Code scanning (CodeQL)
- [x] Secret scanning

### Step 2 — Authenticate GitHub CLI

```bash
gh auth login
gh auth status   # verify you are logged in
```

### Step 3 — Open Agent Mode in VS Code

1. Open your project in VS Code.
2. Open the **Copilot Chat** panel (`Ctrl + Shift + I` or `Cmd + Shift + I`).
3. Switch to **Agent Mode** by clicking the mode selector at the top of the chat panel.

   > If you don't see Agent Mode, make sure you are on the latest version of the
   > GitHub Copilot Chat extension and that it is enabled in your settings.

### Step 4 — Select and Invoke the Agent

1. In the Agent Mode chat input, type **`@security-fixer`** to select the custom agent.
2. Give it a prompt, for example:

   ```
   @security-fixer Scan this repository for open security vulnerabilities,
   create an upgrade plan, apply the fixes, and verify the changes.
   ```

3. The agent will:
   - Run `gh api` commands to fetch open alerts.
   - Call the **`plan-dependabot-fix`** skill to build the upgrade plan.
   - Apply fixes to `pom.xml` / source files / JS / JSP templates.
   - Call the **`code-review-verify`** skill to review and suggest tests.
   - Output a final **Security Fix Report**.

### Step 5 — Review and Test

After the agent finishes:

1. **Review the diff** — inspect every changed file in the Source Control panel.
2. **Run the recommended tests:**

   ```bash
   # Full test suite
   mvn clean test

   # OWASP dependency check (optional)
   mvn org.owasp:dependency-check-maven:check
   ```

3. **Commit and push** once you are satisfied with the changes.

---

## 💡 Example Prompts

| Prompt | What it does |
| ------ | ------------ |
| `@security-fixer Fix all critical Dependabot alerts.` | Focuses only on CRITICAL severity alerts (backend + frontend). |
| `@security-fixer Scan and create an upgrade plan only — do not apply fixes.` | Runs Steps 1–2 only (triage + planning). |
| `@security-fixer Review the files I just changed for security issues.` | Invokes the code-review-verify skill directly. |
| `@security-fixer Rotate all exposed secrets and replace with env vars.` | Targets Secret Scanning alerts specifically. |
| `@security-fixer Upgrade jQuery to the latest safe version across all files.` | Finds and updates all jQuery references (WebJars, npm, CDN). |
| `@security-fixer Find and fix all DOM-based XSS patterns in our JS files.` | Scans `.js` / `.html` / `.jsp` for unsafe jQuery calls. |

---

## ⚙️ Customisation

### Change the AI Model

Edit the `model` field in the frontmatter of any `.md` file:

```yaml
---
model: gpt-4o          # default
model: claude-sonnet-4  # alternative
---
```

### Add More Skills

Create a new `.md` file in `.copilot/skills/` with the standard frontmatter schema:

```yaml
---
name: your-skill-name
description: What this skill does.
model: gpt-4o
---
```

Then reference it from `security-fixer.md` by name.

### Adapt to Gradle

The agent auto-detects `build.gradle` / `build.gradle.kts` if `pom.xml` is absent.
No configuration change is needed.

---

## 🖥️ Tech Stack

| Layer      | Technology                                                                 |
| ---------- | -------------------------------------------------------------------------- |
| **Backend**| Java 17+, Spring Boot 3.x, Maven / Gradle                                 |
| **Frontend**| jQuery (via WebJars, npm, or CDN), JSP / Thymeleaf / static HTML          |
| **Testing**| JUnit 5 + Mockito (backend), Browser console + XSS smoke test (frontend)  |
| **Security**| GitHub Dependabot, CodeQL Code Scanning, Secret Scanning                  |

---

## 🔐 jQuery Security Patterns Detected

The agent and skills are configured to detect and fix these common jQuery security issues:

| Pattern | Risk | Fix |
| ------- | ---- | --- |
| `.html(userInput)` | DOM-based XSS | Replace with `.text()` or sanitise with DOMPurify |
| `.append(userInput)` | DOM-based XSS | Validate/escape input before appending |
| `$("#" + userInput)` | Selector injection | Use `document.getElementById()` or validate input |
| `$.parseJSON()` | Deprecated (removed in jQuery 4) | Use `JSON.parse()` |
| `$.trim()` | Deprecated | Use `String.prototype.trim()` |
| `.click()` shorthand | Deprecated | Use `.on('click', handler)` |
| CDN `<script>` without SRI | Integrity risk | Add `integrity` and `crossorigin` attributes |
| Outdated jQuery version | Known CVEs | Upgrade to latest patch version |

---

## 📜 License

This configuration is provided as-is for internal project use. Adjust to fit your organisation's security policies.
