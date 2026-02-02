# claude-threatmodel

**Automated threat modeling for Claude Code**

```
3 skills • 182 lines • ~2K tokens • Minimal context window impact
```

---

## What This Does

This plugin adds automatic security assessment to Claude Code. When you approve an implementation plan, it runs a threat analysis **before** code gets written—not weeks later when fixes are expensive.

**The key insight:** Security review should happen at the moment of intent, not after deployment.

---

## How It Works

### The Trigger

When you're in Claude Code and approve a plan (`ExitPlanMode`), a hook fires automatically. No commands to remember—security assessment just happens.

### The Subagent Architecture

Here's what makes this different: **threat analysis runs in a subagent**, not in your main conversation.

Why does this matter?

| Approach                   | Context Cost    | Analysis Depth |
| -------------------------- | --------------- | -------------- |
| Direct injection           | ~2K+ tokens     | Limited        |
| **Subagent (this plugin)** | **~200 tokens** | **Unlimited**  |

Traditional security tools inject their entire analysis into your conversation, eating up precious context. This plugin spawns a separate agent that:

1. Runs `/threatmodel:quick` in its own context
2. Scans for vulnerabilities
3. Returns only a brief summary to your main conversation
4. Your context window stays clean for actual work

### The Flow

```
You: "Implement user authentication"
     ↓
Claude: [Creates plan]
     ↓
You: [Approve plan] → ExitPlanMode
     ↓
Hook fires → Subagent spawns
     ↓
Subagent: Runs /threatmodel:quick
          • Scans for hardcoded credentials
          • Checks for injection vulnerabilities
          • Looks for XSS patterns
          • Returns: "Found 2 high risks..."
     ↓
Claude: Continues with security-aware implementation
```

### Risk-Based Escalation

The quick assessment calculates a risk level:

| Risk Level        | What Happens                                                |
| ----------------- | ----------------------------------------------------------- |
| **Critical/High** | Recommends `/threatmodel:full` for complete STRIDE analysis |
| **Medium/Low**    | Shares findings, proceeds with implementation               |

You decide whether to run the full analysis. The plugin doesn't block you—it informs you.

---

## The 3 Skills

| Skill                 | Purpose                      | When It Runs                      |
| --------------------- | ---------------------------- | --------------------------------- |
| `/threatmodel:quick`  | Fast risk scan (~30s)        | **Automatic** after plan approval |
| `/threatmodel:full`   | Complete STRIDE + compliance | On-demand or when escalated       |
| `/threatmodel:status` | Current security posture     | Manual check                      |

### What Quick Scans For

- Hardcoded credentials (`password.*=.*["']`)
- Code injection (`eval(`, `exec(`)
- SQL injection (`query.*+.*req.`)
- XSS vulnerabilities (`innerHTML.*=`)

### What Full Analyzes

- **S**poofing - Can attackers impersonate?
- **T**ampering - Can data be modified?
- **R**epudiation - Can actions be denied?
- **I**nformation Disclosure - Can data leak?
- **D**enial of Service - Can service be disrupted?
- **E**levation of Privilege - Can permissions be gained?

Plus: OWASP Top 10 mapping, SOC2/PCI-DSS compliance checks, baseline snapshots for drift detection.

---

## Installation

### Option 1: Plugin Marketplace (Recommended)

```
/plugin marketplace add josemlopez/claude-threatmodel
/plugin install threatmodel@josemlopez
```

Done. The plugin includes skills and hooks—no configuration needed.

### Option 2: Ask Claude

Just say:

```
Install the threat modeling plugin from https://github.com/josemlopez/claude-threatmodel
```

### Verify It Works

```
/threatmodel:status
```

Or: approve any implementation plan and watch the security assessment run.

<details>
<summary><strong>Development: Test Locally</strong></summary>

```bash
git clone https://github.com/josemlopez/claude-threatmodel.git
claude --plugin-dir ./claude-threatmodel
```

