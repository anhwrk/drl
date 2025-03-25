# Digital Link Resolver (DLR) System Design Documentation
## Table of Contents

- **CDN**: Serves high-traffic content with low latency
- **API Gateway**: Routes requests and handles API management
- **Lambda Functions**: Serverless execution of API handlers
- **S3 Storage**: Highly available, scalable object storage
- **Zero Infrastructure**: No servers to manage, scales automatically

---

## 9. [Future Value-Added Services](#future-value-added-services)

These commercial services can enhance the core open-source implementation:

### Long-term Publishing SLA
- Credential maintenance for extended periods (10+ years)
- Regular verification and validation
- Archival storage management

### Big Data Analytics
- Usage pattern analysis
- Resolution metrics and insights
- Cross-identifier correlation

### Complex Event Processing
- Real-time monitoring of resolution activity
- Trigger-based notifications
- Anomaly detection

### Security Services
- Virus scanning for uploaded content
- Credential verification
- Link validation and monitoring

---

## 10. [Appendix: Data Structures](#appendix-data-structures)

### Link List JSON Structure

```json
{
  "identifier": "string",
  "scheme": "string",
  "links": [
    {
      "linkType": "string",
      "url": "string",
      "title": "string",
      "mimeType": "string",
      "attributes": {
        "property1": "string",
        "property2": "string"
      }
    }
  ],
  "updated": "string (ISO 8601 datetime)"
}
```

### Link Types Registry

The system maintains a registry of standard link types:

| Link Type | Description | Common MIME Types |
|-----------|-------------|------------------|
| productInfo | General product information | text/html |
| certification | Product certifications | application/pdf |
| safetyData | Safety data sheets | application/pdf |
| sustainability | Sustainability information | text/html |
| warranty | Warranty information | text/html, application/pdf |
| instructions | User instructions | text/html, application/pdf |
| support | Technical support | text/html |
| recycling | Recycling information | text/html |

### Storage Path Patterns

| Scheme | Path Pattern | Example |
|--------|--------------|---------|
| gtin | /gtin/{prefix}/{identifier}/links.json | /gtin/95060/9506000134352/links.json |
| abn | /abn/{identifier}/links.json | /abn/51824753556/links.json |
| nlis | /nlis/{identifier}/links.json | /nlis/3ABCD123456/links.json |
