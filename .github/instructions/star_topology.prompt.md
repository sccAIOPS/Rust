# Star Topology Configuration for TheAlgorithms/Rust

## Topology Overview

```yaml
topology: star
hub: .github/copilot-instructions.md
reason: Single-module educational library with consistent patterns
```

## Agent Configuration

### Analysis Cluster
```yaml
cluster: analysis
purpose: Understanding and documenting existing code
agents:
  - name: Senior-Architecture_Analyst
    file: .github/agents/Senior-Architecture_Analyst.agent.md
    outputs: [UML diagrams, C4 models, dependency graphs]
    
  - name: Senior-Design_Pattern_Analyst  
    file: .github/agents/Senior-Design_Pattern_Analyst.agent.md
    outputs: [Pattern documentation, classification reports]
    
  - name: Principal-Algorithm_Analyst
    file: .github/agents/Principal-Algorithm_Analyst.agent.md
    outputs: [Analysis reports, optimization recommendations]
```

### Development Cluster
```yaml
cluster: development
purpose: Planning and implementing new features
agents:
  - name: Senior-Feature_Planner
    file: .github/agents/Senior-Feature_Planner.agent.md
    outputs: [BDD specs, requirements docs, feature plans]
    
  - name: Principal-System_Designer
    file: .github/agents/Principal-System_Designer.agent.md
    outputs: [Architecture designs, component specs, implementation guides]
    
  - name: Senior-Rust_Implementer
    file: .github/agents/Senior-Rust_Implementer.agent.md
    outputs: [Rust source code, tests, documentation]
```

### Quality Cluster
```yaml
cluster: quality
purpose: Ensuring code quality and correctness
agents:
  - name: Senior-Code_Reviewer
    file: .github/agents/Senior-Code_Reviewer.agent.md
    outputs: [Review comments, quality reports]
    
  - name: Senior-QA_Tester
    file: .github/agents/Senior-QA_Tester.agent.md
    outputs: [Test suites, coverage reports, performance benchmarks]
```

## Workflow Definitions

### Workflow: full_development_cycle
```yaml
name: Full Development Cycle
description: Complete lifecycle for new feature development
steps:
  1:
    agent: Senior-Feature_Planner
    input: Feature request
    output: BDD specification, requirements
    
  2:
    agent: Principal-System_Designer
    input: Requirements from step 1
    output: Architecture design, implementation guide
    
  3:
    agent: Senior-Rust_Implementer
    input: Design from step 2
    output: Implemented code with tests
    
  4:
    agent: Senior-Code_Reviewer
    input: Code from step 3
    output: Review feedback
    gate: Must pass before step 5
    
  5:
    agent: Senior-QA_Tester
    input: Reviewed code from step 4
    output: Test report, coverage
```

### Workflow: documentation_analysis
```yaml
name: Documentation Analysis
description: Extract and document existing codebase
steps:
  1:
    agent: Senior-Architecture_Analyst
    input: Repository structure
    output: Architecture diagrams
    
  2:
    agent: Senior-Design_Pattern_Analyst
    input: Code analysis
    output: Pattern documentation
    parallel_with: step 1
    
  3:
    agent: Principal-Algorithm_Analyst
    input: Algorithm implementations
    output: Analysis reports
    parallel_with: step 1, 2
```

### Workflow: quick_implementation
```yaml
name: Quick Implementation
description: Fast implementation for simple algorithms
steps:
  1:
    agent: Senior-Rust_Implementer
    input: Algorithm specification
    output: Implementation with tests
    
  2:
    agent: Senior-Code_Reviewer
    input: Code from step 1
    output: Review approval
```

### Workflow: quality_assurance
```yaml
name: Quality Assurance
description: Review and test existing implementations
steps:
  1:
    agent: Senior-Code_Reviewer
    input: Code to review
    output: Review report
    
  2:
    agent: Senior-QA_Tester
    input: Reviewed code
    output: Test suite and report
```

## Quality Gates

```yaml
gates:
  pre_implementation:
    - Feature plan approved
    - Design document complete
    - No duplicate functionality
    
  pre_merge:
    - cargo fmt passes
    - cargo clippy passes (no warnings)
    - cargo test passes
    - Code review approved
    - Documentation complete
    
  post_merge:
    - All tests still pass
    - No regression in coverage
```

## Inter-Agent Communication

```yaml
handoff_protocols:
  planner_to_designer:
    - BDD specification
    - Functional requirements
    - Non-functional requirements
    
  designer_to_implementer:
    - API specification
    - Implementation guide
    - Test case outline
    
  implementer_to_reviewer:
    - Source code
    - Tests
    - Documentation
    
  reviewer_to_tester:
    - Reviewed code
    - Review comments
    - Identified edge cases
```

## Metrics

```yaml
success_metrics:
  code_quality:
    - clippy_warnings: 0
    - fmt_issues: 0
    
  test_coverage:
    - line_coverage: ">80%"
    - branch_coverage: ">70%"
    
  documentation:
    - public_api_docs: "100%"
    - examples: "required"
```