Note: `--plugin-dir` loads the plugin for that session only.

</details>

<details>
<summary><strong>Advanced: Manual Installation</strong></summary>

For shorter skill names (`/tm-quick` instead of `/threatmodel:quick`):

```bash
cp -r skills/quick ~/.claude/skills/tm-quick
cp -r skills/full ~/.claude/skills/tm-full
cp -r skills/status ~/.claude/skills/tm-status
```

Then copy hooks from `hooks/hooks.json` to `~/.claude/settings.json`, updating skill references from `/threatmodel:quick` to `/tm-quick`.

</details>

---

## Technical Details

### The Hook

The plugin registers a `PostToolUse` hook that matches `ExitPlanMode`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "ExitPlanMode",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"hookSpecificOutput\": {\"additionalContext\": \"SECURITY CHECKPOINT: Launch a subagent to run /threatmodel:quick...\"}}'"
          }
        ]
      }
    ]
  }
}
```

When triggered, this injects context telling Claude to spawn a Task subagent for the security scan. The subagent does the heavy lifting; your conversation gets only the summary.

### Output Structure (Full Analysis)

```
.threatmodel/
├── config.yaml
├── state/
│   ├── assets.json        # Discovered components
│   ├── dataflows.json     # Data movement
│   ├── threats.json       # STRIDE analysis
│   ├── controls.json      # Security controls found
│   ├── gaps.json          # Missing controls
│   └── compliance.json    # Framework mapping
├── reports/
│   ├── risk-report.md
│   └── executive-summary.md
└── baseline/
    └── snapshot-{date}.json
```

### Size & Footprint

| Component | Lines   | Est. Tokens |
| --------- | ------- | ----------- |
| quick     | 52      | ~600        |
| full      | 77      | ~900        |
| status    | 53      | ~600        |
| **Total** | **182** | **~2,100**  |

Designed to minimize context window consumption.

---

## Project Structure

```
claude-threatmodel/
├── .claude-plugin/
│   └── plugin.json        # Plugin manifest
├── skills/
│   ├── quick/             # /threatmodel:quick
│   ├── full/              # /threatmodel:full
│   └── status/            # /threatmodel:status
├── hooks/
│   └── hooks.json         # Auto-trigger on ExitPlanMode
├── shared/
│   ├── schemas/           # JSON schemas
│   ├── frameworks/        # OWASP, SOC2, PCI-DSS mappings
│   └── templates/         # Config templates
└── README.md
```

---

## FAQ

**Does this slow down my workflow?**
No. The quick scan takes ~30 seconds and runs in a subagent. You can continue working.

**Does this consume my context window?**
Minimal impact (~200 tokens). The analysis runs in a separate subagent context.

**When exactly does it trigger?**
After you approve a plan (`ExitPlanMode`). Not during planning—only after you say "proceed."

**What if I don't have an existing threat model?**
Works without one. Uses heuristic pattern matching for common vulnerabilities.

**Quick vs Full—when to use which?**

|              | `/threatmodel:quick`    | `/threatmodel:full`          |
| ------------ | ----------------------- | ---------------------------- |
| Time         | ~30 seconds             | ~5 minutes                   |
| Trigger      | Automatic               | Manual/escalated             |
| Output       | Summary to conversation | Files in `.threatmodel/`     |
| Scope        | Top threats only        | Complete STRIDE + compliance |
| Context cost | ~200 tokens             | Report saved to disk         |

---

## Why This Matters

| Benefit           | Description                                                 |
| ----------------- | ----------------------------------------------------------- |
| **Frictionless**  | No commands to remember. Security happens automatically.    |
| **Context-Aware** | Claude implements with security knowledge, reducing rework. |
| **Auditable**     | Compliance mapping and drift detection built in.            |

Security doesn't have to be a roadblock. Make it part of the plan.

---

[GitHub](https://github.com/josemlopez/claude-threatmodel) • [Issues](https://github.com/josemlopez/claude-threatmodel/issues) • MIT License
