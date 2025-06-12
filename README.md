# Amazon-DynamoDB
Amazon DynamoDB is a fully managed, serverless, NoSQL database service provided by AWS, designed to deliver high performance, scalability, and reliability for applications requiring fast and predictable access to data. It supports both key-value and document data models, making it highly flexible for handling structured, semi-structured, or unstructured data.

## Use Cases
**Ideal for applications requiring high scalability, such as:**
- Financial services (e.g., live trading, loan management, transaction ledgers)
- Gaming (e.g., game state, leaderboards)
- Streaming (e.g., metadata indexing, recommendations)
- IoT (e.g., real-time data processing)
- Mobile apps and content management systems

## Key Concepts of DynamoDB
1. Serverless
2. NoSQL Database
3. Fully Managed
4. Performance
5. Scalability
6. Capabilities
7. Security
8. Resilience
9. Pricing
10. Service Integration 

<details>
  <summary>Click to View Explained Componenets of DynamoDB</summary>
  
### Serverless
- DynamoDB requires no server management. AWS handles provisioning, patching, and scaling, with pay-as-you-go pricing. It scales automatically to meet demand and can scale to zero without cold starts.

### NoSQL Database
- Supports key-value and document data models with a flexible, schema-less design. It does not support JOIN operations, encouraging denormalization, but offers strong read consistency and ACID transactions.

### Fully Managed
- AWS manages setup, configuration, maintenance, high availability, hardware provisioning, security, backups, and monitoring, allowing developers to focus on application development.

### Performance
- Delivers single-digit millisecond latency at any scale, optimized for high-performance workloads. It can handle over 10 trillion requests per day and peaks exceeding 20 million requests per second.

### Scalability 
- Scales horizontally by automatically distributing data across multiple servers, supporting virtually unlimited storage and throughput. It offers two capacity modes: on-demand (automatic scaling) and provisioned (manual capacity settings).

### Capabilities
- _**Global Tables:**_ Enables multi-active replication across AWS regions for low-latency access and 99.999% availability.
- **_ACID Transactions:_** Supports atomic, consistent, isolated, and durable transactions across one or more tables, suitable for financial transactions or order processing.
- _**Change Data Capture (CDC)**:_ Uses DynamoDB Streams and Amazon Kinesis to capture data changes for real-time processing.
- _**Secondary Indexes**:_ Global and local secondary indexes enable flexible querying without impacting performance.

### Security
- Integrates with AWS Identity and Access Management (IAM) for fine-grained access control.
- Encrypts data at rest using AWS Key Management Service (KMS), with options for AWS-owned keys (free) or customer-managed keys (fee-based).
- Complies with standards like HIPAA, PCI DSS, and GDPR.

### Resilience
- Replicates data across three Availability Zones for 99.99% availability.
- Offers continuous backups with point-in-time recovery (up to 35 days) and on-demand backups for additional protection.

### Pricing
- Charges based on read capacity units (RCUs), write capacity units (WCUs), and storage.
- On-demand mode charges per operation; provisioned mode allows predefined throughput.
- Free tier includes 25 GB of storage, 25 WCUs, and 25 RCUs, supporting up to 200 million requests per month.

### Service Integrations
- Integrates with AWS Lambda for serverless compute (e.g., triggers on data changes).
- Supports data import/export with Amazon S3.
- Enables zero-ETL integration with Amazon Redshift and OpenSearch for analytics.
- DynamoDB Accelerator (DAX) provides caching for up to 10x read performance improvement.
- _**Access Methods:**_ Supports AWS Management Console, AWS CLI, NoSQL Workbench, and DynamoDB APIs for interaction.

</details>

---

## Comparison Between DynamoDB and RDS
Amazon RDS (Relational Database Service) is a managed relational database service supporting SQL databases like MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Amazon Aurora. While both DynamoDB and RDS are fully managed by AWS, they cater to different needs based on data models, scalability, and use cases. The table below compares their key features for clear documentation.

<details>
  <summary>Comparison Table Between DynamoDB and RDS</summary>

### DynamoDB vs Amazon RDS – Feature Comparison

