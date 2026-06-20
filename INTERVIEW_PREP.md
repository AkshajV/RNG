# CSPM Tool — Deep Explainer

> This document explains everything about your project in plain English.
> Read it like a textbook. When you're done, you should be able to answer
> any question an interviewer throws at you about this project — not by
> memorizing, but by actually understanding what you built and why.

---

## ⚠️ What you ACTUALLY need to know vs. what's just reference

**MUST know (they will ask):**
- What a CSPM is (one sentence)
- What each of the 5 frameworks IS in plain English (not their control numbers)
- What a crosswalk is and why it saves money
- What your tool does end-to-end (the pipeline)
- Why you made design choices (read-only, decorator pattern, drift split)
- What drift detection is and why it matters
- The honest weaknesses

**NICE to know (makes you sound sharp but not expected of a fresher):**
- Which services the 30 controls cover (IAM, S3, EC2, etc.)
- One example control you can explain deeply (pick CIS 1.5 — root MFA — it's the easiest)
- What the Capital One breach was (IMDSv1 + SSRF)

**DON'T memorize (nobody expects this, even senior people Google it):**
- Specific NIST control numbers (IA-2(1), AU-2, SC-7...)
- Specific PCI requirement numbers (8.4.2, 10.2.1...)
- Specific SOC 2 criteria numbers (CC6.1, CC7.2...)
- Specific ISO control numbers (A.8.5, A.8.20...)
- The exact API calls each scanner makes

The numbers are in this document as REFERENCE — so if you ever want to
look one up before an interview, it's here. But you'll never be quizzed
on them. What matters is the concepts.

---

## Table of Contents

### Part 1: Your Project (what you built and why)
1. [What even IS a CSPM?](#1-what-even-is-a-cspm)
2. [Why you built this](#2-why-you-built-this)
3. [The 5 frameworks — in plain English](#3-the-5-frameworks--in-plain-english)
4. [How the whole thing works (the pipeline)](#4-how-the-whole-thing-works-the-pipeline)
5. [Architecture — why it's shaped this way](#5-architecture--why-its-shaped-this-way)
6. [Every design decision and WHY](#6-every-design-decision-and-why)
7. [The 30 controls — what they actually check](#7-the-30-controls--what-they-actually-check)
8. [Drift detection — the headliner feature](#8-drift-detection--the-headliner-feature)
9. [The dashboard — what each page does](#9-the-dashboard--what-each-page-does)
10. [Technologies used and WHY each one](#10-technologies-used-and-why-each-one)
11. [Vocabulary glossary](#11-vocabulary-glossary)
12. [Interview questions and answers (project-specific)](#12-interview-questions-and-answers)
13. [Honest weaknesses (what you'd improve)](#13-honest-weaknesses-what-youd-improve)
14. [Your resume bullet points — expanded](#14-your-resume-bullet-points--expanded)

### Part 2: GRC & Security Fundamentals (beyond the project)
15. [The CIA Triad](#15-the-cia-triad--the-foundation-of-everything)
16. [What a GRC analyst actually DOES](#16-what-a-grc-analyst-actually-does-day-to-day)
17. [Risk management — deciding what to fix first](#17-risk-management--how-you-decide-what-to-fix-first)
18. [Common attack patterns your controls prevent](#18-common-attack-patterns-you-should-know)
19. [The audit process — what actually happens](#19-the-audit-process--what-actually-happens)
20. [Security concepts they'll quiz you on](#20-security-concepts-theyll-quiz-you-on)
21. [Compliance vs. Security — they're NOT the same](#21-compliance-vs-security--theyre-not-the-same)
22. [Questions that aren't about your project](#22-questions-they-ask-that-arent-about-your-project)
23. [Behavioral / soft-skill questions](#23-behavioral--soft-skill-questions)
24. [Questions YOU should ask the interviewer](#24-questions-you-should-ask-the-interviewer)
25. [One-page cheat sheet (read 10 min before)](#25-one-page-cheat-sheet-review-this-10-minutes-before-the-interview)

---

## 1. What even IS a CSPM?

**CSPM = Cloud Security Posture Management.**

Break that down:
- **Cloud** — you're checking things in AWS (could be Azure/GCP too, but yours is AWS).
- **Security** — you're checking whether things are configured safely.
- **Posture** — the "stance" of your security. Like posture when sitting — are you slouching (misconfigured) or sitting straight (hardened)?
- **Management** — you're not just checking once. You're tracking it over time, seeing if it gets better or worse.

**In simple terms:** A CSPM scans your cloud account and tells you "hey, your S3 bucket is public" or "your root account doesn't have MFA" — things that are misconfigured and could get you hacked.

**Real-world examples of CSPMs:**
- Wiz (~$12B valuation) — the hottest one right now
- Palo Alto Prisma Cloud
- AWS Security Hub (Amazon's own, built-in)
- Orca Security
- Lacework

These all cost $50k-$500k/year. Yours does the same core thing for free, locally, and you can explain every line.

---

## 2. Why you built this

**The honest answer (which is also the interview answer):**

> "Off-the-shelf CSPMs are black boxes that cost six figures. I wanted to
> understand what's actually happening under the hood — how do you go from
> 'scan an AWS account' to 'here's your compliance posture across five
> frameworks'? Building it myself means I can explain every step of that
> pipeline, which is exactly what a GRC analyst or security engineer needs
> to understand."

**What this demonstrates to an interviewer:**
- You understand the FULL lifecycle: detect → assess → map → score → visualize → track
- You're not just clicking buttons in a GUI tool — you understand the mechanics
- You can work with AWS APIs, compliance frameworks, and data visualization
- You think about the analyst's workflow, not just the technology

---

## 3. The 5 frameworks — in plain English

### What IS a "framework" anyway?

A framework is just a checklist that some authority published saying "if you do all these things, we consider you secure." Different industries and countries have different checklists. That's literally it.

The annoying part: they all say basically the same things ("use MFA," "encrypt data," "log everything") but with different numbering systems and different levels of detail. That's why crosswalks exist — to say "this CIS check for root MFA is the same thing as that NIST identity requirement and that PCI authentication requirement." Same concept, different label.

### The 5 frameworks in your tool:

---

### CIS AWS Foundations Benchmark v2.0

**What:** A checklist of ~200 AWS-specific configuration checks. Written by the Center for Internet Security (a nonprofit). Free to download.

**Who uses it:** Security teams doing cloud hardening. It's the most popular AWS security checklist because it's free, specific ("check this exact setting"), and prescriptive ("here's exactly how to fix it").

**Why it's the base of your scanner:** It's the most granular — it tells you exactly what API to call and what value to check. The other four frameworks are more abstract ("ensure access controls" — okay, but HOW?). CIS gives you the HOW.

**Your tool checks 30 of the ~200 CIS controls.** That's a deliberate scope — the 30 most impactful and automatable ones.

---

### NIST SP 800-53 Rev 5

**What:** The United States government's master catalog of security controls. ~1,000 controls organized into 20 families (AC = Access Control, AU = Audit, IA = Identification/Authentication, etc.).

**Who uses it:**
- Every U.S. federal agency (it's mandatory)
- Any company that wants FedRAMP authorization (to sell to the government)
- Defense contractors (via CMMC, which builds on NIST)

**Why it's in your tool:** It's the "gold standard" reference. If you can map your evidence to NIST, you can speak to any U.S. government compliance requirement.

**Key thing to know (reference only — don't memorize):** NIST controls have a family prefix + number. If you ever see one in a document, here's how to read it:
- AC = Access Control family
- IA = Identification and Authentication family
- AU = Audit family
- SC = System and Communications Protection family
- The number after the dash is the specific control (e.g., IA-2)
- A number in parentheses is an "enhancement" — a sub-requirement (e.g., IA-2(1) = the MFA enhancement)

---

### SOC 2 Trust Services Criteria (2017)

**What:** Published by the AICPA (American Institute of Certified Public Accountants). Defines 5 categories: Security, Availability, Processing Integrity, Confidentiality, Privacy. The controls are called "Common Criteria" (CC).

**Who uses it:** SaaS companies. If you sell software to enterprises, they WILL ask for your SOC 2 Type II report before signing a contract. It's the "cost of doing business" audit.

**Type I vs Type II:**
- Type I = "are the controls designed properly?" (point-in-time check)
- Type II = "have the controls been operating effectively for 6-12 months?" (the real one — shows consistency over time)

**Why it's in your tool:** Nearly every B2B SaaS company needs SOC 2. If you're in GRC at a tech company, you will deal with SOC 2.

**Key thing to know:** CC = Common Criteria (SOC 2's numbering system). You don't need to know specific CC numbers. Just know that SOC 2 covers 5 categories and the Security one (which your tool addresses) is the most commonly audited.

---

### ISO/IEC 27001:2022 Annex A

**What:** An international standard for building an Information Security Management System (ISMS). Annex A is the actual list of controls (93 controls in the 2022 version, organized into 4 themes).

**Who uses it:**
- Multinational companies (it's the global standard, not just U.S.)
- European companies especially
- Any company selling to regulated industries outside the U.S.

**The 4 themes (don't memorize numbers — just know the categories):**
- Organizational controls (policies, roles, responsibilities)
- People controls (screening, training, remote work)
- Physical controls (offices, data centers)
- Technological controls ← this is where all your CIS checks map

**Why it's in your tool:** If a company is global, they probably have ISO 27001. It's the international version of "prove you're secure."

**Good to know:** The 2022 version reorganized everything from the old 2013 version. Your tool uses 2022 (the current one). If an interviewer mentions ISO 27001, just know it exists, it's international, and your checks map to the "technological controls" theme.

---

### PCI DSS v4.0.1

**What:** Payment Card Industry Data Security Standard. Mandated by Visa, Mastercard, Amex, Discover for ANY organization that stores, processes, or transmits cardholder data.

**Who uses it:**
- Banks and payment processors
- E-commerce companies
- Fintechs
- Literally any company that touches credit card numbers

**12 top-level requirements:**
1. Install and maintain network security controls (firewalls)
2. Apply secure configurations (no vendor defaults)
3. Protect stored account data (encryption at rest)
4. Protect cardholder data in transit (TLS)
5. Protect against malicious software (anti-malware)
6. Develop and maintain secure systems (patching, SDLC)
7. Restrict access by business need-to-know (least privilege)
8. Identify users and authenticate access (MFA, passwords)
9. Restrict physical access to cardholder data
10. Log and monitor all access (audit trails)
11. Test security systems and processes (vuln scanning, pen testing)
12. Support information security with policies and programs

**Why it's in your tool:** Companies like Visa, Mastercard, JPMorgan, Stripe — they all need PCI. It's one of the most financially punishing frameworks (non-compliance = fines from the card brands + liability for breaches).

**QSA = Qualified Security Assessor** — the person who audits PCI compliance. Like a CPA but for credit card security.

---

### Why a crosswalk is the most valuable thing in GRC

Imagine you're a GRC analyst and your company has SOC 2, ISO 27001, AND PCI DSS audits this year. Without a crosswalk, you collect evidence three separate times for "do you have MFA?" With a crosswalk, you collect it ONCE and say:

> "Here's our MFA evidence. It satisfies SOC 2 CC6.1, ISO A.8.5, PCI 8.4.2, and NIST IA-2(1)."

**One piece of evidence, four audits satisfied.** That's the economics of a crosswalk. Your tool does this automatically for 30 controls × 5 frameworks.

---

## 4. How the whole thing works (the pipeline)

Here's what happens end to end when you type `.venv/bin/python scripts/run_scan.py`:

```
Step 1: AUTHENTICATE
  └── Uses your AWS credentials (via AWS_PROFILE environment variable)
  └── No passwords stored in code — uses the standard AWS credential chain

Step 2: DISCOVER CHECKS
  └── The orchestrator auto-discovers all @check-decorated methods
  └── Finds 30 checks across 8 scanner classes (IAM, S3, EC2, CloudTrail, EBS, RDS, KMS, VPC)

Step 3: RUN EACH CHECK (with fault isolation)
  └── For each check:
      ├── Call read-only AWS APIs (describe_*, get_*, list_*)
      ├── For each resource found:
      │   ├── Evaluate: does this resource pass or fail this control?
      │   └── Emit a Finding (PASS/FAIL/MANUAL/ERROR + evidence)
      └── If the check throws an error → catch it, emit ERROR, move on
          (one broken check doesn't kill the whole scan)

Step 4: SCORE
  └── Count PASS/FAIL per framework using the crosswalk
  └── Compute pass percentages

Step 5: PERSIST
  └── Write everything to data/scans/scan_<timestamp>.json
  └── ~4MB file, ~9,500 findings on a typical dev account

Step 6: DASHBOARD (separate step — user launches Streamlit)
  └── Reads the JSON
  └── Enriches findings with title/severity/framework from framework_mappings.py
  └── Renders 5 pages of interactive visualizations
```

**The key insight:** Steps 1-5 are the "scanner." Step 6 is the "dashboard." They're completely decoupled — you could swap the dashboard for a CLI report, a PDF generator, or a Slack bot without touching the scanner.

---

## 5. Architecture — why it's shaped this way

```
cspm-tool/
├── scanners/           ← The "WHAT to check" layer
│   ├── base_scanner.py     (shared infrastructure: @check decorator, Finding creation)
│   ├── iam_scanner.py      (12 IAM controls)
│   ├── s3_scanner.py       (3 S3 controls)
│   ├── ec2_scanner.py      (4 EC2 network controls: SSH/RDP/default SG/IMDSv2)
│   ├── cloudtrail_scanner.py (5 logging controls)
│   ├── ebs_scanner.py      (1 EBS encryption control)
│   ├── rds_scanner.py      (3 RDS controls)
│   ├── kms_scanner.py      (1 KMS key rotation control)
│   └── vpc_scanner.py      (1 VPC flow logs control)
│
├── engine/             ← The "HOW to run" layer
│   ├── finding.py          (dataclasses: Finding, ScanResult, etc.)
│   ├── scan_orchestrator.py (auto-discovers checks, runs them, handles errors)
│   ├── scoring.py          (counts PASS/FAIL per framework)
│   └── drift_detector.py   (compares two scans → regressed/resolved/persisting)
│
├── frameworks/         ← The "WHAT DOES IT MEAN" layer
│   └── framework_mappings.py (30 controls × 5 frameworks — titles, severities, crosswalk IDs)
│
├── remediation/        ← The "HOW TO FIX IT" layer
│   └── remediation_library.py (30 entries: risk explanation + Terraform/CLI fix snippet)
│
├── storage/            ← The "WHERE TO PUT IT" layer
│   └── json_store.py       (read/write scan JSONs to data/scans/)
│
├── dashboard/          ← The "HOW TO SEE IT" layer
│   ├── app.py              (Streamlit entry point + scan picker)
│   ├── data_loader.py      (ALL data logic — pages are presentation only)
│   └── pages/
│       ├── 01_Overview.py
│       ├── 02_Frameworks.py
│       ├── 03_Controls.py
│       ├── 04_Resources.py
│       └── 05_Drift.py
│
├── tests/              ← The "DID ANYTHING BREAK" layer
│   └── test_remediation_coverage.py (AST-based regression test)
│
├── scripts/            ← The "HOW TO RUN IT" layer
│   └── run_scan.py
│
└── data/scans/         ← The "OUTPUT" (generated, not checked in)
    └── scan_<timestamp>.json
```

### Why this layering?

**Separation of concerns.** Each folder has ONE job. If you need to:
- Add a new check → touch `scanners/` only
- Fix a framework mapping → touch `frameworks/` only
- Change how the dashboard looks → touch `dashboard/pages/` only
- Change how data is stored → touch `storage/` only

**No folder imports "up" the stack.** Scanners don't know about the dashboard. The engine doesn't know about Streamlit. This means you could rip out the entire dashboard and replace it with a CLI tool or an API, and the scanner/engine/storage would work unchanged.

---

## 6. Every design decision and WHY

### Why read-only? Why never mutate AWS?

**Decision:** The scanner only calls `describe_*`, `get_*`, `list_*` APIs. Never `create_*`, `update_*`, `delete_*`, `put_*`.

**Why:**
- **Safety.** If your tool can modify infrastructure, one bug = production outage. Security teams will never approve running a tool that can change things.
- **Trust.** "It's read-only by design" is one sentence that makes any security team relax.
- **Principle of least privilege.** The scanner only needs ReadOnlyAccess. It doesn't need admin permissions.

**What you rejected:** Auto-remediation (tool that finds AND fixes). Impressive in demos, terrifying in production. If your auto-fix has a bug and strips rules from a production security group, you just caused a security incident.

**Interview phrasing:** "A scanner that also remediates is a scanner that can cause outages. I separated detection from remediation intentionally — the tool tells you what's wrong and gives you a fix template, but the human decides when and whether to apply it."

---

### Why the @check decorator pattern?

**Decision:** Each scanner check is a method decorated with `@BaseScanner.check("CIS X.Y")`. The orchestrator automatically finds and runs all decorated methods.

**Why:**
- **Zero boilerplate to add a new check.** Write the method, put the decorator on it, done. The orchestrator auto-discovers it.
- **Fault isolation.** The orchestrator wraps each check in try/except. If one check crashes (API timeout, permission error), the other 29 still run.
- **Self-documenting.** You can see every control a scanner implements by looking at its decorators.

**What you rejected:** A manual list of checks you have to update every time you add one. Humans forget. The decorator makes it impossible to add a check without registering it.

**The pattern in code:**
```python
@BaseScanner.check("CIS 1.5")
def check_root_mfa(self) -> list[Finding]:
    """Ensures MFA is enabled on the root account."""
    # ... check logic ...
```

**Interview phrasing:** "I used a decorator-based registry so adding a new security check is one method + one decorator. The orchestrator auto-discovers it, wraps it in fault isolation, and includes it in the next scan. No registration step to forget."

---

### Why JSON for storage (not a database)?

**Decision:** Scan results are stored as JSON files in `data/scans/`.

**Why:**
- **Simplicity.** No database server to run, no schema migrations, no connection strings.
- **Portable.** The entire scan history is a folder of files. Copy the folder = copy the history.
- **Human-readable.** You can open any scan in a text editor and see exactly what happened.
- **Good enough for the scale.** A single account produces ~4MB per scan. Even 100 scans = 400MB. You'd need thousands before JSON becomes a bottleneck.

**What you'd switch to at scale:** SQLite (single-file database, no server needed) once you have 50+ scans and want to query across them without loading everything into memory.

**Interview phrasing:** "JSON is the right choice for a single-user local tool. If this became a team tool with hundreds of scans, I'd switch to SQLite — still no server, but gives you query capabilities across scans without loading them all into memory."

---

### Why Streamlit (not React, Flask, etc.)?

**Decision:** The dashboard is built with Streamlit.

**Why:**
- **Python-native.** The scanner is Python. The data processing is pandas. Streamlit lets you build the UI in the same language — no JavaScript, no frontend framework, no separate build step.
- **Time-to-value.** A Streamlit page is 50-100 lines of Python. A React equivalent would be 500+ lines across multiple files with a build system.
- **Data-science-friendly.** Streamlit is designed for DataFrames. `st.dataframe(df)` and you have a sortable, searchable table.
- **Good enough for a portfolio piece.** You're not shipping a SaaS product. You're demonstrating that you can build a full pipeline. The UI just needs to be functional and clear.

**What you'd switch to for production:** React + a proper API backend. Streamlit doesn't scale for multi-user concurrent access. But for a local portfolio tool, it's perfect.

**Interview phrasing:** "Streamlit let me build a functional dashboard in Python without context-switching to JavaScript. The scanner and the dashboard share the same language, same data types, same mental model. For a portfolio piece that's meant to demonstrate the pipeline — not win a UI design award — it's the right tool."

---

### Why one `data_loader.py` instead of logic in each page?

**Decision:** ALL data loading, transformation, joining, and computation lives in `dashboard/data_loader.py`. The page files (01-05) are presentation ONLY.

**Why:**
- **DRY (Don't Repeat Yourself).** Multiple pages need the same data (findings DataFrame, control summary, severity breakdown). If each page did its own loading, you'd have 5 slightly different copies of the same logic.
- **One place to fix bugs.** When the `pass_percentage` computation was wrong, I fixed it in one place and all 5 pages were fixed.
- **Testable.** You can test `data_loader.py` without launching Streamlit. You can't easily test logic embedded inside Streamlit page renders.

**The pattern:**
```python
# data_loader.py — does all the thinking
def findings_dataframe(filepath) -> pd.DataFrame:
    # loads, enriches, joins, returns

# 03_Controls.py — just renders
df = findings_dataframe(filepath)
st.dataframe(df)  # presentation only
```

**Interview phrasing:** "I separated data logic from presentation. `data_loader.py` is the single layer all 5 pages read from. If every page did its own JSON parsing and DataFrame munging, I'd have five slightly-different copies of the same logic and five places to fix the same bug."

---

### Why frozen dataclasses?

**Decision:** Core data structures (`DriftRow`, `DriftReport`, `Remediation`, `ControlMapping`) are frozen dataclasses (`@dataclass(frozen=True)`).

**Why:**
- **Immutability.** Once created, they can't be accidentally modified. A bug in one function can't corrupt data that another function depends on.
- **Hashable.** Frozen dataclasses can be used in sets and as dict keys. Mutable ones can't.
- **Intent-communicating.** When a developer sees `frozen=True`, they immediately know "this is a value object — compute it once, read it many times, never modify it."

**What you rejected:** Regular dicts. Dicts have no schema — you'd get KeyError bugs at runtime when someone misspells a key. Dataclasses catch that at definition time.

---

### Why evidence compaction (PASS = ARN only, FAIL = full evidence)?

**Decision:** PASS findings store only the resource ARN. FAIL findings store full evidence (the actual API response showing what's wrong).

**Why:**
- **Scale.** A single scan produces ~9,500 findings. ~98% are PASSes. If every PASS stored the full API response, the JSON would be 50+ MB instead of 4 MB.
- **Value asymmetry.** Nobody reads the evidence of a PASS — "it's fine" is all you need. But the evidence of a FAIL is critical — the analyst needs to see WHY it failed to know how to fix it.

**Interview phrasing:** "PASS findings are boring — you just need to know the resource was evaluated. FAIL findings are actionable — the analyst needs the evidence to understand what's wrong. Compacting PASSes down to just an ARN keeps scan files tractable without losing any analytical value."

---

### Why exclude MANUAL and ERROR from drift?

**Decision:** When comparing two scans, MANUAL and ERROR findings are completely ignored.

**Why:**
- **MANUAL** = "there were no resources to evaluate." This is the N/A pattern. Example: CIS 1.20 checks IAM Access Analyzer, but if no analyzer exists, you can't PASS or FAIL — you emit MANUAL. If one scan finds 0 analyzers and the next finds 0 analyzers, that's not "drift" — nothing changed.
- **ERROR** = a transient scan failure (API timeout, permission error). If scan A gets an error on CIS 3.1 because CloudTrail throttled you, and scan B doesn't, that's not real posture change — it's just noise.
- **Including them = ghost drift.** You'd get false "changes" between every scan pair, drowning out the real signal.

**Interview phrasing:** "MANUAL means 'couldn't evaluate,' ERROR means 'check crashed.' Neither represents real posture change. If I included them, the drift page would show phantom regressions every run — an analyst would learn to ignore it, which defeats the purpose."

---

### Why split "regressed" from "new-resource fail"?

**Decision:** New FAILs are split into two buckets:
- **Regressed** = the resource existed before and PASSED, now it FAILS
- **New-resource fail** = the resource didn't exist in the baseline at all

**Why:**
- **Different root causes.** Regression means something CHANGED on existing infrastructure (someone edited a security group rule, removed MFA, disabled encryption). New-resource means someone provisioned something new that was never configured correctly.
- **Different fixes.** Regression → "who changed it? Check the CloudTrail logs. Was it IaC drift or a manual hand-edit?" New-resource → "our provisioning template/process is missing this control. Fix the template."
- **Different urgency.** Regression is scarier — something that WAS secure is now NOT. New-resource is bad but expected — new things often need hardening.

**Interview phrasing:** "Engineers triage regressions differently from new-resource failures. A regression means existing infrastructure degraded — probably a manual change or IaC drift. A new-resource failure means the provisioning process has a gap. Different root cause, different fix, different urgency."

---

## 7. The 30 controls — what they actually check

> **You don't need to memorize all 30.** This section is a REFERENCE.
> Skim it so you recognize what each one does if it comes up, but the
> only thing you need to be able to do in an interview is:
> 1. Name the SERVICE AREAS you cover (IAM, S3, EBS, RDS, CloudTrail, KMS, VPC, EC2)
> 2. Explain ONE control deeply if asked (pick root MFA — it's the easiest)
> 3. Explain the CATEGORIES of things you check (identity, storage encryption, logging, network access)

### IAM (Identity & Access Management) — 12 controls

| Control | What it checks | Why it matters |
|---|---|---|
| CIS 1.3 | Are there IAM users with unused credentials (>90 days)? | Dormant credentials are free attack surface. If nobody's using them, an attacker can. |
| CIS 1.4 | Are there access keys that haven't been rotated (>90 days)? | Old keys = old compromises still work. Rotation limits the blast radius of a leak. |
| CIS 1.5 | Does the root account have MFA enabled? | Root = god-mode. No MFA on root = one phished password away from total account takeover. |
| CIS 1.6 | Is hardware MFA used on root (vs virtual/SMS)? | SMS can be SIM-swapped. Virtual MFA can be copied if the seed is stolen. Hardware tokens (YubiKey) can't be remotely cloned. |
| CIS 1.7 | Is root actually being used (access key / console login)? | Root should NEVER be used for daily work. Any root activity is suspicious. |
| CIS 1.8 | Is the password policy strong enough? | Weak passwords = brute-forceable. CIS wants 14+ chars, mixed case, symbols, no reuse. |
| CIS 1.10 | Is MFA enabled for every IAM user with console access? | Any user without MFA = phishing target. One compromised user = foothold. |
| CIS 1.13 | Does any user have more than one active access key? | Two active keys = one is probably forgotten. Forgotten keys get leaked in git commits. |
| CIS 1.14 | Are access keys rotated within 90 days? | Same as 1.4 but specifically for the rotation schedule. |
| CIS 1.16 | Are any policies attached directly to users (vs groups/roles)? | Direct attachment = ungovernable. Groups let you manage permissions at scale; direct policies create snowflakes. |
| CIS 1.17 | Is the support role configured? | AWS Support needs a role to troubleshoot. If it doesn't exist, you can't get help during an incident. |
| CIS 1.20 | Is IAM Access Analyzer enabled? | Access Analyzer tells you what resources are exposed externally. Without it, you're blind to public buckets, cross-account roles, etc. |

### S3 (Storage) — 3 controls

| Control | What it checks | Why it matters |
|---|---|---|
| CIS 2.1.1 | Is server-side encryption enabled on all buckets? | Unencrypted S3 = data readable by anyone who accesses the underlying storage layer (defense-in-depth). |
| CIS 2.1.2 | Is server access logging enabled on all buckets? | Without access logs, you can't tell who read/wrote what in your bucket. Essential for forensics after a breach. |
| CIS 2.1.5 | Is S3 Block Public Access enabled on all buckets? | Public S3 buckets are the #1 cause of cloud data breaches. Capital One, Twitch, dozens more. Block Public Access is the kill switch. |

### EBS (Block Storage) — 1 control

| Control | What it checks | Why it matters |
|---|---|---|
| CIS 2.2.1 | Is EBS volume encryption enabled by default? | EBS volumes are your EC2 hard drives. Unencrypted = if someone steals a snapshot, they can read everything. |

### RDS (Databases) — 3 controls

| Control | What it checks | Why it matters |
|---|---|---|
| CIS 2.3.1 | Is storage encryption enabled on RDS instances? | Your database has your most sensitive data. Unencrypted storage = readable if the underlying disk is compromised. |
| CIS 2.3.2 | Is auto minor version upgrade enabled? | Minor versions contain security patches. Without auto-upgrade, you fall behind on patches and become vulnerable to known CVEs. |
| CIS 2.3.3 | Are RDS instances NOT publicly accessible? | A database exposed to the internet is an immediate target. All DB access should route through your VPC. |

### CloudTrail (Audit Logging) — 5 controls

| Control | What it checks | Why it matters |
|---|---|---|
| CIS 3.1 | Is CloudTrail enabled in all regions? | CloudTrail = your audit log. If it's only in one region, an attacker can operate from another region invisibly. |
| CIS 3.2 | Is log file validation enabled? | Without validation, an attacker who gets access can delete/modify their tracks and you'd never know. |
| CIS 3.3 | Is the CloudTrail S3 bucket public? | If your audit logs are public, an attacker can READ them to learn your infrastructure... or DELETE them to cover tracks. |
| CIS 3.4 | Is CloudTrail integrated with CloudWatch Logs? | CloudWatch = real-time alerting. Without it, you only find out about breaches when you manually review logs (which nobody does). |
| CIS 3.7 | Is KMS encryption enabled on CloudTrail logs? | Encrypting logs means even someone with S3 access can't read them without the KMS key — defense in depth. |

### KMS (Encryption Key Management) — 1 control

| Control | What it checks | Why it matters |
|---|---|---|
| CIS 3.8 | Is annual key rotation enabled on CMKs? | Key rotation limits the blast radius of a compromised key. If you rotate annually and a key leaks, only one year of data is at risk. |

### VPC/EC2 (Network) — 4 controls

| Control | What it checks | Why it matters |
|---|---|---|
| CIS 5.1 | Are VPC flow logs enabled? | Flow logs = your network traffic audit trail. Without them, you can't investigate "who talked to whom" during an incident. |
| CIS 5.2 | Does any security group allow SSH (port 22) from 0.0.0.0/0? | SSH open to the world = brute-force attempts within seconds. The most-scanned port on the internet. |
| CIS 5.3 | Does any security group allow RDP (port 3389) from 0.0.0.0/0? | RDP open to the world = ransomware entry point. Every major ransomware family uses exposed RDP. |
| CIS 5.4 | Does the default security group have any rules? | The default SG gets attached to anything without an explicit SG. If it has rules, you're granting permissions you didn't intend. |
| CIS 5.6 | Do EC2 instances require IMDSv2? | IMDSv1 = the Capital One breach. An SSRF attack can steal instance credentials. IMDSv2 adds a token requirement that blocks this attack. |

---

## 8. Drift detection — the headliner feature

### Why drift is the best interview talking point

Most CSPMs only show a snapshot: "here's your posture right now." Drift answers the harder question: "is your security getting better or worse over time?"

That's the question a CISO asks in a weekly security meeting. That's the question auditors ask during SOC 2 Type II (which specifically requires you to demonstrate controls operating CONSISTENTLY over time, not just at one point).

### How it works

**Identity key:** `(control_id, resource_arn)`

This means: "for THIS specific check, against THIS specific resource — what happened?"

**Four buckets:**

| Bucket | What it means | Example |
|---|---|---|
| **Regressed** | Was PASS → now FAIL | Someone disabled MFA on root. Used to be fine, now it's broken. |
| **New-resource fail** | Resource didn't exist before → now it exists and it's failing | Someone launched an EC2 instance without IMDSv2. New resource, born misconfigured. |
| **Resolved** | Was FAIL → now PASS (or resource is gone) | Someone enabled encryption on that RDS instance. Problem fixed. |
| **Persisting** | FAIL in both scans | That KMS key still doesn't have rotation enabled. Known issue, still in the backlog. |

### Why this matters in interviews

> "The drift page turns the tool from a scanner into a security program.
> A scanner gives you a snapshot. Drift gives you a narrative: we had 172
> persisting failures last week, 1 regression (root MFA flapped), 6 new
> resources that were born misconfigured, and 1 resolution. That's the
> weekly security briefing slide — generated automatically from two scan
> files."

---

## 9. The dashboard — what each page does

### Home (app.py)
**What:** A scan picker (dropdown of all scan files) and a project blurb.
**Why it exists:** Every other page reads from `session_state["scan_filepath"]`. Home is where you pick which scan all other pages render.

### Overview (01_Overview.py)
**What:** The "executive summary" page. Shows:
- Total findings, pass/fail/manual/error counts
- Pass % per framework (the 5 "framework cards")
- Severity breakdown of failures (CRITICAL/HIGH/MEDIUM/LOW)
- Top failing controls

**Why it exists:** A CISO has 30 seconds. This page answers "are we in good shape?" at a glance.

### Frameworks (02_Frameworks.py)
**What:** Pick a framework (CIS/NIST/SOC 2/ISO/PCI), see how each CIS control maps to it.
**Why it exists:** The crosswalk is the intellectual core of the tool. This page makes it visible: "which PCI requirements am I satisfying with this CIS scan?"

### Controls (03_Controls.py)
**What:** All 30 controls in a table (filterable by severity, fails-only, text search). Drill into any control to see:
- Underlying findings (PASS/FAIL per resource)
- Raw evidence JSON
- Remediation snippet (Terraform/CLI to fix it)

**Why it exists:** This is the analyst's daily driver. "Which controls are failing? Show me the evidence. How do I fix it?"

### Resources (04_Resources.py)
**What:** Same data, different angle. Instead of "which controls are failing?" it asks "which RESOURCES have the most issues?" Drill into a resource to see every finding against it.
**Why it exists:** Engineers remediate one resource at a time (one IAM user, one S3 bucket, one EC2 instance). This page matches how they actually work.

### Drift (05_Drift.py)
**What:** Pick two scans (baseline + current). See what changed: regressions, new-resource failures, resolutions, persisting backlog.
**Why it exists:** The weekly security review. "Did we get better or worse since last scan?"

---

## 10. Technologies used and WHY each one

| Technology | What it is | Why I used it | What I'd use at scale |
|---|---|---|---|
| **Python 3.13** | Programming language | Industry standard for security tooling and data analysis. boto3 (AWS SDK) is Python-native. | Same — Python is the right choice for this domain. |
| **boto3** | AWS SDK for Python | The official way to call AWS APIs from Python. Every scanner uses it. | Same. |
| **pandas** | Data manipulation library | Makes groupby/pivot/filter operations trivial. The dashboard lives on DataFrames. | Same for analysis; might add SQL for queries. |
| **Streamlit** | Python web framework for data apps | Fastest path from "I have a DataFrame" to "I have an interactive dashboard." No JS needed. | React + FastAPI for multi-user production. |
| **dataclasses** | Python's built-in structured data | Type-safe, self-documenting, hashable (when frozen). Better than raw dicts. | Same — or Pydantic if input validation matters. |
| **JSON** | Data storage format | Human-readable, no server needed, good enough for single-user local tool. | SQLite or PostgreSQL for query capabilities. |
| **AST module** | Python's built-in code parser | Powers the regression test without importing boto3. Reads scanner code statically. | Same. |
| **logging** | Python's built-in logging | Standard practice. Every function logs entry/exit for debugging. | Same, probably with structured JSON logging. |

---

## 11. Vocabulary glossary

| Term | Plain English | Example |
|---|---|---|
| **Posture** | How secure your configuration currently is | "Our posture improved — failures dropped from 200 to 180." |
| **Control** | A single security requirement | "Root account must have MFA" is one control. |
| **Finding** | The result of checking one control against one resource | "Root MFA check against the root account → PASS." |
| **Evidence** | The raw proof of why something passed or failed | The actual API response showing MFA device is attached. |
| **Crosswalk** | A mapping between frameworks — "this CIS check also satisfies this NIST requirement" | One check (root MFA) satisfies requirements in all 5 frameworks at once. |
| **Drift** | Change in posture between two scans | "Root MFA regressed from PASS to FAIL since last week." |
| **Remediation** | The fix for a finding | A Terraform snippet or AWS CLI command that resolves the issue. |
| **Benchmark** | A published standard of best practices | CIS AWS Foundations Benchmark v2.0. |
| **Hardening** | The process of securing configuration | "We hardened the account by enabling MFA everywhere." |
| **Least privilege** | Give only the minimum permissions needed | Don't give admin access if someone only needs read access. |
| **Defense in depth** | Multiple layers of security | Encrypt AND restrict access AND log — not just one. |
| **Blast radius** | How much damage a compromise causes | Rotating keys limits blast radius — only recent data is at risk. |
| **Attack surface** | Everything an attacker could target | Open ports, public buckets, unused credentials = attack surface. |
| **SG** | Security Group — AWS's virtual firewall | Controls what traffic can reach your EC2 instances. |
| **VPC** | Virtual Private Cloud — your private network in AWS | Your isolated network. Resources inside can talk to each other. |
| **ARN** | Amazon Resource Name — unique ID for any AWS resource | Like a social security number but for cloud resources. |
| **IMDSv2** | Instance Metadata Service version 2 | The secure way for EC2 instances to get their own credentials. v1 was exploited in the Capital One breach. |
| **SSRF** | Server-Side Request Forgery | Attack where you trick a server into making requests on your behalf (how Capital One was breached via IMDSv1). |
| **IaC** | Infrastructure as Code (Terraform, CloudFormation) | Managing infrastructure through code files, not clicking in the console. |
| **CMK** | Customer Managed Key (in KMS) | An encryption key YOU control the lifecycle of (vs AWS managing it). |
| **QSA** | Qualified Security Assessor | The person who audits PCI compliance. Like a CPA but for credit card security. |
| **FedRAMP** | Federal Risk and Authorization Management Program | Uses NIST 800-53 as its backbone. Required to sell cloud services to the US government. |

---

## 12. Interview questions and answers

### "Tell me about this project."

> "I built a read-only Cloud Security Posture Management scanner. It evaluates
> an AWS account against 30 controls from the CIS AWS Foundations Benchmark,
> maps each finding to five compliance frameworks — CIS, NIST 800-53, SOC 2,
> ISO 27001, and PCI DSS — and surfaces the results through an interactive
> dashboard. It also tracks drift between scans so you can see whether posture
> is improving or degrading over time.
>
> The key design choice is that it's read-only — it describes posture, it never
> changes it. And the crosswalk means one scan produces evidence usable across
> five different compliance audits."

---

### "Walk me through what happens when a scan runs."

> "The orchestrator auto-discovers all check methods via a decorator registry,
> then iterates through them. Each check calls read-only AWS APIs — describe,
> get, list — and emits a Finding per resource: PASS, FAIL, MANUAL, or ERROR.
> Each check is wrapped in fault isolation, so if one check crashes from an API
> timeout, the other 29 still complete. After all checks run, the scoring engine
> uses the framework crosswalk to compute pass percentages per framework, and
> the whole result is written to a JSON file."

---

### "Why not just use AWS Security Hub?"

> "Three reasons. First, Security Hub is a managed service — you get pass/fail
> but not visibility into how it decided. For a GRC analyst preparing for an
> audit, you need to show the evidence chain, not just the verdict. Second,
> Security Hub locks you into AWS's mapping choices. My crosswalk is explicit
> and auditable — I can explain exactly why a particular CIS check maps to a
> particular NIST or PCI requirement. Third, building it myself means I
> actually understand the mechanics of the pipeline, not just the output."

---

### "What's the hardest part of this project?"

Pick one depending on what the role cares about:

**For a security role:**
> "The IAM credential report. AWS rate-limits generation to once per 4 hours,
> and four of my controls need it. I cache one fetch per scan so I don't hit
> the limit. If I didn't, the second control to request it would get throttled
> and the scan would look broken."

**For a GRC role:**
> "The framework crosswalk. Mappings are subjective — one CIS control partially
> satisfies 5-10 controls in NIST. I had to research each framework's source
> document and pick the 2-4 most directly applicable mappings per control. I
> cite my sources so an auditor or QSA can verify."

**For an engineering role:**
> "Drift detection. You can't just diff two JSON files because every scan
> generates fresh UUIDs. I had to define a stable identity key — control_id
> plus resource_arn — then categorize changes into four buckets: regressions,
> new-resource failures, resolutions, and persisting backlog. The
> regressed-vs-new-resource split is important because they have different
> root causes and different fixes."

---

### "How would you handle multi-region?"

> "Currently regional services only scan the active session region. To go
> multi-region, the orchestrator would iterate ec2.describe_regions(), create
> a session per region, and run regional scanners in each. The global services
> — IAM, S3, CloudTrail — already return account-wide results regardless of
> region, so they'd only run once. The main challenge would be deduplication:
> some resources (like S3 buckets) are global but reported in a specific region."

---

### "What would you do differently?"

> "Three things. First, I'd add a proper database — even SQLite — instead of
> flat JSON files. JSON is fine for 10 scans but doesn't scale for trending
> across hundreds. Second, I'd add filtering to the scan itself — in a large
> account you might only care about one VPC or one team's resources. Third,
> I'd add multi-region from day one rather than retrofitting it."

---

### "Is this production-ready?"

> "No, and intentionally not. It's designed to demonstrate the full pipeline:
> discovery, evaluation, evidence collection, framework mapping, scoring,
> visualization, and drift detection. Making it production-ready would mean
> adding authentication, multi-tenancy, a real database, CI/CD, scheduling,
> and probably rewriting the scanner to run as a Lambda on a cron. That's a
> team project — this is a portfolio piece that proves I understand all the
> moving parts."

---

### "What did you learn?"

> "Two big things. First, compliance frameworks look very different on paper
> but say remarkably similar things — the value of a crosswalk is making that
> overlap explicit. Second, the detective control pipeline (find → assess →
> score → report → track) is more complex than it looks. Getting drift right
> — distinguishing regressions from new resources, excluding noise from
> MANUAL/ERROR — required thinking carefully about what 'change' actually means
> in a security context."

---

### "Tell me about the PCI DSS mapping."

> "PCI DSS v4.0.1 has 12 top-level requirements — things like 'identify and
> authenticate access,' 'log and monitor everything,' 'install network security
> controls.' I mapped each CIS control to whichever PCI requirement it most
> directly provides evidence for. So MFA checks map to the authentication
> requirement, CloudTrail checks map to the logging requirement, security group
> checks map to the network controls requirement. The mapping is conservative —
> I only listed requirements where the CIS check directly provides evidence,
> not tangentially related ones. And I cited my source (v4.0.1, which is
> current — the old v3.2.1 retired in 2024)."

---

### "How do you handle false positives?"

> "The MANUAL status handles the 'can't tell' case — when there are no
> resources to evaluate, I emit MANUAL rather than a misleading PASS. For
> actual false positives in FAIL findings, the evidence JSON is attached so
> the analyst can verify. The tool doesn't suppress findings — it shows you
> everything and lets you decide. In a production version, I'd add an
> 'acknowledged' status where an analyst can mark a finding as accepted risk
> with a justification note."

---

### "What does your regression test do?"

> "It AST-parses the scanner files to extract every control_id string literal
> without importing boto3 — so it runs without AWS credentials. Then it asserts
> that set is EQUAL to the framework_mappings keys AND the remediation library
> keys. Equal, not subset — extra entries are drift too. This caught a real bug:
> when the scanner added CIS 5.2 and 5.3 as separate checks, the test flagged
> that the mappings file was missing them."

---

## 13. Honest weaknesses (what you'd improve)

Interviewers LOVE this question. Having a real answer (not "nothing, it's perfect") is a green flag.

1. **Single-region only for EC2/VPC/RDS/EBS.** An attacker would just operate from a different region. Multi-region is the biggest correctness gap.

2. **No alerting.** The tool tells you what's wrong but doesn't proactively notify you. In production you'd want a Slack/email alert when a CRITICAL regression appears.

3. **No acknowledged/accepted-risk workflow.** Sometimes a finding is "yes I know, we accept that risk." Right now there's no way to mark that — it shows up as a persisting failure forever.

4. **JSON doesn't scale.** Fine for 10 scans, painful for trending over 100. SQLite would solve this without adding server complexity.

5. **No RBAC on the dashboard.** Anyone who can access port 8501 sees everything. In production you'd want role-based access (security team sees everything, app team sees only their resources).

6. **Crosswalk is manually maintained.** If NIST releases Rev 6 or CIS releases v3.0, someone has to manually update the mappings. Ideally you'd import from a machine-readable crosswalk feed (OSCAL format, if available).

---

## 14. Your resume bullet points — expanded

Here's how each bullet translates into interview talking points:

---

**Bullet: "Built a read-only CSPM scanner evaluating 30 CIS AWS Foundations Benchmark v2.0 controls"**

What this means to an interviewer:
- You know what CIS is and which version is current (v2.0)
- You know how to call AWS APIs safely (read-only)
- You can scope a project (30 controls, not 200)
- "Read-only" signals security awareness

Follow-up they'll ask: "Why 30 and not all 200?"
> "Scope management. 30 covers the most impactful and automatable controls
> across IAM, storage, logging, and networking. The remaining controls are
> either manual-only (physical access, legal agreements) or niche
> (organization-level policies that need Control Tower). These 30 give 80%
> of the security value."

---

**Bullet: "Mapped findings to NIST 800-53 Rev 5, SOC 2 TSC 2017, ISO 27001:2022, and PCI DSS v4.0.1"**

What this means to an interviewer:
- You know the current version of all four frameworks (not outdated ones)
- You understand why crosswalks exist (one evidence set, multiple audits)
- You can cite source documents

Follow-up: "How did you decide which NIST controls to map?"
> "I used CIS's own published mapping to NIST 800-53 as a starting point, then
> verified against the NIST control catalog. NIST organizes controls into
> families — Access Control, Audit, Identification & Authentication, etc. So
> for each CIS check, I identified which family it logically belongs to. MFA
> checks go to the Identity family, logging checks go to the Audit family,
> network checks go to System Protection. You don't need to memorize the
> numbers — you just need to understand which domain each control lives in."

---

**Bullet: "Implemented drift detection comparing scan pairs"**

What this means to an interviewer:
- You understand that snapshots aren't enough — trends matter
- You can design comparison logic (identity keys, bucket categorization)
- You think about the analyst's workflow (regression ≠ new resource)

Follow-up: "What's the identity key for drift?"
> "Control_id plus resource_arn. A Finding UUID won't work because every scan
> generates fresh UUIDs. But 'CIS 1.5 against arn:aws:iam::root' is stable
> across scans — it means the same check against the same resource."

---

**Bullet: "Built Streamlit dashboard with framework crosswalk visualization, per-control drill-in, and remediation snippets"**

What this means to an interviewer:
- You can present security data to non-technical stakeholders
- You think about the full loop (detect → assess → fix)
- You understand different user personas (CISO wants overview, analyst wants drill-in, engineer wants remediation)

Follow-up: "Who's the audience for this dashboard?"
> "Three personas. The CISO gets the Overview page — 30 seconds to answer 'are
> we in good shape?' The GRC analyst gets Frameworks — proof that one scan
> satisfies multiple audits. The security engineer gets Controls and Resources
> — drill into failures, see evidence, get a fix snippet."

---

## Final Advice

1. **Don't memorize — understand.** If you understand WHY a crosswalk exists, you can answer any question about it. If you just memorize specific control numbers, you'll freeze when they ask "why that mapping?" Understand the logic: "MFA checks map to the identity/authentication family in every framework."

2. **It's okay to say "I'd look that up."** You don't need to know every control number by memory. You need to know the methodology: "I'd identify which domain the check falls in — identity, logging, network — then find the corresponding family in the target framework."

3. **Lead with the WHY.** Don't say "I used a decorator registry." Say "I needed fault isolation so one broken check doesn't kill the scan, and auto-discovery so adding a new check is one decorator and done. A decorator registry gives me both."

4. **Own the weaknesses.** When they ask "what would you improve?" — single-region, no alerting, JSON at scale. These show you think critically about your own work.

5. **The project proves a pipeline, not a product.** Don't oversell it. It's a demonstration that you understand the full security lifecycle. That's what entry-level roles care about — can you think through the whole problem?

---
---

# Part 2: GRC & Security Fundamentals (Beyond the Project)

> Everything below is stuff they might ask that ISN'T covered by your
> project. Your project shows you can build a security tool. This section
> shows you understand the FIELD you're trying to work in.

---

## 15. The CIA Triad — the foundation of everything

Every single security control, framework, and tool exists to protect one or more of these three things:

**Confidentiality** — Only authorized people can SEE the data.
- Encryption (at rest and in transit)
- Access controls (who can read what)
- Data classification (public vs internal vs restricted)
- Example: S3 Block Public Access protects confidentiality. If a bucket is public, unauthorized people can see data.

**Integrity** — The data hasn't been TAMPERED with.
- Checksums and hashing (detect if something changed)
- Digital signatures (prove who wrote it)
- Version control and audit trails
- Example: CloudTrail log file validation protects integrity. If someone deletes their tracks, the validation hash will fail.

**Availability** — The data/system is ACCESSIBLE when needed.
- Redundancy and backups
- DDoS protection
- Disaster recovery
- Example: RDS multi-AZ protects availability. If one data center goes down, the database fails over.

**How to use this in an interview:**

If they ask "why does this control matter?" you can always frame it as:
> "This protects [confidentiality/integrity/availability] because..."

Example: "Why does MFA on root matter?"
> "It protects confidentiality and integrity — without MFA, a single
> compromised password gives an attacker full access to read AND modify
> everything in the account."

---

## 16. What a GRC analyst actually DOES day-to-day

Your tool automates the DETECTION part. But a GRC analyst's job is much broader:

### The GRC lifecycle (your tool covers steps 1-2)

```
1. IDENTIFY risks and requirements    ← your scanner does this
2. ASSESS current state (gap analysis) ← your dashboard shows this
3. REMEDIATE (fix or accept risks)     ← your remediation snippets help here
4. DOCUMENT (policies, procedures)     ← not in your tool
5. AUDIT (prove compliance to third parties) ← your crosswalk supports this
6. MONITOR continuously               ← your drift detection does this
```

### A typical week for a GRC analyst:

**Monday:** Check the risk register. Any new risks reported? Any deadlines approaching for remediation items?

**Tuesday:** Meet with engineering team about 3 open findings from last month's scan. Two are fixed (collect evidence, close the ticket). One is delayed — document the reason, update the timeline.

**Wednesday:** Prep for upcoming SOC 2 audit. Auditor sent a "prepared by client" (PBC) list — 50 items they need evidence for. Pull screenshots, exports, config dumps. Your CSPM tool would generate some of this evidence automatically.

**Thursday:** Write/update a security policy. The "Password Policy" needs updating because the company moved to passwordless auth. Draft the change, get approval from the CISO, publish it.

**Friday:** Vendor risk assessment. A new SaaS tool wants to process customer data. Review their SOC 2 report, check their security practices, rate the risk, make a recommendation to leadership.

### Key GRC documents you should know exist:

| Document | What it is | Plain English |
|---|---|---|
| **Risk Register** | A spreadsheet/database of all known risks | The backlog of "things that could go wrong" with severity, owner, and status |
| **Policy** | A high-level statement of intent | "We require MFA on all accounts" — the WHAT, not the HOW |
| **Standard** | The specific implementation detail | "MFA must be hardware token for privileged accounts, virtual MFA for standard users" — the HOW |
| **Procedure** | Step-by-step instructions | "To enroll MFA: go to IAM > Security Credentials > Assign MFA device..." |
| **Control** | A safeguard that reduces risk | MFA itself IS the control. The policy says you need it, the standard says what kind, the procedure says how to set it up. |
| **Evidence** | Proof a control is working | A screenshot showing MFA is enabled, an API response, a scan result — this is what your tool produces |
| **PBC list** | "Prepared by client" — auditor's shopping list | "Show me evidence of items 1-50" — the list of evidence an auditor needs you to provide |
| **POA&M** | Plan of Action & Milestones | "We know this is broken, here's our plan to fix it by [date]" — how you handle findings you can't fix immediately |

### How YOUR project fits into GRC work:

> "My CSPM tool automates the IDENTIFY and ASSESS steps. It produces
> the evidence an analyst would normally collect manually — screenshots
> of settings, API responses showing configurations. The crosswalk then
> maps that evidence to the specific framework requirements the auditor
> is asking about. Instead of spending 3 days pulling evidence for a
> SOC 2 audit, the analyst can point to the scan results."

---

## 17. Risk Management — how you decide what to fix first

### The risk equation:

```
Risk = Likelihood × Impact
```

- **Likelihood** = how probable is it that this vulnerability gets exploited?
- **Impact** = if it IS exploited, how bad is it?

### Why your tool uses CRITICAL/HIGH/MEDIUM/LOW:

Your severity levels roughly encode BOTH factors:

| Severity | Meaning | Example from your tool |
|---|---|---|
| CRITICAL | High likelihood AND high impact. Fix immediately. | Root account without MFA — any phished password = total account takeover |
| HIGH | Either very likely OR very impactful. Fix this sprint. | SSH open to 0.0.0.0/0 — bots scan for this automatically within minutes |
| MEDIUM | Moderate likelihood and moderate impact. Plan to fix. | KMS key rotation not enabled — limited blast radius, not actively scanned |
| LOW | Low likelihood OR low impact. Fix when convenient. | Support role not configured — only matters during an incident |

### Risk acceptance — when you DON'T fix something:

Not every finding gets fixed. Sometimes the business says "we accept that risk." That's VALID as long as:
1. The risk is documented (in the risk register)
2. Someone with authority formally accepted it (signed off)
3. There's a review date (don't accept forever — revisit quarterly)
4. Compensating controls exist (other protections that reduce the risk)

**Interview question:** "What if an engineering team refuses to fix a finding?"

> "I'd first make sure they understand the risk — not just the technical
> finding, but the business impact. If they still won't fix it, I'd
> escalate to their manager and the CISO. If leadership formally accepts
> the risk, I document it in the risk register with an owner, a
> justification, and a review date. Accepted risk isn't ignored risk —
> it's risk someone took responsibility for."

---

## 18. Common attack patterns you should know

You don't need to be a pentester. But you should understand the attacks your controls PREVENT:

### Credential-based attacks (IAM controls prevent these)

| Attack | How it works | Which control prevents it |
|---|---|---|
| **Credential stuffing** | Attacker uses leaked passwords from other sites | MFA (even if password is leaked, they can't log in without the token) |
| **Brute force** | Automated guessing of passwords | Strong password policy + account lockout |
| **Access key leakage** | Dev accidentally commits AWS key to GitHub | Key rotation (limits blast radius) + unused key cleanup |
| **Privilege escalation** | Attacker with low-level access finds a way to get admin | Least privilege + no direct policy attachment to users |

### Network attacks (VPC/EC2 controls prevent these)

| Attack | How it works | Which control prevents it |
|---|---|---|
| **Port scanning → exploitation** | Bots scan 0.0.0.0/0 for open SSH/RDP, then brute-force | Restrict SSH/RDP to known CIDRs or bastion SGs |
| **SSRF → credential theft** | Attacker tricks a web app into calling the metadata service | IMDSv2 (requires a token, which SSRF can't easily get) |
| **Lateral movement** | Attacker on one host moves to others via open internal ports | Default SG with no rules (limits internal movement) |

### Data exposure (S3/EBS/RDS controls prevent these)

| Attack | How it works | Which control prevents it |
|---|---|---|
| **Public bucket exfiltration** | Attacker finds a public S3 bucket, downloads everything | Block Public Access |
| **Snapshot theft** | Attacker with partial access copies an unencrypted EBS snapshot | EBS encryption (snapshot is useless without the key) |
| **Database exposure** | Publicly accessible RDS instance gets brute-forced | RDS not publicly accessible + VPC-only access |

### Audit evasion (CloudTrail controls prevent these)

| Attack | How it works | Which control prevents it |
|---|---|---|
| **Log deletion** | Attacker deletes CloudTrail logs to hide activity | Log file validation (detects tampering) + KMS encryption (can't read to know what to delete) |
| **Regional blindspot** | Attacker operates from a region where logging isn't enabled | Multi-region CloudTrail |
| **Insider cover-up** | Employee with access modifies logs | Log file integrity validation (hash chain breaks if modified) |

### The Capital One breach (2019) — your go-to example:

Know this one cold because your tool has a control (CIS 5.6 / IMDSv2) that directly prevents it:

1. **What happened:** A former AWS employee exploited an SSRF vulnerability in Capital One's WAF (web application firewall).
2. **How:** The SSRF let her make requests FROM the WAF TO the EC2 metadata service (IMDSv1), which happily returned the instance's IAM role credentials — no authentication required.
3. **What she got:** Temporary credentials for an IAM role that had read access to S3 buckets containing 100 million customer records.
4. **What prevents it:** IMDSv2 requires a PUT request with a TTL header to get a session token FIRST, then you use that token to fetch metadata. SSRF attacks can't easily make PUT requests or handle multi-step token flows.
5. **The lesson:** "One misconfiguration (IMDSv1) + one vulnerability (SSRF) + one over-permissioned role = 100 million records. Defense in depth means fixing ALL three layers, not just one."

---

## 19. The audit process — what actually happens

### How a SOC 2 Type II audit works (the most common one):

```
Month 1-2:  READINESS ASSESSMENT
            - Auditor tells you what they'll need (PBC list)
            - You identify gaps and fix them
            - This is where your CSPM tool shines — automated gap finding

Month 3-8:  OBSERVATION PERIOD (6 months minimum)
            - Controls must be operating CONSISTENTLY
            - This is why drift detection matters — you need to prove
              controls didn't lapse during the period
            - Auditor may pull samples from any point in the period

Month 9:    EVIDENCE COLLECTION
            - Pull evidence for every control in scope
            - Screenshots, exports, scan results, policies
            - Your tool's JSON evidence is exactly this

Month 10:   FIELDWORK
            - Auditor reviews evidence, asks questions, tests controls
            - They may re-run checks themselves to verify your evidence
            - "Walk me through how this control works day-to-day"

Month 11:   REPORT DRAFTING
            - Auditor writes findings (exceptions)
            - You respond to exceptions with remediation plans

Month 12:   REPORT ISSUED
            - Clean report = no exceptions (rare)
            - Most reports have a few exceptions with management responses
            - The report goes to your customers (they asked for it)
```

### Why your drift page matters for audits:

SOC 2 Type II specifically requires that controls operated CONSISTENTLY over the observation period. If your MFA check passed in month 1 but failed in month 4, that's an exception — even if it passes again in month 9.

> "My drift detection shows whether controls maintained their passing
> state across the observation period. If something regressed and was
> fixed, I can show both events — the auditor sees the regression AND
> the resolution, with timestamps. That's transparency — and auditors
> reward it."

---

## 20. Security concepts they'll quiz you on

### Encryption at rest vs. in transit:

| | At rest | In transit |
|---|---|---|
| **What** | Data stored on disk/in S3/in a database | Data moving between systems (API calls, web traffic) |
| **How** | AES-256 encryption with KMS keys | TLS 1.2+ (HTTPS) |
| **Your controls** | CIS 2.1.1 (S3), 2.2.1 (EBS), 2.3.1 (RDS), 3.7 (CloudTrail logs) | Not covered in your scanner (would be a future enhancement) |
| **Why both** | Defense in depth. If someone steals a disk, encryption at rest protects it. If someone intercepts network traffic, TLS protects it. |

### Authentication vs. Authorization:

| | Authentication (AuthN) | Authorization (AuthZ) |
|---|---|---|
| **Question it answers** | "WHO are you?" | "WHAT are you allowed to do?" |
| **How** | Password + MFA, certificates, SSO | IAM policies, roles, permissions |
| **Your controls** | CIS 1.5 (root MFA), 1.8 (password policy), 1.10 (user MFA) | CIS 1.16 (no direct policy attachment — use groups/roles) |
| **Real-world analogy** | Showing your badge at the door | The badge only opens certain rooms, not all of them |

### Preventive vs. Detective vs. Corrective controls:

| Type | What it does | Examples from your tool |
|---|---|---|
| **Preventive** | Stops bad things from happening | Block Public Access (prevents public S3), default SG with no rules (prevents accidental exposure) |
| **Detective** | Finds bad things that happened | CloudTrail (logs all API calls), VPC flow logs (records network traffic), your CSPM scanner itself |
| **Corrective** | Fixes bad things after they're found | Your remediation snippets, key rotation (limits damage of a compromised key) |

> "My tool is primarily a DETECTIVE control — it finds misconfigurations.
> But the controls it checks are a mix of all three types. And the
> remediation library helps with the CORRECTIVE step."

### Shared responsibility model (AWS):

| | AWS is responsible for | YOU are responsible for |
|---|---|---|
| **Security OF the cloud** | Physical data centers, hardware, network infrastructure, hypervisor | — |
| **Security IN the cloud** | — | Your configurations, your IAM policies, your data, your encryption choices, your network rules |

> "My tool checks the 'security IN the cloud' side — the things YOU
> configure. AWS guarantees the physical security and the hypervisor.
> But if you leave a bucket public or don't enable MFA, that's on you.
> My scanner finds those misconfigurations."

### Zero Trust (buzzword they might ask about):

**Old model:** "Trust everything inside the network perimeter. Once you're in, you're trusted."

**Zero Trust:** "Never trust, always verify. Every request is authenticated and authorized regardless of where it comes from."

Principles:
- Verify explicitly (always authenticate, even internal traffic)
- Least privilege access (give minimum permissions needed)
- Assume breach (design systems assuming attackers are already inside)

How your tool relates:
> "Several of my controls enforce Zero Trust principles. CIS 5.4 (strip
> the default SG) assumes you shouldn't trust internal traffic by default.
> CIS 1.16 (no direct policy attachment) enforces least privilege. CIS 5.6
> (IMDSv2) assumes the network might be compromised (SSRF) and requires
> explicit token auth even for internal metadata requests."

---

## 21. Compliance vs. Security — they're NOT the same

This is a nuanced point that makes you sound mature:

**Compliance** = meeting the minimum requirements of a framework. Checking boxes. Passing an audit.

**Security** = actually being protected against threats. Reducing real risk.

**The gap:**
- You can be COMPLIANT but NOT SECURE. (You checked all the boxes but an attacker still gets in because the boxes didn't cover their attack vector.)
- You can be SECURE but NOT COMPLIANT. (Your security is excellent but you didn't document it in the way the auditor requires.)

**Interview answer:**
> "My tool helps with both but they're different goals. The crosswalk
> helps with compliance — mapping evidence to framework requirements so
> you can pass audits efficiently. But the actual security value comes
> from the controls themselves — finding misconfigurations that an
> attacker would exploit. Compliance tells you if you'll pass an audit.
> Security tells you if you'll survive an attack. Ideally you want both."

---

## 22. Questions they ask that AREN'T about your project

### "What's the OWASP Top 10?"

The top 10 web application security risks. Published by the Open Web Application Security Project. The 2021 version:

1. **Broken Access Control** — users can do things they shouldn't
2. **Cryptographic Failures** — sensitive data not properly encrypted
3. **Injection** — SQL injection, XSS, command injection
4. **Insecure Design** — flawed architecture (not just bugs)
5. **Security Misconfiguration** — THIS IS WHAT YOUR TOOL FINDS (in cloud infra, not web apps)
6. **Vulnerable Components** — using libraries with known CVEs
7. **Authentication Failures** — weak login mechanisms
8. **Data Integrity Failures** — no verification of updates/data
9. **Logging Failures** — YOUR CLOUDTRAIL CONTROLS address this at the infra level
10. **SSRF** — YOUR IMDSv2 CONTROL prevents the most common cloud SSRF exploit

> "My tool is focused on infrastructure misconfiguration — item #5 in the
> OWASP Top 10, but at the cloud layer rather than the application layer.
> It also addresses #9 (logging) and #10 (SSRF via IMDSv2)."

### "What's a vulnerability vs. a threat vs. a risk?"

| Term | What it is | Example |
|---|---|---|
| **Vulnerability** | A weakness that COULD be exploited | SSH open to 0.0.0.0/0 |
| **Threat** | Someone/something that WOULD exploit it | Automated bots scanning for open SSH |
| **Risk** | The combination: likelihood of threat × impact of exploitation | High — bots WILL find it, and SSH access = full host compromise |

> "My tool finds VULNERABILITIES (misconfigurations). The THREAT is
> implicit — we know attackers scan for these things. The RISK is what
> I communicate through severity levels."

### "What's the difference between a pentest and a vulnerability scan?"

| | Vulnerability scan | Penetration test |
|---|---|---|
| **What** | Automated tool that checks for known weaknesses | Human expert actively trying to break in |
| **Depth** | Wide but shallow — checks configurations, known CVEs | Deep — chains multiple weaknesses together, tests business logic |
| **Output** | List of findings (like your CSPM tool) | Narrative of how they got in, what they accessed, what an attacker could do |
| **Frequency** | Continuous/daily | Quarterly or annually |
| **Your tool is** | A vulnerability scan (configuration-focused, automated, broad) | NOT a pentest (no exploitation, no chaining) |

### "What certifications are relevant?"

If they ask what you're pursuing or planning:

| Cert | What it proves | Effort | Best for |
|---|---|---|---|
| **CompTIA Security+** | Broad security fundamentals | 1-2 months study | Entry-level security roles. Very common HR filter. |
| **ISC2 CC** | Entry-level security concepts (free exam) | 2-4 weeks | Getting past HR when you have zero certs |
| **AWS Cloud Practitioner** | You understand AWS basics | 1-2 weeks | Pairs well with your project (shows you know the platform) |
| **AWS Security Specialty** | Deep AWS security knowledge | 2-3 months | Aspirational — after you have some experience |
| **CISA** | Audit/GRC focused | 3-6 months | GRC-specific roles. Requires experience (or waiver). |

---

## 23. Behavioral / soft-skill questions

### "Tell me about a time you solved a complex problem."

Use your project:
> "When building drift detection, I realized you can't just diff two JSON
> files because every scan generates fresh UUIDs. The naive approach
> doesn't work. I had to step back and ask 'what does identity mean
> across scans?' The answer was the combination of which check ran
> against which resource — that's stable even when IDs change. Then I
> realized not all new failures are the same — a resource that REGRESSED
> from passing is different from a brand-new resource that was born
> broken. That distinction matters because they have different root
> causes. The process was: understand the problem deeply, find the right
> abstraction, then implement it simply."

### "How do you handle disagreements with team members?"

> "I lead with data, not opinions. If I believe a security finding needs
> to be fixed and an engineer disagrees, I'd show them the evidence —
> what the misconfiguration is, what attack it enables, what the impact
> could be. If they still disagree, I'd escalate to a risk acceptance
> process rather than forcing the issue. Security works through
> influence and transparency, not authority."

### "How do you prioritize when everything is urgent?"

> "Severity × exposure × ease of fix. A CRITICAL finding on a
> production system that's one CLI command to fix goes first. A MEDIUM
> finding on a dev system that requires an architecture change goes to
> the backlog. My tool's severity levels help with the first factor, and
> the remediation snippets help teams estimate the third. The second
> factor — exposure — requires business context that a tool can't
> provide."

### "Where do you see yourself in 5 years?"

For GRC/security roles:
> "I want to grow from doing the hands-on technical work — running scans,
> collecting evidence, mapping controls — into designing the program
> itself. Eventually I'd want to own a compliance program end-to-end:
> deciding which frameworks we pursue, building the automation, managing
> auditor relationships, and reporting risk to leadership."

---

## 24. Questions YOU should ask the interviewer

Asking smart questions shows you understand the field:

1. **"What frameworks are you currently certified or auditing against?"** — Shows you know there are choices and they're not all the same.

2. **"How much of your compliance evidence collection is automated vs. manual?"** — Shows you think about efficiency (your project is literally about this).

3. **"How does the security team communicate risk to engineering?"** — Shows you understand the relationship between security and engineering.

4. **"What's the biggest gap in your current security posture that you're working to close?"** — Shows you think proactively, not just reactively.

5. **"Is this role more audit-prep focused or continuous-monitoring focused?"** — Shows you know GRC has different flavors and you want to understand the day-to-day.

---

## 25. One-page cheat sheet (review this 10 minutes before the interview)

**Your project in 30 seconds:**
> Read-only CSPM scanner. 30 CIS controls. 5 frameworks. Drift detection.
> Remediation snippets. Streamlit dashboard. Everything mapped, nothing mutated.

**The 5 frameworks in 5 seconds each:**
- CIS = the prescriptive AWS checklist (what we scan against)
- NIST 800-53 = US government's control catalog (FedRAMP)
- SOC 2 = the SaaS audit (enterprise customers demand it)
- ISO 27001 = the international standard (global companies)
- PCI DSS = the credit card standard (Visa, banks, e-commerce)

**Crosswalk in one sentence:**
> One scan produces evidence that satisfies five different audits.

**Drift in one sentence:**
> Shows whether security is getting better or worse over time.

**3 design choices to defend:**
1. Read-only = safe for any environment, even production
2. Decorator registry = auto-discovery + fault isolation
3. Regressed vs. new-resource = different root causes need different fixes

**3 weaknesses to own:**
1. Single-region only
2. No alerting
3. JSON doesn't scale

**If you don't know the answer:**
> "I'm not sure about the specific details, but I know it falls in the
> [identity/logging/network/encryption] domain, and I'd look it up in
> [the NIST catalog / the CIS benchmark / the framework docs]."

