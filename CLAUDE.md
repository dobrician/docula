# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Docula** is a microservices architecture project consisting of:

1. **Microservices Infrastructure** - A self-hosted architecture designed to run on a single bare-metal server (128 GB RAM) with future scalability to multiple servers/datacenters
2. **SMTP Forwarder** - A Go-based SMTP proxy/relay server that acts as an intermediary between SMTP clients and external SMTP servers

## Current State

This repository is currently in the documentation/planning phase. The `docs/` directory contains architectural specifications and implementation plans. No code has been implemented yet.

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

### TDD-First Development Workflow

**MANDATORY**: Before writing any production code, follow this workflow:

1. **Write Test First** (RED)
   - Create test file if it doesn't exist (`*_test.go`)
   - Write a test that describes the behavior you want
   - Run test and confirm it fails

2. **Implement Minimal Code** (GREEN)
   - Write just enough code to pass the test
   - Run test and confirm it passes
   - Commit with message: "feat: [feature] - tests passing"

3. **Refactor** (REFACTOR)
   - Improve code quality, structure, performance
   - Run tests to ensure they still pass
   - Commit with message: "refactor: [description]"

4. **Repeat** for next feature/behavior

**Example TDD Session**:
```bash
# 1. Write test
vim internal/auth/auth_test.go
go test ./internal/auth  # Should fail

# 2. Implement
vim internal/auth/auth.go
go test ./internal/auth  # Should pass

# 3. Refactor
vim internal/auth/auth.go
go test ./internal/auth  # Should still pass

# 4. Check coverage
go test -cover ./internal/auth
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
