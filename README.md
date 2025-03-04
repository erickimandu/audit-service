# Audit Service

<div align="center">

[![Build Status](https://img.shields.io/github/workflow/status/org/audit-service/main)](https://github.com/org/audit-service/actions)
[![Code Coverage](https://img.shields.io/codecov/c/github/org/audit-service/main)](https://codecov.io/gh/org/audit-service)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Docker Pulls](https://img.shields.io/docker/pulls/org/audit-service)](https://hub.docker.com/r/org/audit-service)
[![API Docs](https://img.shields.io/badge/API%20Docs-Swagger-green)](https://api.example.com/docs)

**A forensic-grade audit trail system with powerful search capabilities**

</div>

## 📋 Overview

The Audit Service is a robust, enterprise-grade system designed to maintain an immutable record of all activities within your application ecosystem. It provides comprehensive audit trails with advanced search capabilities, allowing for efficient investigation, compliance monitoring, and security incident response.

Built with a dual database/Elasticsearch search strategy, the service offers powerful search capabilities while maintaining high performance and availability, even during infrastructure disruptions.

### Why Use This Audit Service?

✅ **Regulatory Compliance**: Meet requirements for various regulations (SOX, GDPR, HIPAA, PCI-DSS)  
✅ **Security Monitoring**: Detect and investigate suspicious activities  
✅ **Incident Response**: Provide detailed forensic evidence for investigations  
✅ **Operational Intelligence**: Gain insights into system usage patterns  
✅ **Accountability**: Create clear trails of responsibility for all actions  

## 🌟 Key Features

### Comprehensive Audit Trails
- **Immutable Records**: Tamper-proof storage of all audit events
- **Detailed Context Capture**: Records who, what, when, where, and how
- **Snapshot Storage**: Captures before/after states for change operations
- **Cross-Service Correlation**: Links related activities across microservices

### Powerful Search Capabilities
- **Full-Text Search**: Find audit records containing specific text across all fields
- **Field Filtering**: Filter by service, action, actor, resource, and other fields
- **Fuzzy Matching**: Find results despite typos or variations
- **Type-Ahead Suggestions**: Autocomplete functionality for search fields
- **Relevance Ranking**: Results ordered by relevance to search query
- **Result Highlighting**: Shows where search terms matched in results

### Resilient Architecture
- **Dual Implementation Strategy**: Database + Elasticsearch for high availability
- **Event-Driven Design**: Kafka-based event collection for reliability
- **Graceful Degradation**: Falls back to database search if Elasticsearch is unavailable
- **Scalable Performance**: Horizontal scaling for high-volume environments

## 🚀 Getting Started

### Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
- [Docker](https://www.docker.com/get-started) and [Docker Compose](https://docs.docker.com/compose/install/)
- [PostgreSQL 14+](https://www.postgresql.org/download/) (or use Docker)
- [Redis 7.0+](https://redis.io/download) (or use Docker)
- [Kafka 3.0+](https://kafka.apache.org/downloads) (or use Docker)
- [Elasticsearch 7.17+](https://www.elastic.co/downloads/elasticsearch) (optional, or use Docker)

### Quick Start with Docker

The fastest way to get started is using Docker Compose:

```bash
# Clone the repository
git clone https://github.com/org/audit-service.git
cd audit-service

# Start the service and dependencies
docker-compose up -d
```

The service will be available at http://localhost:5001.

### Local Development Setup

```bash
# Clone the repository
git clone https://github.com/org/audit-service.git
cd audit-service

# Start the dependencies with Docker
docker-compose -f docker-compose.deps.yml up -d

# Restore dependencies
dotnet restore

# Apply database migrations
dotnet ef database update --project src/AuditService.Infrastructure

# Run the service
dotnet run --project src/AuditService
```

## 📐 Architecture

The Audit Service follows a clean, layered architecture based on Domain-Driven Design principles:

```
┌─────────────────────────────────────────────────────────┐
│                    API Layer                            │
│  ┌─────────────────┐ ┌─────────────────┐ ┌───────────┐  │
│  │  REST Controllers │ │  Search API     │ │  Health   │  │
└──┴─────────────────┴─┴─────────────────┴─┴───────────┴──┘
              │                 │                │
              ▼                 ▼                ▼
┌─────────────────────────────────────────────────────────┐
│                Application Layer                        │
│  ┌─────────────────┐ ┌─────────────────┐ ┌───────────┐  │
│  │  Audit Service   │ │  Search Service  │ │  Event    │  │
└──┴─────────────────┴─┴─────────────────┴─┴───────────┴──┘
              │                 │                │
              ▼                 ▼                ▼
┌─────────────────────────────────────────────────────────┐
│                  Domain Layer                           │
│  ┌─────────────────┐ ┌─────────────────┐ ┌───────────┐  │
│  │  Domain Models   │ │  Domain Events   │ │  Validation│  │
└──┴─────────────────┴─┴─────────────────┴─┴───────────┴──┘
              │                 │                │
              ▼                 ▼                ▼
┌─────────────────────────────────────────────────────────┐
│               Infrastructure Layer                      │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌────┐ │
│  │ Repository  │ │ Kafka       │ │Search       │ │Cache│ │
└──┴─────────────┴─┴─────────────┴─┴─────────────┴─┴────┴─┘
        │               │               │            │
        ▼               ▼               ▼            ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────┐
│  PostgreSQL  │ │    Kafka     │ │ Elasticsearch │ │Redis │
└──────────────┘ └──────────────┘ └──────────────┘ └──────┘
```

The Search Service uses a dual implementation strategy:

```
┌───────────────────┐      ┌─────────────────────────┐
│                   │      │                         │
│  Search API       │──────▶  Search Service         │
│  Controller       │      │  (Interface)            │
│                   │      │                         │
└───────────────────┘      └──────────────┬──────────┘
                                          │
                                          │
              ┌─────────────────────────────────────────────┐
              │                                             │
              ▼                                             ▼
┌─────────────────────────────┐            ┌─────────────────────────────┐
│                             │            │                             │
│  Database Search Service    │            │  Elasticsearch Search       │
│  (Basic Implementation)     │            │  Service (Advanced)         │
│                             │            │                             │
└──────────────┬──────────────┘            └──────────────┬──────────────┘
               │                                          │
               ▼                                          ▼
┌─────────────────────────────┐            ┌─────────────────────────────┐
│                             │            │                             │
│  PostgreSQL Database        │            │  Elasticsearch              │
│                             │            │                             │
└─────────────────────────────┘            └─────────────────────────────┘
```

## 🔍 Usage Examples

### Recording Audit Events

```csharp
// Inject the audit event producer
private readonly IAuditEventProducer _auditEventProducer;

// Record an audit event
await _auditEventProducer.SendAuditEventAsync(new AuditEventData
{
    ServiceId = "user_service",
    Action = "user.create",
    ActorId = "admin123",
    ActorType = "user",
    IpAddress = "192.168.1.1",
    ResourceType = "user",
    ResourceId = "user456",
    TraceId = Activity.Current?.Id,
    AdditionalData = new Dictionary<string, object>
    {
        { "email", "user@example.com" },
        { "department", "Sales" },
        { "userType", "employee" }
    },
    After = new Dictionary<string, object>
    {
        { "status", "active" },
        { "roles", new[] { "user", "reporter" } }
    }
});
```

### Searching Audit Records

```csharp
// Inject the search service
private readonly ISearchService _searchService;

// Search for specific user actions
var criteria = new AuditSearchCriteria
{
    SearchText = "password", // Full-text search
    ActorId = "admin123",    // Filter by actor
    ResourceType = "user",
    StartTime = DateTime.UtcNow.AddDays(-30),
    EnableFuzzySearch = true,
    ContextFilters = new Dictionary<string, string>
    {
        { "department", "Sales" }
    }
};

var results = await _searchService.SearchAuditRecordsAsync(criteria);
```

### REST API Examples

#### Search Audit Records

```http
POST /api/search/audit
Content-Type: application/json

{
  "searchText": "configuration change",
  "startTime": "2025-01-01T00:00:00Z",
  "endTime": "2025-02-01T00:00:00Z",
  "serviceId": "config_service",
  "actorId": "admin123",
  "resourceType": "system_settings",
  "contextFilters": {
    "environment": "production"
  },
  "enableFuzzySearch": true
}
```

#### Get Search Suggestions

```http
GET /api/search/suggest?field=serviceId&query=auth
```

## ⚙️ Configuration

The service is configured via environment variables or appsettings.json:

```json
{
  "ConnectionStrings": {
    "AuditDatabase": "Host=postgres;Database=audit_db;Username=audituser;Password=auditpass;",
    "Redis": "redis:6379,password=redispass"
  },
  "Kafka": {
    "BootstrapServers": "kafka:9092",
    "GroupId": "audit-service",
    "AuditTopic": "audit.events"
  },
  "Elasticsearch": {
    "Urls": ["http://elasticsearch:9200"],
    "AuditIndexName": "audit-records",
    "Username": "elastic",
    "Password": "changeme"
  },
  "IdentityServer": {
    "Authority": "http://identity-server:5000",
    "Audience": "audit_service"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

## 🔒 Security

The Audit Service implements several security measures:

- **Authentication**: OAuth2/JWT for secure API access
- **Authorization**: Role-based access control for audit data
- **Data Protection**: Encrypted storage for sensitive data
- **Immutability**: Tamper-proof audit records
- **Secure Communication**: TLS for all API endpoints
- **Input Validation**: Comprehensive request validation

## 📦 Deployment

### Docker

```bash
# Build the Docker image
docker build -t org/audit-service:latest .

# Run the container
docker run -p 5001:80 --name audit-service org/audit-service:latest
```

### Kubernetes

```bash
# Apply Kubernetes manifests
kubectl apply -f kubernetes/audit-service.yaml
```

### Environment Variables

Key environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `ASPNETCORE_ENVIRONMENT` | Runtime environment | Production |
| `ConnectionStrings__AuditDatabase` | PostgreSQL connection string | - |
| `ConnectionStrings__Redis` | Redis connection string | - |
| `Kafka__BootstrapServers` | Kafka server addresses | localhost:9092 |
| `Elasticsearch__Urls` | Elasticsearch server addresses | - |

## 📊 Monitoring

The service exposes the following monitoring endpoints:

- `/health`: Health check endpoint (returns 200 OK if healthy)
- `/health/ready`: Readiness check for Kubernetes
- `/health/live`: Liveness check for Kubernetes
- `/metrics`: Prometheus metrics endpoint

## 🧪 Testing

```bash
# Run unit tests
dotnet test tests/AuditService.Tests

# Run integration tests
dotnet test tests/AuditService.IntegrationTests
```

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/amazing-feature`
3. Commit your changes: `git commit -m 'Add some amazing feature'`
4. Push to the branch: `git push origin feature/amazing-feature`
5. Open a Pull Request

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 📞 Support

For support, please contact [support@example.com](mailto:support@example.com) or open an issue on GitHub.

---

<div align="center">

**Audit Service** - Secure, Scalable, Compliant

[Example Organization](https://example.com) | [Documentation](https://docs.example.com) | [API Reference](https://api.example.com/docs)

</div>
