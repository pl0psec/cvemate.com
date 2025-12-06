# CveMate Risk Scoring System

> **EXPERIMENTAL**
>
> This scoring methodology is experimental and actively evolving. We welcome feedback, suggestions, and recommendations from the security community to improve it.
>
> **[Share your feedback](https://github.com/pl0psec/cvemate.com/issues)**

---

CveMate calculates an **Overall Risk Score** (0-100) for each vulnerability by combining multiple threat indicators into a single actionable metric. This score helps security teams prioritize remediation efforts effectively.

## Overview

The risk score is calculated in two phases:

1. **Base Score Calculation** - A weighted sum of core vulnerability metrics
2. **Conditional Rules** - Non-linear adjustments for specific threat scenarios

---

## Base Score Calculation

The base score is computed as a weighted sum of the following factors:

| Factor | Weight | Description |
|--------|--------|-------------|
| **CVSS** | 40 pts | Base severity score (0-10 normalized to 0-40) |
| **EPSS** | 20 pts | Exploit Prediction Scoring System probability (0-1 normalized to 0-20) |
| **CISA KEV** | 15 pts | Listed in CISA Known Exploited Vulnerabilities catalog |
| **Ransomware** | 15 pts | Confirmed use in ransomware campaigns (must be "Known", not "Unknown") |
| **Public Exploits** | 10 pts | Publicly available exploit code exists |

### Formula

```
Base Score = (CVSS / 10) × 40
           + EPSS × 20
           + (KEV ? 15 : 0)
           + (Ransomware ? 15 : 0)
           + (Exploits ? 10 : 0)
```

### Example

For a vulnerability with:
- CVSS: 9.8
- EPSS: 0.85 (85%)
- In CISA KEV: Yes
- Ransomware: No
- Public Exploits: Yes

```
Base Score = (9.8 / 10) × 40 + 0.85 × 20 + 15 + 0 + 10
           = 39.2 + 17 + 15 + 0 + 10
           = 81.2
```

---

## Conditional Rules

After calculating the base score, conditional rules are applied to handle specific threat scenarios. Rules are evaluated in priority order (highest first), and some rules can override the score entirely.

### Rule Definitions

| Rule | Priority | Trigger Conditions | Effect |
|------|----------|-------------------|--------|
| **AI Emergency** | 115 | AI system detected emergency-level threat | Set Critical, min 95 |
| **Perfect Storm** | 110 | CVSS 10 + EPSS ≥90% + KEV + Ransomware | Set Critical, min 100 |
| **Telegram + Critical CVSS** | 105 | CVSS ≥9 + Telegram activity detected | Set Critical, min 90 |
| **Critical Exploit** | 100 | CVSS 10 + Public exploit exists | Set Critical, min 90 |
| **Active Exploitation** | 90 | KEV + Exploits + CVSS ≥7 | +25 pts, min 60 |
| **KEV + Ransomware** | 85 | In CISA KEV + Known ransomware usage | Set Critical, min 90 |
| **Ransomware Threat** | 80 | Ransomware + CVSS ≥6 | ×1.3 multiplier |
| **KEV Alone** | 75 | Listed in CISA KEV | Set Critical, min 75 |
| **Ransomware Alone** | 75 | Known ransomware campaign usage | Min 50 |
| **High CVSS Alone** | 70 | CVSS ≥9 | Min 70 |

### Rule Effects

- **Set Critical**: Forces the vulnerability to critical level (base score 85)
- **Min Risk**: Ensures score doesn't fall below this value
- **Add Points**: Adds fixed points to the score
- **Multiplier**: Multiplies the current score

---

## Risk Levels

The final score maps to risk levels as follows:

| Score Range | Level | Description |
|-------------|-------|-------------|
| 75-100 | **Critical** | Immediate action required |
| 50-74 | **High** | Prioritize for near-term remediation |
| 25-49 | **Medium** | Schedule for remediation |
| 10-24 | **Low** | Address during regular maintenance |
| 0-9 | **Minimal** | Monitor only |

---

## Data Sources

The scoring system integrates data from multiple authoritative sources:

### Vulnerability Data & CVSS Scores
- **NVD** - National Vulnerability Database
- **CVE Numbering Authorities (CNAs)** - Primary vulnerability reporters including vendors and security researchers
- **Ecosystem databases** - GitHub Advisory Database, OSV, and language-specific security databases
- **Distribution security teams** - Red Hat, Debian, Ubuntu, Microsoft, Oracle, and other vendor security advisories

### Exploit & Threat Intelligence
- **EPSS** - Exploit Prediction Scoring System from FIRST.org
- **CISA KEV** - Known Exploited Vulnerabilities catalog
- **Exploit Databases** - ExploitDB, Metasploit modules
- **Threat Intelligence** - Dark web monitoring, Telegram channels

---

## Interpretation Guidelines

### When to Act Immediately (Score 75+)

- **Listed in CISA KEV** - confirmed active exploitation in the wild
- Known ransomware campaign usage
- Critical CVSS (9.0+) with additional threat indicators
- AI-detected emergency threats

### When to Prioritize (Score 50-74)

- High severity (CVSS 7-8.9) with exploit code available
- Known ransomware usage without KEV listing
- High EPSS percentile indicating likely future exploitation

### When to Schedule (Score 25-49)

- Medium severity without active exploitation
- No public exploits available
- Lower probability of exploitation

### When to Monitor (Score < 25)

- Low severity vulnerabilities
- No indicators of active exploitation
- Theoretical vulnerabilities with no PoC

---

## Limitations

- Scores are based on available public data and may not reflect zero-day threats
- EPSS predictions are probabilistic and updated daily
- Dark web intelligence may have delays in detection
- Scores should be combined with organizational context (asset criticality, exposure)

---

## Future: Risk Profiles

We are exploring the concept of **Risk Profiles** that would allow organizations to re-compute vulnerability scores based on their specific infrastructure and risk tolerance.

### Profiles Under Consideration

| Profile | Description | Use Case |
|---------|-------------|----------|
| **Default** | Balanced assessment for typical networks | General purpose |
| **Paranoid** | Zero tolerance - any threat indicator triggers maximum score | Critical infrastructure, banking, healthcare |
| **Internet-Facing** | Higher weight on remote exploitability and public exploits | Web servers, APIs, public applications |
| **Air-Gapped** | Reduced weight on network-based attacks | Isolated systems, offline infrastructure |

### Example: Paranoid Profile

In a paranoid profile, any of these conditions would result in a score of 100:
- CVSS Critical (9.0+)
- EPSS percentile above 85%
- Listed in CISA KEV
- Public exploit available

### We Need Your Input

This feature is in the brainstorming phase. We want to understand:

- What risk profiles would be most useful for your organization?
- What factors should influence score adjustments for each profile?
- Are there industry-specific profiles that would be valuable?

**[Share your ideas and requirements](https://github.com/pl0psec/cvemate.com/issues)**