| **Feature**      | **DynamoDB**                                                                    | **Amazon RDS**                                                                                       |
| ---------------- | ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Type**         | NoSQL, fully managed                                                            | Relational (SQL), supports multiple engines (Aurora, MySQL, MariaDB, PostgreSQL, Oracle, SQL Server) |
| **Data Model**   | Key-value and document                                                          | Relational, with tables, rows, and columns                                                           |
| **Schema**       | Flexible, schema-less                                                           | Predefined schema                                                                                    |
| **Scalability**  | Horizontal scaling, virtually unlimited storage and throughput                  | Vertical scaling, or horizontal with read replicas                                                   |
| **Performance**  | Single-digit millisecond latency, optimized for high-throughput NoSQL workloads | Optimized for relational queries and transactions, up to 40,000 IOPS (Aurora)                        |
| **Use Cases**    | Real-time bidding, shopping carts, mobile apps, gaming, IoT, unstructured data  | Enterprise apps, CRM, e-commerce, data warehouses, traditional apps                                  |
| **Availability** | Replicated across 3 Availability Zones, 99.99% SLA; global tables (99.999%)     | Multi-AZ deployments optional, active-passive failover, 99.9% SLA                                    |
| **Security**     | IAM integration, AWS KMS encryption (AWS-owned key free)                        | IAM for MySQL/PostgreSQL, AWS KMS encryption, SSL support                                            |
| **Backups**      | Point-in-time recovery (PITR) up to 35 days, on-demand backups                  | PITR via automatic backups, snapshots stored in Amazon S3                                            |
| **Storage Size** | Virtually unlimited                                                             | Aurora: 128 TB, MySQL/MariaDB/Oracle/PostgreSQL: 64 TB, SQL Server: 16 TB                            |
| **Querying**     | Fast key-based access, secondary indexes, limited join support                  | Complex SQL queries, including joins and advanced data manipulations                                 |
| **Pricing**      | Pay for reads (4KB), writes (1KB), storage; on-demand/provisioned modes         | Monthly fee per instance, pay-as-you-go, reserved instances cheaper                                  |
| **Consistency**  | Eventual or strong consistency options                                          | Strong consistency (ACID transactions)                                                               |
| **Maintenance**  | Fully managed, no manual intervention needed                                    | Fully managed, but some engine-specific configurations required                                      |

---

### Detailed Comparison Insights
- **Data Model and Schema**
  - **DynamoDB:** Its schema-less design allows for dynamic data structures, making it ideal for applications with evolving requirements, such as mobile apps or IoT. Data is stored as key-value pairs or JSON-like documents.
  - **RDS:** Requires a predefined schema, enforcing structured data with tables, rows, and columns. This is suitable for applications needing relational integrity, like financial systems.
    
- **Scalability**
  - **DynamoDB:** Scales horizontally by automatically partitioning data across servers, supporting massive datasets and high request rates (e.g., over 20 million requests per second).
  - **RDS:** Primarily scales vertically by upgrading instance types. Horizontal scaling is possible with read replicas, but it’s less seamless than DynamoDB’s approach.

- **Performance**
  - **DynamoDB:** Optimized for NoSQL workloads, delivering low-latency access for key-based queries. DAX caching can reduce read times from milliseconds to microseconds.
  - **RDS:** Excels in relational workloads, supporting complex SQL queries and transactions. Performance depends on the database engine and instance type (e.g., Aurora offers up to 40,000 IOPS).

- **Use Cases**
  - **DynamoDB:** Best for high-traffic, unstructured data applications like real-time bidding, gaming, or IoT. It powers Amazon’s Prime Day and Alexa, handling over 1 billion requests per hour.
  - **RDS:** Suited for structured data applications requiring complex queries, such as CRM, e-commerce, or data warehousing.
    
- **Availability and Resilience**
  - **DynamoDB:** 3 AZ replication by default, Automatically replicates data across three Availability Zones, with global tables offering multi-region replication for higher availability.
  - **RDS:**  Multi-AZ deployments are optional, using active-passive failover, which may incur additional costs and complexity.

- **Cost**
  - **DynamoDB:** Charges based on read/write operations and storage, with on-demand pricing offering flexibility but potentially higher costs for unpredictable workloads.
  - **RDS:** Charges per instance, with reserved instances offering cost savings for predictable workloads. Storage and I/O operations add to the cost.

