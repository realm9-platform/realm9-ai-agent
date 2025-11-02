# Realm9 AI Agent System

> Multi-layer AI agent architecture for intelligent Terraform infrastructure management and Kubernetes observability

[![License](https://img.shields.io/badge/license-Proprietary-blue.svg)](LICENSE)
[![AI](https://img.shields.io/badge/ai-Multi--LLM%20%7C%20BYOK-purple)](https://realm9.app)
[![Protocol](https://img.shields.io/badge/protocol-MCP-green)](https://modelcontextprotocol.io/)
[![Platform](https://img.shields.io/badge/platform-Realm9-blue)](https://github.com/realm9-platform/realm9)

## Overview

The Realm9 AI Agent System is a comprehensive implementation combining three key components:

1. **AI Terraform Code Editor** - LLM-powered conversational interface for infrastructure-as-code with BYOK (Bring Your Own Key)
2. **Model Context Protocol (MCP) Server** - Standardized tool provider with 45+ infrastructure management tools
3. **Kubernetes Observability Agent** - Self-deploying cluster monitoring with OTLP collectors

This architecture enables natural language infrastructure management while maintaining security, auditability, and GitOps workflows.

### Bring Your Own Key (BYOK)
Realm9 supports a **Bring Your Own Key** model, allowing you to use your own API keys from your preferred LLM provider. This ensures:
- **Data sovereignty** - Your infrastructure conversations stay within your LLM account
- **Cost control** - You manage and optimize your LLM spending directly
- **Provider choice** - Switch between providers based on your needs
- **Compliance** - Meet data residency and privacy requirements

**Supported LLM Providers**:
- OpenAI (GPT-4o, GPT-4o-mini, GPT-5)
- Anthropic (Claude 4.5 Sonnet, Claude 4.1 Opus, Claude 4.5 Haiku)
- Azure OpenAI (all OpenAI models via Azure)
- Google Vertex AI (Gemini models) - Coming Q1 2025
- AWS Bedrock (Claude, Llama models) - Coming Q1 2025

## System Architecture

```
┌────────────────────────────────────────────────────────────┐
│                     Realm9 Platform UI                      │
│  ┌──────────────┬──────────────┬─────────────────────────┐│
│  │ Terraform    │   Agent      │   Observability         ││
│  │ Editor       │   Chat       │   Dashboard             ││
│  └──────────────┴──────────────┴─────────────────────────┘│
├────────────────────────────────────────────────────────────┤
│                      Agent Server                           │
│  ┌──────────────────────────────────────────────────────┐ │
│  │  LLM Integration + Tool Call Processing             │ │
│  │  - Conversation management (Redis-backed)            │ │
│  │  - Iterative tool execution                          │ │
│  │  - Frontend/backend tool separation                  │ │
│  └──────────────────────────────────────────────────────┘ │
├────────────────────────────────────────────────────────────┤
│              MCP Server (Model Context Protocol)            │
│  ┌─────────────┬────────────────┬─────────────────────┐  │
│  │  Database   │  File System   │   Terraform Exec    │  │
│  │  Queries    │  Management    │   & Monitoring      │  │
│  │  (45+ standardized tools)                           │  │
│  └─────────────┴────────────────┴─────────────────────┘  │
├────────────────────────────────────────────────────────────┤
│            Redis State Layer (TTL-managed)                  │
│  - Unified editor state (files, UI)                        │
│  - Chat history                                             │
│  - Agent sessions                                           │
│  - Tool call queues                                         │
├────────────────────────────────────────────────────────────┤
│                  PostgreSQL Database                        │
│  - Agent registrations (Kubernetes clusters)               │
│  - Observability services & deployments                    │
│  - Terraform projects, workspaces, runs                    │
├────────────────────────────────────────────────────────────┤
│              Kubernetes Cluster (User's)                    │
│  ┌──────────────────────────────────────────────────────┐ │
│  │  realm9-agent (Helm deployed)                        │ │
│  │  - Config polling                                     │ │
│  │  - OTLP collector deployment                         │ │
│  │  - Heartbeat reporting                               │ │
│  └──────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────┘
```

## Component 1: AI Terraform Code Editor

### What It Does

The AI Terraform Code Editor provides a conversational interface for managing infrastructure-as-code. Engineers can describe what they want to build in natural language, and the AI agent generates, modifies, and executes Terraform code.

### Key Features

**Natural Language to Terraform**
- Engineers describe infrastructure requirements in plain English
- AI agent reads existing project files and understands current state
- Generates terraform configuration following best practices
- Updates files in editor with validation
- Explains changes and provides guidance

**Intelligent Code Understanding**
- Parses existing Terraform to understand current infrastructure
- Preserves existing resources when adding new ones
- Follows Terraform best practices
- Validates before making changes

**GitOps Integration**
- All changes sync to Git repository
- Commit messages generated by AI
- Branch management support
- Pull request workflow ready

### Architecture

The agent system uses a multi-layer architecture:
- **Agent Server**: Manages LLM conversations and tool orchestration
- **Tool Execution**: Separates backend tools (immediate execution) from frontend tools (user-in-the-loop)
- **State Management**: Redis-based ephemeral state with automatic TTL cleanup
- **Security**: Multi-tenant isolation with organization-scoped access

## Component 2: Model Context Protocol (MCP) Server

### What It Does

The MCP Server provides a standardized interface for AI agents to access infrastructure management tools. It implements the [Model Context Protocol](https://modelcontextprotocol.io/) specification, allowing the agent to discover and call tools without knowing implementation details.

### Why MCP?

**Tool Abstraction**:
- Agent doesn't need direct database or system access
- Tools can be upgraded without changing agent code
- Consistent interface across all tools
- Easy to add new capabilities

**Standardization**:
- MCP is an emerging standard for AI tool access
- Works with any LLM provider (OpenAI, Anthropic, Azure OpenAI, etc.)
- Portable across agent frameworks
- Enables easy switching between LLM providers

### Tool Categories

The MCP server provides 45+ tools across several categories:
- **Database Query Tools**: Project details, workspace information, cloud credentials
- **File Management Tools**: Terraform file operations, Git status, file tree navigation
- **Execution & Monitoring Tools**: Terraform plan/apply, run logs, health checks
- **Infrastructure Tools**: Validation, deployment, Git operations

### Security Model

- Agent cannot bypass tool interface to access systems directly
- All queries filtered by organization for multi-tenant isolation
- Redis TTL auto-cleanup prevents data leakage
- No cross-project or cross-organization access possible

## Component 3: Kubernetes Observability Agent

### What It Does

The Kubernetes Observability Agent is a self-deploying monitoring system that runs inside user clusters. Once installed via Helm, it autonomously deploys OpenTelemetry collectors for each observability service, forwards logs/metrics/traces to RO9 Observability, and reports health status.

### Agent Lifecycle

1. User creates observability service in Realm9 UI
2. System generates installation command with API key
3. User runs Helm command in their cluster
4. Agent starts and authenticates with API key
5. Agent polls for configuration updates
6. Agent deploys OTLP collectors as needed
7. Collectors forward telemetry to RO9
8. Agent sends heartbeat with deployment status
9. UI shows real-time status and health

### Security Features

**API Key Security**:
- Cryptographically random generation
- SHA-256 hashed storage (never plaintext)
- HTTPS-only transmission
- One-time display at creation
- Rotation supported via UI

**Network Security**:
- Agents make outbound calls only (no inbound firewall rules needed)
- No webhooks or inbound connections required
- Works across NAT, firewalls, and air-gapped environments

## Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **AI Models (BYOK)** | OpenAI, Anthropic, Azure OpenAI | Intelligence & reasoning with your API keys |
| **Tool Protocol** | Model Context Protocol (MCP) | Standardized tool interface |
| **Agent Runtime** | Express.js (Node.js) | HTTP server for agent logic |
| **MCP Server** | Express.js + StreamableHTTP | Tool provider |
| **State Management** | Redis (ioredis) | Session, chat, editor state |
| **Database** | PostgreSQL + Prisma | Agent registry, projects, services |
| **Container Runtime** | Docker | Agent & collector images |
| **Orchestration** | Kubernetes | Job execution, agent deployment |
| **Package Manager** | Helm 3 | Kubernetes deployment |
| **Version Control** | GitHub/GitLab APIs | Git integration |
| **Observability** | OpenTelemetry (OTLP) | Logs, metrics, traces collection |

## Key Architectural Patterns

### Pattern 1: Tool Abstraction via MCP
- Agent discovers tools at runtime from MCP server
- Tools are black boxes to the agent (no implementation knowledge)
- Easy to add new tools without modifying agent code
- Standardized error handling and result formatting

### Pattern 2: Frontend/Backend Tool Separation
- **Backend tools**: Execute immediately, return data
- **Frontend tools**: Pause agent, request UI execution, resume with result
- Enables human-in-the-loop workflows
- Maintains responsive UI during long operations

### Pattern 3: Redis-Centric Ephemeral State
- All session state in Redis (not database)
- TTL-based auto-cleanup (no manual cleanup needed)
- Fast access (sub-millisecond latency)
- Scales horizontally (Redis Cluster)

### Pattern 4: Polling-Based Agent Communication
- Agents make OUTBOUND calls only (no inbound firewall rules)
- Backend never calls agent (no webhook complexity)
- Simple deployment (no load balancer, ingress, certificates)
- Works across NAT, firewalls, air-gapped environments

### Pattern 5: Multi-Tenant Isolation
- All agent registrations scoped to organization
- Services filtered by organization in all queries
- API keys hashed and compared securely
- No cross-organization data leakage possible

## Benefits

### For Platform Engineering Teams
- **Reduced Complexity**: Single platform for environment management, Terraform, and observability
- **Faster Onboarding**: Natural language interface lowers barrier to entry
- **Cost Savings**: Bring your own LLM keys, pay only for what you use
- **Data Control**: All infrastructure conversations stay in your LLM account

### For Organizations
- **Compliance Ready**: BYOK model meets data residency requirements
- **Vendor Flexibility**: Switch between LLM providers without platform changes
- **Security First**: Multi-tenant isolation, encrypted keys, audit logging
- **Scalability**: Proven architecture handles enterprise workloads

## Security Considerations

### API Key Management
- Cryptographically random generation
- SHA-256 hashed in database (never plaintext)
- HTTPS-only transmission
- One-time display at creation
- Rotation supported via UI

### Data Protection
- All agent configurations filtered by organization
- Redis TTL auto-cleanup prevents abandoned data
- No sensitive data in Redis (keys, credentials)
- Database encryption at rest (provider-dependent)

### Network Security
- Agents make outbound calls only
- No inbound connections to user clusters
- API endpoints require authentication
- Rate limiting on all public endpoints

## Roadmap

### Current (v1.0)
- [x] AI Terraform code editor with multi-LLM support (BYOK)
- [x] OpenAI (GPT-4o, GPT-4o-mini, GPT-5)
- [x] Anthropic (Claude 4.5 Sonnet, Claude 4.1 Opus, Claude 4.5 Haiku)
- [x] Azure OpenAI (all OpenAI models via Azure)
- [x] MCP server with 45+ tools
- [x] Kubernetes agent deployment
- [x] OTLP collector auto-deployment
- [x] Heartbeat & health monitoring
- [x] Redis-based session management

### Q1 2025
- [ ] Google Vertex AI (Gemini models)
- [ ] AWS Bedrock (Claude, Llama models)
- [ ] Advanced Terraform plan analysis
- [ ] Multi-region agent support
- [ ] Prometheus metrics export
- [ ] Custom collector configurations

### Q2 2025
- [ ] Azure AKS native support
- [ ] GCP GKE native support
- [ ] Agent auto-update mechanism
- [ ] Advanced RBAC for agent tools
- [ ] Cost optimization recommendations

## Getting Started

For more information about deploying Realm9, visit:
- [Realm9 Platform](https://github.com/realm9-platform/realm9) - Main platform repository
- [RO9 Observability](https://github.com/realm9-platform/ro9-observability) - Log analytics platform
- [Realm9 Multi-Cloud](https://github.com/realm9-platform/realm9-multi-cloud) - Multi-cloud management
- [Realm9 Security](https://github.com/realm9-platform/realm9-enterprise-security) - Security architecture

### Contact

For enterprise deployments, custom integrations, or technical support:
- **Website**: https://realm9.app
- **Email**: sales@realm9.app

## License

Copyright © 2025 Realm9. All rights reserved.

---

**Realm9 AI Agent System** - *Intelligent Infrastructure Management*

Part of the [Realm9 Platform](https://github.com/realm9-platform/realm9)
