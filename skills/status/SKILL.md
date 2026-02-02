---
name: status
description: Show threat model status - asset counts, threat distribution, control verification, compliance coverage.
allowed-tools: Read, Glob
---

# Threat Model Status

## Usage

```
/threatmodel:status [--format text|json]
```

## Output

Shows current state of `.threatmodel/`:

```
═══════════════════════════════════════════════════════════
                 THREAT MODEL STATUS
═══════════════════════════════════════════════════════════

Assets: 14 (3 data-stores, 6 services, 3 clients, 2 integrations)
Data Flows: 22 (8 cross trust boundaries)
Trust Boundaries: 5

THREATS: 47 total
  Critical: 5 | High: 12 | Medium: 18 | Low: 12

CONTROLS: 29 required
  Implemented: 18 (62%) | Partial: 7 | Missing: 4

GAPS: 11 total
  Critical: 2 | High: 4

COMPLIANCE:
  OWASP: 82% | SOC2: 88%

TOP PRIORITY:
  1. [CRITICAL] MFA not enforced
  2. [CRITICAL] SQL injection in legacy module
  3. [HIGH] Rate limiting missing
═══════════════════════════════════════════════════════════
```

## Process

1. **Check for `.threatmodel/`** - If missing, show getting started guide
2. **Read state files**: assets.json, threats.json, controls.json, gaps.json, compliance.json
3. **Calculate statistics**: counts, percentages, distributions
4. **Show top priority items**: unmitigated critical/high threats and gaps
5. **Suggest next actions**: based on incomplete phases
