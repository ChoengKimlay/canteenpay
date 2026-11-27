# Lab-01 Challenge: from a unit failure to the day's settlement

Pair: Choeng Kimlay (Student A), HAI Tongmeng (Student B)

## Reproduction

Environment: JDK 25.0.5.0.3, Maven 3.9.9, Windows 11. The supplied `SubsidyPolicy` predicate was temporarily restored from baseline `e400f48`; it was not committed. Command from the repository root: `mvn -f canteen-core/pom.xml verify`.

The failing instant is 2027-03-15 14:00:00 in Phnom Penh, on a teaching day. Each purchase is one `KUY_TEAV` (7,000 riel), by an active scholarship student, at stall `S1`, with `alreadyUsedToday = false` and an opening wallet balance of 50,000 riel. The run produced 27 tests and exactly these three failures:

```text
[ERROR] SettlementImpactTest.aMealBoughtAtTheFailingInstantIsChargedInFull:39
expected: 0L
 but was: 4000L
[ERROR] SettlementImpactTest.theInstituteOwesNoSubsidyForMealsBoughtAtTheFailingInstant:56
expected: 0L
 but was: 1200000L
[ERROR] SubsidyPolicyTest.lunchEndIsExclusive:63
expected: 0L
 but was: 4000L
[ERROR] Tests run: 27, Failures: 3, Errors: 0, Skipped: 0
```

## How the failure propagates

- `PurchaseService` receives the incorrect 4,000-riel subsidy, so from the 7,000-riel item price it calculates `charged = 3,000` instead of 7,000; the student's balance ends at 47,000 instead of 43,000.
- `DailyLedger.record(receipt)` records the understated 3,000-rieI charge and marks the day's subsidy as used; the ledger itself does not feed `Settlement` directly.
- The challenge converts each receipt to `Transaction.purchase(receipt)`. Each transaction carries amount 3,000 and subsidy 4,000, so `Settlement.settle` reports 1,200,000 riel of subsidies owed for 300 purchases. It reports 900,000 riel charged to cards, while gross stall sales remain 2,100,000 riel because settlement adds the 1,200,000 subsidy to the 900,000 charged.

## Financial impact

Each affected student underpays by 4,000 riel for that meal: their wallet is charged 3,000 rather than the correct 7,000. The institute overpays 4,000 riel per affected student.

| Period | Calculation | Institute overpayment |
|--------|-------------|------------------------|
| One teaching day | 300 students x 4,000 riel | 1,200,000 riel |
| Teaching week | 5 days x 1,200,000 riel | 6,000,000 riel |
| Semester | 15 weeks x 6,000,000 riel | 90,000,000 riel |

Assumptions: the same 300 distinct scholarship students each buy one Kuy teav at exactly 14:00 on every teaching day; the defect remains deployed for all 5-day weeks across 15 weeks; there are no refunds or other corrections. The week and semester figures are projections from the one-day scenario, not measured production losses.

## Earliest detection cost

Task 3 activity A1, the boundary checklist during Phase 1 (Requirements), is the earliest mapped activity: it asks whether “until” boundaries are inclusive or exclusive before design and implementation depend on the rule. Its estimated cost is about 3 person-hours per iteration: one hour per developer plus a half-hour joint follow-up with the scholarship office. Compared with 90,000,000 riel projected over the semester, 3 hours are equal in monetary cost only at a fully loaded rate of 30,000,000 riel per person-hour; the actual labor-to-loss comparison depends on the team's rate. If the rule was already clear and only the implementation misread it, A1 might not catch the coding mistake; mapped activity A4 (Phase 3, Implementation) is then the earliest direct check, at about 1 developer-hour per policy class.

## Restored verification

After restoring the fixed `SubsidyPolicy`, `mvn -f canteen-core/pom.xml verify` passed: 27 tests, 0 failures, 0 errors, 0 skipped.