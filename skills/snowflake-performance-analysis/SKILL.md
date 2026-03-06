---
name: snowflake-performance-analysis
description: Analyze and optimize Snowflake query performance, warehouse sizing, memory spilling, partition pruning, cache hit rates, and cost optimization through detailed profiling and recommendations.
allowed-tools: "*"
---

# Snowflake Performance Analysis

**When to invoke:** Use when queries are slow, warehouses seem undersized/oversized, investigating performance issues, or when asked to "optimize performance", "analyze query", "check warehouse utilization", or "why is this slow".

## Role
You are a Snowflake performance optimization expert specializing in query profiling, warehouse sizing, and identifying performance bottlenecks.

## Workflow

### 1. Identify the Problem Scope
- **Slow specific query:** Get query ID or SQL text
- **General slowness:** Check recent query history patterns
- **Warehouse issues:** Identify which warehouse(s) involved
- **Cost concerns:** Focus on compute spend patterns

### 2. Query Profile Analysis
If specific query provided:

```sql
-- Get query profile details
SELECT 
    query_id,
    query_text,
    database_name,
    schema_name,
    warehouse_name,
    warehouse_size,
    execution_status,
    total_elapsed_time/1000 as total_elapsed_sec,
    bytes_scanned,
    bytes_spilled_to_local_storage,
    bytes_spilled_to_remote_storage,
    bytes_written,
    bytes_written_to_result,
    partitions_scanned,
    partitions_total,
    percentage_scanned_from_cache,
    rows_produced,
    compilation_time/1000 as compilation_sec,
    execution_time/1000 as execution_sec,
    queued_provisioning_time/1000 as queued_provisioning_sec,
    queued_repair_time/1000 as queued_repair_sec,
    queued_overload_time/1000 as queued_overload_sec
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE query_id = '<query_id>'
ORDER BY start_time DESC
LIMIT 1;
```

**Key metrics to analyze:**
- **Spilling:** `bytes_spilled_to_local_storage` or `bytes_spilled_to_remote_storage` > 0 indicates warehouse too small
- **Cache hit rate:** `percentage_scanned_from_cache` (higher is better)
- **Partition pruning:** `partitions_scanned / partitions_total` (lower ratio is better)
- **Queueing:** Any `queued_*_time` > 0 indicates contention
- **Data scanned:** `bytes_scanned` indicates table scan efficiency

### 3. Warehouse Utilization Analysis

```sql
-- Check warehouse usage patterns (last 7 days)
SELECT 
    warehouse_name,
    DATE_TRUNC('hour', start_time) as hour,
    COUNT(*) as query_count,
    AVG(total_elapsed_time)/1000 as avg_elapsed_sec,
    SUM(total_elapsed_time)/1000 as total_elapsed_sec,
    AVG(bytes_scanned) as avg_bytes_scanned,
    SUM(CASE WHEN bytes_spilled_to_local_storage > 0 
             OR bytes_spilled_to_remote_storage > 0 
        THEN 1 ELSE 0 END) as queries_with_spilling,
    AVG(percentage_scanned_from_cache) as avg_cache_hit_rate
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE warehouse_name IS NOT NULL
  AND start_time >= dateadd('days', -7, current_timestamp())
GROUP BY warehouse_name, hour
ORDER BY warehouse_name, hour DESC;
```

**Warehouse health indicators:**
- **Spilling frequency:** > 5% queries spilling = warehouse too small
- **Queue times:** Consistent queueing = need more warehouses or multi-cluster
- **Cache hit rate:** < 50% average = poor cache reuse patterns
- **Utilization gaps:** Long idle periods = review auto-suspend settings

### 4. Table/Schema Performance Analysis

```sql
-- Check table clustering and pruning efficiency
SELECT 
    table_name,
    AVG(partitions_scanned * 100.0 / NULLIF(partitions_total, 0)) as avg_partition_scan_pct,
    COUNT(*) as query_count,
    AVG(total_elapsed_time)/1000 as avg_elapsed_sec
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE partitions_total > 0
  AND database_name = '<database>'
  AND schema_name = '<schema>'
  AND start_time >= dateadd('days', -7, current_timestamp())
GROUP BY table_name
HAVING avg_partition_scan_pct > 10  -- Scanning >10% of partitions
ORDER BY avg_partition_scan_pct DESC;
```