---

### Choosing Between DynamoDB and RDS
- **_Choose DynamoDB when:_**
  - Your application handles unstructured or semi-structured data.
  - You need high scalability and low-latency performance for large-scale workloads.
  - Serverless architecture and minimal operational overhead are priorities.
  - Examples: Gaming platforms, IoT data processing, real-time analytics.
 
- **_Choose RDS when:_**
  - Your application requires structured data and complex SQL queries or joins.
  - You need a relational database with strong consistency for transactions.
  - You’re using traditional enterprise applications or existing SQL-based systems.
  - Examples: CRM systems, e-commerce platforms, financial applications.
 
### Example Use Case Scenarios
- **DynamoDB:** A mobile gaming company uses DynamoDB to store player profiles and game states, leveraging its scalability to handle millions of concurrent users during peak events.
- **RDS**: An e-commerce platform uses RDS (MySQL) to manage product catalogs, customer data, and order transactions, relying on SQL joins for reporting and analytics.

---

</details>

---

## Key Features of DynamoDB

### 1. Capacity Modes
DynamoDB controls how much data your app can read or write using **capacity modes**.

- **On-Demand Mode**:
  - No setup needed—DynamoDB adjusts to your app’s needs.
  - Pay only for what you use.
  - Great for apps with unknown traffic (e.g., a new game).
  - **Example**: Handles 1,000 users without any manual changes.

- **Provisioned Mode**:
  - You set limits for reads and writes.
  - Cheaper if you know your traffic (e.g., 100 users/hour).
  - **Warning**: Too-low limits cause **throttling** (requests fail).
  - **Fix**: Use **auto-scaling** to adjust limits automatically.
  - **Free Tier**: 25 read and 25 write units (enough for small apps).

### 2. Consistency Models
DynamoDB keeps multiple data copies for safety. You choose how “up-to-date” your data is when reading.

- **Eventual Consistency** (Default):
  - Faster and cheaper.
  - Updates spread slowly, so you might see old data briefly.
  - **Example**: Delete a profile, but it shows up for a second before disappearing.
  - Best for apps where speed matters (e.g., social media).

- **Strong Consistency**:
  - Slower, uses more capacity (2x eventual).
  - Guarantees the latest data.
  - **Example**: See the latest balance after a bank transfer.
  - Add a parameter to your read request to use this.

**Note**: 1 strong consistent read = 2 eventual consistent reads.

### 3. Read and Write Requests
DynamoDB measures work in **read request units (RRUs)** or **write request units (WRUs)** for on-demand mode, and **read capacity units (RCUs)** or **write capacity units (WCUs)** for provisioned mode. Item size affects costs.

- **Read Requests**:
  - **GetItem**: Reads one item (up to 400 KB).
    - 1 RRU/RCU = 1 strong read or 2 eventual reads of 4 KB.
    - **Example**: 2.5 KB item = 4 KB (1 RRU/RCU strong, 0.5 RRU/RCU eventual).
    - 9 KB item = 12 KB (3 RRU/RCUs strong, 1.5 RRU/RCUs eventual).
  - **Transactional Read**: All operations succeed or fail together. Uses 2x units.
  - **BatchGetItem**: Reads up to 100 items. Each rounds to 4 KB.
    - **Example**: Items of 1 KB, 5 KB, 9.5 KB = 4 KB + 8 KB + 12 KB = 24 KB.
  - **Query**: Reads items with the same key. Rounds to 4 KB.
    - **Example**: 49 KB total = 52 KB.
  - **Scan**: Reads all items. Uses capacity for all accessed items.

- **Write Requests**:
  - **PutItem**: Writes or overwrites one item.
    - 1 WRU/WCU = 1 write of 1 KB.
    - **Example**: 5 KB item = 5 WRU/WCUs. 0.5 KB = 1 WRU/WCU.
  - **Transactional Write**: All operations succeed or fail together. Uses 2x units.
  - **BatchWriteItem**: Writes/deletes up to 25 items (max 16 MB).
  - **UpdateItem**: Changes one item. Uses larger size (before or after).
    - **Example**: 3 KB to 5 KB = 5 WRU/WCUs.
  - **DeleteItem**: Removes one item. Uses item size or 1 WRU/WCU if it doesn’t exist.

