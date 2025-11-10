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

## Development Commands (Future)

When the SMTP Forwarder is implemented, these commands will be used:

```bash
# Install dependencies
go mod download

# Run in development
go run cmd/server/main.go

# Build for production
go build -o smtp-forwarder cmd/server/main.go

# Run tests (when implemented)
go test ./...

# Test specific package
go test ./internal/smtp
```

## Implementation Guidelines

### When Implementing SMTP Forwarder

1. **Configuration**: All configuration must be loaded from `.env` file, never hardcoded
2. **Security Requirements**:
   - Mandatory authentication for incoming connections
   - TLS/STARTTLS support required
   - Input validation for email addresses and content
   - Comprehensive logging of all connections and authentication attempts
3. **Error Handling**: All forwarding errors must be logged with context (from, to, timestamp, error)
4. **Session Management**: Each SMTP session must properly track `from`, `to[]`, and message data before forwarding

### When Implementing Microservices

1. **Containerization**: Each service must have its own Dockerfile
2. **Configuration**: Use Docker Compose for local development orchestration
3. **Load Balancing**: Route all external traffic through Nginx Proxy Manager
4. **Task Processing**: Use message broker (Valkey/RabbitMQ) for any background job processing
5. **Data Storage**: Use MongoDB for JSON document storage

## Testing Approach

### SMTP Forwarder Testing
- Manual testing with telnet for SMTP protocol verification
- Integration testing with actual SMTP clients (PHPMailer examples provided)
- Unit tests for authentication, session management, and forwarding logic

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
