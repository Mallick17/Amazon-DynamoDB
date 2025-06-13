# DynamoDB Global Secondary Indexes (GSIs) Guide
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

---
---



# DynamoDB Local Secondary Indexes (LSIs) Guide
## 1. Introduction to Local Secondary Indexes (LSIs)

- **Definition**: LSIs are a DynamoDB feature that allows querying on alternative sort keys within the same partition key as the primary table, enabling more flexible query patterns without scanning the entire table.
- **Purpose**: Extend the sort key functionality to non-key attributes, allowing efficient range queries and direct lookups on those attributes, reducing latency and read capacity unit (RCU) consumption.
- **Prerequisites**: Familiarity with DynamoDB basics, including:
  - **Partition Key**: Determines the partition where data is stored.
  - **Sort Key**: Sorts items within the same partition for range queries.
  - **Global Secondary Indexes (GSIs)**: Secondary tables with different partition keys (for comparison).
  - **Partitioning**: How DynamoDB distributes data across physical partitions.
- **Use Case Example**:
  - **Scenario**: A forum post table where queries by `ForumName` (partition key) and `LastPostDateTime` (sort key) are efficient, but a new requirement demands querying by `Subject`.
  - **Problem**: Without LSIs, querying by `Subject` requires a slow and costly scan operation.
  - **Solution**: An LSI on `Subject` enables direct lookups or range queries within a specific `ForumName`.

## 2. Understanding the Example Table

- **Context**: The transcript uses a forum post table from the DynamoDB AWS website to illustrate LSIs.
- **Table Description**:
  - Represents forum posts on AWS forums (e.g., S3, EC2, RDS).
  - Each row corresponds to a unique forum post with a specific subject and the timestamp of the most recent post.
  - **Schema**:
    - **Partition Key**: `ForumName` (e.g., S3, EC2, RDS).
    - **Sort Key**: `LastPostDateTime` (timestamp, e.g., `2025-05-01 10:00:00`).
    - **Attribute**: `Subject` (title of the forum post, e.g., “How to use S3 buckets”).
- **Example Table**: Forum post table used in the transcript.

  | ForumName (Partition Key) | LastPostDateTime (Sort Key) | Subject                     |
  |---------------------------|-----------------------------|-----------------------------|
  | S3                        | 2025-05-01 10:00:00         | AA: How to use S3 buckets   |
  | S3                        | 2025-05-02 12:30:00         | BB: S3 pricing questions    |
  | S3                        | 2025-05-03 15:45:00         | CC: S3 security best practices |
  | S3                        | 2025-05-04 09:20:00         | DD: S3 data transfer issues |
  | EC2                       | 2025-05-01 11:00:00         | AA: EC2 instance types      |
  | RDS                       | 2025-05-02 14:00:00         | AA: RDS backup strategies   |

  - **Explanation**:
    - `ForumName` (e.g., S3) groups posts by forum.
    - `LastPostDateTime` sorts posts within each forum by recency.
    - `Subject` is a non-key attribute describing the post topic.
    - Multiple posts in the same forum (e.g., S3) have different subjects and timestamps.

## 3. Query Capabilities with Primary Table Schema

- **Default Query Behavior**:
  - The primary table supports fast queries using the partition key (`ForumName`) and optional sort key (`LastPostDateTime`).
  - The sort key enables range queries (e.g., greater than, less than, between).
