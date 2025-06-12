# DynamoDB Global Secondary Indexes (GSIs) Guide
This document provides a detailed and easy-to-understand guide on DynamoDB Global Secondary Indexes (GSIs), focusing on their purpose, functionality, usage, and potential pitfalls. It includes all examples from the provided transcript, presented in table format for clarity, and explains concepts in simple language. The guide covers related topics such as throttling, query expressions, and capacity units to help developers effectively use GSIs for flexible querying in DynamoDB tables.

## 1. Introduction to Global Secondary Indexes (GSIs)
- **Definition**: GSIs allow querying on attributes other than the table’s primary key, enabling flexible query patterns without scanning the entire table.
- **Purpose**: Provide direct lookups on non-primary key attributes, reducing latency and costs compared to scan operations.
- **Use Case Example**:
  - **Scenario**: A bank account table where queries by `AccountID` (primary key) are fast, but a new requirement demands querying by `OriginCountry`.
  - **Problem**: Without GSIs, querying by `OriginCountry` requires a slow and costly scan operation.
  - **Solution**: A GSI on `OriginCountry` enables direct lookups.

## 2. Problem with Non-Primary Key Queries

- **Challenge**: DynamoDB only supports fast lookups using the primary key (partition key or partition key + sort key).
- **Example**:
  - **Query**: “Find all rows where `OriginCountry = Germany`.”
  - **Issue**: Without knowing the `AccountID`, a direct lookup is impossible.
  - **Naive Approach**: Use a **scan operation** with a **filter expression**.
    - **Scan Operation**: Reads every row in the table.
    - **Filter Expression**: Returns only rows matching the condition (e.g., `OriginCountry = Germany`).
    - **Drawbacks**:
      - High latency for large tables (thousands to billions of rows).
      - High cost due to consuming read capacity units (RCUs) for every scanned row.

- **Example Table**: Bank account table used in the transcript.

  | AccountID (Partition Key) | CreationDate       | OriginCountry |
  |---------------------------|--------------------|---------------|
  | 1                         | 2023-01-15        | US            |
  | 2                         | 2023-02-10        | UK            |
  | 3                         | 2023-03-05        | US            |
  | 4                         | 2023-04-20        | Germany       |

  - **Scan Operation Process**:
    - Scans row 1: `OriginCountry = US` → No match.
    - Scans row 2: `OriginCountry = UK` → No match.
    - Scans row 3: `OriginCountry = US` → No match.
    - Scans row 4: `OriginCountry = Germany` → Match, returns row.
  - **Problem**: Scanning all rows is inefficient for large tables, increasing latency and RCU costs.

## 3. How GSIs Solve the Problem

- **Functionality**: GSIs create a secondary table (index) with a different partition key, allowing direct lookups on non-primary key attributes.
- **Benefits**:
  - **Lower Latency**: Avoids scanning the entire table.
  - **Reduced Costs**: Uses fewer RCUs than scans.
  - **Query Flexibility**: Supports new query patterns without changing the primary table’s schema.
- **Example**:
  - **Primary Table**: Partition key is `AccountID`.
  - **GSI**: Partition key is `OriginCountry`.
  - **Query**: “Get all rows where `OriginCountry = Germany`.”
    - Result: Directly retrieves `AccountID = 4` from the GSI without scanning.

## 4. How GSIs Work

### 4.1 Creating a GSI
- **Process**:
  - Define a new partition key (and optional sort key) for the GSI.
  - Specify the GSI name (e.g., `OriginCountryIndex`).
  - Choose **projected attributes** (optional subset of table attributes to include).
  - Set read and write capacity units (RCUs/WCUs).
- **Example** (from AWS console):
  - **Partition Key**: `OriginCountry`.
  - **Index Name**: `OriginCountryIndex` (can be any name).
  - **Projected Attributes**: Include `AccountID`, `CreationDate` (exclude others if not needed).
  - **Capacity**: Configure RCUs/WCUs for the GSI.

### 4.2 GSI Structure
- **Mechanism**: DynamoDB creates a secondary table (GSI) that reorganizes data based on the new partition key.
- **Example Table**: GSI with `OriginCountry` as partition key.

  | OriginCountry (Partition Key) | AccountID | CreationDate       |
  |------------------------------|-----------|--------------------|
  | US                           | 1         | 2023-01-15        |
  | UK                           | 2         | 2023-02-10        |
  | US                           | 3         | 2023-03-05        |
  | Germany                      | 4         | 2023-04-20        |

  - **Explanation**:
    - The GSI reindexes data with `OriginCountry` as the partition key.
    - Querying `OriginCountry = Germany` directly retrieves `AccountID = 4`.