### 4. Why Item Size Matters & Item Size Calculations

- Item size affects performance and cost. 
- Items that are too big can:
  - **Slow down** your app (reads/writes take longer).
  - **Cost more** because you pay for data used.
  - **Cause crashes** if the database grows too fast.
- **Tip**: Keep items small (e.g., avoid large text or images) to save money and keep your app fast.
  
<details>
  <summary>Click to View How Sizes Are Calculated</summary>
  
## Here’s how sizes are calculated:

- **Strings**: 
  - English letters = 1 byte (e.g., “test” = 4 bytes).
  - Symbols: “$” = 1 byte, “£” = 2 bytes.
  - Other languages may use 2–4 bytes.

- **Numbers**: 
  - Formula: (attribute name length) + (1 byte per two digits) + (1 byte).
  - **Example**: “112” = 3 bytes. “-112” = 4 bytes. “123” = 3 bytes.

- **Binary**: 1 byte per byte.
  - **Example**: 4-byte binary = 4 bytes.

- **Boolean**: True/false = 1 byte.

- **Null**: 1 byte.

- **List/Map**: 3 bytes + element sizes + 1 byte per element/pair.
  - **Example**: List [“a”, “b”] = 3 + 1 + 1 + 2 = 7 bytes.

- **Item Example**:
  - Attribute names: `pk`, `sk`, `str`, etc. = 28 bytes.
  - Values: “testid” (6 bytes), “test” (4 bytes), “example” (7 bytes), binary “QQ==” (4 bytes), true (1 byte), null (1 byte), number “112” (3 bytes), list [“a”, “b”] (7 bytes), map {“x”: 1} (5 bytes).
  - Total: 28 + 21 + 22 = **71 bytes**.

- **Reads**: Round to next 4 KB (e.g., 2.5 KB = 4 KB).
- **Writes**: Round to next 1 KB (e.g., 1.5 KB = 2 KB).

</details>

---

# DynamoDB Partition Key Selection Guide

This document provides a clear and detailed guide on choosing the right partition key for Amazon DynamoDB tables, explaining key concepts in simple language, their implications, and strategies for optimal performance. It includes all examples from the provided transcript and additional context to ensure clarity and completeness.

## 1. Importance of Choosing the Right Partition Key

Selecting an appropriate partition key (and sort key, if applicable) is critical for designing efficient DynamoDB tables. The choice impacts:

- **Query Flexibility**: The partition key determines the types of queries you can perform. A poorly chosen key may limit your ability to query data efficiently for long-term use cases.
- **Throttling Risks**: Incorrect key selection can lead to uneven data distribution, causing "hot partitions" where some partitions are overutilized, leading to throttling (request rejection due to insufficient provisioned resources).

### Example
- A table with a poorly designed partition key might concentrate data on a single partition, causing throttling when many requests target that partition, even if the table has sufficient overall capacity.

## 2. Key Definitions

Understanding DynamoDB's key terminology is essential before designing a table schema.

### 2.1 Primary Key
- **Definition**: A required attribute (or combination of attributes) that uniquely identifies each item in a DynamoDB table.
- **Types**:
  - **Simple Primary Key**: Consists of only a partition key.
  - **Composite Primary Key**: Consists of a partition key and a sort key (also called a range key).
- **Example** (from DynamoDB Developer Guide):
  - **Table Schema**:
    - **Partition Key**: `AccountID` (e.g., values: 1, 2, 3).
    - **Sort Key** (optional): `Date` (e.g., specific dates like `2023-10-01`, `2023-10-02`).
    - Each item is uniquely identified by either the `AccountID` alone (simple primary key) or the combination of `AccountID` and `Date` (composite primary key).

### 2.2 Partition Key
- **Definition**: A key used by DynamoDB to determine the physical storage location of an item using a hashing function.
- **Function**:
  - The partition key value (e.g., `AccountID = 1`) is input into a hashing function.
  - The hashing function outputs a physical location on one of DynamoDB’s partitions (logical groups of machines hosting data).
  - This enables fast data retrieval by directly identifying the partition containing the item.
