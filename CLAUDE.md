# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Docula** is a microservices architecture project consisting of:

1. **Microservices Infrastructure** - A self-hosted architecture designed to run on a single bare-metal server (128 GB RAM) with future scalability to multiple servers/datacenters
2. **SMTP Forwarder** - A Go-based SMTP proxy/relay server that acts as an intermediary between SMTP clients and external SMTP servers

## Current State

This repository is currently in the documentation/planning phase. The `docs/` directory contains architectural specifications and implementation plans. No code has been implemented yet.

## Expert Review Agents (Mandatory Consultation)

**CRITICAL**: This project uses specialized read-only expert agents that MUST be consulted at specific points in the development workflow. These agents provide reviews, critiques, and recommendations but do not write code directly.

### Agent 1: CI/CD Expert

**Role**: GitHub Actions and CI/CD Pipeline Specialist

**Expertise**:
- GitHub Actions workflow optimization
- Multi-stage pipeline design (lint, test, build, deploy)
- Caching strategies for faster builds
- Matrix builds for multi-platform testing
- Security best practices in CI/CD
- Secrets management
- Artifact handling and versioning
- Performance optimization (parallel jobs, job dependencies)
- Cost optimization (avoiding unnecessary runs)

**When to Consult** (MANDATORY):
- Before creating or modifying any `.github/workflows/*.yml` file
- When designing the CI/CD strategy
- After implementing new test types (to integrate into pipeline)
- When pipeline runs are slow or inefficient
- Before adding new deployment steps
- When implementing caching strategies
- When setting up automated releases

**Review Checklist**:
- Pipeline efficiency and run time
- Proper job dependencies and parallelization
- Caching strategy (Go modules, build artifacts)
- Security scanning integration
- Test result reporting
- Code coverage reporting and enforcement
- Branch protection alignment
- Deployment strategies (staging, production)
- Rollback capabilities

**Consultation Format**:
```
Use Task tool with prompt:
"Act as a CI/CD expert. Review the following GitHub Actions workflow
and provide critique on: efficiency, best practices, security, caching,
parallelization, and cost optimization. Suggest specific improvements.

[paste workflow content or describe planned pipeline]"
```

### Agent 2: Test Automation Expert

