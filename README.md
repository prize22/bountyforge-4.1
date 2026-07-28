  
# bountyforge

> All-round bug bounty skill for Claude Code — parallelized agents for smart contract audits (EVM, Move, Solana, TRON), web/API security, local tooling orchestration, and submission-ready reports for HackerOne, Bugcrowd, Intigriti & Immunefi.

---

## What It Does

Bounty Forge spins up **8 specialized security agents + 7 S-Class modules + 15 Intelligence agents in parallel (30 total)**, each attacking a different surface of your target. Findings are deduplicated, gate-evaluated, CVSS-scored, and formatted into a submission-ready report — in minutes.

| Agent | Covers |
|---|---|
| Web / API | Auth bypass, IDOR, XSS, SSRF, SQLi, CSV injection, open redirect, path traversal, parameter pollution, GraphQL, CORS |
| Smart Contract | EVM, Move/Aptos, Solana, TRON — structural & chain-specific bugs |
| Access Control | Role bypass, init hijack, confused deputy, proxy admin |
| Business Logic | State machine abuse, workflow skip, limit bypass, payment logic |
| Crypto / Math | Overflow, precision loss, signature replay, EIP-712, nonce issues |
| Race Conditions | Front-running, sandwich, TOCTOU, rotation window races |
| Economic Security | Flash loans, oracle manipulation, inflation attacks, DeFi tokenomics |
| Recon | Subdomain takeover, secret leaks, cloud misconfig, chain explorer recon |
| Shadow Logic | Business logic flaws, state machine violations, semantic anomalies |
| Patch-Gap | Incomplete patches, variant vulnerabilities, "else branch" bugs |
| Second-Order | Stored→reflected vulns, temporal bugs, queue/cache poisoning |
| Weirdness | Statistical anomalies, hidden endpoints, timing side-channels |
| Semantic Taint | Static source→sink tracking, mass assignment, auth bypass |
| Ghost State | Race conditions, TOCTOU, single-packet attacks, concurrency |
| Zero-Context | Novel WAF bypass generation, semantic payload mutation |
| Attack Surface Intelligence | Hidden APIs, GraphQL, WebSockets, mobile APIs, admin routes, debug endpoints, staging envs |
| JavaScript Intelligence | JS bundle analysis, source maps, undocumented endpoints, API schemas, feature flags |
| Permission Graph | User/role/permission/resource mapping, BOLA/IDOR via authorization graph |
| Invariant Violation | Business rule inference, ownership/balance/quota violations, illegal state transitions |
| Cross-Service Data Flow | Data flow tracking across APIs, DBs, queues, workers, caches, microservices |
| Dangerous Pattern Mining | Insecure coding patterns, repeated anti-patterns, vulnerable design structures |
| Exploit Chain | Multi-step attack correlation, A→B→C chain building from scattered findings |
| Assumption Breaker | Ownership, sequencing, trust boundary, validation consistency testing |
| Behavior Diff | Guest vs auth, role diffs, mobile vs web, feature flag diffs, API version diffs |
| Patch Regression | Incomplete fixes, inconsistent remediation, recurring patterns via code reuse |
| Hypothesis Generator | Attack hypothesis generation, evidence-based prioritization, agent dispatch |
| Evidence Correlation | Multi-source correlation, confidence boosting, duplicate reduction |
| Payload Evolution | Adaptive payload generation, response-based learning, context-aware inputs |
| Emergent Behavior | Feature interaction analysis, unexpected state combinations, cross-component bugs |
| Hidden Capability | Undocumented endpoint prediction, naming convention inference, schema deduction |

**Supported targets:** web/API, smart contracts, infrastructure, supply chain, internal tooling, binary analysis, AI/LLM features, CI/CD pipelines

**Local tooling:** When Claude Code execution is enabled, BountyForge can orchestrate local CLI tools like `nmap`, `ffuf`, `amass`, `sqlmap`, `gobuster`, `curl`, `httpx`, `wfuzz`, `zap`, `burpsuite`, and other installed scanners/fuzzers.

**Payload coverage:** Designed to explore unlimited payload variants for SQL injection, CSV injection, open redirect, XSS, SSRF, command injection, template injection, path traversal, deserialization, prototype pollution, auth bypass, business logic abuse, IDOR, CSRF, response splitting, cache poisoning, HTTP smuggling, GraphQL abuse, LLM prompt injection, and more.

**Reference setup files:** `references/setup.md` and `references/local-tooling.md` contain the actual Deepseek CLI, local tooling, and vulnerability environment instructions the skill uses.