- **Example Queries**:
  1. **Direct Lookup by Partition Key**:
     - **Query**: “Get all records where `ForumName = S3`.”
     - **Result**: Returns all four S3 rows.
     - **Table**: Query result for `ForumName = S3`.

       | ForumName | LastPostDateTime    | Subject                     |
       |-----------|---------------------|-----------------------------|
       | S3        | 2025-05-01 10:00:00 | AA: How to use S3 buckets   |
       | S3        | 2025-05-02 12:30:00 | BB: S3 pricing questions    |
       | S3        | 2025-05-03 15:45:00 | CC: S3 security best practices |
       | S3        | 2025-05-04 09:20:00 | DD: S3 data transfer issues |

  2. **Range Query with Sort Key**:
     - **Query**: “Get all records where `ForumName = S3` and `LastPostDateTime` > 15 days old (before 2025-04-28).”
     - **Result**: Returns S3 rows older than 15 days (assuming current date is 2025-05-13).
     - **Table**: Query result for `ForumName = S3` and `LastPostDateTime` before 2025-04-28.

       | ForumName | LastPostDateTime    | Subject                     |
       |-----------|---------------------|-----------------------------|
       | S3        | 2025-05-01 10:00:00 | AA: How to use S3 buckets   |

     - **Explanation**: The sort key enables efficient filtering by date (e.g., `LastPostDateTime` < 2025-04-28).

- **Limitation**:
  - Only one sort key (`LastPostDateTime`) can be defined per table.
  - Querying by other attributes (e.g., `Subject`) is inefficient without LSIs.

## 4. Problem with Querying Non-Sort Key Attributes

- **Challenge**: Querying by a non-key attribute like `Subject` requires a scan operation, which is inefficient.
- **Example Query**: “Find all records where `ForumName = S3` and `Subject = AA`.”
  - **Issue**: The primary table’s sort key (`LastPostDateTime`) cannot filter by `Subject`.
  - **Naive Approach**: Scan the entire table with a filter expression.
    - **Process**:
      - Scans all rows.
      - Applies filter: `ForumName = S3` and `Subject = AA`.
      - Returns matching row(s).
    - **Drawbacks**:
      - High latency for large tables.
      - Consumes RCUs for every scanned row, increasing costs.
  - **Better Approach**: Query by `ForumName = S3`, then filter by `Subject = AA`.
    - Still examines all S3 rows (4 in this case), less efficient than a direct lookup.
  - **Optimal Solution**: Use an LSI on `Subject` for direct lookup.

## 5. How LSIs Solve the Problem

- **Functionality**: LSIs allow an alternative sort key (e.g., `Subject`) within the same partition key (`ForumName`), enabling direct lookups or range queries on that attribute.
- **Benefits**:
  - **Lower Latency**: Avoids scanning all rows in a partition.
  - **Reduced Costs**: Minimizes RCU usage compared to scans or filters.
  - **Query Flexibility**: Supports efficient queries on non-sort key attributes within the same partition.
- **Example**:
  - **Primary Table**: Partition key = `ForumName`, sort key = `LastPostDateTime`.
  - **LSI**: Partition key = `ForumName`, sort key = `Subject`.
  - **Query**: “Get all rows where `ForumName = S3` and `Subject = AA`.”
    - **Result**: Directly retrieves the matching row.
    - **Table**: LSI query result.

      | ForumName | Subject                   | LastPostDateTime    |
      |-----------|---------------------------|---------------------|
      | S3        | AA: How to use S3 buckets | 2025-05-01 10:00:00 |

## 6. How LSIs Work

### 6.1 Creating an LSI
- **Process**:
  - Define an LSI at table creation time (cannot be added later).
  - Specify the same partition key as the primary table (`ForumName`).
  - Choose a different sort key (e.g., `Subject`).
  - Select projected attributes (optional subset of table attributes).
- **Example**:
  - **LSI Configuration**:
    - Partition Key: `ForumName`.
    - Sort Key: `Subject`.
    - Index Name: `SubjectIndex`.
    - Projected Attributes: Include `LastPostDateTime` (optional).