**Role**: Test Architecture and Automation Specialist (Uncle Bob's Test Standards)

**Expertise**:
- Test architecture and organization (F.I.R.S.T principles)
- Test maintainability and refactoring
- Test smells and anti-patterns
- Mocking strategies and dependency injection
- Test fixture management
- Integration test design
- E2E test reliability
- Performance test methodology
- Test data management
- Test coverage quality (not just quantity)

**When to Consult** (MANDATORY):
- Before writing any test suite (unit, integration, e2e, performance)
- When tests become difficult to maintain
- When test coverage is high but quality is questionable
- When tests are flaky or slow
- Before implementing mocking strategies
- When designing test data fixtures
- When tests smell (tight coupling, brittle tests, etc.)
- During test refactoring sessions

**Review Checklist**:
- Test follows F.I.R.S.T principles (Fast, Independent, Repeatable, Self-validating, Timely)
- Tests are readable and maintainable
- Proper use of table-driven tests
- Appropriate mocking (not over-mocked, not under-mocked)
- Test naming conventions (clear, descriptive)
- Test organization and structure
- DRY violations in tests
- Test fixtures and helpers properly organized
- Tests verify behavior, not implementation
- Edge cases and error paths covered

**F.I.R.S.T Principles**:
- **Fast**: Tests run quickly (<100ms for unit tests)
- **Independent**: Tests don't depend on each other
- **Repeatable**: Same results every time, any environment
- **Self-validating**: Pass or fail, no manual interpretation
- **Timely**: Written before or with the code (TDD)

**Consultation Format**:
```
Use Task tool with prompt:
"Act as a test automation expert following Uncle Bob's principles.
Review the following test architecture/implementation:

[describe test structure or paste test code]

Critique:
1. F.I.R.S.T principles compliance
2. Maintainability and readability
3. Test architecture and organization
4. Mocking strategy appropriateness
5. Test smells and anti-patterns
6. Suggest architectural refactoring if needed"
```

### Agent 3: Clean Code Expert (Uncle Bob Standards)

**Role**: Code Quality and Clean Code Principles Enforcer

**Expertise**:
- SOLID principles
- Clean code principles (naming, functions, comments, formatting)
- Code smells detection
- Refactoring techniques
- Design patterns (appropriate usage)
- Separation of concerns
- Dependency management
- Error handling patterns
- Code readability and expressiveness
- Function/method size and complexity

**When to Consult** (MANDATORY):
- After implementing any component (before marking as complete)
- During refactoring phase of TDD cycle
- When code feels "smelly" or complex
- Before code review/PR submission
- When function exceeds 20 lines
- When cyclomatic complexity is high
- When considering design pattern usage
- When architecting new components

**Review Checklist (Uncle Bob's Standards)**:
- **Naming**: Intention-revealing, pronounceable, searchable
- **Functions**: Small (< 20 lines), do one thing, one level of abstraction
- **Comments**: Code is self-documenting, comments explain "why" not "what"
- **Formatting**: Consistent, vertical openness, proper indentation
- **Error Handling**: Don't return null, use error types, handle at appropriate level
- **SOLID Principles**:
  - Single Responsibility: Each function/struct has one reason to change
  - Open/Closed: Open for extension, closed for modification
  - Liskov Substitution: Subtypes must be substitutable for their base types
  - Interface Segregation: Many small interfaces over one large interface
  - Dependency Inversion: Depend on abstractions, not concretions
- **Code Smells**: Long functions, large structs, duplicated code, inappropriate intimacy
- **Go-Specific**: Proper error handling, effective use of interfaces, idiomatic Go

**Consultation Format**:
```
Use Task tool with prompt:
"Act as Uncle Bob (Robert C. Martin). Review this code with your
strictest clean code standards:

[paste code or describe component]

Provide critique on:
1. SOLID principles violations
2. Code smells and anti-patterns
3. Function size and complexity
4. Naming quality
5. Comments (necessary vs unnecessary)
6. Error handling patterns
7. Design pattern usage
8. Specific refactoring recommendations"
```

## Expert Agent Integration Workflow

### TDD Cycle with Expert Consultation

```
┌─────────────────────────────────────────────────────┐
│  1. PLAN (Task: Plan Agent)                         │
│     - Design component with test strategy            │
│     ↓                                                 │
│  2. CONSULT: Test Automation Expert                  │
│     - Review test architecture before implementation │
│     ↓                                                 │
│  3. RED: Write Failing Test                          │
│     ↓                                                 │
│  4. GREEN: Implement Minimal Code                    │
│     ↓                                                 │
│  5. CONSULT: Clean Code Expert                       │
│     - Review implementation for code smells          │
│     ↓                                                 │
│  6. REFACTOR: Improve Based on Expert Feedback       │
│     ↓                                                 │
│  7. Verify Tests Still Pass                          │
│     ↓                                                 │
│  8. REPEAT for next feature                          │
└─────────────────────────────────────────────────────┘
```

### CI/CD Setup with Expert Consultation

```
┌─────────────────────────────────────────────────────┐
│  1. Design CI/CD Pipeline (document requirements)    │
│     ↓                                                 │
│  2. CONSULT: CI/CD Expert                            │
│     - Review pipeline design                         │
│     - Get optimization recommendations               │
│     ↓                                                 │
│  3. Implement Workflow                               │
│     ↓                                                 │
│  4. CONSULT: CI/CD Expert                            │
│     - Review implemented workflow                    │
│     - Verify best practices                          │
│     ↓                                                 │
│  5. Test and Optimize                                │
│     ↓                                                 │
│  6. CONSULT: CI/CD Expert (if issues found)          │
└─────────────────────────────────────────────────────┘
```

### Code Review Checklist (Before PR)

**MANDATORY** - Consult all three experts before submitting PR:

1. **Test Automation Expert**: Review all test code
2. **Clean Code Expert**: Review all implementation code
3. **CI/CD Expert**: Verify CI/CD passes and is optimal

## Agent Consultation Examples

### Example 1: Implementing Authentication Module

```bash
# Step 1: Plan with Test Automation Expert
Task: "As a test automation expert, review this test architecture
for SMTP authentication module:
- TestAuthPlain_ValidCredentials
- TestAuthPlain_InvalidUsername
- TestAuthPlain_InvalidPassword
- TestAuthPlain_EmptyCredentials
- TestAuthPlain_SQLInjectionAttempt
- TestAuthPlain_RateLimiting

Suggest architectural improvements and additional test cases."

# Step 2: Implement following TDD
# (write tests, implement code)

# Step 3: Review with Clean Code Expert
Task: "Act as Uncle Bob. Review this authentication implementation:
[paste code]
Critique SOLID principles, naming, function size, error handling."

# Step 4: Refactor based on feedback
# Step 5: Verify tests pass
```

### Example 2: Setting Up CI/CD

```bash
# Step 1: Consult CI/CD Expert
Task: "As a CI/CD expert, design an optimal GitHub Actions workflow
for Go SMTP forwarder with:
- Linting (golangci-lint)
- Unit tests (with coverage)
- Integration tests (with testcontainers)
- E2E tests
- Performance tests
- Security scanning
- Multi-arch builds (amd64, arm64)

Provide workflow structure with best practices for caching and parallelization."

# Step 2: Implement workflow based on recommendations

# Step 3: Review implementation
Task: "As a CI/CD expert, review this implemented workflow:
[paste .github/workflows/ci.yml]
Critique and suggest optimizations."
```

### Example 3: Test Refactoring

```bash
# When tests become hard to maintain
Task: "As a test automation expert, review these tests:
[paste test code]

Tests are becoming brittle and hard to maintain. Suggest architectural
refactoring to improve:
1. Test fixtures management
2. Mock setup reduction
3. DRY violations
4. Test readability
5. Test independence"
```

## Enforcement Rules

### For Claude Code Instances

When working on this project, you MUST:

1. **Always consult appropriate expert before major work**:
   - Writing test suites → Test Automation Expert
   - Implementing code → Clean Code Expert
   - CI/CD work → CI/CD Expert

2. **Use Task tool for consultation** (read-only review):
   ```
   Task tool with subagent_type="general-purpose"
   Prompt includes: "Act as [expert role]..."
   ```

3. **Document expert feedback** in commit messages when addressing their recommendations

4. **Never skip expert consultation** for the mandatory checkpoints listed above

5. **Re-consult if significant changes** are made after initial review

6. **Before any PR/commit**, verify:
   - [ ] Test Automation Expert reviewed test architecture
   - [ ] Clean Code Expert approved implementation
   - [ ] CI/CD Expert verified pipeline (if CI/CD changes)

### Violation Consequences

If expert consultation is skipped:
- Code may not meet enterprise standards
- Tests may become unmaintainable
- CI/CD may be inefficient or insecure
- Technical debt will accumulate

**Remember**: These experts prevent technical debt BEFORE it's created.

## Project Architecture

### Microservices Infrastructure (docs/architecture.md)

**Core Components:**
- **Containerization**: Docker for isolation, Docker Compose for orchestration
- **Load Balancing**: Nginx Proxy Manager (NPM) as reverse proxy with round-robin, least connections, and weighted strategies
- **Message Broker**: Valkey (modern Redis alternative) or RabbitMQ for distributed task queues handling millions of JSON jobs
- **Database**: MongoDB Community Edition for document-based JSON storage with replication and sharding support

**Key Architecture Principle**: Start with single-server deployment, maintain ability to scale horizontally to multiple servers/datacenters.

### SMTP Forwarder (docs/smtp-forwarder.md)

**Purpose**: Proxy SMTP server that receives authenticated connections from clients and relays messages to external SMTP servers.

**Flow**: `Client (Outlook/Thunderbird/PHPMailer) → SMTP Forwarder (port 2587) → External SMTP Server`

**Planned Project Structure** (when implemented):
```
smtp-forwarder/
├── cmd/server/main.go          # Entry point
├── internal/
│   ├── config/config.go        # Environment configuration
│   ├── smtp/                   # SMTP server logic
│   │   ├── server.go
│   │   ├── session.go
│   │   └── forwarder.go
│   └── auth/auth.go            # Authentication
├── pkg/logger/logger.go        # Logging utilities
├── .env.example
├── Makefile
└── go.mod
```

**Key Dependencies** (when implemented):
- `github.com/emersion/go-smtp` - SMTP protocol implementation
- `github.com/joho/godotenv` - Environment configuration
- `github.com/sirupsen/logrus` - Logging

**Configuration**: Uses `.env` file with variables for:
- Ingestion SMTP (port, security, credentials)
- External SMTP relay (host, port, security, credentials)

## Development Commands

When the SMTP Forwarder is implemented, these commands will be used:

### Build and Run
```bash
# Install dependencies
go mod download

# Run in development
go run cmd/server/main.go

# Build for production
go build -o smtp-forwarder cmd/server/main.go

# Build with optimizations
go build -ldflags="-s -w" -o smtp-forwarder cmd/server/main.go
```

### Testing Commands
```bash
# Run all tests
go test ./...

# Run tests with coverage
go test -coverprofile=coverage.out ./...

# View coverage in browser
go tool cover -html=coverage.out

# Run tests with race detector
go test -race ./...

# Run tests verbosely
go test -v ./...

# Run specific test
go test -v -run TestAuthPlain ./internal/auth

# Run unit tests only
go test -short ./...

# Run integration tests
go test -tags=integration ./test/integration/...

# Run e2e tests
go test -tags=e2e ./test/e2e/...

# Run performance tests
go test -bench=. -benchmem ./...

# Run performance tests with profiling
go test -bench=. -cpuprofile=cpu.prof -memprofile=mem.prof ./...

# Test with timeout
go test -timeout 30s ./...

# Run tests in parallel
go test -parallel 4 ./...
```

### Code Quality
```bash
# Format code
go fmt ./...

# Lint code
golangci-lint run

# Vet code
go vet ./...

# Security scan
gosec ./...

# Check for vulnerabilities
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...

# Static analysis
go install honnef.co/go/tools/cmd/staticcheck@latest
staticcheck ./...
```

### Coverage Analysis
```bash
# Generate and view coverage
go test -coverprofile=coverage.out ./... && go tool cover -html=coverage.out

# Coverage by function
go tool cover -func=coverage.out

# Check coverage threshold (80%)
go test -coverprofile=coverage.out ./... && \
    go tool cover -func=coverage.out | grep total | awk '{print $3}' | sed 's/%//' | \
    awk '{if ($1 < 80) {print "Coverage below 80%"; exit 1} else {print "Coverage OK"}}'
```

### Makefile Targets (when implemented)
```bash
# Run all tests
make test

# Run tests with coverage
make test-coverage

# Run integration tests
make test-integration

# Run e2e tests
make test-e2e

# Run performance tests
make test-performance

# Run all quality checks
make quality

# Build
make build

# Clean
make clean
```

## Implementation Guidelines

### TDD-First Development Workflow (With Expert Consultation)

**MANDATORY**: Before writing any production code, follow this workflow with expert consultation:

#### Phase 1: Planning & Test Architecture (RED Phase Start)

1. **Consult Test Automation Expert FIRST**
   ```bash
   # Use Task tool to consult expert
   Task: "As a test automation expert following Uncle Bob's principles,
   review this proposed test architecture for [component name]:

   Planned Tests:
   - [list test cases]

   Critique: F.I.R.S.T compliance, test organization, mocking strategy,
   and suggest additional test cases or architectural improvements."
   ```

2. **Create Test File** (based on expert feedback)
   - Create test file if it doesn't exist (`*_test.go`)
   - Implement test structure recommended by expert

3. **Write Failing Test** (RED)
   - Write test that describes the behavior you want
   - Run test and confirm it fails for the right reason
   ```bash
   go test ./internal/[component]  # Should fail
   ```

#### Phase 2: Implementation (GREEN Phase)

4. **Implement Minimal Code**
   - Write just enough code to pass the test
   - Focus on making it work, not making it perfect
   ```bash
   go test ./internal/[component]  # Should pass
   ```

5. **Verify Tests Pass**
   ```bash
   go test -v -race ./internal/[component]
   go test -cover ./internal/[component]
   ```

#### Phase 3: Review & Refactor (REFACTOR Phase)

6. **Consult Clean Code Expert**
   ```bash
   Task: "Act as Uncle Bob. Review this implementation with strictest
   clean code standards:

   [paste implementation code]

   Critique: SOLID principles, code smells, function size, naming,
   error handling, and provide specific refactoring recommendations."
   ```

7. **Refactor Based on Expert Feedback**
   - Address all code smells and violations
   - Improve naming, structure, complexity
   - Apply SOLID principles
   ```bash
   go test ./internal/[component]  # Must still pass
   ```

8. **Final Verification**
   ```bash
   # Run all tests
   go test -v -race ./...

   # Check coverage
   go test -coverprofile=coverage.out ./...
   go tool cover -func=coverage.out

   # Lint and format
   go fmt ./...
   golangci-lint run
   ```

9. **Commit with Expert Feedback Noted**
   ```bash
   git commit -m "feat: [component] - implement [feature]

   - Implemented following TDD methodology
   - Test Automation Expert: [key recommendations applied]
   - Clean Code Expert: [key refactorings applied]
   - Coverage: X%
   - All tests passing with race detector"
   ```

#### Phase 4: Repeat

10. **Iterate for Next Feature**
    - Return to Phase 1 for next behavior/feature
    - Build incrementally with continuous expert review

**Complete TDD Example Session with Experts**:
```bash
# ============================================
# PHASE 1: PLANNING & TEST ARCHITECTURE
# ============================================

# Step 1: Consult Test Automation Expert
# (Use Task tool with expert consultation prompt)

# Step 2: Create test file based on expert feedback
touch internal/auth/auth_test.go

# Step 3: Write failing test (RED)
vim internal/auth/auth_test.go
# Write: TestAuthPlain_ValidCredentials
go test ./internal/auth  # Should fail ❌

# ============================================
# PHASE 2: IMPLEMENTATION
# ============================================

# Step 4: Implement minimal code (GREEN)
vim internal/auth/auth.go
go test ./internal/auth  # Should pass ✅

# Step 5: Verify
go test -v -race ./internal/auth
go test -cover ./internal/auth

# ============================================
# PHASE 3: REVIEW & REFACTOR
# ============================================

# Step 6: Consult Clean Code Expert
# (Use Task tool with expert consultation prompt)

# Step 7: Refactor based on feedback
vim internal/auth/auth.go
# Apply: Better naming, extract functions, reduce complexity
go test ./internal/auth  # Must still pass ✅

# Step 8: Final verification
go test -v -race ./...
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out
golangci-lint run

# Step 9: Commit
git add .
git commit -m "feat(auth): implement plain authentication

- Implemented ValidCredentials validation using TDD
- Test Automation Expert: Added edge cases for empty credentials
- Clean Code Expert: Extracted validateCredentials(), improved naming
- Coverage: 92%
- All tests passing with race detector"

# ============================================
# PHASE 4: REPEAT
# ============================================
# Go back to Phase 1 for next test case (InvalidCredentials)
```

### When Implementing SMTP Forwarder

1. **TDD Mandatory**: Write tests before implementation code (see TDD workflow above)
2. **Configuration**: All configuration must be loaded from `.env` file, never hardcoded
3. **Security Requirements**:
   - Mandatory authentication for incoming connections
   - TLS/STARTTLS support required
   - Input validation for email addresses and content
   - Comprehensive logging of all connections and authentication attempts
4. **Error Handling**: All forwarding errors must be logged with context (from, to, timestamp, error)
5. **Session Management**: Each SMTP session must properly track `from`, `to[]`, and message data before forwarding

### When Implementing Microservices

1. **Containerization**: Each service must have its own Dockerfile
2. **Configuration**: Use Docker Compose for local development orchestration
3. **Load Balancing**: Route all external traffic through Nginx Proxy Manager
4. **Task Processing**: Use message broker (Valkey/RabbitMQ) for any background job processing
5. **Data Storage**: Use MongoDB for JSON document storage

## Test Driven Development (TDD) Methodology

**CRITICAL**: All code in this project MUST be developed using Test Driven Development (TDD). This is a non-negotiable requirement.

### TDD Workflow (Red-Green-Refactor)

1. **RED**: Write a failing test first
   - Write the test before any implementation code
   - Test should fail because the feature doesn't exist yet
   - Ensure the test fails for the right reason

2. **GREEN**: Write minimal code to make the test pass
   - Implement only enough code to pass the test
   - Focus on making it work, not making it perfect
   - Verify all tests pass

3. **REFACTOR**: Improve the code while keeping tests green
   - Clean up code duplication
   - Improve naming and structure
   - Optimize performance if needed
   - Ensure all tests still pass

### Test Structure and Organization

```
smtp-forwarder/
├── internal/
│   ├── config/
│   │   ├── config.go
│   │   └── config_test.go
│   ├── smtp/
│   │   ├── server.go
│   │   ├── server_test.go
│   │   ├── session.go
│   │   ├── session_test.go
│   │   └── forwarder_test.go
│   └── auth/
│       ├── auth.go
│       └── auth_test.go
├── test/
│   ├── integration/          # Integration tests
│   ├── e2e/                  # End-to-end tests
│   ├── performance/          # Performance/load tests
│   ├── fixtures/             # Test data and fixtures
│   └── mocks/                # Mock implementations
└── .github/
    └── workflows/
        └── ci.yml            # CI/CD pipeline
```

## Testing Requirements (Enterprise Level)

### 1. Unit Tests
**Coverage Target**: Minimum 85%, Target 90%+

**Requirements**:
- Every function must have corresponding unit tests
- Test all code paths (happy path, error cases, edge cases)
- Use table-driven tests for multiple scenarios
- Mock external dependencies (network, filesystem, databases)
- Tests must be fast (<100ms per test)
- Tests must be isolated and independent

**Example Structure**:
```go
func TestAuthPlain(t *testing.T) {
    tests := []struct {
        name          string
        username      string
        password      string
        expectedError error
    }{
        {"valid credentials", "user", "pass", nil},
        {"invalid username", "wrong", "pass", ErrInvalidCredentials},
        {"invalid password", "user", "wrong", ErrInvalidCredentials},
        {"empty username", "", "pass", ErrInvalidCredentials},
        {"empty password", "user", "", ErrInvalidCredentials},
        {"sql injection attempt", "admin' OR '1'='1", "pass", ErrInvalidCredentials},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Test implementation
        })
    }
}
```

### 2. Integration Tests
**Location**: `test/integration/`

**Requirements**:
- Test interactions between components
- Use real dependencies where practical (with test containers)
- Test database operations with test MongoDB instance
- Test SMTP protocol interactions with mock SMTP server
- Verify proper error propagation across component boundaries
- Test configuration loading from .env files

**Key Integration Test Scenarios**:
- Config → Server → Session → Forwarder pipeline
- Authentication → Session creation → Message forwarding
- TLS handshake → Authentication → Message delivery
- Error handling across component boundaries

### 3. End-to-End Tests
**Location**: `test/e2e/`

**Requirements**:
- Test complete user workflows
- Use real SMTP clients (Go SMTP client, testcontainers)
- Verify actual email delivery to test SMTP server
- Test with various email clients (simulate PHPMailer, Thunderbird behavior)
- Validate logs and metrics are generated correctly

**Critical E2E Scenarios**:
- **Golden Path**: Client connects → authenticates → sends email → forwarded successfully
- **Authentication Failure**: Invalid credentials rejected
- **Connection Security**: TLS/STARTTLS handshake verification
- **Multiple Recipients**: Email sent to multiple recipients
- **Large Messages**: Handle emails up to configured size limit
- **Concurrent Connections**: Multiple simultaneous client connections

### 4. Performance Tests
**Location**: `test/performance/`

**Requirements**:
- Load testing with increasing concurrent connections
- Measure throughput (emails/second)
- Memory usage profiling
- CPU usage under load
- Latency measurements (P50, P95, P99)
- Identify bottlenecks and resource limits

**Performance Targets** (Milestone 1):
- Handle 100 concurrent connections
- Process 1000 emails/minute
- P95 latency < 500ms
- Memory usage < 256MB under normal load
- No memory leaks during 24h stress test

**Tools**:
```bash
# Benchmarking
go test -bench=. -benchmem ./...

# Profiling
go test -cpuprofile=cpu.prof -memprofile=mem.prof
go tool pprof cpu.prof

# Load testing
# Use custom load testing tool or k6
```

### 5. Edge Cases and Error Scenarios

**Must Test**:
- Network failures (connection drops, timeouts)
- External SMTP server unavailable
- Malformed email addresses
- Invalid SMTP commands
- Authentication rate limiting
- Connection limits exceeded
- TLS certificate errors
- Disk space exhausted (for logging)
- OOM conditions (very large emails)
- Race conditions (concurrent access)
- Graceful shutdown (in-flight messages)

### 6. Security Tests

**Requirements**:
- SQL injection attempts (in email addresses)
- Command injection attempts
- Email header injection
- Path traversal attempts (in attachments)
- Buffer overflow attempts
- Brute force authentication attempts
- TLS downgrade attacks
- Man-in-the-middle scenarios

### 7. Smoke Tests

**Location**: `test/smoke/`

**Purpose**: Quick validation that basic functionality works after deployment

**Smoke Test Suite** (must complete in <30 seconds):
- Server starts successfully
- Health check endpoint responds
- Can establish connection to SMTP port
- Authentication works with valid credentials
- Can send a simple test email
- Logs are being written
- Metrics endpoint accessible (when implemented)

## Code Coverage Requirements

### Coverage Targets
- **Unit Tests**: 85% minimum, 90%+ target
- **Integration Tests**: 70% of critical paths
- **Overall Coverage**: 80% minimum

### Coverage Tools
```bash
# Generate coverage report
go test -coverprofile=coverage.out ./...

# View coverage in browser
go tool cover -html=coverage.out

# Coverage by package
go test -coverprofile=coverage.out ./... && go tool cover -func=coverage.out

# Fail if coverage below threshold
go test -coverprofile=coverage.out ./... && \
    go tool cover -func=coverage.out | grep total | awk '{print $3}' | sed 's/%//' | \
    awk '{if ($1 < 80) exit 1}'
```

### Excluded from Coverage
- Generated code
- Main entry points (cmd/server/main.go) - tested via E2E
- Trivial getters/setters
- Test utilities and mocks

## CI/CD Testing Pipeline

### GitHub Actions Workflow

**On Every Push/PR**:
1. **Lint** - golangci-lint
2. **Unit Tests** - Fast feedback (<2 minutes)
3. **Integration Tests** - With test containers
4. **Security Scan** - gosec, nancy
5. **Code Coverage** - Upload to Codecov
6. **Build** - Verify builds on linux/amd64, linux/arm64

**On Main Branch**:
7. **E2E Tests** - Full workflow validation
8. **Performance Tests** - Regression detection
9. **Docker Build** - Multi-arch images
10. **Smoke Tests** - Against built artifacts

**Nightly**:
11. **Extended Performance Tests** - Long-running scenarios
12. **Security Penetration Tests** - Automated security testing
13. **Dependency Vulnerability Scan** - Check for CVEs

### CI/CD Configuration Example

```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.21'

      - name: Lint
        run: golangci-lint run --timeout 5m

      - name: Unit Tests
        run: go test -v -race -coverprofile=coverage.out ./...

      - name: Integration Tests
        run: go test -v -tags=integration ./test/integration/...

      - name: Code Coverage
        run: |
          go tool cover -func=coverage.out
          # Fail if below 80%
          go tool cover -func=coverage.out | grep total | awk '{if ($3+0 < 80.0) exit 1}'

      - name: Security Scan
        run: |
          go install github.com/securego/gosec/v2/cmd/gosec@latest
          gosec ./...

      - name: Build
        run: go build -o smtp-forwarder cmd/server/main.go

      - name: E2E Tests
        run: go test -v -tags=e2e ./test/e2e/...

      - name: Performance Tests
        run: go test -v -tags=performance -timeout 30m ./test/performance/...
```

## Testing Best Practices

### Do's ✅
- Write tests first (TDD)
- Use table-driven tests for multiple scenarios
- Test error paths, not just happy paths
- Use meaningful test names (TestFunctionName_Scenario_ExpectedBehavior)
- Mock external dependencies
- Use test fixtures for complex test data
- Run tests in parallel where possible
- Use testify/assert for better assertions
- Clean up resources in defer statements
- Test concurrent scenarios with race detector

### Don'ts ❌
- Don't skip writing tests
- Don't test implementation details
- Don't use sleep() for timing (use proper synchronization)
- Don't share state between tests
- Don't write tests that depend on execution order
- Don't mock everything (test real integrations when practical)
- Don't ignore flaky tests (fix them)
- Don't commit code without running full test suite
- Don't disable the race detector
- Don't sacrifice test quality for coverage percentage

## Manual Testing (SMTP Forwarder Specific)

While automated tests are primary, manual testing is useful for verification:

### Telnet Testing
```bash
telnet localhost 2587
EHLO localhost
AUTH PLAIN <base64_encoded_credentials>
MAIL FROM:<test@example.com>
RCPT TO:<recipient@example.com>
DATA
Subject: Test
From: test@example.com
To: recipient@example.com

Test message
.
QUIT
```

### PHPMailer Testing
- Integration testing with actual SMTP clients (PHPMailer examples provided in docs)
- Test with real email clients (Thunderbird, Outlook) when available

## Language and Documentation

Documentation is written in Romanian (RO). When working on this project:
- Maintain consistency with existing documentation language
- Comments in code should follow Go conventions (English is standard for Go code comments)
- Commit messages and technical communications can be in Romanian or English

## Roadmap Context

The SMTP Forwarder has a phased development plan:
- **Milestone 1**: Basic server, authentication, forwarding, .env config, basic logging
- **Milestone 2**: Rate limiting, retry queue, message persistence, web dashboard, Prometheus metrics
- **Milestone 3**: Multiple external servers, spam filtering, header modification, webhooks, REST API

When implementing features, consider the milestone context and maintain compatibility with future enhancements.

---

## Quick Reference: Expert Consultation Triggers

**Use this checklist to know when to consult each expert:**

### 🧪 Test Automation Expert - Consult When:
- [ ] Planning any new test suite (unit/integration/e2e/performance)
- [ ] Before writing first test for a component
- [ ] Tests are becoming hard to maintain
- [ ] Test coverage is high but tests feel brittle
- [ ] Implementing mocking strategy
- [ ] Tests are flaky or slow
- [ ] During test refactoring
- [ ] Test architecture needs review

**Quick Prompt**:
```
Task: "As a test automation expert following Uncle Bob's principles,
review [describe test scenario]. Critique F.I.R.S.T compliance,
maintainability, architecture, and suggest improvements."
```

### 🎯 Clean Code Expert - Consult When:
- [ ] After implementing any component (before marking complete)
- [ ] During REFACTOR phase of TDD cycle
- [ ] Function exceeds 20 lines
- [ ] Code feels complex or "smelly"
- [ ] Before submitting PR
- [ ] Considering design pattern usage
- [ ] Architecting new components
- [ ] High cyclomatic complexity detected

**Quick Prompt**:
```
Task: "Act as Uncle Bob. Review this code with strictest clean code
standards: [paste code]. Critique SOLID, code smells, naming, function
size, and provide refactoring recommendations."
```

### 🚀 CI/CD Expert - Consult When:
- [ ] Creating any `.github/workflows/*.yml` file
- [ ] Modifying existing workflows
- [ ] Adding new test types to pipeline
- [ ] Pipeline runs are slow
- [ ] Before adding deployment steps
- [ ] Implementing caching strategy
- [ ] Setting up automated releases
- [ ] Pipeline efficiency needs improvement

**Quick Prompt**:
```
Task: "As a CI/CD expert, review this GitHub Actions workflow:
[paste workflow or describe plan]. Critique efficiency, best practices,
security, caching, parallelization, and suggest improvements."
```

---

## Pre-Commit Checklist

Before committing ANY code, verify:

- [ ] **TDD Followed**: Tests written before implementation
- [ ] **Test Automation Expert**: Consulted for test architecture
- [ ] **Clean Code Expert**: Consulted for implementation review
- [ ] **All Tests Pass**: `go test -v -race ./...`
- [ ] **Coverage Met**: `>80%` overall, `>85%` for units
- [ ] **Linting Clean**: `golangci-lint run`
- [ ] **Formatted**: `go fmt ./...`
- [ ] **Race Detector**: No race conditions found
- [ ] **Security Scan**: `gosec ./...` passed

---

## Pre-PR Checklist

Before submitting Pull Request:

- [ ] **All Pre-Commit Checks**: Passed
- [ ] **CI/CD Expert**: Consulted if workflow changes
- [ ] **Integration Tests**: All passing
- [ ] **E2E Tests**: All passing
- [ ] **Performance Tests**: No regressions
- [ ] **Documentation**: Updated if needed
- [ ] **Commit Messages**: Include expert feedback notes
- [ ] **Branch Name**: Descriptive (feature/component-name, fix/issue-description)
- [ ] **PR Description**: Clear description of changes and expert consultations

---

## Git and Development Practices

- **Always use GitHub CLI** instead of plain git commands for GitHub operations
- **Use descriptive branch names**: `feature/smtp-authentication`, `fix/tls-handshake-error`, `test/integration-auth`
- **Commit message format**:
  ```
  type(scope): brief description

  - Detailed changes
  - Expert consultation notes
  - Coverage metrics
  ```
  Types: `feat`, `fix`, `test`, `refactor`, `docs`, `chore`, `perf`