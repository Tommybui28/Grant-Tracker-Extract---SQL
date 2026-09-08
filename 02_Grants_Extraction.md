# Grants Extraction — `RV_Grants` + All 47 GrantID Tables

Extracts eligible grants from `RV_Grants` and every related table carrying a `GrantID` column.

---

## Criteria

| Rule | Detail |
|---|---|
| Status | Not `Deleted`, not `Pre-Submission` |
| Outcome | Not `Invited`, not `Rejected` |
| SubmittedOn | Not older than 9 years (i.e. within the last 9 years) |

---

## Step 1 — Eligible grants only

```sql
SELECT g.*
INTO   #EligibleGrants
FROM   RV_Grants AS g
WHERE  g.Status  NOT IN ('Deleted', 'Pre-Submission')
  AND  g.Outcome NOT IN ('Invited', 'Withdrawn')
  AND  NOT (g.Outcome = 'Rejected' AND g.SubmittedOn < DATEADD(YEAR, -9, GETDATE()));
```

---

## Step 2 — Full extraction script (rerun-safe)

Uses `CONVERT(NVARCHAR(50), …)` on both sides of every comparison so a single
non-GUID value can never abort the batch with a `uniqueidentifier` conversion error.

```sql
/*═══════════════════════════════════════════════════════════════
  RV_Grants extraction — eligible grants + all 47 related tables
═══════════════════════════════════════════════════════════════*/

IF OBJECT_ID('tempdb..#EligibleGrants') IS NOT NULL DROP TABLE #EligibleGrants;

SELECT g.*
INTO   #EligibleGrants
FROM   RV_Grants AS g
WHERE  g.Status  NOT IN ('Deleted', 'Pre-Submission')
  AND  g.Outcome NOT IN ('Invited', 'Rejected')
  AND  g.SubmittedOn >= DATEADD(YEAR, -9, GETDATE());

-- Parent table
SELECT * FROM #EligibleGrants;

-- All related tables (filtered by eligible GrantID)
SELECT t.* FROM RV_BudgetItems                                AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_BudgetItemsAppFormControlGridRowValues     AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_BudgetItemsAppFormControlValues            AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ChangeRequests                             AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_Claims                                     AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_CustomConflicts                            AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_FinancialMonitoringByQuarter               AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_FinancialMonitoringData_By_Financial_Year  AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantActivationChecks                      AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantAppFormControlGridRowValues           AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantAppFormControlValues                  AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantClassifications                       AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantContacts                              AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantContractDocuments                     AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantContracts                             AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantDocuments                             AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantEmailReceived                         AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantEmailSent                             AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantExchangeRateHistory                   AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantExcludedReviewers                     AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantLeadApplicantChanges                  AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantOrganisationChanges                   AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantOrganisations                         AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantPayees                                AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantRecommendedReviewers                  AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantRejectionReasons                      AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantReviewMeetings                        AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantStaffPositions                        AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantStatusHistoryView                     AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_GrantYears                                 AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_OverallActiveGrantFinanceFigures           AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_PaymentCheckpoints                         AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ProgrammeGrants                            AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ProgressReports                            AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ProgressReportsAppFormControlGridRowValues AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ProgressReportsAppFormControlValues        AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ReportingID_GrantChecklists                AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ReportingID_GrantInitialRequirements       AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ReportingID_Grants                         AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ReportingID_Grants_FormGrids               AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ReportingID_Grants_IncludingSubForms       AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_Reviews                                    AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ReviewsAppFormControlGridRowValues         AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ReviewsAppFormControlValues                AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_RO_Grants                                  AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_ScheduledPayments                          AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_Suspensions                                AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);
SELECT t.* FROM RV_Transactions                               AS t WHERE CONVERT(NVARCHAR(50), t.GrantID) IN (SELECT CONVERT(NVARCHAR(50), ID) FROM #EligibleGrants);

DROP TABLE #EligibleGrants;
```

---

## Related tables (47)

| # | Table | # | Table |
|---|---|---|---|
| 1 | RV_BudgetItems | 25 | RV_GrantRecommendedReviewers |
| 2 | RV_BudgetItemsAppFormControlGridRowValues | 26 | RV_GrantRejectionReasons |
| 3 | RV_BudgetItemsAppFormControlValues | 27 | RV_GrantReviewMeetings |
| 4 | RV_ChangeRequests | 28 | RV_GrantStaffPositions |
| 5 | RV_Claims | 29 | RV_GrantStatusHistoryView |
| 6 | RV_CustomConflicts | 30 | RV_GrantYears |
| 7 | RV_FinancialMonitoringByQuarter | 31 | RV_OverallActiveGrantFinanceFigures |
| 8 | RV_FinancialMonitoringData_By_Financial_Year | 32 | RV_PaymentCheckpoints |
| 9 | RV_GrantActivationChecks | 33 | RV_ProgrammeGrants |
| 10 | RV_GrantAppFormControlGridRowValues | 34 | RV_ProgressReports |
| 11 | RV_GrantAppFormControlValues | 35 | RV_ProgressReportsAppFormControlGridRowValues |
| 12 | RV_GrantClassifications | 36 | RV_ProgressReportsAppFormControlValues |
| 13 | RV_GrantContacts | 37 | RV_ReportingID_GrantChecklists |
| 14 | RV_GrantContractDocuments | 38 | RV_ReportingID_GrantInitialRequirements |
| 15 | RV_GrantContracts | 39 | RV_ReportingID_Grants |
| 16 | RV_GrantDocuments | 40 | RV_ReportingID_Grants_FormGrids |
| 17 | RV_GrantEmailReceived | 41 | RV_ReportingID_Grants_IncludingSubForms |
| 18 | RV_GrantEmailSent | 42 | RV_Reviews |
| 19 | RV_GrantExchangeRateHistory | 43 | RV_ReviewsAppFormControlGridRowValues |
| 20 | RV_GrantExcludedReviewers | 44 | RV_ReviewsAppFormControlValues |
| 21 | RV_GrantLeadApplicantChanges | 45 | RV_RO_Grants |
| 22 | RV_GrantOrganisationChanges | 46 | RV_ScheduledPayments |
| 23 | RV_GrantOrganisations | 47 | RV_Suspensions |
| 24 | RV_GrantPayees | — | RV_Transactions |

---

## Notes & watch-outs

- **`NULL` handling:** `NOT IN` returns `NULL` (row excluded) if `Status`/`Outcome` is `NULL`. Add `OR g.Outcome IS NULL` to keep blanks. Rows with a `NULL` `SubmittedOn` are also dropped by the `>=` comparison.
- **Views vs tables:** `RV_GrantStatusHistoryView` and `RV_OverallActiveGrantFinanceFigures` look like views by naming convention. They query fine as long as they expose a `GrantID` column.
- **Result sets:** the script returns 48 separate result sets (parent + 47 related).