- **Example**:
  - **Table with Partition Key `AccountID`**:
    - Item with `AccountID = 1` → Hash function → Stored on Partition 1.
    - Item with `AccountID = 2` → Hash function → Stored on Partition 2.
    - Item with `AccountID = 3` → Hash function → Stored on Partition 1.
  - If a table has hundreds of GB of data, DynamoDB may create multiple partitions (e.g., 10) to distribute data.

### 2.3 Sort Key (Range Key)
- **Definition**: An optional key that sorts items with the same partition key value in a specific order on the same partition.
- **Function**:
  - Items with the same partition key are stored together on the same partition, sorted by the sort key value (e.g., `Date` in ascending order).
  - Enables range queries (e.g., retrieve items within a date range).
  - The combination of partition key and sort key must be globally unique.
- **Example**:
  - **Table with Partition Key `AccountID` and Sort Key `Date`**:
    - Items with `AccountID = 2`:
      - `AccountID = 2`, `Date = 2023-10-01`.
      - `AccountID = 2`, `Date = 2023-10-02`.
    - Both items are stored on the same partition (determined by `AccountID = 2`) but sorted by `Date`.
    - Query example: "Get all items with `AccountID = 2` where `Date` is between `2023-10-01` and `2023-10-03`."
      - DynamoDB performs a direct lookup on the partition for `AccountID = 2` and filters by the date range, achieving low latency.

## 3. Partitioning and Performance

### 3.1 Partitions
- **Definition**: Logical separations of data across groups of machines in DynamoDB to handle large datasets.
- **How It Works**:
  - DynamoDB distributes data across multiple partitions based on the partition key’s hash value.
  - For a table with hundreds of GB of data, DynamoDB might create 10 partitions, spreading data to balance storage and access.

### 3.2 Throttling and Hot Partitions
- **Throttling**: Occurs when requests exceed the provisioned read or write capacity units (RCUs/WCUs) for a partition.
- **Hot Partitions**:
  - When many requests target items with the same partition key, they hit the same partition, potentially exhausting its capacity.
  - **Example**:
    - **Table Setup**:
      - Provisioned capacity: 400 RCUs total.
      - Two partitions: Each gets 200 RCUs.
      - Items with `AccountID = 2` are stored on Partition 2.
    - **Scenario**:
      - A burst of requests targets `AccountID = 2`, requiring >200 RCUs.
      - Partition 2 is overloaded, but Partition 1 (with 200 RCUs) is idle and cannot assist.
      - Result: Throttling occurs, and requests are rejected despite the table’s total 400 RCUs.
- **Impact**: Hot partitions reduce performance and waste provisioned capacity.

### 3.3 Adaptive Capacity (Introduced May 2019)
- **Definition**: A DynamoDB feature that mitigates hot partition issues by dynamically redistributing capacity.
- **How It Works**:
  - If a partition (e.g., Partition 2) is overburdened, DynamoDB borrows unused capacity from underutilized partitions (e.g., Partition 1).
  - **Example**:
    - Partition 1: 200 RCUs, idle.
    - Partition 2: 200 RCUs, overwhelmed by requests for `AccountID = 2`.
    - Adaptive capacity borrows RCUs from Partition 1 to boost Partition 2’s capacity.
- **Limits**:
  - Maximum per partition: 3,000 RCUs or 1,000 WCUs.
  - If requests exceed these limits, throttling occurs despite adaptive capacity.
- **Impact**: Solves hot partition issues for ~99% of use cases, reducing the need for complex key distribution strategies.

## 4. Choosing Between Simple and Composite Primary Keys

To select the right key schema, ask these questions:

### 4.1 Question 1: Are You Always Performing Quick Lookups of a Known, Globally Unique Key?
- **Scenario**:
  - You always query by a single, unique identifier (e.g., UUID, `AccountID`).
  - Example: Retrieving a user’s profile by `UserID` (e.g., `UserID = "abc123"`).
- **Recommendation**:
  - Use a **Simple Primary Key** (partition key only).
  - No sort key is needed, as queries are direct lookups (e.g., `GetItem` for `UserID = "abc123"`).
