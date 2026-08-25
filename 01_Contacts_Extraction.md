# Contacts Extraction — `RV_Contacts` + Linked Tables

Extracts eligible contacts from `RV_Contacts` and their linked tables
(`RV_Qualifications`, `RV_EmploymentHistory`, `RV_DynamicFieldValues`).

---

## Criteria

| Rule | Detail |
|---|---|
| Active | `= 'Y'` |
| Deceased | `= 'N'` |
| Email | Must **not** contain `cctechnology.com`, `symplectic.co.uk`, `prostatecanceruk.org` |
| Contact type | Must have at least one row in `RV_ContactTypes` |
| Single type | Exclude if the only type is `Staff Member`, `Peer reviewer`, `Department Head` or `Finance Officer` |
| Multiple types | Always keep |
| Grants | Exclude if **all** linked grants are `Outcome = 'Rejected'` **and** `SubmittedOn` older than 9 years |

---

## Full query (CTE version — single result set)

```sql
/*───────────────────────────────────────────────────────────────
  Step 1: Base RV_Contacts filter
───────────────────────────────────────────────────────────────*/
WITH FilteredContacts AS (
    SELECT c.*
    FROM        RV_Contacts AS c
    WHERE       c.Active   = 'Y'
      AND       c.Deceased = 'N'
      AND       c.Email NOT LIKE '%cctechnology.com%'
      AND       c.Email NOT LIKE '%symplectic.co.uk%'
      AND       c.Email NOT LIKE '%prostatecanceruk.org%'
),

/*───────────────────────────────────────────────────────────────
  Step 2: ContactType rule
    - Must have at least one ContactType
    - If exactly 1 type   -> keep only if NOT an excluded role
    - If more than 1 type -> always keep
───────────────────────────────────────────────────────────────*/
QualifyingTypes AS (
    SELECT   ct.ContactID
    FROM     RV_ContactTypes AS ct
    GROUP BY ct.ContactID
    HAVING   COUNT(*) > 1
          OR MAX(ct.Type) NOT IN (
                 'Staff Member',
                 'Peer reviewer',
                 'Department Head',
                 'Finance Officer'
             )
),

/*───────────────────────────────────────────────────────────────
  Step 3: Grants exclusion
    Exclude contacts whose grants are ALL:
        Outcome = 'Rejected' AND SubmittedOn older than 9 years
    (Contacts with no grants are NOT excluded here.)
───────────────────────────────────────────────────────────────*/
ContactsToExclude AS (
    SELECT   gc.ContactID
    FROM     RV_GrantContacts AS gc
    JOIN     RV_Grants        AS g ON gc.GrantID = g.ID
    GROUP BY gc.ContactID
    HAVING   COUNT(*) = SUM(
                 CASE
                     WHEN g.Outcome = 'Rejected'
                      AND g.SubmittedOn < DATEADD(YEAR, -9, GETDATE())
                     THEN 1 ELSE 0
                 END
             )
),

/*───────────────────────────────────────────────────────────────
  Step 4: Final qualifying contact set
───────────────────────────────────────────────────────────────*/
EligibleContacts AS (
    SELECT   c.*
    FROM     FilteredContacts AS c
    INNER JOIN QualifyingTypes AS qt ON c.ID = qt.ContactID
    WHERE    c.ID NOT IN (SELECT ContactID FROM ContactsToExclude)
)

SELECT * FROM EligibleContacts;
```

> **CTE scope:** a CTE is only visible to the *single* statement immediately following it.
> For multiple output tables, use the staging-table version below.

---

## Staging-table version (recommended — separate result sets)

```sql
IF OBJECT_ID('tempdb..#EligibleContacts') IS NOT NULL DROP TABLE #EligibleContacts;

WITH FilteredContacts AS (
    SELECT c.*
    FROM   RV_Contacts AS c
    WHERE  c.Active   = 'Y'
      AND  c.Deceased = 'N'
      AND  c.Email NOT LIKE '%cctechnology.com%'
      AND  c.Email NOT LIKE '%symplectic.co.uk%'
      AND  c.Email NOT LIKE '%prostatecanceruk.org%'
),
QualifyingTypes AS (
    SELECT   ct.ContactID
    FROM     RV_ContactTypes AS ct
    GROUP BY ct.ContactID
    HAVING   COUNT(*) > 1
          OR MAX(ct.Type) NOT IN ('Staff Member','Peer reviewer','Department Head','Finance Officer')
),
ContactsToExclude AS (
    SELECT   gc.ContactID
    FROM     RV_GrantContacts AS gc
    JOIN     RV_Grants        AS g ON gc.GrantID = g.ID
    GROUP BY gc.ContactID
    HAVING   COUNT(*) = SUM(CASE WHEN g.Outcome = 'Rejected'
                                  AND g.SubmittedOn < DATEADD(YEAR, -9, GETDATE())
                             THEN 1 ELSE 0 END)
)
SELECT   c.*
INTO     #EligibleContacts
FROM     FilteredContacts AS c
INNER JOIN QualifyingTypes AS qt ON c.ID = qt.ContactID
WHERE    c.ID NOT IN (SELECT ContactID FROM ContactsToExclude);


-- Output: parent + linked tables
SELECT *    FROM #EligibleContacts;
SELECT q.*  FROM RV_Qualifications     AS q WHERE q.ContactID IN (SELECT ID FROM #EligibleContacts);
SELECT e.*  FROM RV_EmploymentHistory  AS e WHERE e.ContactID IN (SELECT ID FROM #EligibleContacts);
SELECT d.*  FROM RV_DynamicFieldValues AS d WHERE d.EntityID  IN (SELECT ID FROM #EligibleContacts);

DROP TABLE #EligibleContacts;
```

---

## Notes & watch-outs

- **Email `NULL`s:** `NULL NOT LIKE '...'` evaluates to `NULL`, so contacts with a blank email are excluded. Add `OR c.Email IS NULL` to keep them.
- **">9 years":** uses `SubmittedOn < DATEADD(YEAR, -9, GETDATE())` — strictly older than 9 years.
- **Grants exclusion:** a contact is only dropped if **every** linked grant is rejected-and-old. One recent or non-rejected grant keeps them in.
- **Row explosion:** don't join Qualifications × EmploymentHistory × DynamicFieldValues in one query — a contact with 3 qualifications and 2 jobs returns 6 rows. Keep them as separate result sets.
