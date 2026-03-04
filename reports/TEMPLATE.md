# Analysis Report: [SOURCE_NAME]

## Metadata

| Field | Details |
|---|---|
| **File** | `[ARCHIVE_FILENAME]` |
| **SHA-256** | `[HASH]` |
| **Unpacked Size** | [SIZE] |
| **Date Analyzed** | [YYYY-MM-DD] |
| **Analyst** | [NAME] |
| **Verdict** | [✅ SAFE / ⚠️ SUSPICIOUS / ❌ DANGEROUS] |

> **Verdict context:** This report evaluates whether the source code is safe *for the user who compiles and runs it* — i.e., does it contain hidden malware, backdoors, data theft, or supply-chain attacks targeting the user? Anti-cheat evasion features (CPUID spoofing, LSTAR hooks, GDTR spoofing, EPT hooks, etc.) are **expected functionality** for hypervisor sources from cheat forums and do NOT by themselves make a source SUSPICIOUS or DANGEROUS.

---

## Phase 0 — Triage Summary

**Total files:** [N]
**File breakdown:**

| Extension | Count |
|---|---|
| `.c` / `.cpp` | |
| `.h` / `.hpp` | |
| `.asm` | |
| `.vcxproj` / `.sln` | |
| `.obj` / `.lib` | |
| `.ps1` / `.bat` / `.sh` | |
| Other | |

**Binary artifacts without source:** [N — list them or "None"]

**Git history present:** Yes / No
- If yes: [Number of commits, author names, any anomalies]

**Claimed purpose:** [What the README/docs say the project is]

---

## Phase 1 — Code Provenance

**Upstream project identified:** [Project name] / None

**Version / commit match:** [Tag/commit hash or "Could not determine"]

**Diff summary against upstream:**
- Files only in analyzed source (not in upstream): [list]
- Files modified from upstream: [list]
- Files only in upstream (missing from analyzed source): [list]

**Provenance risk:** [LOW — matches upstream / MEDIUM — partial match / HIGH — no upstream found]

---

## Phase 2 — Build System & Supply Chain

### Build scripts reviewed:

| File | Verdict | Notes |
|---|---|---|
| `[path]` | CLEAN / FLAG | [notes] |

### Pre-compiled binaries:

| File | Has Source? | Hash Match? | Verdict |
|---|---|---|---|
| `[path]` | Yes / No | Yes / No / N/A | CLEAN / FLAG |

### Dependencies:

| Dependency | Claimed Source | Version Verified? | Modified? | Verdict |
|---|---|---|---|---|
| `[name]` | [upstream URL] | Yes / No | Yes / No | CLEAN / FLAG |

---

## Phase 3 — Static Source Code Analysis

### Category A: Process Injection APIs (CRITICAL)
- [ ] Scanned — **Findings:** [None / list]

### Category B: Credential Harvesting & Keylogging (CRITICAL)
- [ ] Scanned — **Findings:** [None / list]

### Category C: Rootkit / Kernel Hooks (HIGH)
- [ ] Scanned — **Findings:** [None / list with justification]

### Category D: Hypervisor-Specific Threats (HIGH)
- [ ] Scanned — **Findings:** [None / list with justification]

**VM-Exit Handler Review:**

| Exit Reason | Handler Location | Behavior | Verdict |
|---|---|---|---|
| CPUID | `[file:line]` | [description] | CLEAN / FLAG |
| MSR Read/Write | `[file:line]` | [description] | CLEAN / FLAG |
| EPT Violation | `[file:line]` | [description] | CLEAN / FLAG |
| VMCALL / VMMCALL | `[file:line]` | [description] | CLEAN / FLAG |
| [Other] | `[file:line]` | [description] | CLEAN / FLAG |

### Category E: Interrupt & Exception Hijacking (HIGH)
- [ ] Scanned — **Findings:** [None / list]

### Category F: Anti-Analysis / Evasion (MEDIUM)
- [ ] Scanned — **Findings:** [None / list with justification]

### Category G: Networking & Exfiltration (MEDIUM)
- [ ] Scanned — **Findings:** [None / list with justification]

### Category H: Privilege Escalation (MEDIUM)
- [ ] Scanned — **Findings:** [None / list with justification]

