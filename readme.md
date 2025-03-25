# Digital Link Resolver (DLR)

## Overview

The Digital Link Resolver (DLR) is a lightweight, high-performance system for resolving identifiers to links. It serves as an open-source reference implementation of the ISO standard for link resolution while supporting various identifier schemes such as GS1 GTINs, ABNs, and others.

This implementation prioritizes speed, horizontal scalability, and cost-effectiveness. It focuses on delivering the core resolver functionality first, with a roadmap for additional capabilities in later releases.

## Key Features

### MVP (Current Release)
- ISO standard-compliant link resolution
- GS1 Digital Link support
- Fast, performant read operations
- Horizontally scalable architecture
- RESTful API for resolving identifiers to link lists

### Planned Future Enhancements
- Write API for managing link lists
- Credential publishing and retrieval
- Advanced administrative features
- Analytics and monitoring tools
- Link validation and verification

## Architecture

The DLR is built using a hexagonal architecture pattern (ports and adapters) with:

- **API Layer**: Routes requests to appropriate use cases
- **Business Logic**: Implements resolution logic using domain objects
- **Repository Layer**: Interfaces with storage system 
- **Storage**: Uses object storage (S3/MinIO) rather than traditional databases for performance and scalability

### Development Environment

Docker Compose
├── App Container (Node.js/Express)
│   ├── Domain Objects
│   ├── Use Cases
│   ├── Repository
│   └── Unit Tests
└── MinIO (Object Store)

### Production Environment
```
Client Request
└── CDN
    └── API Gateway
        └── Lambda Functions
            └── S3 Object Storage
```

## Getting Started

### Prerequisites
- Docker and Docker Compose
- Node.js (v14 or later)
- AWS CLI (for deployment to production)

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/digital-link-resolver.git
   cd digital-link-resolver
   ```

2. Start the development environment:
   ```bash
   docker-compose up -d
   ```

3. Access the API documentation:
   ```
   http://localhost:3000/api-docs
   ```

## API Reference

### Resolver API
- `GET /resolver/{scheme}/{identifier}` - Resolve an identifier to a list of links
- `GET /resolver/{scheme}/{identifier}?linkType={type}` - Resolve an identifier to links of a specific type

## Usage Examples

### Resolving a GS1 GTIN
```bash
curl -X GET "http://localhost:3000/resolver/gtin/9506000134352"
```

### Resolving an identifier with a specific link type
```bash
curl -X GET "http://localhost:3000/resolver/gtin/9506000134352?linkType=productInfo"
```

## Development

### Running Tests
```bash
npm test
```

### Building for Production
```bash
npm run build
```

## Deployment

The DLR is designed to be deployed using serverless architecture for optimal performance and cost-effectiveness:

1. Build the project
2. Deploy to your AWS environment using the provided CloudFormation templates
3. Configure your CDN to point to the API Gateway endpoint

Detailed deployment instructions can be found in the [Deployment Guide](./docs/deployment.md).

## License

This project is licensed under the [MIT License](LICENSE).

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details on how to contribute to this project.

## Acknowledgements

- ISO for the Digital Link standard
- GS1 for the Digital Link specification
- All contributors to this open-source implementation
