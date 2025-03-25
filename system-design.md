# Digital Link Resolver (DLR) System Design

## High-Level Architecture

### Overview Diagram

```
                         ┌───────────────────────────────────────────────────────┐
                         │                                                       │
                         │                  Client Applications                  │
                         │                                                       │
                         └───────────────┬───────────────────────┬───────────────┘
                                         │                       │
                                         ▼                       ▼
┌───────────────────────┐      ┌───────────────────┐   ┌───────────────────┐
│                       │      │                   │   │                   │
│   Registry Systems    │      │   Public DLR API  │   │  Admin Write API  │
│   (GS1, ATO, etc.)    │──┬──▶│   (Read-only)     │   │  (Authenticated)  │
│                       │  │   │                   │   │                   │
└───────────────────────┘  │   └─────────┬─────────┘   └──────────┬────────┘
                           │             │                        │
                           │             │                        │
                           │             ▼                        ▼
                           │   ┌───────────────────────────────────────────┐
                           │   │                                           │
                           │   │           Business Logic Layer            │
                           │   │                                           │
                           │   └───────────────────┬───────────────────────┘
                           │                       │
                           │                       │
                           │                       ▼
                           │   ┌───────────────────────────────────────────┐
                           │   │                                           │
                           │   │           Repository Layer                │
                           │   │                                           │
                           │   └───────────────────┬───────────────────────┘
                           │                       │
                           │                       │
                           │                       ▼
                           │   ┌───────────────────────────────────────────┐
                           └──▶│                                           │
                               │        Object Storage (S3/MinIO)          │
                               │                                           │
                               └───────────────────────────────────────────┘
```

## Component Descriptions

### Client Applications
- End-user applications that need to resolve identifiers to links
- May include mobile apps scanning barcodes, web applications, or other systems
- Interact primarily with the Public DLR API

### Registry Systems
- External systems that own and manage identifiers (e.g., GS1 for GTINs, ATO for ABNs)
- Have their own authentication and management interfaces
- Push updates to the DLR through the Admin Write API or direct object storage access

### Public DLR API (Read-only)
- Implements ISO and GS1 standards for link resolution
- High-performance, horizontally scalable
- Provides endpoints for resolving identifiers to link lists
- Can filter by link type

### Admin Write API (Authenticated)
- Secured API for updating link data
- Used by registry systems or authorized administrators
- Includes endpoints for creating, updating, and deleting link lists

### Business Logic Layer
- Contains domain objects and use cases
- Implements the link resolution methods (ISO, GS1, etc.)
- Validates inputs and formats outputs

### Repository Layer
- Abstracts the storage operations
- Translates between domain objects and storage format
- Optimizes read/write patterns

### Object Storage (S3/MinIO)
- Stores link lists as JSON objects
- Organized in a hierarchical structure for fast access
- Horizontally scalable and cost-effective

## Deployment Architecture

### Development Environment
```
┌─────────────────────────────────────┐
│           Docker Compose            │
│                                     │
│  ┌────────────┐     ┌────────────┐  │
│  │            │     │            │  │
│  │    App     │     │   MinIO    │  │
│  │ Container  │     │  Object    │  │
│  │            │     │   Store    │  │
│  │            │     │            │  │
│  └────────────┘     └────────────┘  │
│                                     │
└─────────────────────────────────────┘
```

### Production Environment
```
┌───────────────┐
│               │
│    Users      │
│               │
└───────┬───────┘
        │
        ▼
┌───────────────┐     ┌───────────────┐
│               │     │               │
│     CDN       │────▶│  API Gateway  │
│               │     │               │
└───────────────┘     └───────┬───────┘
                              │
                              ▼
                      ┌───────────────┐
                      │               │
                      │    Lambda     │
                      │  Functions    │
                      │               │
                      └───────┬───────┘
                              │
                              ▼
                      ┌───────────────┐
                      │               │
                      │      S3       │
                      │               │
                      └───────────────┘
```

## Data Flow

1. **Identifier Resolution**:
   - Client requests link list for an identifier
   - Public API determines the scheme and link resolution method
   - Repository constructs the appropriate object path
   - Object is retrieved from storage and returned

2. **Link List Update**:
   - Registry system authenticates with Admin API
   - Admin API validates the update request
   - Business logic formats the link list
   - Repository writes to the appropriate location in storage

## Extension Points

### Future Value-Added Services
```
┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐
│                    │  │                    │  │                    │
│  Long-term         │  │  Big Data          │  │  Complex Event     │
│  Publishing SLA    │  │  Analytics         │  │  Processing        │
│                    │  │                    │  │                    │
└────────────────────┘  └────────────────────┘  └────────────────────┘
            │                     │                      │
            └─────────────────────┼──────────────────────┘
                                  │
                                  ▼
                      ┌────────────────────┐
                      │                    │
                      │  Core DLR System   │
                      │                    │
                      └────────────────────┘
```

## Credential Publishing (Future)
```
┌────────────────────┐
│                    │
│  Identity Owner    │
│                    │
└──────────┬─────────┘
           │
           ▼
┌────────────────────┐     ┌────────────────────┐
│                    │     │                    │
│  Credential        │────▶│  DLR with          │
│  Issuing System    │     │  Credential Store  │
│                    │     │                    │
└────────────────────┘     └────────────────────┘
                                     │
                                     ▼
                           ┌────────────────────┐
                           │                    │
                           │  End Users /       │
                           │  Verifiers         │
                           │                    │
                           └────────────────────┘
```