### 4.3 Data Synchronization
- **Behavior**: DynamoDB synchronizes the primary table and GSI for insert, update, and delete operations.
- **Example**:
  - **Primary Table Update**: Change `OriginCountry` from `Germany` to `France` for `AccountID = 4`.
  - **GSI Update**: Propagated to update `OriginCountry = Germany` to `France` in the GSI.
  - **Note**: Propagation is eventually consistent (no guaranteed SLA).

### 4.4 Querying a GSI
- **Syntax**: Similar to primary table queries, but specify the GSI name.
- **Example**:
  - **Query Expression**: `Query` with `IndexName = OriginCountryIndex` and `KeyConditionExpression: OriginCountry = Germany`.
  - **Result**: Retrieves `AccountID = 4`, `CreationDate = 2023-04-20`.

## 5. GSI Rules and Considerations

GSIs follow the same rules as primary DynamoDB tables, with specific considerations:

1. **Partition Key Distribution**:
   - The GSI’s partition key must ensure even data distribution to avoid hot partitions.
   - **Example**:
     - If most accounts have `OriginCountry = US`, the `US` partition may become a hotspot, causing throttling.
     - Solution: Choose a partition key with diverse values or use sharding techniques.

2. **Read and Write Capacity Units (RCUs/WCUs)**:
   - GSIs require separate RCU/WCU settings from the primary table.
   - **Example**:
     - GSI needs 100 RCUs for reads and 50 WCUs for writes, based on query load.
     - Primary table may have different settings (e.g., 200 RCUs, 100 WCUs).

3. **Throttling**:
   - GSIs can throttle independently of the primary table.
   - **Example**:
     - GSI with insufficient WCUs throttles writes, impacting primary table writes.

## 6. DynamoDB Capacity Units Explained

- **Read Capacity Units (RCUs)**:
  - Measure read throughput.
  - 1 RCU = One strongly consistent read (or two eventually consistent reads) of up to 4 KB per second.
  - **Example**:
    - Querying a 4 KB item from the GSI (`OriginCountry = Germany`) consumes 1 RCU (strongly consistent).
- **Write Capacity Units (WCUs)**:
  - Measure write throughput.
  - 1 WCU = One write of up to 1 KB per second.
  - **Example**:
    - Updating a 1 KB item in the primary table and GSI consumes 1 WCU each.
- **Importance**:
  - Configure sufficient RCUs/WCUs for both primary table and GSIs to prevent throttling.
  - Use auto-scaling to adjust capacity dynamically.

## 7. GSI Gotchas and Best Practices

GSIs provide flexibility but have pitfalls to avoid, as highlighted in the transcript.

### 7.1 Increased Write Costs
- **Issue**: Each write to the primary table triggers a write to all GSIs, multiplying WCU consumption.
- **Example**:
  - **Setup**: Primary table + 5 GSIs.
  - **Operation**: Update `AccountID = 4` (1 KB item).
  - **Cost Table**:

    | Table/Index        | WCUs Consumed |
    |--------------------|---------------|
    | Primary Table      | 1 WCU         |
    | GSI 1              | 1 WCU         |
    | GSI 2              | 1 WCU         |
    | GSI 3              | 1 WCU         |
    | GSI 4              | 1 WCU         |
    | GSI 5              | 1 WCU         |
    | **Total**          | **6 WCUs**    |

  - **Explanation**: One write operation consumes 6 WCUs (1 for primary + 1 per GSI).
- **Best Practice**: Use only necessary GSIs to minimize write costs.

### 7.2 Maximum GSI Limit
- **Limit**: Up to 20 GSIs per table (increased from 5 in December 2018).
- **Example**:
  - A table with GSIs for `OriginCountry`, `CreationDate`, and other attributes cannot exceed 20 GSIs.
- **Best Practice**: Plan GSIs to cover essential query patterns within the limit.

### 7.3 Eventual Consistency
- **Issue**: GSI updates are eventually consistent, with potential delays in propagation.
- **Example**:
  - **Primary Table Update**: Change `OriginCountry` from `Germany` to `France` for `AccountID = 4`.
  - **GSI Delay**: GSI may temporarily return `OriginCountry = Germany` (stale data).
  - **Risk**: Race conditions when reading from GSI and updating the primary table.
    - **Scenario**:
      - Application reads stale GSI data (`OriginCountry = Germany`).
      - Updates primary table based on stale data, causing inconsistencies.
- **Best Practice**:
  - Avoid using GSI data as the source of truth for updates.
  - Use strongly consistent reads on the primary table when consistency is critical.

