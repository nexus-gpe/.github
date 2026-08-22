<p align="center">
  <img src="https://raw.githubusercontent.com/nexus-gpe/.github/main/profile/assets/banner.png" alt="NEXUS Game Performance Engine" width="100%">
</p>

<h3 align="center">Measurable. Reversible. Verified.</h3>

<p align="center">
  NEXUS is a Windows performance engine for gamers. It reads your hardware and your games, applies optimizations that can be measured, backs every change up before it touches anything, and proves the result with real frame-time benchmarks.
</p>

<br>

### What NEXUS does

- **Boosts, explained.** Every optimization says what it changes, why it helps, and when it won't. Switch it on, switch it off. No placebo tweaks, ever.
- **Snapshot before, verify after.** The original value is stored before a change and restored with one click. A change that cannot be verified is rolled back on the spot.
- **Benchmarks, not promises.** Frame times come from the graphics kernel while your game runs: average, 1% and 0.1% lows, p95/p99 latency. Before/after comparisons use your numbers, on your PC.
- **Game aware.** Per-game settings, boosts, benchmarks and backups. Fortnite today; more titles follow the same rules.
- **Anti-cheat safe by construction.** Overlay and capture never touch the game process.

### How it's built

| Part | Stack |
| --- | --- |
| Desktop app | Electron · React · TypeScript |
| Performance Core | .NET 8, runs as a separate process; owns every Windows operation |
| Server | Fastify · SQLite: accounts, admin panel, release feed |
| Installer & patcher | NEXUS installs and updates itself. No wizard framework. |

Renderer and core talk over a typed, validated IPC contract; the renderer has no shell or filesystem access.

### Principles

1. If it cannot be measured, it does not ship.
2. If it cannot be reversed, it does not ship.
3. Say why something is unavailable instead of hiding it.
