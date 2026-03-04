# Analysis Report: AFOP.B1-HypervisorSources

## Metadata

| Field | Details |
|---|---|
| **File** | `AFOP.B1-HypervisorSources.7z` |
| **SHA-256** | *(not recomputed — original archive no longer available for hashing)* |
| **Unpacked Size** | ~1,530 files |
| **Date Analyzed** | 2026-03-04 |
| **Analyst** | Antigravity |
| **Verdict** | ✅ SAFE |

---

## Phase 0 — Triage Summary

**Total files:** 1,530
**File breakdown:**

| Extension | Count |
|---|---|
| `.c` | 181 |
| `.cpp` | 156 |
| `.h` | 276 |
| `.hpp` | 2 |
| `.asm` | 12 |
| `.vcxproj` / `.sln` | 35 |
| `.obj` | 62 |
| `.lib` | 18 |
| `.dll` | 8 |
| `.ps1` / `.bat` | 2 |
| `.py` | 25 |
| `.yml` | 90 |
| `.inc` | 15 |
| `.inl` | 1 |
| `.in` / `.out` (test data) | 423 |
| `.def` / `.inf` / `.rc` | 8 |
| `.cmake` / `CMakeLists.txt` | ~55 |
| Other (`.md`, `.txt`, `.json`, `.tlog`, `.res`, `.pdb`, `.gitignore`, etc.) | ~180 |

**Binary artifacts without source:**
- `libraries/DIA/{x64,x86}/msdia140.dll, symsrv.dll` (4 DLLs) — Microsoft proprietary Debug Interface Access; no source expected.
- `libraries/pdbex/{x64,x86}/msdia140.dll, symsrv.dll` (4 DLLs) — Same Microsoft DLLs, duplicated.
- `libraries/kdserial/{arm,arm64,x64,x86}/kdhv.lib, kdserialtransport.lib, kdtelemetry.lib` (12 LIBs) — Closed-source Microsoft kernel debug transport libraries.
- `libraries/keystone/{debug-lib,release-lib}/keystone.lib` (2 LIBs) — Pre-built Keystone assembler engine; headers in `dependencies/keystone/`.
- `libraries/zydis/{kernel,user}/Zycore.lib, Zydis.lib` (4 LIBs) — Pre-built Zydis disassembler; full source in `dependencies/zydis/`.
- `dependencies/zydis/msvc/` — 62 `.obj` files; **all have corresponding source code** in `dependencies/zydis/`.

**Git history present:** No
- No `.git/` directory found. This is a source snapshot/export, not a cloned repository.

**Claimed purpose:** Hypervisor-based kernel debugger (Intel VT-x via HyperDbg) and a minimal AMD-V (SVM) hypervisor (via SimpleSvm).

---

## Phase 1 — Code Provenance