### 7.4 Throttling Due to Insufficient WCUs
- **Issue**: Insufficient WCUs on a GSI can throttle primary table writes, as GSI writes are part of the transaction.
- **Example**:
  - **Setup**:
    - Primary table: 100 WCUs.
    - GSI: 50 WCUs.
  - **Operation**: Write to `AccountID = 4` triggers a GSI write.
  - **Result**: GSI throttling (due to insufficient WCUs) causes the primary table write to fail.
- **Best Practice**:
  - Set GSI WCUs ≥ primary table WCUs.
  - Enable auto-scaling and use the “same settings as primary table” option for GSI write capacity.

### 7.5 Separate Monitoring for GSIs
- **Requirement**: GSIs have independent metrics (e.g., error counts, latency, throttling).
- **Example**:
  - **Scenario**: GSI experiences high latency due to insufficient RCUs, while the primary table performs well.
  - **Monitoring Table**:

    | Metric             | Primary Table | GSI (OriginCountryIndex) |
    |--------------------|---------------|--------------------------|
    | ThrottledRequests  | 0             | 10 (due to low WCUs)     |
    | ReadLatency        | 5 ms          | 20 ms (insufficient RCUs)|
    | ErrorCount         | 0             | 2 (failed queries)       |

  - **Explanation**: GSI issues (e.g., throttling) may not affect the primary table but require separate monitoring.
- **Best Practice**:
  - Set up CloudWatch alarms for GSI metrics (e.g., `ThrottledRequests`, `ReadLatency`).
  - Monitor GSIs as critical components, similar to the primary table.

## 8. Additional DynamoDB Concepts

### 8.1 DynamoDB Query Expression
- **Definition**: A condition to retrieve items based on key attributes (partition/sort key).
- **Example**:
  - **GSI Query**: `KeyConditionExpression: OriginCountry = Germany` on `OriginCountryIndex`.
  - **Result**: Retrieves `AccountID = 4`, `CreationDate = 2023-04-20`.
  - **Table**: GSI query result.

    | OriginCountry | AccountID | CreationDate       |
    |---------------|-----------|--------------------|
    | Germany       | 4         | 2023-04-20        |

- **Difference from Filter Expression**: Queries are efficient, targeting key attributes directly.

### 8.2 DynamoDB Scan Operation
- **Definition**: Reads every item in a table or GSI, optionally with a filter expression.
- **Example**:
  - **Scan with Filter**: `FilterExpression: OriginCountry = Germany`.
  - **Process**: Scans all rows, returns only `AccountID = 4`.
  - **Table**: Scan operation result.

    | AccountID | CreationDate       | OriginCountry |
    |-----------|--------------------|---------------|
    | 4         | 2023-04-20        | Germany       |

  - **Drawbacks**: High RCU consumption and latency for large tables.

### 8.3 DynamoDB Filter Expression
- **Definition**: Filters non-key attributes after a query or scan, less efficient than key conditions.
- **Example**:
  - **Scan with Filter**: `FilterExpression: OriginCountry = Germany`.
  - **Process**: Scans all rows, filters for `OriginCountry = Germany`.
  - **Table**: Same as scan operation result above.
- **Limitation**: Consumes RCUs for all scanned items, even those not returned.

### 8.4 DynamoDB Throttling
- **Definition**: Occurs when requests exceed provisioned RCUs/WCUs.
- **Example**:
  - **Setup**: GSI with 50 WCUs receives 60 WCU-worth of writes.
  - **Result**: Throttling occurs, potentially failing primary table writes.
- **Mitigation**:
  - Increase RCUs/WCUs or enable auto-scaling.
  - Ensure even partition key distribution.

## 9. Summary
- **GSIs Overview**:
  - Enable queries on non-primary key attributes (e.g., `OriginCountry`).
  - Create a secondary table synced with the primary table.
  - Reduce latency and costs compared to scans.
- **Key Benefits**:
  - Flexible querying for new use cases.
  - Efficient direct lookups on indexed attributes.
- **Best Practices**:
  - Use minimal GSIs (max 20) to control costs.
  - Set GSI WCUs ≥ primary table WCUs.
  - Monitor GSI metrics separately.
  - Handle eventual consistency to avoid race conditions.
- **Avoid Pitfalls**:
  - Increased write costs (1 write = 1 + number of GSIs writes).
  - Throttling due to insufficient GSI capacity.
  - Stale data from eventual consistency.
- **When to Use GSIs**:
  - For queries on non-primary key attributes.
  - When scans are too slow or costly for large tables.

