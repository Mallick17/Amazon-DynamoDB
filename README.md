# Amazon-DynamoDB
Amazon DynamoDB is a fully managed, serverless, NoSQL database service provided by AWS, designed to deliver high performance, scalability, and reliability for applications requiring fast and predictable access to data. It supports both key-value and document data models, making it highly flexible for handling structured, semi-structured, or unstructured data.
- Below are the key concepts and features of DynamoDB, ideal for documentation and understanding its capabilities.

## Key Concepts of DynamoDB
### Serverless
- DynamoDB requires no server management. AWS handles provisioning, patching, and scaling, with pay-as-you-go pricing. It scales automatically to meet demand and can scale to zero without cold starts.

### NoSQL Database
- Supports key-value and document data models with a flexible, schema-less design. It does not support JOIN operations, encouraging denormalization, but offers strong read consistency and ACID transactions.

### Fully Managed
- AWS manages setup, configuration, maintenance, high availability, hardware provisioning, security, backups, and monitoring, allowing developers to focus on application development.

### Performance
- Delivers single-digit millisecond latency at any scale, optimized for high-performance workloads. It can handle over 10 trillion requests per day and peaks exceeding 20 million requests per second.
