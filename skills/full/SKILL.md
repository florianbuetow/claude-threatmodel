---
name: full
description: Complete threat modeling workflow. Discovers assets, analyzes threats (STRIDE), verifies controls, maps compliance, generates reports.
allowed-tools: Read, Write, Glob, Grep, Bash(mkdir:*), Bash(ls:*)
---

# Full Threat Model

## Usage

```
/threatmodel:full [--docs <path>] [--compliance owasp,soc2,pci-dss]
```

## What It Does

Runs complete threat modeling in 5 phases:

1. **Initialize** - Read docs, discover assets, map data flows, identify trust boundaries
2. **Threats** - Apply STRIDE to each component, build attack trees, calculate risk scores
3. **Verify** - Search code for security controls, collect evidence, document gaps
4. **Compliance** - Map to OWASP Top 10, SOC2, PCI-DSS frameworks
5. **Report** - Generate risk report, executive summary, create baseline

## Output Structure

```
.threatmodel/
├── config.yaml
├── state/
│   ├── assets.json          # Discovered components
│   ├── dataflows.json       # Data movement
│   ├── threats.json         # STRIDE analysis
│   ├── controls.json        # Security controls found
│   ├── gaps.json            # Missing controls
│   └── compliance.json      # Framework mapping
├── diagrams/
│   └── architecture.mmd     # Mermaid diagram
├── reports/
│   ├── risk-report.md
│   └── executive-summary.md
└── baseline/
    └── snapshot-{date}.json
```

## STRIDE Categories

- **S**poofing - Can attacker impersonate?
- **T**ampering - Can data be modified?
- **R**epudiation - Can actions be denied?
- **I**nformation Disclosure - Can data leak?
- **D**enial of Service - Can service be disrupted?
- **E**levation of Privilege - Can permissions be gained?

## Risk Scoring

`Risk = Likelihood (1-5) × Impact (1-5)`

| Score | Level    |
| ----- | -------- |
| 16-25 | Critical |
| 10-15 | High     |
| 5-9   | Medium   |
| 1-4   | Low      |

## Process

1. **Scan documentation** at --docs path for architecture info
2. **Extract assets**: services, data stores, clients, integrations
3. **Map data flows**: source → destination, protocols, encryption
4. **Identify trust boundaries**: network, privilege, environment
5. **Apply STRIDE** to each asset and trust boundary crossing
6. **Search codebase** for control implementations (auth, validation, encryption)
7. **Document gaps** where controls are missing
8. **Map to compliance** frameworks
9. **Generate reports** with prioritized findings
10. **Create baseline** snapshot for drift detection