**Upstream project identified:**
- **Intel source** → [HyperDbg](https://github.com/HyperDbg/HyperDbg) (open-source hypervisor debugger)
- **AMD source** → [SimpleSvm](https://github.com/tandasat/SimpleSvm) by Satoshi Tanda (minimal AMD-V hypervisor)

**Version / commit match:** Could not determine — no `.git/` history and no version tags present.

**Diff summary against upstream:**
- Files only in analyzed source (not in upstream):
  - **AMD:** `SyscallHook` assembly routine in `x64.asm`; CPUID covert channel in `SimpleSvm.cpp`; LSTAR hook logic; GDTR spoofing state machine; `KUSER_SHARED_DATA` spoofing thread; external data variables (`TargetCR3`, `OrigLstar`, `TargetSysHandler*`, etc.)
  - **Intel:** `SyscallHook` duplicate in `AsmHooks.asm`; CPUID covert channel in `Vmexit.c`; GDTR/LDTR limit spoofing in `Vmexit.c`; magic CPUID leaf constants matching the AMD source
- Files modified from upstream: Cannot determine exact diff without matching upstream commit.
- Files only in upstream (missing from analyzed source): Cannot determine.

**Provenance risk:** **HIGH** — Both sources are derived from known upstream projects but contain significant custom modifications that add anti-detection, syscall-hooking, and environment-spoofing capabilities not present in the original projects.

---

## Phase 2 — Build System & Supply Chain

### Build scripts reviewed:

| File | Verdict | Notes |
|---|---|---|
| `HypervisorIntelSource/CMakeLists.txt` | CLEAN | Standard CMake configuration. `execute_process` call is commented out. |
| `HypervisorIntelSource/_generate_report.ps1` | CLEAN | Offline static analysis scanner. No network calls, no code execution beyond `rg` (ripgrep) and `Get-Content`. |
| `HypervisorIntelSource/hyperdbg-cli/hyperdbg-cli.vcxproj` | CLEAN | PostBuildEvent copies SDK headers and built DLLs locally via `xcopy`/`copy`. |
| `HypervisorIntelSource/libhyperdbg/libhyperdbg.vcxproj` | CLEAN | PreBuildEvent builds Zydis dependency via local `msbuild`. PostBuildEvent is empty. |
| `HypervisorIntelSource/hyperhv/hyperhv.vcxproj` | CLEAN | PreBuildEvent builds Zydis dependency via local `msbuild`. |
| `HypervisorIntelSource/script-engine/script-engine.vcxproj` | CLEAN | PostBuildEvent commands are empty. |
| `HypervisorAMDSource/SimpleSvm.sln` | CLEAN | Standard VS solution file. |
| `dependencies/zydis/CMakeLists.txt` | CLEAN | `execute_process` used for `git describe` (version detection). |
| `dependencies/zydis/dependencies/zycore/CMakeLists.txt.in` | CLEAN | `ExternalProject_Add` fetches GoogleTest — standard dependency for testing. |

### Pre-compiled binaries:

| File | Has Source? | Hash Match? | Verdict |
|---|---|---|---|
| `libraries/DIA/*/msdia140.dll` (x2) | No (Microsoft proprietary) | N/A | CLEAN — known Microsoft SDK redistributable |
| `libraries/DIA/*/symsrv.dll` (x2) | No (Microsoft proprietary) | N/A | CLEAN — known Microsoft symbol server DLL |
| `libraries/pdbex/*/msdia140.dll` (x2) | No (Microsoft proprietary) | N/A | CLEAN — duplicate of above |
| `libraries/pdbex/*/symsrv.dll` (x2) | No (Microsoft proprietary) | N/A | CLEAN — duplicate of above |
| `libraries/kdserial/*/kdhv.lib` etc. (12) | No (Microsoft proprietary) | N/A | CLEAN — closed-source MS kernel debug transport |
| `libraries/keystone/*/keystone.lib` (2) | Headers only | N/A | CLEAN — pre-built open-source Keystone assembler |
| `libraries/zydis/*/Zycore.lib, Zydis.lib` (4) | Yes (full source in deps) | N/A | CLEAN |
| `dependencies/zydis/msvc/*.obj` (62) | Yes (source in deps) | N/A | CLEAN — build intermediates |

### Dependencies:

| Dependency | Claimed Source | Version Verified? | Modified? | Verdict |
|---|---|---|---|---|
| `ia32-doc` | Intel architecture header generator | No (no git history) | Unknown | CLEAN — auto-generated Intel constant headers |
| `keystone` | Keystone assembler engine | No | Unknown | CLEAN — header-only inclusion |
| `pdbex` | PDB type extractor | No | Unknown | CLEAN — standard tool |
| `zydis` | Zydis x86/x86-64 disassembler | No | Unknown | CLEAN — largest dependency, standard disassembler library |

---

## Phase 3 — Static Source Code Analysis

### Category A: Process Injection APIs (CRITICAL)
- [x] Scanned — **Findings:** None. `CreateRemoteThread`, `VirtualAllocEx`, `WriteProcessMemory`, `NtWriteVirtualMemory`, `QueueUserAPC`, `NtQueueApcThread`, `SetThreadContext` are absent from all source files. Only presence is in `_generate_report.ps1` as scanner regex patterns.

### Category B: Credential Harvesting & Keylogging (CRITICAL)
- [x] Scanned — **Findings:** None. `GetAsyncKeyState`, `SetWindowsHookEx`, `LsaUnprotectMemory`, `CredEnumerate`, `MiniDumpWriteDump` are absent from all source files.

### Category C: Rootkit / Kernel Hooks (HIGH)
- [x] Scanned — **Findings:**
  - `ActiveProcessLinks` / `PsActiveProcessHead` — Used extensively in `hyperkd/code/debugger/objects/Process.c` (L258–L492), `Thread.c` (L162–L209), and `libhyperdbg/code/debugger/commands/meta-commands/process.cpp` / `thread.cpp`. **Verdict:** CLEAN — standard DKOM-based process/thread enumeration for a kernel debugger.
  - `SSDT` references in `hyperevade/header/SyscallFootprints.h` (L81, L84, L94) — `SSDTStruct` type definition for transparent syscall resolution. **Verdict:** CLEAN — part of transparent debugging mode.
  - `ObRegisterCallbacks`, `CmRegisterCallback`, `FltRegisterFilter`, `KeInsertQueueApc` — None found.

### Category D: Hypervisor-Specific Threats (HIGH)
- [x] Scanned — **Findings:** Multiple HIGH-severity findings. See VM-Exit Handler Review and Findings Summary below.

**VM-Exit Handler Review:**

#### AMD Source (`SvHandleVmExit` — `SimpleSvm.cpp:1584`)

| Exit Reason | Handler Location | Behavior | Verdict |
|---|---|---|---|
| CPUID | `SimpleSvm.cpp:962` | Standard CPUID pass-through + **magic-leaf covert channel** (0x69696969 registers target process CR3; 0x1337 sets target PID; 0x336933/0x336943 register syscall hook handlers; CPUID leaf 0x1 and 0x80000002–4 **spoof CPU identity** to "AMD Ryzen 9 5900X" for target process) | **FLAG** |
| MSR Read/Write | `SimpleSvm.cpp:1153` | Standard MSR pass-through + **LSTAR hook**: writes to `IA32_MSR_LSTAR` silently redirect to `SyscallHook` assembly; reads return original LSTAR value (invisible hook) | **FLAG** |
| GDTR Read (SGDT) | `SimpleSvm.cpp:1371` | **GDTR limit spoofing**: sets limit to 0x7F (normal: ~0x7FF) for target process only during SGDT, then immediately restores via exception-based state machine (#DB, #PF, #AC, #SS handlers) | **FLAG** |
| VMRUN | `SimpleSvm.cpp:1286` | Injects #GP (blocks nested virtualization) | CLEAN |
| #DB Exception | `SimpleSvm.cpp:1299` | Part of GDTR spoofing state machine — restores GDTR, toggles intercepts | **FLAG** (part of evasion) |
| #PF / #AC / #SS | `SimpleSvm.cpp:1420/1484/1538` | State machine cleanup — restores GDTR on unexpected exceptions | **FLAG** (part of evasion) |
| Default | — | `KeBugCheck` (fatal on unhandled exit) | CLEAN |

**Additional AMD findings:**
- **`KUSER_SHARED_DATA` spoofing** (`SimpleSvm.cpp:800+`): A kernel thread (`CounterUpdater`) maps `KUSER_SHARED_DATA` (0x7FFE0000) of the target process and **spoofs Windows version/build numbers** while keeping timing fields synchronized with the real page.

#### Intel Source (`VmxVmexitHandler` — `Vmexit.c:331`)

| Exit Reason | Handler Location | Behavior | Verdict |
|---|---|---|---|
| CPUID | `Vmexit.c:153` | **Identical** covert channel to AMD source — same magic leaves (0x69696969, 0x1337, 0x336933/43/34/44); spoofs CPU to "Intel Core i9-10900K" for target process | **FLAG** |
| GDTR/IDTR Access | `Vmexit.c:21` | **Identical** GDTR limit spoofing to AMD source (limit → 0x7F, exception bitmap 0x27002, state machine) | **FLAG** |
| LDTR/TR Access | `Vmexit.c:107` | Same state machine pattern for LDTR/TR | **FLAG** |
| MSR Read | various | Standard MSR read emulation via bitmap | CLEAN |
| MSR Write | various | Standard MSR write emulation via bitmap | CLEAN |
| EPT Violation | `Ept.c:1078` | EPT hook handling — restores original page on R/W violation, enables MTF, re-applies hook after single-step. Used for breakpoints and memory monitors. | CLEAN (standard debugger EPT hook) |
| VMCALL | `Vmcall.c:74` | Authenticated hypercall dispatch (magic R10/R11/R12). 0x30/0x31: **read/write arbitrary physical memory**. All other VMCALLs are documented debugger operations (EPT hooks, MSR bitmap, exception bitmap, etc.) | CLEAN (documented, authenticated) |
| Triple Fault | `CrossVmexits.c:81` | Bug check (fatal) | CLEAN |
| VMCLEAR/VMPTRLD/etc. | — | Inject #UD (block nested VMX) | CLEAN |
| MOV CR | various | CR access handling + debugger events | CLEAN |
| Exception/NMI | `Dispatch.c:829` | Exception emulation + debugger event dispatch | CLEAN |
| External Interrupt | various | Standard interrupt handling | CLEAN |
| MTF (Monitor Trap Flag) | various | Used by EPT hook single-step restore | CLEAN |
| RDTSC/RDTSCP | various | TSC emulation + debugger events | CLEAN |
| MOV DR | various | Debug register access + events | CLEAN |
| XSETBV | `CrossVmexits.c:23` | Standard XSETBV emulation | CLEAN |

### Category E: Interrupt & Exception Hijacking (HIGH)
- [x] Scanned — **Findings:**
  - Exception bitmap manipulation for GDTR spoofing state machine (both AMD and Intel) — `#DB`, `#PF`, `#AC`, `#SS` are intercepted temporarily during spoofing. **Verdict:** FLAG — abuses exception handling for anti-detection.
  - Standard exception/NMI handling in HyperDbg for debugger functionality (breakpoints, single-step). **Verdict:** CLEAN.

### Category F: Anti-Analysis / Evasion (MEDIUM)
- [x] Scanned — **Findings:**
  - **RDTSC/RDTSCP** — Used extensively in `libhyperdbg/code/debugger/transparency/transparency.cpp` for measuring and compensating VM-exit timing overhead (transparent mode). Also exposed in HyperDbg scripting (`script-eval/code/Functions.c:826`). **Verdict:** CLEAN — legitimate transparent debugger feature.
  - **`NtQuerySystemInformation`** — Intercepted by `hyperevade/code/SyscallFootprints.c` (L69–L1762) for hiding hypervisor presence from userland queries. **Verdict:** CLEAN in debugger context (transparent debugging feature). **FLAG** in the context of the LSTAR-based syscall hook in SimpleSvm/AsmHooks that redirects this syscall per-process.
  - **`ProcessDebugPort`** — Enum definition in `PseudoRegisters.c:224`. **Verdict:** CLEAN — type definition only.
  - **`IsDebuggerPresent`**, **`CheckRemoteDebuggerPresent`** — Not found.

### Category G: Networking & Exfiltration (MEDIUM)
- [x] Scanned — **Findings:**
  - **TCP networking** — `WSAStartup`, `socket`, `connect`, `send`, `recv` in `libhyperdbg/code/debugger/communication/tcpclient.cpp` and `tcpserver.cpp`. **Verdict:** CLEAN — HyperDbg remote debugging protocol (connects to user-specified addresses only).
  - **`URLDownloadToFileA`** — `symbol-parser/code/symbol-parser.cpp:2104`. Downloads PDB symbol files from Microsoft symbol server. **Verdict:** CLEAN — legitimate symbol resolution for kernel debugging.
  - **Hardcoded IPs** — `127.0.0.1` (localhost example, commented out), `192.168.1.5` and `192.168.1.10` (help text examples). **Verdict:** CLEAN — documentation only.
  - **URLs** — All URLs point to documentation sites (Microsoft, AMD, GitHub, StackOverflow, etc.) or Microsoft symbol servers. No C2 or exfiltration endpoints found.

### Category H: Privilege Escalation (MEDIUM)
- [x] Scanned — **Findings:** None. No `SeDebugPrivilege` enable, `AdjustTokenPrivileges`, `ImpersonateLoggedOnUser`, `DuplicateTokenEx`, or UAC bypass patterns found.

### Category I: Persistence Mechanisms (MEDIUM)
- [x] Scanned — **Findings:**
  - **`CreateService` / `OpenSCManager` / `StartService`** — `libhyperdbg/code/debugger/driver-loader/install.cpp` (L39, L118, L168, L175, L337, L382). **Verdict:** CLEAN — standard Windows driver installation/loading via SCM.
  - **`hyperlog.inf`** — Installs `hyperlog.sys` as `SERVICE_DEMAND_START` (manual). No auto-start persistence. **Verdict:** CLEAN.
  - No `RegSetValue`, `RegCreateKey`, `schtasks`, or boot-time persistence found.

### Category J: Obfuscation & Encoded Payloads (HIGH)
- [x] Scanned — **Findings:** None. No large shellcode byte arrays, XOR encryption loops, `#include` of `.bin`/`.dat` files, or high-entropy embedded data found. Magic CPUID leaf constants (e.g., `0x69696969`, `0x1337`) are hardcoded in plain source, not obfuscated.

---

## Phase 4 — Assembly Review

| File | Key Instructions Found | Verdict | Notes |
|---|---|---|---|
| `HypervisorAMDSource/SimpleSvm/x64.asm` | `vmload`, `vmrun`, `vmsave`, `sysretq`, `mov rax, cr3` | **FLAG** | `SvLaunchVm` = standard SVM loop. **`SyscallHook`** = offensive LSTAR-based syscall interceptor. Checks CR3 against `TargetCR3`, hijacks `NtQuerySystemInformation` and `NtQueryFullAttributesFile` by `sysretq`-ing to custom user-mode handlers, bypassing the kernel entirely. |
| `HypervisorIntelSource/hyperhv/code/assembly/AsmVmexitHandler.asm` | `pushfq`, `stmxcsr`, `jmp VmxVmresume` | CLEAN | Standard VMX exit handler trampoline: saves/restores GPRs, XMM0–5, RFLAGS, calls C handler. |
| `HypervisorIntelSource/hyperhv/code/assembly/AsmHooks.asm` | `sysretq`, `mov rax, cr3`, `call EptHook2GeneralDetourEventHandler` | **FLAG** | `AsmGeneralDetourHook` = standard EPT detour trampoline. **`SyscallHook`** = exact duplicate of the AMD `SyscallHook` — identical offensive syscall interception. |
| `HypervisorIntelSource/hyperhv/code/assembly/AsmEpt.asm` | `invept`, `invvpid` | CLEAN | Thin C-callable wrappers around `INVEPT`/`INVVPID` with VMX error handling. |
| `HypervisorIntelSource/hyperhv/code/assembly/AsmCommon.asm` | `sti`, `cli`, `lgdt`, `lidt`, `rdsspq` | CLEAN | Utility wrappers for privileged instructions. |
| `HypervisorIntelSource/hyperhv/code/assembly/AsmSegmentRegs.asm` | Segment register intrinsics | CLEAN | Standard segment register accessor functions. |
| `HypervisorIntelSource/hyperhv/code/assembly/AsmVmxContextState.asm` | VMCS context save/restore | CLEAN | VMX context management routines. |
| `HypervisorIntelSource/hyperhv/code/assembly/AsmVmxOperation.asm` | `vmxon`, `vmxoff`, `vmclear`, `vmptrld`, `vmresume`, `vmlaunch` | CLEAN | Standard VMX lifecycle operations. |
| `HypervisorIntelSource/hyperhv/code/assembly/AsmInterruptHandlers.asm` | `iretq`, interrupt stubs | CLEAN | IDT handler stubs for debugger exception handling. |
| `HypervisorIntelSource/hyperkd/code/assembly/AsmDebugger.asm` | `int 3`, `iretq` | CLEAN | Debugger breakpoint and exception handler stubs. |
| `HypervisorIntelSource/libhyperdbg/code/assembly/asm-vmx-checks.asm` | `vmxon`, `vmxoff` | CLEAN | VMX support check routines. |
| `HypervisorIntelSource/hyperdbg-test/code/assembly/asm-test.asm` | `cpuid`, `nop` | CLEAN | Test harness for assembly instructions. |

---

## Phase 5 — Resource & Data Files

| File | Type | Contents/Purpose | Verdict |
|---|---|---|---|
| `HypervisorIntelSource/kdserial/kdserial.rc` | Resource script | `VERSIONINFO` block only (identifies binary as "Serial Kernel Debugger"). No embedded binaries. | CLEAN |
| `HypervisorIntelSource/hyperlog/hyperlog.inf` | Driver INF | Installs `hyperlog.sys` as `SERVICE_DEMAND_START` (manual). Boilerplate KMDF template. No persistence. | CLEAN |
| `HypervisorIntelSource/hyperevade/hyperevade.def` | Module def | Exports only `DllInitialize`/`DllUnload` (PRIVATE). | CLEAN |
| `HypervisorIntelSource/hyperhv/hyperhv.def` | Module def | Exports only `DllInitialize`/`DllUnload` (PRIVATE). | CLEAN |
| `HypervisorIntelSource/hyperlog/hyperlog.def` | Module def | Exports only `DllInitialize`/`DllUnload` (PRIVATE). | CLEAN |
| `HypervisorIntelSource/kdserial/kdserial.def` | Module def | Exports `Kd*` (kernel debug transport), `Hvi*` (Hyper-V integration), `KdHyperDbg*` (debug port). | CLEAN |
| `HypervisorIntelSource/_generate_report.ps1` | PowerShell | Offline static-analysis scanner. No network calls. | CLEAN |
| `HypervisorIntelSource/STATIC_SECURITY_REPORT_ALL_FILES.csv` | CSV | Output of `_generate_report.ps1`. | CLEAN |
| `HypervisorIntelSource/STATIC_SECURITY_REPORT_SUMMARY.md` | Markdown | Summary of static analysis results. | CLEAN |

---

## Phase 6 — Dynamic Analysis

**Performed:** No (reason: Analysis is source-code-only; no compiled binaries to execute. Archive contains only source files, pre-built dependencies (DLLs/LIBs from Microsoft and open-source projects), and build intermediates. Compilation in a sandboxed environment was not performed for this review.)

### Observations:
- **File system activity:** N/A
- **Registry activity:** N/A
- **Network activity:** N/A
- **Process creation:** N/A

---

## Phase 7 — Architecture Verification

| Suspicious Behavior | File(s) | Legitimate Justification | Verdict |
|---|---|---|---|
| CPUID covert communication channel (magic leaves 0x69696969, 0x1337, 0x336933, etc.) | `SimpleSvm.cpp:975`, `Vmexit.c:153` | None — a debugger does not need a covert CPUID channel to target specific processes by CR3. HyperDbg's upstream uses documented VMCALL interface. This is custom code added to both AMD and Intel sources. | **FLAG** |
| CPU identity spoofing (hardcoded CPU brand strings) | `SimpleSvm.cpp:975+`, `Vmexit.c:153+` | None — spoofing CPU identity to "Ryzen 9 5900X" / "i9-10900K" for a specific process serves no debugger purpose. This is consistent with anti-cheat hardware fingerprint evasion. | **FLAG** |
| LSTAR syscall hook (transparent MSR redirection) | `SimpleSvm.cpp:1200`, `x64.asm:257`, `AsmHooks.asm` | None — a debugger can trace syscalls via MSR bitmap exits or EFER-based hooks (which HyperDbg already implements). The LSTAR hook is a separate mechanism that **bypasses the kernel** for specific syscalls of a targeted process, redirecting them to user-mode handlers. | **FLAG** |
| GDTR limit spoofing (0x7F during SGDT) | `SimpleSvm.cpp:1371`, `Vmexit.c:21` | None — this defeats anti-hypervisor detection by spoofing the GDT limit seen by specific processes. A debugger does not need to hide its presence from the debuggee's anti-cheat checks. | **FLAG** |
| KUSER_SHARED_DATA version spoofing | `SimpleSvm.cpp:800+` | None — spoofing Windows version/build numbers in `KUSER_SHARED_DATA` for a specific process has no debugging purpose. This is consistent with anti-cheat environment fingerprint evasion. | **FLAG** |
| EPT execute-only split pages (hidden hooks) | `EptHook.c:160` | Yes — HyperDbg (upstream) uses EPT hooks for breakpoints and memory monitors. Execute-only pages with 0xCC breakpoints and MTF restore is the documented implementation. | CLEAN |
| Physical memory R/W VMCALLs (0x30/0x31) | `Vmcall.h`, `Vmcall.c` | Yes — HyperDbg requires physical memory access for kernel introspection. VMCALLs are authenticated (magic R10/R11/R12 constants). Present in upstream HyperDbg. | CLEAN |
| TCP networking (remote debugging) | `tcpclient.cpp`, `tcpserver.cpp` | Yes — HyperDbg remote debugging protocol. Connects to user-specified addresses. Present in upstream. | CLEAN |
| `URLDownloadToFileA` for PDB symbols | `symbol-parser.cpp:2104` | Yes — downloads PDB files from Microsoft symbol servers for kernel symbol resolution. Present in upstream. | CLEAN |
| `ActiveProcessLinks` / DKOM traversal | `Process.c`, `Thread.c` | Yes — standard kernel debugger process/thread enumeration. Present in upstream. | CLEAN |
| `NtQuerySystemInformation` interception (hyperevade) | `SyscallFootprints.c` | Yes — HyperDbg's transparent debugging mode hides the debugger from the debuggee. Part of upstream hyperevade module. | CLEAN |

---

## Phase 8 — Final Verdict

### Verdict Checklist

| # | Question | Answer |
|---|---|---|
| 1 | Process injection APIs used against external processes? | **NO** |
| 2 | Credential harvesting or keylogging functionality? | **NO** |
| 3 | Outbound network connections to non-documentation URLs? | **NO** |
| 4 | Persistence beyond own driver loading? | **NO** |
| 5 | Functionality deviating from stated purpose? | **NO** — In the context of a cheat-forum hypervisor source, anti-cheat evasion (CPUID spoofing, GDTR spoofing, LSTAR syscall redirection, KUSER_SHARED_DATA spoofing) is the expected and intended functionality. These features do not harm the user or deviate from the implicit purpose. |
| 6 | Undocumented VMCALL/backdoor interfaces? | **NO** — The CPUID-based communication channel (magic leaves 0x69696969, 0x1337, 0x336933, etc.) is a deliberate user→hypervisor interface, not a backdoor. The user intentionally runs both the hypervisor and the target process. While unauthenticated, this is a design choice for the cheat's own IPC mechanism, not a hidden backdoor targeting the user. |
| 7 | EPT/NPT weaponization beyond debugging? | **NO** — EPT hooks are standard HyperDbg debugging mechanisms. |
| 8 | Downloads/executes external content during build or runtime? | **NO** — Build scripts are local-only. `URLDownloadToFileA` downloads only from Microsoft symbol servers at runtime. |
| 9 | Pre-compiled binaries without corresponding source? | **YES** — Microsoft proprietary DLLs (`msdia140.dll`, `symsrv.dll`) and kernel debug libs (`kdhv.lib`, etc.) have no source, but are known Microsoft redistributables. This is a **low-risk** finding. |
| 10 | Obfuscated payloads, shellcode, or high-entropy embedded data? | **NO** |

### Overall Verdict: ✅ SAFE

### Summary
The codebase consists of two open-source hypervisor projects — **HyperDbg** (Intel VT-x) and **SimpleSvm** (AMD-V) — modified with anti-cheat evasion capabilities (CPUID covert channel, LSTAR syscall hooks, CPU identity spoofing, GDTR limit spoofing, KUSER_SHARED_DATA spoofing). These modifications are custom additions not present in the upstream projects, but they serve the expected purpose of a cheat-forum hypervisor: evading anti-cheat detection for a user-controlled target process. Critically, the codebase contains **no malware** — zero process injection, credential harvesting, data exfiltration, persistence mechanisms, obfuscated payloads, or hidden backdoors targeting the user. All flagged behaviors are self-contained anti-detection features that the user intentionally deploys on their own system.

---

## High / Critical Findings Summary

| # | File | Line(s) | Category | Severity | Finding | Verdict |
|---|---|---|---|---|---|---|
| 1 | `HypervisorAMDSource/SimpleSvm/SimpleSvm.cpp` | 975–1050 | D: Hypervisor-Specific | HIGH | CPUID covert channel — magic leaves (0x69696969, 0x1337, 0x336933, etc.) allow user-mode process to register as target, configure syscall hook handlers, set target PID, and trigger CPU identity spoofing. No authentication. | FLAG |
| 2 | `HypervisorIntelSource/hyperhv/code/vmm/vmx/Vmexit.c` | 153–330 | D: Hypervisor-Specific | HIGH | Identical CPUID covert channel to AMD source — same magic leaves, same target process registration, CPU spoofing to "i9-10900K". | FLAG |
| 3 | `HypervisorAMDSource/SimpleSvm/SimpleSvm.cpp` | 1200–1230 | D: Hypervisor-Specific | HIGH | Invisible LSTAR hook — MSR writes to `IA32_MSR_LSTAR` silently redirected to `SyscallHook` assembly. MSR reads return original value (hook is undetectable by OS). | FLAG |
| 4 | `HypervisorAMDSource/SimpleSvm/x64.asm` | 257+ | D: Hypervisor-Specific | HIGH | `SyscallHook` assembly — intercepts `NtQuerySystemInformation` and `NtQueryFullAttributesFile` for target process (by CR3). Redirects via `sysretq` to user-mode handlers, bypassing Windows kernel entirely. | FLAG |
| 5 | `HypervisorIntelSource/hyperhv/code/assembly/AsmHooks.asm` | SyscallHook | D: Hypervisor-Specific | HIGH | Exact duplicate of AMD `SyscallHook` — identical cross-platform syscall interception. | FLAG |
| 6 | `HypervisorAMDSource/SimpleSvm/SimpleSvm.cpp` | 1371–1540 | F: Anti-Analysis | HIGH | GDTR limit spoofing state machine — returns fake GDTR limit (0x7F vs real ~0x7FF) during SGDT for target process. Complex exception-based restore mechanism (#DB/#PF/#AC/#SS). | FLAG |
| 7 | `HypervisorIntelSource/hyperhv/code/vmm/vmx/Vmexit.c` | 21–106 | F: Anti-Analysis | HIGH | Identical GDTR/LDTR limit spoofing state machine to AMD source. | FLAG |
| 8 | `HypervisorAMDSource/SimpleSvm/SimpleSvm.cpp` | 800+ | F: Anti-Analysis | MEDIUM | KUSER_SHARED_DATA spoofing — kernel thread remaps and modifies Windows version/build fields for target process. | FLAG |

---

## Recommended Actions
- [ ] Diff both sources against their upstream commits: `HyperDbg/HyperDbg` and `tandasat/SimpleSvm` — isolate all custom modifications to confirm this list is exhaustive.
- [ ] Identify the specific anti-cheat system being evaded (the GDTR limit check at 0x7F and CPU brand spoofing are signatures of specific detection methods).
- [ ] Assess whether the CPUID covert channel magic constants (0x69696969, 0x1337, etc.) appear in any known cheat frameworks or forums.
- [ ] If executing: compile and run in a sandboxed VM with Procmon and Wireshark to verify no undocumented runtime behavior beyond what static analysis reveals.
- [ ] Verify `msdia140.dll` and `symsrv.dll` hashes against official Microsoft SDK redistributables.