### 6.2 LSI Structure
- **Mechanism**: LSIs create an alternate index within the same partition, sorted by the new sort key.
- **Example Table**: LSI with `ForumName` as partition key and `Subject` as sort key.

  | ForumName (Partition Key) | Subject (Sort Key)          | LastPostDateTime    |
  |---------------------------|-----------------------------|---------------------|
  | S3                        | AA: How to use S3 buckets   | 2025-05-01 10:00:00 |
  | S3                        | BB: S3 pricing questions    | 2025-05-02 12:30:00 |
  | S3                        | CC: S3 security best practices | 2025-05-03 15:45:00 |
  | S3                        | DD: S3 data transfer issues | 2025-05-04 09:20:00 |
  | EC2                       | AA: EC2 instance types      | 2025-05-01 11:00:00 |
  | RDS                       | AA: RDS backup strategies   | 2025-05-02 14:00:00 |

  - **Explanation**:
    - The LSI maintains the same partition key (`ForumName`) but sorts items by `Subject`.
    - Querying `ForumName = S3` and `Subject = AA` directly retrieves the matching row.

### 6.3 Querying an LSI
- **Syntax**: Similar to primary table queries, but specify the LSI name.
- **Example**:
  - **Query Expression**: `Query` with `IndexName = SubjectIndex`, `KeyConditionExpression: ForumName = S3 AND Subject = AA`.
  - **Result**: Retrieves the row with `ForumName = S3`, `Subject = AA`.

### 6.4 Java Implementation Example
- **Context**: The transcript provides a Java code snippet using DynamoDB Mapper to query an LSI.
- **Code**:
  ```java
  // Define conditions
  Condition rangeKeyCondition = new Condition()
      .withAttributeValueList(new AttributeValue().withS("CCC"))
      .withComparisonOperator(ComparisonOperator.EQ);

  // Create query expression
  DynamoDBQueryExpression<LastPost> queryExpression = new DynamoDBQueryExpression<LastPost>()
      .withHashKeyValues(new LastPost().withForumName("S3"))
      .withRangeKeyCondition("Subject", rangeKeyCondition)
      .withIndexName("SubjectIndex");

  // Perform query
  List<LastPost> results = mapper.query(LastPost.class, queryExpression);
  ```
- **Explanation**:
  - Queries the LSI (`SubjectIndex`) for `ForumName = S3` and `Subject = CCC`.
  - **Result Table**:

    | ForumName | Subject                     | LastPostDateTime    |
    |-----------|-----------------------------|---------------------|
    | S3        | CC: S3 security best practices | 2025-05-03 15:45:00 |

  - **Note**: The transcript uses `CCC` as an example, adjusted to `CC` to match the table.

## 7. LSI Query Approaches Compared

The transcript outlines three approaches to querying `ForumName = S3` and `Subject = AA`.

1. **Naive Scan Approach**:
   - **Method**: Scan the entire table with a filter expression (`ForumName = S3 AND Subject = AA`).
   - **Process**:
     - Scans all rows (S3, EC2, RDS).
     - Filters for `ForumName = S3` and `Subject = AA`.
   - **Result Table**:

     | ForumName | LastPostDateTime    | Subject                   |
     |-----------|---------------------|---------------------------|
     | S3        | 2025-05-01 10:00:00 | AA: How to use S3 buckets |

   - **Drawbacks**:
     - High RCU consumption (scans all rows).
     - Slow for large tables.

2. **Better Query Approach**:
   - **Method**: Query by `ForumName = S3`, then apply a filter expression for `Subject = AA`.
   - **Process**:
     - Queries `ForumName = S3` (returns 4 rows).
     - Filters for `Subject = AA` (examines all 4 rows).
   - **Result Table**: Same as above.
   - **Drawbacks**:
     - Still examines all S3 rows, less efficient than direct lookup.

3. **Optimal LSI Approach**:
   - **Method**: Use LSI with `ForumName` as partition key and `Subject` as sort key.
   - **Process**: Direct lookup for `ForumName = S3` and `Subject = AA`.
   - **Result Table**: Same as above.
   - **Benefits**:
     - Minimizes RCU usage (direct lookup).
     - Faster and more cost-effective.

## 8. LSI Limitations

- **1. Defined at Table Creation**:
  - LSIs must be specified when creating the table; they cannot be added or modified later.
  - **Example**: To query by `Subject`, the LSI must be defined during table creation with `Subject` as the sort key.