- **Benefits**:
  - Simplifies schema design.
  - Optimizes performance for direct lookups.

### 4.2 Question 2: Is Your Key Non-Unique, or Do You Need Range Queries?
- **Scenario**:
  - Multiple items share the same partition key value, or you need to query a range of values (e.g., dates, numbers).
  - Example: Retrieving all orders for `AccountID = 2` between `2023-10-01` and `2023-10-03`.
- **Recommendation**:
  - Use a **Composite Primary Key** (partition key + sort key).
  - Partition key: `AccountID`.
  - Sort key: `Date` (enables sorting and range queries).
- **Benefits**:
  - Supports queries like “greater than,” “less than,” or “between” on the sort key.
  - Efficient for fetching multiple items with the same partition key.

### Example Queries
- **Simple Primary Key**:
  - Query: `GetItem` for `AccountID = 1`.
  - Result: Directly retrieves the item from the partition determined by `AccountID = 1`.
- **Composite Primary Key**:
  - Query: `Query` for `AccountID = 2` and `Date` between `2023-10-01` and `2023-10-03`.
  - Result: DynamoDB locates the partition for `AccountID = 2` and filters items by the date range.

## 5. Strategies for High-Throughput Applications

For applications requiring thousands of RCUs/WCUs, additional strategies ensure even data distribution to avoid hot partitions.

### 5.1 Add Prefixes or Suffixes to Partition Keys
- **Purpose**: Distribute items with the same logical key across multiple partitions.
- **Method**:
  - Append a random or calculated prefix/suffix to the partition key.
  - Example:
    - Instead of `AccountID = 2`, use `AccountID = 2-A`, `2-B`, `2-C`.
    - Hashing function distributes `2-A`, `2-B`, `2-C` across different partitions.
- **Query Impact**: Applications must know the prefix/suffix to query (e.g., query `2-A`, `2-B`, etc., separately).
- **Example**:
  - Original: All items with `AccountID = 2` on one partition.
  - Modified: `AccountID = 2-1`, `2-2` spread across multiple partitions, reducing throttling.

### 5.2 Use Composed Partition Keys
- **Purpose**: Create a unique partition key by combining multiple attributes.
- **Method**:
  - Combine `AccountID` with another attribute (e.g., `Region`, `OrderType`).
  - Example: Partition key = `AccountID#Region` (e.g., `2#US`, `2#EU`).
- **Benefits**:
  - Ensures even distribution across partitions.
  - Simplifies queries if the combined key is predictable.
- **Example**:
  - Original: `AccountID = 2` (all items on one partition).
  - Composed: `AccountID#Region = 2#US`, `2#EU` (spread across partitions).

### 5.3 DynamoDB Accelerator (DAX)
- **Definition**: A caching layer for DynamoDB to bypass partition capacity limits.
- **How It Works**:
  - Adds an in-memory cache (backed by EC2 instances) in front of DynamoDB.
  - Reduces read load on partitions, bypassing the 3,000 RCU limit per partition.
- **Trade-Off**:
  - Additional cost for EC2 instances.
  - Best for read-heavy workloads.
- **Example**:
  - A high-traffic application querying `AccountID = 2` frequently uses DAX to cache results, reducing direct DynamoDB requests.

## 6. Additional Considerations
- **Adaptive Capacity Limitations**:
  - Cannot exceed 3,000 RCUs or 1,000 WCUs per partition.
  - Plan for high-throughput applications beyond these limits using the above strategies.
- **Research Resources**:
  - For complex use cases, refer to AWS documentation and community articles on partition key design and high-throughput strategies.
  - Example topics: Advanced sharding techniques, optimizing for write-heavy workloads.

## 7. Summary
- **Choose Simple Primary Key** for unique, direct lookups (e.g., `UserID`).
- **Choose Composite Primary Key** for non-unique keys or range queries (e.g., `AccountID` + `Date`).
- **Handle High Throughput**:
  - Use prefixes/suffixes or composed keys to distribute data.
  - Consider DAX for caching in read-heavy applications.
- **Leverage Adaptive Capacity** to mitigate hot partitions for most use cases.
- **Plan Ahead**: Design schemas with long-term query patterns and scalability in mind to avoid throttling and performance issues.















