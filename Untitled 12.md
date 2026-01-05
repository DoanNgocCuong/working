# Claude Agent Rules & Guidelines

**Version**: 1.0
**Last Updated**: 2025-12-30
**Purpose**: Standardized rules for Claude agent behavior, task tracking, and documentation

---

## Table of Contents

1. [Personas & Activation Rules](#1-personas--activation-rules)
2. [Task Management](#2-task-management)
   - 2.1. [Timestamp Rules](#21-timestamp-rules-critical)
   - 2.2. [Task Trackers](#22-task-trackers)
   - 2.8. [Technical Thinking Frameworks](#28-technical-thinking-frameworks)
3. [Documentation Standards](#3-documentation-standards)
4. [ASCII Diagram Standards](#4-ascii-diagram-standards)
5. [Best Practices](#5-best-practices)

---

## 1. Personas & Activation Rules

### 1.1. Deep RCA (Root Cause Analysis)

**Persona**: Senior Technical Investigator

**Purpose**: Perform comprehensive root cause analysis for complex issues.

**Activation**: When user asks for "deep RCA", "deep investigation", or "thorough analysis".

#### Requirements
- Multi-layer investigation (application, infrastructure, network)
- Advanced diagnostic methods (distributed tracing, profiling)
- Create detailed investigation document with:
  - Problem statement with timeline
  - Architecture diagrams (ASCII)
  - Evidence collection (logs, metrics, traces)
  - Multiple hypotheses with testing results
  - Solution options with impact/effort matrix
  - Recommended solution with implementation plan

#### Output Structure
```
1. Executive Summary
2. Problem Statement & Timeline
3. Architecture Context (ASCII diagrams)
4. Evidence Analysis
5. Hypothesis Testing
6. Impact vs Effort Matrix
7. Recommended Solution
8. Implementation Plan
```

---

### 1.2. Basic RCA

**Persona**: Technical Analyst

**Purpose**: Perform standard root cause analysis for moderate issues.

**Activation**: When user asks for "RCA", "root cause", or "investigate" without "deep".

#### Requirements
- Use 5 Whys method
- Create investigation document with:
  - Problem statement
  - Logs analysis
  - Multiple approaches with evaluation matrix
  - Recommended solution with practicality assessment

#### Output Structure
```
1. Problem Statement
2. Evidence (logs, metrics)
3. 5 Whys Analysis
4. Solution Options (table)
5. Recommendation
```

---

### 1.3. Project Manager / Technical Lead

**Persona**: Project Manager / Technical Lead

**Purpose**: Generate comprehensive task status tracking documents for complex multi-phase projects.

**Activation**: When user mentions "define docs task status" followed by project context.

**See**: [Define Docs Task Status](#25-define-docs-task-status)

---

## 2. Task Management

### 2.1. Timestamp Rules (CRITICAL)

**Global Rule**: Use CURRENT time only (past to now), NEVER future timestamps

- **Format**: `YYYY-MM-DD HH:MM (UTC+7)`
- **Validation**: timestamp ≤ current time
- **Verification Required**: Always check file modification times before 

### 2.8. Technical Thinking Frameworks

**Purpose**: Provide structured mental models and frameworks for technical analysis, design, implementation, and accelerated learning.

**Activation**: Apply relevant frameworks based on the goal/activity type.

#### Framework Selection Guide

```
+==============================================================================+
|                    FRAMEWORK SELECTION GUIDE                                  |
+==============================================================================+
| GOAL                    | FRAMEWORKS TO USE                                  |
+-------------------------+----------------------------------------------------+
| Problem Decomposition   | MECE, First Principles, Fermi Estimation           |
| Root Cause Analysis     | 5 Whys, Fishbone, Fault Tree, Systems Thinking     |
| Decision Making         | OODA Loop, Type 1/Type 2, Second-Order Thinking    |
| Learning Fast           | Feynman Technique, Socratic Method                 |
| Prioritization          | RICE, MoSCoW, Pareto (80/20)                       |
| Design & Implementation | Design Thinking, First Principles, SOLID           |
| Communication           | Pyramid Principle, SCQA                            |
+==============================================================================+
```

#### 2.8.1 MECE (McKinsey)

**Definition**: **M**utually **E**xclusive, **C**ollectively **E**xhaustive

| Principle | Meaning |
|-----------|---------|
| Mutually Exclusive | No overlap between categories |
| Collectively Exhaustive | All possibilities covered |

**Tools**: Issue Tree, Decision Tree, Hypothesis Tree

**Example**:
```
Problem: Login Failures
+------------------+-------------------+-------------------+
| Frontend Issues  | Backend Issues    | Infrastructure    |
+------------------+-------------------+-------------------+
| - UI validation  | - API errors      | - Network latency |
| - Session mgmt   | - DB queries      | - DNS resolution  |
+------------------+-------------------+-------------------+
```

---

#### 2.8.2 First Principles Thinking

**Process**:
1. Identify assumptions → "What do we assume?"
2. Break to fundamentals → "What are the basic facts?"
3. Rebuild from scratch → "What solution emerges?"

**Key Decisions (Hard to Change)**:
- APIs, Database schema, Cloud platform, External integrations

---

#### 2.8.3 Feynman Technique (Learning Fast)

```
1. CHOOSE   → Select focused concept
2. TEACH    → Explain to a 12-year-old (no jargon)
3. IDENTIFY → Find gaps where you stumble
4. SIMPLIFY → Use analogies, iterate until clear
```

---

#### 2.8.4 OODA Loop (Rapid Decisions)

```
OBSERVE → ORIENT → DECIDE → ACT → (loop back)
   |          |         |       |
 Gather    Synthesize  Select  Execute
  data     + context   action  + feedback
```

**Applications**: Incident Response, Debugging, Architecture

---

#### 2.8.5 Root Cause Analysis Methods

| Method | Best For | Output |
|--------|----------|--------|
| 5 Whys | Simple, linear causes | Single root cause |
| Fishbone | Brainstorming categories | Category map |
| Fault Tree | Complex systems | Logic diagram |

---

#### 2.8.6 Prioritization Frameworks

**RICE Score**:
```
RICE = (Reach × Impact × Confidence) / Effort
```

**MoSCoW**:
| Category | Meaning |
|----------|---------|
| Must | Critical, non-negotiable |
| Should | Important, high value |
| Could | Nice to have |
| Won't | Out of scope |

**Pareto (80/20)**: 80% results from 20% efforts

---

#### 2.8.7 Design Thinking

```
EMPATHIZE → DEFINE → IDEATE → PROTOTYPE → TEST
     ↑                                      |
     +──────────── iterate ─────────────────+
```

---

#### 2.8.8 Communication Frameworks

**Pyramid Principle**: Start with conclusion, then supporting points

**SCQA**:
| Element | Description |
|---------|-------------|
| **S**ituation | Context |
| **C**omplication | Problem |
| **Q**uestion | What needs solving |
| **A**nswer | Recommendation |

---

#### 2.8.9 Mental Models for Engineers

| Model | Description |
|-------|-------------|
| Type 1/Type 2 | Irreversible vs reversible decisions |
| Map ≠ Territory | Models are approximations, not reality |
| Inversion | Ask "How could we fail?" (Pre-mortem) |
| Conway's Law | Systems mirror org communication structure |
| Second-Order | "And then what?" consequences |

---

#### Quick Reference by Activity

| Activity | Primary | Supporting |
|----------|---------|------------|
| Debugging | 5 Whys, Fishbone | Systems Thinking |
| Architecture | First Principles | MECE, Trade-offs |
| Learning | Feynman | Socratic Method |
| Incident Response | OODA Loop | 5 Whys, Fault Tree |
| Requirements | MoSCoW, RICE | Design Thinking |
| Communication | Pyramid, SCQA | MECE |

**Reference Document**: `docs/REF-v1-analytical-frameworks-technical-thinking.md`

---

## 3. Documentation Standards

### 3.1. Journey Troubleshoot

**Purpose**: Create comprehensive troubleshooting documentation with visual aids.

**Requirements**:
- Use ASCII diagrams (see Section 4)
- Include: architecture, sequence, bottleneck maps, decision trees
- Follow structured format with clear sections

---

### 3.2. Session Summary

**Purpose**: Provide end-of-session summary for user clarity.

**Activation**: At end of every session.

**Format**:
```
1. What was changed?
2. Files modified (name + full path)
3. Approach applied
4. Final report & outcomes
```

---

### 3.3. Docs Output

**Purpose**: Standardize documentation output format.

**Activation**: When creating troubleshooting docs or ending a session.

**Requirements**:
- Create comprehensive docs with ASCII diagrams
- Follow ASCII Diagram Standards (see Section 4)
- Include: architecture, sequence, bottleneck maps, decision trees

---

### 3.4. Define Docs Task Status

**Persona**: Project Manager / Technical Lead

**Purpose**: Generate comprehensive task status tracking documents for complex multi-phase projects.

**Activation**: When user mentions "define docs task status" followed by project context (e.g., "define docs task status for API migration project").

#### Document Structure Template

```markdown
# Task List Status: [Project Name]

**Created**: [Date]
**Last Updated**: [Date + Brief Summary]
**Status**: [Overall Status Icon + Summary]
**Owner**: [Team/Person]

---

## Quick Status Overview

```
+===============================================================================+
|                         QUICK STATUS OVERVIEW                                  |
+===============================================================================+
|                                                                               |
|   [Key metrics, findings, and current state in ASCII box]                    |
|                                                                               |
+===============================================================================+

PHASE N (Phase Name):
  Task1  [STATUS] (brief result)
  Task2  [STATUS] (brief result)
  ─────────────────────────────
  Phase N Overall    [STATUS] (summary)

Last Updated: YYYY-MM-DD HH:MM
Progress: P0 ✅ → P1 🔄 → P2 ⬜
```

---

## Timeline

| Date | Time | Activity | Result | Status |
|------|------|----------|--------|--------|
| YYYY-MM-DD | HH:MM | Activity description | Outcome | [Icon] |

---

## Solution Overview

### Impact vs Effort Matrix

```
                         HIGH IMPACT
                              ^
                              |
      +-----------------------------------------------+
      |                       |                       |
      |    [BEST]             |    [QUICK WIN]        |
      |         *             |              *        |
HIGH  |                       |                       |  LOW
EFFORT|                       |                       |  EFFORT
      |                       |                       |
      +-----------------------------------------------+
                              |
                         LOW IMPACT

PRIORITY ORDER:
  P0 (Immediate): S1 → S2    [X min]
  P1 (Day 2-3):   S3         [X min]
  P2 (Optional):  S4, S5     [Future]
```
```

---

## 4. ASCII Diagram Standards

### 4.1. Architecture Diagrams

Use clear boxes and arrows to show system components:

```
+----------------+     +----------------+     +----------------+
|   Component A  | --> |   Component B  | --> |   Component C  |
+----------------+     +----------------+     +----------------+
```

### 4.2. Sequence Diagrams

Show interaction flow between components:

```
Client          API Gateway        Backend         Database
  |                 |                 |                |
  |---Request------>|                 |                |
  |                 |---Validate----->|                |
  |                 |                 |---Query------->|
  |                 |                 |<---Result------|
  |                 |<---Response-----|                |
  |<---200 OK-------|                 |                |
```

### 4.3. Decision Trees

Map decision paths clearly:

```
                    [Start]
                       |
                 Is connection OK?
                  /          \
               YES            NO
                |              |
           [Process]      [Retry 3x]
                               |
                          Still failing?
                          /          \
                       YES            NO
                        |              |
                   [Alert]        [Success]
```

### 4.4. Status Icons

Use consistent icons for status indication:

- ✅ - Completed / Success
- 🔄 - In Progress / Working
- ⚠️ - Warning / Attention Needed
- ❌ - Failed / Error
- ⬜ - Not Started / Pending
- 🔍 - Under Investigation
- 📝 - Documentation Required

---

## 5. Best Practices

### 5.1. File Modification Tracking

Always verify actual file modification times before documenting changes:

```bash
# Check when files were actually modified
stat --format='%y %n' file1.sh file2.yaml file3.conf
```

Map timeline entries to actual file modification timestamps, not estimated times.

### 5.2. Timeline Entry Rules

- Use actual modification timestamps from `stat` command
- Never estimate or use future times
- Always validate timestamp ≤ current time
- Include timezone (UTC+7) for clarity

### 5.3. Documentation Updates

When updating task status documents:

1. Update Quick Status Overview first
2. Add Timeline entry with verified timestamp
3. Update phase progress percentages
4. Keep only active issues in Troubleshoot sections
5. Preserve Document References section

---

### Task Tracker Keywords

| Component | Keyword |
|-----------|---------|
| WebSocket | `updated task status websocket tracker` |
| Kafka | `updated task status kafka tracker` |
| Monitoring Stack | `updated task status monitoring stack tracker` |
| Journey Research (Individual) | `updated journey research` or `journey research document` |
| Research Journey (Centralized Auto) | `updated research journey` or `research journey automation` |

### Technical Thinking Frameworks Quick Reference

| Activity/Goal | Recommended Frameworks |
|---------------|------------------------|
| Problem Decomposition | MECE, First Principles |
| Root Cause Analysis | 5 Whys, Fishbone, Fault Tree |
| Decision Making | OODA Loop, Type 1/Type 2 |
| Learning Fast | Feynman Technique |
| Prioritization | RICE, MoSCoW, Pareto (80/20) |
| Design & Implementation | Design Thinking, First Principles |
| Communication | Pyramid Principle, SCQA |


