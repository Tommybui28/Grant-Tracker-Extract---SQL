# Schema Discovery & Troubleshooting

Helper queries used to find related tables and diagnose type-mismatch errors during the extraction.

---

## 1. Find every table with a foreign key to a parent table

Reads real FK metadata, so nothing is missed or guessed.

```sql
SELECT
    fk.name                              AS FK_Name,
    OBJECT_NAME(fk.parent_object_id)     AS ChildTable,
    cChild.name                          AS ChildColumn,
    OBJECT_NAME(fk.referenced_object_id) AS ParentTable,
    cParent.name                         AS ParentColumn
FROM        sys.foreign_keys AS fk
JOIN        sys.foreign_key_columns AS fkc ON fk.object_id = fkc.constraint_object_id
JOIN        sys.columns AS cChild  ON fkc.parent_object_id     = cChild.object_id
                                  AND fkc.parent_column_id     = cChild.column_id
JOIN        sys.columns AS cParent ON fkc.referenced_object_id = cParent.object_id
                                  AND fkc.referenced_column_id = cParent.column_id
WHERE       OBJECT_NAME(fk.referenced_object_id) = 'RV_Grants'
ORDER BY    ChildTable;
```

---

## 2. Find tables by column name (when FKs aren't enforced)

```sql
SELECT   t.name AS TableName, c.name AS ColumnName
FROM     sys.columns AS c
JOIN     sys.tables  AS t ON c.object_id = t.object_id
WHERE    c.name = 'GrantID'
ORDER BY t.name;
```

Swap `'GrantID'` for `'ContactID'`, `'EntityID'`, etc. as needed.

---

## 3. Check column data types across tables

Use this to spot type mismatches before they break a batch.

```sql
SELECT   t.name AS TableName, c.name AS ColumnName, ty.name AS DataType
FROM     sys.columns c
JOIN     sys.tables  t  ON c.object_id     = t.object_id
JOIN     sys.types   ty ON c.user_type_id  = ty.user_type_id
WHERE    c.name = 'GrantID'
ORDER BY ty.name, t.name;
```

---

## Troubleshooting: `Msg 8169 — Conversion failed when converting from a character string to uniqueidentifier`

**Cause.** The parent key is a `uniqueidentifier` (GUID) while a child table's `GrantID` is
`varchar`/`nvarchar` containing at least one value that isn't a valid GUID (empty string, `'N/A'`,
an integer, etc.). `uniqueidentifier` has higher type precedence, so SQL Server implicitly converts
the text column to a GUID for the comparison — and errors on the first bad value.

**Knock-on effects.**
- The batch **aborts**, so any statements after the failing one never run.
- A trailing `DROP TABLE #Temp` never runs, so the temp table **still exists** in the session —
  you can resume from the failure point without rebuilding it.

**Fix A — safe convert on the offending table only**

```sql
SELECT t.*
FROM   RV_RO_Grants AS t
WHERE  TRY_CONVERT(uniqueidentifier, t.GrantID) IN (SELECT ID FROM #EligibleGrants);
```

`TRY_CONVERT` turns any non-GUID string into `NULL` (which simply won't match) instead of erroring.

**Fix B — bulletproof pattern for every table (recommended for full reruns)**

```sql
WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants)
```

Both sides become plain text, so no GUID conversion is ever attempted. Default collation makes the
match case-insensitive, which is fine for GUID strings, and it works regardless of which side is the
GUID.

**Reverse case.** If the parent `ID` is the `varchar` and the child `GrantID` is the GUID, flip it:

```sql
WHERE t.GrantID IN (SELECT TRY_CONVERT(uniqueidentifier, ID) FROM #EligibleGrants)
```

---

## Locating the failing statement

The error's `Line NN` refers to the line within the batch, not the table order. Count the successful
`(N rows affected)` messages to identify the last statement that completed — the failing statement is
the next one in sequence.
