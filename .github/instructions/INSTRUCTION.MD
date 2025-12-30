# Instructions Directory Guide

This directory contains topology and workflow instructions for orchestrating multiple agents.

## Available Topology: Star

The **Star Topology** is used for this repository because:
- Single-module library structure
- Consistent coding patterns across all categories
- No complex inter-service dependencies
- Educational focus with clear boundaries

## Topology Diagram

```
                    ┌─────────────────────────────────────┐
                    │   copilot-instructions.md (Hub)     │
                    │   - Orchestration rules             │
                    │   - Project standards               │
                    │   - Workflow chains                 │
                    └───────────────┬─────────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
┌───────────────┐         ┌─────────────────┐         ┌─────────────────┐
│ Analysis      │         │ Development     │         │ Quality         │
│ Agents        │         │ Agents          │         │ Agents          │
├───────────────┤         ├─────────────────┤         ├─────────────────┤
│ Architecture  │         │ Feature_Planner │         │ Code_Reviewer   │
│ Design_Pattern│         │ System_Designer │         │ QA_Tester       │
│ Algorithm     │         │ Implementer     │         │                 │
└───────────────┘         └─────────────────┘         └─────────────────┘
```

## When to Use Each Workflow

### Analysis Workflows (Understanding)

| Goal | Agent Chain | Output |
|------|-------------|--------|
| Understand architecture | Architecture_Analyst | UML diagrams, C4 models |
| Find patterns | Design_Pattern_Analyst | Pattern documentation |
| Optimize algorithm | Algorithm_Analyst | Analysis report |

### Development Workflows (Building)

| Goal | Agent Chain | Output |
|------|-------------|--------|
| Plan new feature | Feature_Planner → System_Designer | Requirements, design docs |
| Implement feature | System_Designer → Implementer | Working code |
| Quick fix | Implementer | Code change |

### Quality Workflows (Validating)

| Goal | Agent Chain | Output |
|------|-------------|--------|
| Review PR | Code_Reviewer | Review comments |
| Test feature | QA_Tester | Test suite |
| Full validation | Code_Reviewer → QA_Tester | Review + tests |

## Full Development Lifecycle

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   PLAN      │ -> │   DESIGN    │ -> │ IMPLEMENT   │ -> │   REVIEW    │ -> │    TEST     │
│             │    │             │    │             │    │             │    │             │
│ Feature     │    │ System      │    │ Rust        │    │ Code        │    │ QA          │
│ Planner     │    │ Designer    │    │ Implementer │    │ Reviewer    │    │ Tester      │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

## Escalation Paths

| Issue Type | Primary Agent | Escalate To |
|------------|---------------|-------------|
| Complexity issues | Algorithm_Analyst | System_Designer |
| Design questions | System_Designer | Architecture_Analyst |
| Implementation bugs | Implementer | Code_Reviewer |
| Test failures | QA_Tester | Algorithm_Analyst |
