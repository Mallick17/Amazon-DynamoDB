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