**Report formats:** HackerOne · Bugcrowd · Intigriti · Immunefi · Generic

---

## Installation

### Claude Code (terminal)

```bash
git clone https://github.com/Gabson0x/bountyforge.git ~/.claude/skills/bountyforge
```

Start a fresh Claude Code session — skills load at startup.

### Claude.ai (web/app)

1. Go to **Customize → Skills**
2. Make sure **Code execution** is enabled in **Settings → Capabilities**
3. Upload the `.skill` file from [Releases](https://github.com/Gabson0x/bountyforge/releases)

---

## Enabling Claude Code Execution (local execution)

To allow BountyForge to run local tools and subagents, enable Claude Code (local execution) in your Claude environment and grant the skill permission to execute shell commands.

macOS / Linux (Claude Code client):

1. Start Claude Code with code-execution enabled (follow your Claude Code client docs).
2. Ensure the shell that launches Claude Code has the tools you want on `PATH` (e.g., `nmap`, `ffuf`, `sqlmap`).
3. If using project-specific env vars, source your project file before starting Claude Code:

```bash
source .env            # or project.env
claude start           # or the command your Claude Code client uses
```

Windows (Claude.app / PowerShell):

1. Open PowerShell as the user that runs Claude.
2. Set any project env vars or tokens (example shown in `references/setup.md`).
3. Launch Claude with code execution enabled from the same session so it inherits the environment.

Notes:
- If you are using the web/app variant, go to **Settings → Capabilities** and toggle **Code execution** on. Some deployments require you to enable a "subagent" or "local tooling" checkbox; consult your Claude distribution docs.
- Always start Claude from the shell that has your project environment loaded so `deepseek export`, `nmap`, and other tools are available to the skill.
- For privacy and safety, grant execution permission only for trusted skills and projects.


## Deepseek Pro Setup (Claude CLI)

BountyForge includes `references/setup.md` so the skill can use the same Deepseek CLI environment and local tooling configuration during audit runs.

### Mac / Linux

In your project shell or project file, export:

```bash
export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
export ANTHROPIC_MODEL=deepseek-v4-pro
export ANTHROPIC_DEFAULT_OPUS_MODEL=deepseek-v4-pro
export ANTHROPIC_DEFAULT_SONNET_MODEL=deepseek-v4-pro
export ANTHROPIC_DEFAULT_HAIKU_MODEL=deepseek-v4-flash
export CLAUDE_CODE_SUBAGENT_MODEL=deepseek-v4-flash
export CLAUDE_CODE_EFFORT_LEVEL=max
export ANTHROPIC_AUTH_TOKEN="your-deepseek-pro-token"
```

Then bind the current repo:

```bash
deepseek export --project . --key "$ANTHROPIC_AUTH_TOKEN" --mode pro
```

Start Claude CLI from the same shell so the environment variables are active.

### Windows (PowerShell)

Use these variables in the current session:

```powershell
$env:ANTHROPIC_BASE_URL = "https://api.deepseek.com/anthropic"
$env:ANTHROPIC_MODEL = "deepseek-v4-pro"
$env:ANTHROPIC_DEFAULT_OPUS_MODEL = "deepseek-v4-pro"
$env:ANTHROPIC_DEFAULT_SONNET_MODEL = "deepseek-v4-pro"
$env:ANTHROPIC_DEFAULT_HAIKU_MODEL = "deepseek-v4-flash"
$env:CLAUDE_CODE_SUBAGENT_MODEL = "deepseek-v4-flash"
$env:CLAUDE_CODE_EFFORT_LEVEL = "max"
$env:ANTHROPIC_AUTH_TOKEN = "your-deepseek-pro-token"

deepseek export --project . --key $env:ANTHROPIC_AUTH_TOKEN --mode pro
```

For persistence, add the same variables to your PowerShell profile.

> Note: this setup is intended for temporary use inside a project or shell session so the Deepseek CLI export command can be applied without modifying the core skill files.

---

## Usage

### How to use Bounty Forge

1. **Choose a precise scope.** For bug bounty work, point the skill at the exact contract file, API endpoint list, or web target you are testing.
2. **Enable code execution.** When Claude Code can run locally, Bounty Forge can orchestrate local tools like `nmap`, `ffuf`, `amass`, `sqlmap`, `gobuster`, `curl`, `httpx`, and `zap`.
3. **Run an audit command.** Use the examples below and include `--file-output` or `--cvss` when you want a formatted deliverable.
4. **Review findings.** The skill will classify and gate each finding, but always validate exploitability before submitting.
5. **Generate a report.** Use the built-in report mode to turn confirmed findings into platform-ready output.

### Audit a contract or repo

```
audit contracts/
```
```
run bountyforge on src/usdc.move --platform immunefi --cvss --file-output
```
>/bountyforge <targetfile>
```
check this contract for vulns --platform h1 --cvss
```

### Web / API target

```
/bountyforge on https://api.target.com
```
```
find vulns in this API — [paste endpoints / Swagger / JS bundle]
```

### Generate a report from findings

```
write a HackerOne report for this finding: [paste notes]
```
```
generate immunefi report --cvss: [describe the vuln]
```

---

## Flags

| Flag | Description |
|---|---|
| `--platform h1` | Format output for HackerOne |
| `--platform immunefi` | Immunefi template |
| `--platform bugcrowd` | Bugcrowd format |
| `--platform intigriti` | Intigriti format |
| `--cvss` | Include full CVSS 3.1 vector string + justification |
| `--file-output` | Save report to `bountyforge-report-[timestamp].md` |
| `--full` | Run all 8 agents regardless of detected file type |
| `--shadow-logic` | Enable behavioral baseline + anomaly detection |
| `--patch-gap` | Ingest recent security commits, hunt unpatched variants |
| `--second-order` | Map storage→consumption flows for temporal bugs |
| `--weirdness` | Build endpoint baseline, find statistical outliers |
| `--semantic-taint` | Run static source→sink analysis on available code |
| `--ghost-state` | Single-packet race + timing side-channel analysis |
| `--zero-context` | Generate novel WAF bypass payloads on-the-fly |
| `--economic-fuzz` | Simulate flash loans, sandwich attacks, oracle manipulation |
| `--autonomous` | Enable Turn 5 feedback loop for self-evolving hunts |
| `--all-modules` | Enable ALL S-Class modules regardless of target |
| `--intelligence` | Enable all 15 Intelligence Agents |
| `--attack-surface` | Deep attack surface mapping |
| `--js-intel` | JavaScript bundle intelligence extraction |
| `--permission-graph` | Build authorization graph |
| `--invariant` | Business rule inference and violation detection |
| `--cross-service` | Microservice data flow tracking |
| `--dangerous-pattern` | Code anti-pattern mining |
| `--exploit-chain` | Automatic chain correlation |
| `--assumption-breaker` | Developer assumption testing |
| `--behavior-diff` | Cross-role/cross-platform behavior comparison |
| `--patch-regression` | Incomplete fix detection |
| `--hypothesis` | AI hypothesis generation |
| `--evidence-correlation` | Multi-source finding correlation |
| `--payload-evolution` | Adaptive payload learning |
| `--emergent-behavior` | Feature interaction analysis |
| `--hidden-capability` | Undocumented functionality prediction |

---

## How It Works

```
Discover files / scope
        ↓
Build agent bundles (source + agent instructions)
        ↓
Spawn 8 agents + S-Class modules + Intelligence agents in parallel
        ↓
Deduplicate findings by (Target | location | bug-class)
        ↓
Gate evaluation: Refutation → Reachability → Trigger → Impact
        ↓
CVSS 3.1 scoring
        ↓
Submission-ready report
        ↓
[--autonomous] Feedback Oracle → New hypotheses → Re-rank → Re-execute
```

With `--autonomous`, BountyForge enters a self-evolving loop after the initial report: the Feedback Oracle ingests every finding, generates new attack hypotheses from anomalies and taint flows, ranks them by Expected Value `(probability × impact) / effort`, and executes the top hypothesis automatically until no high-EV targets remain.

Every finding passes four gates before it's confirmed:

1. **Refutation** — can the attack be concretely blocked by an existing guard?
2. **Reachability** — can the vulnerable state exist in a live deployment?
3. **Trigger** — can an unprivileged actor execute it profitably?
4. **Impact** — is there material harm to an identifiable victim?

Fail any gate → rejected or demoted to a lead for manual review.


---

## S-Class Modules (v5.1)

BountyForge 5.1 adds **9 autonomous Python modules** that detect bug classes checklist-based approaches miss. These modules run as embedded agents or standalone tools.

| Module | What It Finds |
|---|---|
| `shadow_logic.py` | Price manipulation, workflow bypass, state machine violations, mass assignment |
| `patch_gap.py` | Incomplete fixes, variant vulnerabilities, "else branch" bugs left after patches |
| `second_order_detector.py` | Stored XSS via admin dashboards, SQLi via analytics, SSRF via thumbnail services |
| `weirdness_scorer.py` | Hidden debug endpoints, timing side-channels, unauthorized mutation acceptance |
| `semantic_taint.py` | Mass assignment, auth bypass, SQLi, RCE, SSTI via static source→sink tracking |
| `economic_fuzzer.py` | Flash loan attacks, sandwich exploits, oracle manipulation, MEV extraction |
| `zero_context_mutator.py` | Novel WAF bypasses, context-specific XSS, JSON/prototype pollution |
| `ghost_state_hunter.py` | Double-spend, coupon reuse, inventory overselling, TOCTOU race conditions |
| `feedback_oracle.py` | Closed-loop learning — every success/failure updates future hypotheses |

### How S-Class modules integrate

- **Turn 3** — Spawn standard agents + S-Class agents based on target triggers
- **Turn 5** — Autonomous feedback loop (when `--autonomous` is set): ingest findings, generate new hypotheses, rank by Expected Value, execute top hypothesis, repeat
- **Feedback Oracle** — Learns from every PoC: failures suggest alternatives, successes spawn sibling tests, duplicates improve dedup rules

### New A→B Chains (v5.1)

- **Chain 13** — Patch-Gap Variant → Same-class exploit (incomplete fixes)
- **Chain 14** — Second-Order Stored → Admin Dashboard XSS → ATO
- **Chain 15** — Semantic Taint → Mass Assignment → IDOR → Data Exfil
- **Chain 16** — Race Condition → Double-Spend → Financial Loss
- **Chain 17** — Behavioral Anomaly → Hidden Endpoint → Debug Feature → RCE
- **Chain 18** — Economic Fuzzing → Flash Loan Manipulation → Protocol Insolvency


---

## Intelligence Agents (v5.1)

BountyForge 5.1 adds **15 Intelligence Agents** that do not replace the 15 Vulnerability Agents — they **feed** them. Intelligence agents discover, map, correlate, and hypothesize. Vulnerability agents exploit, validate, and prove.

### The Intelligence Pipeline

```
Attack Surface Intelligence
        ↓
JavaScript Intelligence
        ↓
Hidden Capability
        ↓
Hypothesis Generator
        ↓
Permission Graph
        ↓
Behavior Diff
        ↓
Invariant Violation
        ↓
Cross-Service Data Flow
        ↓
Dangerous Pattern Mining
        ↓
Assumption Breaker
        ↓
Payload Evolution
        ↓
Evidence Correlation
        ↓
Exploit Chain
        ↓
Patch Regression
```

### What Each Intelligence Agent Does

| Agent | Purpose |
|---|---|
| **Attack Surface Intelligence** | Discovers hidden APIs, GraphQL endpoints, WebSockets, mobile APIs, admin routes, debug endpoints, feature-flagged functionality, and staging environments. Correlates assets across DNS, JS, API responses, and infrastructure. |
| **JavaScript Intelligence** | Analyzes JS bundles and source maps to recover undocumented endpoints, API schemas, GraphQL operations, hidden routes, feature flags, object models, and client-side authorization logic. |
| **Permission Graph** | Models application authorization by building relationships between users, roles, permissions, ownership, and resources. Identifies inconsistent auth checks and BOLA/IDOR opportunities through permission mapping. |
| **Invariant Violation** | Infers expected business rules (ownership, balances, workflow rules, quotas, state transitions) and detects operations that violate them. |
| **Cross-Service Data Flow** | Tracks attacker-controlled data across APIs, databases, queues, workers, caches, and microservices. Identifies second-order processing paths. |
| **Dangerous Pattern Mining** | Detects insecure coding patterns and repeated anti-patterns across similar code paths, even when no public CVE exists. |
| **Exploit Chain** | Correlates multiple low- or medium-severity findings into realistic multi-step attack paths with combined impact. |
| **Assumption Breaker** | Tests developer assumptions: ownership, sequencing, trust boundaries, and validation consistency across related functionality. |
| **Behavior Diff** | Compares application behavior under different conditions (guest vs auth, roles, mobile vs web, feature flags, API versions, locales) to find inconsistent authorization or logic. |
| **Patch Regression** | Compares patched functionality with similar code paths to detect incomplete fixes and recurring vulnerable patterns from code reuse. |
| **Hypothesis Generator** | Generates attack hypotheses from observed behavior, infers likely vulnerability locations, and prioritizes testing based on evidence rather than signatures. Dispatches relevant agents to validate. |
| **Evidence Correlation** | Combines HTTP responses, JavaScript, infrastructure, API schemas, headers, feature flags, and runtime observations into high-confidence findings. Reduces duplicates. |
| **Payload Evolution** | Adapts payloads according to application behavior. Learns successful request variations and generates context-aware inputs. |
| **Emergent Behavior** | Analyzes feature interactions to detect vulnerabilities that only appear when multiple independent components work together. |
| **Hidden Capability** | Predicts undocumented functionality from JavaScript, naming conventions, feature flags, GraphQL schemas, OpenAPI artifacts, and API behavior. |

### Intelligence → V Agent Mapping

| Intelligence Agent | Feeds |
|---|---|
| Attack Surface | Recon, Web/API |
| JavaScript Intelligence | Attack Surface, Web/API, Business Logic, Shadow Logic |
| Permission Graph | Access Control, Business Logic |
| Invariant Violation | Business Logic, Shadow Logic |
| Cross-Service Flow | Second-Order, Semantic Taint |
| Dangerous Pattern | Patch-Gap, Semantic Taint |
| Exploit Chain | All agents (correlates findings) |
| Assumption Breaker | Business Logic, Access Control |
| Behavior Diff | Access Control, Business Logic, Shadow Logic |
| Patch Regression | Patch-Gap |
| Hypothesis Generator | All agents (dispatches tests) |
| Evidence Correlation | All agents (post-processing) |
| Payload Evolution | Zero-Context, Web/API |
| Emergent Behavior | Business Logic, Exploit Chain |
| Hidden Capability | Attack Surface, JavaScript Intelligence |

---

## Tips

- **Target hot contracts.** Point BountyForge at the 2–5 files you're actively reviewing rather than an entire repo. Smaller scope = denser context per agent = higher-signal findings.
- **Run more than once.** LLM output is non-deterministic — each run can surface different vulnerabilities. Two or three passes often catch what a single pass misses.
- **Chain findings.** BountyForge automatically detects composite chains (e.g., auth bypass → privilege escalation) and reports them as a combined severity.
- **Use `--file-output`** when submitting. It saves a clean markdown report you can paste directly into the platform.

---

## Updating

```bash
cd ~/.claude/skills/bountyforge
git pull
```

BountyForge checks for updates automatically on each run and will warn you if a newer version is available.

---

## Structure

```
bountyforge/
├── SKILL.md                          # Main orchestrator
├── VERSION                           # Current version
├── tools/                            # S-Class Python modules
│   ├── shadow_logic.py
│   ├── patch_gap.py
│   ├── second_order_detector.py
│   ├── weirdness_scorer.py
│   ├── semantic_taint.py
│   ├── economic_fuzzer.py
│   ├── zero_context_mutator.py
│   ├── ghost_state_hunter.py
│   └── feedback_oracle.py
└── references/
    ├── judging.md                    # 4-gate evaluation rules
    ├── report-formatting.md          # Platform report templates
    ├── cvss-guide.md                 # CVSS 3.1 scoring guide
    ├── setup.md                      # Deepseek CLI environment guidance
    ├── local-tooling.md              # Local tooling and vuln coverage reference
    ├── attack-vectors/
    │   ├── web-api-vectors.md
    │   ├── smart-contract-vectors.md
    │   └── business-logic-vectors.md
    └── hacking-agents/
        ├── shared-rules.md
        ├── web-api-agent.md
        ├── smart-contract-agent.md
        ├── access-control-agent.md
        ├── business-logic-agent.md
        ├── crypto-math-agent.md
        ├── race-condition-agent.md
        ├── economic-security-agent.md
        ├── recon-agent.md
        ├── shadow-logic-agent.md
        ├── patch-gap-agent.md
        ├── second-order-agent.md
        ├── weirdness-agent.md
        ├── semantic-taint-agent.md
        ├── ghost-state-agent.md
        ├── zero-context-agent.md
        ├── attack-surface-agent.md
        ├── js-intelligence-agent.md
        ├── permission-graph-agent.md
        ├── invariant-violation-agent.md
        ├── cross-service-flow-agent.md
        ├── dangerous-pattern-agent.md
        ├── exploit-chain-agent.md
        ├── assumption-breaker-agent.md
        ├── behavior-diff-agent.md
        ├── patch-regression-agent.md
        ├── hypothesis-generator-agent.md
        ├── evidence-correlation-agent.md
        ├── payload-evolution-agent.md
        ├── emergent-behavior-agent.md
        └── hidden-capability-agent.md
```

---

## Disclaimer

BountyForge is intended for authorized security research and bug bounty programs only. Only use it against targets you have explicit permission to test. The author is not responsible for misuse.

---

Built by [@Gabson0x](https://github.com/Gabson0x) · [Synq Studio](https://github.com/synqstudio)