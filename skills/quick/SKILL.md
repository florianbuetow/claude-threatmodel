---
name: quick
description: Fast threat assessment (~30s). Returns JSON with risk level, top threats, critical gaps. For hooks and CI/CD.
allowed-tools: Read, Glob, Grep
---

# Quick Threat Assessment

## Usage

```
/threatmodel:quick [--focus <files>] [--format json|text]
```

## Output (JSON)

```json
{
  "risk_level": "high",
  "top_threats": [
    {
      "title": "SQL Injection",
      "severity": "critical",
      "target": "src/api.js:45"
    }
  ],
  "critical_gaps": [
    { "title": "Missing input validation", "severity": "high" }
  ],
  "recommendation": "Address 1 critical threat before proceeding."
}
```

## Process

1. **Check `.threatmodel/state/`** - Use existing threats.json, gaps.json if available
2. **Heuristic scan** (if no state) - Grep for security anti-patterns:
   - `password.*=.*["']` (hardcoded creds)
   - `eval\(|exec\(` (code injection)
   - `query.*\+.*req\.` (SQL injection)
   - `innerHTML.*=` (XSS)
3. **Calculate risk**: critical if any critical threat, high if 3+ high, etc.
4. **Output JSON** to stdout

## Risk Levels

| Condition             | Level    |
| --------------------- | -------- |
| Any critical          | critical |
| 3+ high               | high     |
| 1-2 high or 3+ medium | medium   |
| Otherwise             | low      |

## Escalation to full

After completing the assessment, if risk_level is **critical** or **high**:

- Recommend running `/threatmodel:full` for complete STRIDE analysis
- Explain: "Critical/high risks detected. Recommend running /threatmodel:full for complete threat model, compliance mapping, and remediation guidance."

If risk_level is **medium** or **low**:

- Assessment is sufficient, no escalation needed
- Share findings summary with the main agent
