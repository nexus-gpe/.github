<p align="center">
  <img src="https://raw.githubusercontent.com/nexus-gpe/.github/main/profile/assets/banner.png" alt="NEXUS Game Performance Engine" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Windows_11-0a0a0a?style=for-the-badge&logo=windows11&logoColor=beee11" alt="Windows 11">
  <img src="https://img.shields.io/badge/Electron-0a0a0a?style=for-the-badge&logo=electron&logoColor=beee11" alt="Electron">
  <img src="https://img.shields.io/badge/React-0a0a0a?style=for-the-badge&logo=react&logoColor=beee11" alt="React">
  <img src="https://img.shields.io/badge/TypeScript-0a0a0a?style=for-the-badge&logo=typescript&logoColor=beee11" alt="TypeScript">
  <img src="https://img.shields.io/badge/.NET_8-0a0a0a?style=for-the-badge&logo=dotnet&logoColor=beee11" alt=".NET 8">
  <img src="https://img.shields.io/badge/Fastify-0a0a0a?style=for-the-badge&logo=fastify&logoColor=beee11" alt="Fastify">
  <img src="https://img.shields.io/badge/SQLite-0a0a0a?style=for-the-badge&logo=sqlite&logoColor=beee11" alt="SQLite">
</p>

<h3 align="center">Measurable. Reversible. Verified.</h3>

<p align="center">
NEXUS is a game performance engine for Windows. It reads your hardware and your games, applies optimizations whose effect can be measured, backs every change up before it touches anything, and proves the result with real frame-time benchmarks.<br>
<b>No placebo tweaks. No FPS promises. Your numbers, on your PC.</b>
</p>

<br>

## What a boost is

A boost is a switch. Behind every switch: the real current value of the setting, the value NEXUS wants, why that helps, when it won't, and the risk level. Switching it off restores exactly what was there before. A boost that cannot apply on your system says why instead of hiding.

<img src="https://raw.githubusercontent.com/nexus-gpe/.github/main/profile/assets/boost.png" alt="A boost is a switch with an explanation" width="100%">

## How a boost runs

Every switch runs the same transaction. The original value is snapshotted to disk before the first write, the change is verified by reading it back from Windows, and a failed verification rolls the change back on the spot. Multi-step sets unwind in reverse order on the first failure: nothing is ever left half-applied.

<img src="https://raw.githubusercontent.com/nexus-gpe/.github/main/profile/assets/lifecycle.png" alt="Boost lifecycle: detect, explain, snapshot, apply, verify, rollback" width="100%">

```mermaid
%%{init: {'theme':'dark','themeVariables':{'primaryColor':'#141414','primaryBorderColor':'#beee11','primaryTextColor':'#ffffff','lineColor':'#beee11','secondaryColor':'#1b1b1b','tertiaryColor':'#0a0a0a','fontFamily':'Segoe UI, sans-serif'}}}%%
stateDiagram-v2
    direction LR
    [*] --> Detect
    Detect --> NotApplicable: no value / no permission
    Detect --> AlreadyOptimal: matches target
    Detect --> Available
    Available --> Snapshot: switch on
    Snapshot --> Apply
    Apply --> Verify
    Verify --> Applied: read-back matches
    Verify --> RolledBack: mismatch
    Applied --> RolledBack: switch off
    RolledBack --> Available
    NotApplicable --> [*]: reason shown in the app
```

## Architecture

The interface never touches Windows. The renderer is sandboxed with no Node and no shell; it talks to the Electron main process over a typed, exhaustively validated IPC contract; the main process talks to a separate .NET 8 process, the Performance Core, over newline-delimited JSON on stdio. Only the core reads or writes the machine.

<img src="https://raw.githubusercontent.com/nexus-gpe/.github/main/profile/assets/architecture.png" alt="NEXUS architecture: renderer, main process, Performance Core, Windows" width="100%">

```mermaid
%%{init: {'theme':'dark','themeVariables':{'primaryColor':'#141414','primaryBorderColor':'#beee11','primaryTextColor':'#ffffff','actorBorder':'#beee11','actorBkg':'#141414','actorTextColor':'#ffffff','signalColor':'#beee11','signalTextColor':'#ffffff','noteBkgColor':'#1b1b1b','noteTextColor':'#ffffff','noteBorderColor':'#333333','fontFamily':'Segoe UI, sans-serif'}}}%%
sequenceDiagram
    participant UI as Renderer
    participant Main as Main process
    participant Core as Performance Core
    participant Win as Windows
    UI->>Main: optimizer:apply-selection {ids}  (zod-validated)
    Main->>Core: {"id":"42","method":"optimizer.applySelection",...}
    Core->>Win: read current values
    Core->>Core: write snapshot (SHA-256) to disk
    Core->>Win: apply change set
    Core->>Win: read back
    alt every value matches its target
        Core-->>Main: {"id":"42","ok":true,"result":{...,"verified":true}}
    else a read-back differs
        Core->>Win: restore snapshot, reverse order
        Core-->>Main: {"id":"42","ok":false,"error":{"code":"VERIFICATION_FAILED",...}}
    end
    Main-->>UI: Result<ApplyResult>
```

## Benchmarks

FPS claims are easy to make and hard to measure honestly. NEXUS records frame times straight from the graphics kernel (an ETW session on `Microsoft-Windows-DxgKrnl`) while the game runs, counts each frame exactly once, and derives every number from those frame times: time-weighted average, 1% and 0.1% lows by the percentile method, p95 / p99 frame time as the render-latency floor. Too few samples for a statistic? It is withheld, not estimated.

<img src="https://raw.githubusercontent.com/nexus-gpe/.github/main/profile/assets/benchmark.png" alt="Benchmark method and a measured before/after" width="100%">

## Safety model

<img src="https://raw.githubusercontent.com/nexus-gpe/.github/main/profile/assets/safety.png" alt="Safety model" width="100%">

## Stack

| Layer | Technology | Role |
| --- | --- | --- |
| Desktop | Electron · React 19 · TypeScript · Tailwind · Zustand · Framer Motion | Launcher-style UI, per-game workspace, overlay HUD |
| Contracts | Shared TypeScript types · zod schemas | One typed IPC surface, validated on both ends |
| Performance Core | .NET 8 (self-contained, win-x64) · WMI · performance counters · ETW · P/Invoke | Hardware scan, optimizations, snapshots, benchmarks, telemetry |
| Server | Fastify 5 · SQLite · scrypt · rotating refresh tokens | Accounts, admin panel, changelog, release feed |
| Delivery | NEXUS installs, patches and uninstalls itself | Own setup UI; updates download in the background, verify SHA-512, apply on restart |

## Principles

1. **If it cannot be measured, it does not ship.**
2. **If it cannot be reversed, it does not ship.**
3. **Say why something is unavailable** instead of hiding it.
4. **Never touch the game process.** Overlay and capture stay outside it.

<p align="center"><sub>Fortnite is supported today. More titles follow the same rules.</sub></p>