- **2. Maximum of 5 LSIs**:
  - A table can have up to 5 LSIs (possibly increasable via AWS support ticket).
  - **Example**: A forum table might have LSIs for `Subject`, `Author`, etc., but is limited to 5.
- **3. No Additional Cost**:
  - Unlike GSIs, LSIs do not incur extra write costs since they share the same partition and capacity as the primary table.
  - **Example**: Writing to the primary table does not trigger additional WCU consumption for LSIs.

## 9. LSIs vs. GSIs

- **Key Differences**:
  - **Partition Key**:
    - LSIs use the same partition key as the primary table.
    - GSIs use a different partition key, creating a separate table.
  - **Cost**:
    - LSIs: No extra write costs (share primary table’s capacity).
    - GSIs: Each write to the primary table triggers a write to each GSI, multiplying WCU costs.
  - **Flexibility**:
    - LSIs: Limited to 5, defined at table creation.
    - GSIs: Up to 20, can be added after table creation.
  - **Use Case**:
    - LSIs: Best for queries within the same partition (e.g., alternative sort keys like `Subject`).
    - GSIs: Best for queries requiring a different partition key (e.g., `OriginCountry` in a previous example).
- **Example**:
  - **LSI**: Query `ForumName = S3` and `Subject = AA` (same partition key).
  - **GSI**: Query `Subject = AA` across all forums (different partition key).

## 10. Additional DynamoDB Concepts

- **Query Expression**:
  - **Definition**: A condition to retrieve items based on key attributes.
  - **Example**: `KeyConditionExpression: ForumName = S3 AND Subject = AA` on LSI.
  - **Result**: Direct lookup of matching row.
- **Scan Operation**:
  - **Definition**: Reads all items in a table, optionally with a filter expression.
  - **Example**: Scan with `FilterExpression: ForumName = S3 AND Subject = AA`.
  - **Drawback**: High RCU usage, slow for large tables.
- **Filter Expression**:
  - **Definition**: Filters non-key attributes after a query or scan.
  - **Example**: Query `ForumName = S3`, filter `Subject = AA`.
  - **Limitation**: Less efficient than LSI queries.
- **Read Capacity Units (RCUs)**:
  - **Definition**: Measure read throughput (1 RCU = one 4 KB read per second).
  - **Example**: LSI query for `ForumName = S3`, `Subject = AA` consumes minimal RCUs.
- **Write Capacity Units (WCUs)**:
  - **Definition**: Measure write throughput (1 WCU = one 1 KB write per second).
  - **Example**: LSIs share primary table’s WCUs, unlike GSIs.
- **Throttling**:
  - **Definition**: Occurs when requests exceed RCUs/WCUs.
  - **Example**: Insufficient RCUs for an LSI query may cause throttling.
  - **Mitigation**: Use auto-scaling for capacity.

## 11. Summary

- **LSIs Overview**:
  - Enable alternative sort keys (e.g., `Subject`) within the same partition key (`ForumName`).
  - Support efficient range queries and direct lookups.
  - Share the primary table’s partitions and capacity (no extra write costs).
- **Key Benefits**:
  - Optimized queries for non-sort key attributes within a partition.
  - Cost-effective (no additional WCU costs).
  - Low latency for targeted queries.
- **Best Practices**:
  - Define LSIs at table creation for anticipated query patterns.
  - Use up to 5 LSIs for key attributes (e.g., `Subject`, `Author`).
  - Leverage sort key conditions (e.g., equals, starts with) for flexibility.
- **Avoid Pitfalls**:
  - Plan LSIs upfront (cannot add later).
  - Respect the 5 LSI limit.
- **When to Use LSIs**:
  - For queries requiring alternative sort keys within the same partition.
  - When cost efficiency is critical (compared to GSIs).
- **When to Use GSIs Instead**:
  - For queries needing a different partition key.
  - When LSIs cannot be defined (e.g., table already created).


