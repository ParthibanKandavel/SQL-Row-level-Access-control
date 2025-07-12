# Row-Level Access Control (RLAC) Examples in SQL

This document contains SQL implementations of Row-Level Access Control (RLAC) for Snowflake, PostgreSQL, and SQL Server.

---

## 1. Snowflake: Row Access Policy

### Step 1: Create a Row Access Policy
```sql
CREATE OR REPLACE ROW ACCESS POLICY rlp_region_filter
AS (region STRING) RETURNS BOOLEAN ->
    CURRENT_ROLE() = 'SALES_EAST_ROLE' AND region = 'EAST'
    OR CURRENT_ROLE() = 'SALES_WEST_ROLE' AND region = 'WEST';
```

### Step 2: Apply Policy to Table
```sql
ALTER TABLE customer_data
ADD ROW ACCESS POLICY rlp_region_filter ON (region);
```

Now users with the appropriate role will see only region-specific rows.

---

## 2. PostgreSQL: Row-Level Security (RLS)

### Step 1: Enable RLS on Table
```sql
ALTER TABLE customer_data ENABLE ROW LEVEL SECURITY;
```

### Step 2: Create a Policy
```sql
CREATE POLICY regional_policy
ON customer_data
FOR SELECT
TO etl_user
USING (region = current_setting('app.region'));
```

### Step 3: Set Runtime Session Variable
```sql
SET app.region = 'EAST';
```

This allows `etl_user` to see only the data relevant to the region.

---

## 3. SQL Server: Security Predicate

### Step 1: Create Predicate Function
```sql
CREATE FUNCTION fn_security_predicate(@region AS VARCHAR(10))
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN SELECT 1 AS fn_security_predicate_result
WHERE @region = USER_NAME();
```

### Step 2: Create Security Policy
```sql
CREATE SECURITY POLICY RegionFilterPolicy
ADD FILTER PREDICATE dbo.fn_security_predicate(region)
ON dbo.customer_data
WITH (STATE = ON);
```

This will enforce row-level filtering automatically.

---

## Notes:
- Replace hardcoded logic with role/region mapping tables in production.
- Combine with RBAC for full control.
- Test with different roles or session settings.

---

## Sample Table Used:
```sql
CREATE TABLE customer_data (
  customer_id INT,
  region VARCHAR,
  sales_amount DECIMAL
);
```

---

Feel free to modify this template to suit your enterprise RLS needs across platforms.