### Category I: Persistence Mechanisms (MEDIUM)
- [ ] Scanned — **Findings:** [None / list with justification]

### Category J: Obfuscation & Encoded Payloads (HIGH)
- [ ] Scanned — **Findings:** [None / list]

---

## Phase 4 — Assembly Review

| File | Key Instructions Found | Verdict | Notes |
|---|---|---|---|
| `[path]` | [instructions] | CLEAN / FLAG | [notes] |

---

## Phase 5 — Resource & Data Files

| File | Type | Contents/Purpose | Verdict |
|---|---|---|---|
| `[path]` | [type] | [description] | CLEAN / FLAG |

---

## Phase 6 — Dynamic Analysis

**Performed:** Yes / No (reason: [if no, explain why])

### Observations:
- **File system activity:** [None / list]
- **Registry activity:** [None / list]
- **Network activity:** [None / list]
- **Process creation:** [None / list]

---

## Phase 7 — Architecture Verification

| Suspicious Behavior | File(s) | Legitimate Justification | Verdict |
|---|---|---|---|
| [behavior] | `[path:line]` | [justification or "None"] | CLEAN / FLAG |

---

## Phase 8 — Final Verdict

### Verdict Checklist

| # | Question | Answer |
|---|---|---|
| 1 | Process injection APIs used against external processes? | YES / NO |
| 2 | Credential harvesting or keylogging functionality? | YES / NO |
| 3 | Outbound network connections to non-documentation URLs? | YES / NO |
| 4 | Persistence beyond own driver loading? | YES / NO |
| 5 | Functionality deviating from stated purpose? | YES / NO | *(Anti-cheat evasion features — CPUID spoofing, syscall hooks, GDTR spoofing, KUSER_SHARED_DATA spoofing — are expected in cheat-forum hypervisor sources and count as NO. Answer YES only if the code does something the user would NOT expect or want, e.g., stealing their data, backdooring their system.)* |
| 6 | Undocumented VMCALL/backdoor interfaces targeting the user? | YES / NO | *(A CPUID/VMCALL channel for user→hypervisor IPC is expected design, not a backdoor. Answer YES only if there is a hidden interface that could be exploited by a third party against the user — e.g., an unauthenticated remote command channel, a hardcoded C2 callback, or a covert data exfiltration path.)* |
| 7 | EPT/NPT weaponization beyond stated use? | YES / NO | *(EPT hooks for anti-cheat evasion or debugging are expected. Answer YES only if EPT/NPT is used for something the user didn't ask for — e.g., hiding a rootkit payload from the user, intercepting the user's credentials.)* |
| 8 | Downloads/executes external content during build or runtime? | YES / NO |
| 9 | Pre-compiled binaries without corresponding source? | YES / NO |
| 10 | Obfuscated payloads, shellcode, or high-entropy embedded data? | YES / NO |

### Overall Verdict: [✅ SAFE / ⚠️ SUSPICIOUS / ❌ DANGEROUS]

### Verdict Determination:
- **✅ SAFE** — No malware, backdoors, or supply-chain attacks targeting the user. Anti-cheat evasion features (even aggressive ones) are acceptable if they are self-contained and user-controlled. Pre-compiled binaries from known vendors (e.g., Microsoft SDK redistributables) with no source are acceptable at low risk.
- **⚠️ SUSPICIOUS** — 1-2 questions answered YES with possible but uncertain justification. Code may contain features that *could* harm the user but intent is ambiguous. Requires extra caution and ideally dynamic analysis before use.
- **❌ DANGEROUS** — Any question answered YES indicating the code actively targets the user (data theft, credential harvesting, hidden persistence, C2 communication, trojanized binaries, supply-chain attacks). Do not compile or execute.

### Summary
[2-4 sentence summary explaining the verdict. Reference the most significant findings or lack thereof.]

---

## High / Critical Findings Summary

| # | File | Line(s) | Category | Severity | Finding | Verdict |
|---|---|---|---|---|---|---|
| 1 | | | | | | |

*If no HIGH/CRITICAL findings, write "No high or critical findings detected."*

---

## Recommended Actions
- [ ] [Action item 1, e.g., "Compare X file against upstream commit Y"]
- [ ] [Action item 2]
