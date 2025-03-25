# Digital Link Resolver (DLR) Design Documentation

## Table of Contents
1. [Introduction](#introduction)
2. [System Design Overview](#system-design-overview)
   - [High-Level Architecture](#high-level-architecture)
   - [Key Components and Tech Stacks](#key-components-and-tech-stacks)
   - [System Interaction Flow](#system-interaction-flow)
3. [Resolver Architecture Details](#resolver-architecture-details)
   - [Link Resolution Service](#link-resolution-service)
   - [Object Storage Infrastructure](#object-storage-infrastructure)
   - [Resolution Flow](#resolution-flow)
   - [Ensuring Performance](#ensuring-performance)
   - [Data Organization and Caching](#data-organization-and-caching)
   - [High Availability and Fault Tolerance](#high-availability-and-fault-tolerance)
4. [System Design Diagrams](#system-design-diagrams)
   - [Component Interaction Diagram](#component-interaction-diagram)
   - [Sequence Diagram](#sequence-diagram)
5. [Detailed Component Descriptions](#detailed-component-descriptions)
   - [Client Applications](#client-applications)
   - [Registry Systems](#registry-systems)
   - [Public DLR API](#public-dlr-api)
   - [Admin Write API](#admin-write-api)
   - [Business Logic Layer](#business-logic-layer)
   - [Repository Layer](#repository-layer)
   - [Object Storage](#object-storage)
6. [Implementation Details](#implementation-details)
   - [API Endpoints](#api-endpoints)
   - [Authentication and Authorization](#authentication-and-authorization)
   - [Link Resolution Methods](#link-resolution-methods)
   - [Storage Path Strategy](#storage-path-strategy)
   - [Data Format](#data-format)
   - [Error Handling](#error-handling)
7. [Additional Considerations](#additional-considerations)
   - [Security Enhancements](#security-enhancements)
   - [Scalability Strategies](#scalability-strategies)
   - [Monitoring and Logging](#monitoring-and-logging)
8. [Conclusion](#conclusion)
9. [Appendix: Data Structures](#appendix-data-structures)

---

## 1. [Introduction](#introduction)

The Digital Link Resolver (DLR) is a high-performance system designed to resolve identifiers (such as GS1 GTINs, ABNs, etc.) to lists of links containing relevant information about the identified items. This document outlines the comprehensive system design, with a focus on high-performance, horizontally scalable architecture, followed by detailed implementation guidelines. The goal is to provide a clear understanding of how the DLR operates and interacts with other systems, enabling seamless scalability, maintainability, and rapid response times.

---

## 2. [System Design Overview](#system-design-overview)

### 2.1 [High-Level Architecture](#high-level-architecture)

The Digital Link Resolver operates as a lightweight, high-performance service focused on resolving identifiers to links. It uses a hexagonal architecture pattern (ports and adapters) that clearly separates business logic from technical infrastructure concerns. This design enables the system to be horizontally scalable and cost-effective, primarily optimized for high-volume read operations with occasional writes from authorized registry systems.

### 2.2 [Key Components and Tech Stacks](#key-components-and-tech-stacks)

1. **Client Applications**
   - Web Browsers
   - Mobile Apps
   - Supply Chain Systems
   - Scanner Applications

2. **Public DLR API (Read-only)**
   - Node.js/Express
   - REST API endpoints
   - ISO and GS1 standard implementations

3. **Admin Write API (Authenticated)**
   - Secured endpoints for data management
   - Authentication middleware
   - Validation logic

4. **Business Logic Layer**
   - Domain Objects
   - Use Cases
   - Link Resolution Methods (ISO, GS1, etc.)

5. **Repository Layer**
   - Storage Abstraction
   - Path Construction Logic
   - Object Retrieval Optimization

6. **Object Storage**
   - S3/MinIO for production/development
   - JSON object storage
   - Hierarchical path organization

7. **Deployment Infrastructure**
   - Docker Compose (Development)
   - CDN, API Gateway, Lambda, S3 (Production)

### 2.3 [System Interaction Flow](#system-interaction-flow)

1. **Identifier Detection**
   - A user scans a barcode or inputs an identifier in a client application.

2. **Resolver Request**
   - The client application constructs a request to the Public DLR API with the identifier and optional parameters (such as link type).

3. **Scheme Detection**
   - The Public DLR API analyzes the identifier to determine its scheme (e.g., GS1 GTIN, ABN).

4. **Resolution Method Selection**
   - Based on the scheme, the appropriate link resolution method is selected.

5. **Path Construction**
   - The Repository Layer constructs the appropriate path in the object storage where the link list is stored.

6. **Link List Retrieval**
   - The link list is retrieved from object storage as a JSON object.

7. **Response Formatting**
   - The link list is formatted according to the appropriate standard and returned to the client.

8. **Client Processing**
   - The client application processes the link list and presents the available options to the user.

---

## 3. [Resolver Architecture Details](#resolver-architecture-details)

### 3.1 [Link Resolution Service](#link-resolution-service)

#### Overview

The Link Resolution Service is responsible for translating identifiers into lists of related links. It implements multiple resolution methods based on different standards (e.g., ISO, GS1) to handle a variety of identifier schemes. This service is designed to be stateless for horizontal scalability and optimized for high-performance read operations.

#### Resolution Methods

- **ISO Standard Method**: Implements the basic ISO standard for link resolution.
- **GS1 Digital Link Method**: Implements the GS1 standard for resolving GTINs and other GS1 identifiers.
- **Custom Extension Methods**: Framework for supporting additional identifier schemes and resolution methods.

#### Performance Optimization

- **Read Optimization**: The service is specifically optimized for read operations, which are the most common.
- **Stateless Design**: No session state is maintained, allowing for easy horizontal scaling.
- **Response Caching**: Common responses may be cached at multiple levels (CDN, API Gateway).

### 3.2 [Object Storage Infrastructure](#object-storage-infrastructure)

- **Data Organization**: Link lists are stored as JSON objects in a hierarchical path structure.
- **Path Construction**: Paths are constructed based on the identifier scheme and value to ensure efficient retrieval.
- **Storage Technology**: Uses object storage (S3/MinIO) for cost-effective, highly scalable storage.

### 3.3 [Resolution Flow](#resolution-flow)

1. **Receive Request**: The Public DLR API receives a request with an identifier.
2. **Validate Input**: The identifier is validated for format correctness.
3. **Determine Scheme**: The identifier scheme is determined (e.g., GS1 GTIN, ABN).
4. **Select Method**: The appropriate resolution method is selected.
5. **Construct Path**: The repository constructs the storage path for the identifier.
6. **Retrieve Object**: The JSON object containing the link list is retrieved from storage.
7. **Format Response**: The response is formatted according to the standard.
8. **Return Result**: The formatted response is returned to the client.

### 3.4 [Ensuring Performance](#ensuring-performance)

- **Path-Based Access**: Direct path-based access to objects rather than queries for fast retrieval.
- **Lightweight Processing**: Minimal processing during read operations.
- **Content Delivery Network**: Use of CDN for frequently accessed content.
- **Serverless Architecture**: Lambda functions that scale automatically with demand.

### 3.5 [Data Organization and Caching](#data-organization-and-caching)

- **Hierarchical Structure**: Data is organized in a hierarchical structure by scheme and identifier.
- **Multi-Level Caching**: Caching at CDN, API Gateway, and potentially Lambda levels.
- **Write-Through Strategy**: Updates are written through to the storage immediately.

### 3.6 [High Availability and Fault Tolerance](#high-availability-and-fault-tolerance)

- **Multi-Region Deployment**: Production deployment across multiple regions.
- **Read Redundancy**: Multiple paths to read the same data.
- **Graceful Degradation**: System continues to function with reduced capabilities during partial failures.

---

## 4. [System Design Diagrams](#system-design-diagrams)

### 4.1 [Component Interaction Diagram](#component-interaction-diagram)

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

### 4.2 [Sequence Diagram](#sequence-diagram)

```
sequenceDiagram
    participant Client
    participant PublicAPI as Public DLR API
    participant BizLogic as Business Logic
    participant Repo as Repository
    participant Storage as Object Storage
    
    Client->>PublicAPI: GET /resolver/{scheme}/{identifier}
    PublicAPI->>BizLogic: resolveIdentifier(scheme, identifier)
    BizLogic->>Repo: getLinkList(scheme, identifier)
    Repo->>Storage: getObject(path)
    Storage-->>Repo: JSON Object
    Repo-->>BizLogic: Link List
    BizLogic-->>PublicAPI: Formatted Response
    PublicAPI-->>Client: HTTP Response with Links
    
    Note over Client,Storage: Write Flow
    
    participant AdminAPI as Admin Write API
    participant Registry
    
    Registry->>AdminAPI: PUT /admin/{scheme}/{identifier}
    AdminAPI->>BizLogic: validateAndFormat(linkList)
    BizLogic->>Repo: saveLinkList(scheme, identifier, links)
    Repo->>Storage: putObject(path, data)
    Storage-->>Repo: Success
    Repo-->>BizLogic: Success
    BizLogic-->>AdminAPI: Success
    AdminAPI-->>Registry: HTTP 200 OK
```

---

## 5. [Detailed Component Descriptions](#detailed-component-descriptions)

### 5.1 [Client Applications](#client-applications)

Client applications interact with the DLR to resolve identifiers to links.

- **Responsibilities**:
  - Detect or input identifiers (e.g., scan barcodes, read QR codes).
  - Construct and send resolver requests.
  - Process and display the returned link lists.
  - Handle user interactions with the resolved links.

### 5.2 [Registry Systems](#registry-systems)

External systems that manage identifiers and their associated information.

- **Responsibilities**:
  - Maintain authoritative information about identifiers.
  - Authenticate and authorize users for management of their identifiers.
  - Push updates to the DLR through the Admin Write API.

### 5.3 [Public DLR API](#public-dlr-api)

The read-only API for resolving identifiers to link lists.

- **Responsibilities**:
  - Implement standard-compliant resolver endpoints.
  - Validate incoming requests.
  - Select the appropriate resolution method.
  - Return formatted responses.

### 5.4 [Admin Write API](#admin-write-api)

Secured API for managing link lists.

- **Responsibilities**:
  - Authenticate and authorize registry systems.
  - Validate incoming link list data.
  - Coordinate the storage of link lists.
  - Provide feedback on write operations.

### 5.5 [Business Logic Layer](#business-logic-layer)

Contains the core domain logic for the DLR.

- **Responsibilities**:
  - Implement link resolution methods.
  - Validate and format link lists.
  - Apply business rules for resolution.
  - Coordinate between APIs and repository.

### 5.6 [Repository Layer](#repository-layer)

Abstracts the storage operations from the business logic.

- **Responsibilities**:
  - Construct storage paths from identifiers.
  - Translate between domain objects and storage format.
  - Optimize read and write operations.
  - Handle storage-related errors.

### 5.7 [Object Storage](#object-storage)

Stores the link lists as JSON objects.

- **Responsibilities**:
  - Provide fast, reliable access to stored objects.
  - Scale horizontally to handle large volumes of data.
  - Maintain data durability and consistency.

---

## 6. [Implementation Details](#implementation-details)

### 6.1 [API Endpoints](#api-endpoints)

#### Public DLR API

##### `GET /resolver/{scheme}/{identifier}`

Resolves an identifier to a list of links.

**Parameters**
- `scheme`: The identifier scheme (e.g., `gtin`, `abn`).
- `identifier`: The identifier value.
- `linkType` (optional): Filter for specific link types.

**Response**

- **Success (200 OK)**

  ```json
  {
    "identifier": {
      "scheme": "gtin",
      "value": "9506000134352"
    },
    "links": [
      {
        "type": "productInfo",
        "href": "https://example.com/products/9506000134352",
        "title": "Product Information"
      },
      {
        "type": "safetyInfo",
        "href": "https://example.com/safety/9506000134352",
        "title": "Safety Information"
      }
    ]
  }
  ```

- **Failure**
  - `400 Bad Request`: Invalid scheme or identifier.
  - `404 Not Found`: Identifier not found.
  - `500 Internal Server Error`: Server error.

#### Admin Write API

##### `PUT /admin/{scheme}/{identifier}`

Updates the link list for an identifier.

**Headers**
- `Content-Type: application/json`
- `Authorization: Bearer <token>`

**Request Body**

```json
{
  "links": [
    {
      "type": "productInfo",
      "href": "https://example.com/products/9506000134352",
      "title": "Product Information"
    },
    {
      "type": "safetyInfo",
      "href": "https://example.com/safety/9506000134352",
      "title": "Safety Information"
    }
  ]
}
```

**Response**
- **Success (200 OK)**
  ```json
  {
    "message": "Link list updated successfully",
    "identifier": {
      "scheme": "gtin",
      "value": "9506000134352"
    }
  }
  ```

- **Failure**
  - `400 Bad Request`: Invalid link list format.
  - `401 Unauthorized`: Authentication failed.
  - `403 Forbidden`: Not authorized for this identifier.
  - `500 Internal Server Error`: Server error.

### 6.2 [Authentication and Authorization](#authentication-and-authorization)

- **Public API**: No authentication required for resolution.
- **Admin API**: JWT-based authentication with role-based authorization.
- **Registry Integration**: Support for OAuth 2.0 for registry system integration.

### 6.3 [Link Resolution Methods](#link-resolution-methods)

- **ISO Method**: Simple URL construction based on the ISO standard.
- **GS1 Method**: Implements the GS1 Digital Link standard with vocabulary support.
- **Custom Methods**: Extendable framework for additional resolution methods.

### 6.4 [Storage Path Strategy](#storage-path-strategy)

- **Path Format**: `/{scheme}/{partitioned-identifier}/{identifier}.json`
- **Partitioning**: Identifiers may be partitioned by prefix or other strategies to avoid hot spots.
- **Optimization**: Paths are designed to minimize traversal and enable direct access.

### 6.5 [Data Format](#data-format)

Link lists are stored as JSON objects with a standard schema:

```json
{
  "identifier": {
    "scheme": "gtin",
    "value": "9506000134352"
  },
  "links": [
    {
      "type": "productInfo",
      "href": "https://example.com/products/9506000134352",
      "title": "Product Information"
    }
  ],
  "metadata": {
    "lastUpdated": "2023-06-15T10:30:00Z",
    "updatedBy": "registry-system-id"
  }
}
```

### 6.6 [Error Handling](#error-handling)

- **Input Validation**: Comprehensive validation of all inputs.
- **Graceful Degradation**: Return partial results when possible during partial system failures.
- **Standardized Error Responses**: Consistent error formats across all APIs.
- **Retry Mechanism**: Automatic retries for transient failures when appropriate.

---

## 7. [Additional Considerations](#additional-considerations)

### 7.1 [Security Enhancements](#security-enhancements)

- **Rate Limiting**: Implement to prevent abuse of the Public API.
- **HTTPS Only**: Enforce HTTPS for all API endpoints.
- **IP Filtering**: Optional IP filtering for Admin API access.
- **Audit Logging**: Track all write operations for security review.

### 7.2 [Scalability Strategies](#scalability-strategies)

- **Read-Optimized Design**: Architecture is optimized for high-volume read operations.
- **Horizontal Scaling**: All components are designed to scale horizontally.
- **CDN Integration**: Use of CDN for caching frequently accessed content.
- **Regional Deployment**: Deploy to multiple regions for global performance.

### 7.3 [Monitoring and Logging](#monitoring-and-logging)

- **Usage Metrics**: Track resolution requests by scheme and identifier.
- **Performance Monitoring**: Monitor latency, error rates, and throughput.
- **Alert System**: Set up alerts for abnormal patterns or system issues.
- **Access Logs**: Maintain logs of all API access for troubleshooting.

---

## 8. [Conclusion](#conclusion)

The Digital Link Resolver (DLR) is designed to be a high-performance, horizontally scalable system for resolving identifiers to link lists. Its architecture prioritizes read performance, cost efficiency, and extensibility. The open-source core provides essential functionality, while future value-added services can extend its capabilities for specific business needs.

The design enables simple deployment for development and testing, while also supporting enterprise-grade deployment in cloud environments. By leveraging object storage and serverless computing, the system achieves high performance at a reasonable cost, making it suitable for organizations of all sizes.

---

## 9. [Appendix: Data Structures](#appendix-data-structures)

### Link List JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["identifier", "links"],
  "properties": {
    "identifier": {
      "type": "object",
      "required": ["scheme", "value"],
      "properties": {
        "scheme": { "type": "string" },
        "value": { "type": "string" }
      }
    },
    "links": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["type", "href"],
        "properties": {
          "type": { "type": "string" },
          "href": { "type": "string", "format": "uri" },
          "title": { "type": "string" }
        }
      }
    },
    "metadata": {
      "type": "object",
      "properties": {
        "lastUpdated": { "type": "string", "format": "date-time" },
        "updatedBy": { "type": "string" }
      }
    }
  }
}
```

### Storage Directory Structure

```
/
├── gtin/
│   ├── 95/
│   │   ├── 9506000134352.json
│   │   └── 9506000134369.json
│   └── 76/
│       └── 7612345678900.json
├── abn/
│   ├── 12/
│   │   └── 12345678901.json
│   └── 98/
│       └── 98765432109.json
└── custom-scheme/
    └── ...
``` 