**Look for:**
- Tables with high partition scan percentages = poor clustering or missing cluster keys
- Large tables without clustering keys
- Frequent full table scans

### 5. Common Performance Issues Detection

**Issue 1: Memory Spilling**
- **Symptom:** `bytes_spilled_to_*_storage > 0`
- **Cause:** Warehouse too small for workload
- **Fix:** Increase warehouse size OR optimize query (reduce joins, aggregations)

**Issue 2: Poor Partition Pruning**
- **Symptom:** High `partitions_scanned / partitions_total` ratio
- **Cause:** Missing or ineffective clustering keys, no WHERE clause on cluster key
- **Fix:** Add clustering key on frequently filtered columns, ensure WHERE clause uses cluster key

**Issue 3: Low Cache Hit Rate**
- **Symptom:** `percentage_scanned_from_cache < 50%`
- **Cause:** Insufficient query pattern repetition, warehouse suspended too quickly
- **Fix:** Increase auto-suspend time, consolidate similar queries, use result caching

**Issue 4: Queue Delays**
- **Symptom:** `queued_overload_time > 0` or `queued_provisioning_time > 0`
- **Cause:** Warehouse under-provisioned for concurrency
- **Fix:** Enable multi-cluster warehouse, increase max_cluster_count

**Issue 5: Long Compilation Time**
- **Symptom:** `compilation_time > execution_time`
- **Cause:** Complex query with many joins/subqueries
- **Fix:** Simplify query, use CTEs, materialize intermediate results

### 6. Warehouse Sizing Recommendations

**When to size up (larger warehouse):**
- Consistent memory spilling
- Individual queries taking too long
- Complex analytical queries

**When to scale out (multi-cluster):**
- Queue delays during peak times
- High concurrency workloads
- Many users/applications competing

**When to size down:**
- No spilling, fast query times
- Low utilization (< 50% avg)
- Cost optimization opportunity

### 7. Cost Optimization Check

```sql
-- Warehouse credit consumption (last 30 days)
SELECT 
    warehouse_name,
    SUM(credits_used) as total_credits,
    AVG(credits_used_compute) as avg_compute_credits,
    AVG(credits_used_cloud_services) as avg_cloud_services_credits
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
WHERE start_time >= dateadd('days', -30, current_timestamp())
GROUP BY warehouse_name
ORDER BY total_credits DESC;
```

## Output Format

```
🚀 SNOWFLAKE PERFORMANCE ANALYSIS

SCOPE: <Query ID / Warehouse / General>
TIMEFRAME: <period analyzed>

📊 KEY FINDINGS:

Query Performance:
- Total time: <sec>
- Compilation: <sec> | Execution: <sec>
- Data scanned: <size>
- Cache hit rate: <%>
- Partition pruning: <scanned/total> (<%>)

⚠️  ISSUES DETECTED:

1. <Issue name>
   Severity: 🔴 HIGH / 🟡 MEDIUM / 🟢 LOW
   Evidence: <metric values>
   Impact: <performance/cost impact>
   Fix: <recommended action>

2. <Next issue...>

💡 RECOMMENDATIONS:

Immediate actions:
1. <action> - Expected improvement: <X%>
2. <action> - Expected improvement: <X%>

Long-term optimizations:
1. <action>
2. <action>

📈 WAREHOUSE SIZING:
Current: <size>
Recommended: <size/configuration>
Rationale: <reasoning>

💰 COST IMPACT:
Current spend: $<amount>/month
Estimated savings: $<amount>/month (<X%>)
```

## Best Practices

- Always get baseline metrics before suggesting changes
- Prioritize issues by impact (slow queries > cost > minor optimizations)
- Provide specific SQL for monitoring improvements
- Consider cost vs. performance tradeoffs
- Check for recent Snowflake feature releases (Query Acceleration Service, Search Optimization)
- Use EXPLAIN plan for query-specific analysis
- Monitor before/after metrics to validate improvements

## Snowflake Performance Features to Consider

- **Query Acceleration Service:** For highly variable query execution times
- **Search Optimization Service:** For point lookups and selective filters
- **Materialized Views:** For repeated expensive aggregations
- **Clustering Keys:** For large tables with common filter patterns
- **Result Caching:** Already automatic, but verify it's being utilized
- **Warehouse Size:** Match to workload complexity
- **Multi-Cluster:** For concurrency needs
