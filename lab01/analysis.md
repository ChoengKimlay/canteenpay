# Lab-01: failure report and analysis

Pair: Choeng Kimlay (Student A), HAI Tongmeng (Student B)

## FR-01: SubsidyPolicy grants a subsidy at the exclusive 14:00 lunch boundary

Reported by: HAI Tongmeng, 2026-11-27 · Component: canteen-core 1.0.0, SubsidyPolicy
Environment: JDK 25.0.5.0.3, Maven 3.9.9, Windows 11, supplied version `e400f48`

### Steps to reproduce
1. Use an active scholarship student (`CP-20270001-9`), basket `[KUY_TEAV]`, local time `2027-03-15T14:00:00` in Phnom Penh, and `alreadyUsedToday = false`.
2. Call `SubsidyPolicy.subsidy(...)` with those arguments using the pre-fix `SubsidyPolicy` from `e400f48` and the tests from `223ce19`.
   Automated: `SubsidyPolicyTest.lunchEndIsExclusive` (`mvn verify` from `canteen-core`).

### Expected
0 riel. The verified example in the Task 5 brief says: "lunchEndIsExclusive 14:00:00 on 2027-03-15 -> 0". The full case-study prose is not included in this checkout.

### Actual
4,000 riel, derived from the supplied pre-fix predicate and the policy cap. The pre-fix Surefire assertion transcript was not retained, so the exact `expected ... but was ...` line below is reconstructed, not quoted from a captured run: `expected: <0L> but was: <4000L>`.

### Severity and impact
Minor: a scholarship meal at exactly 14:00 on a teaching day can receive an unauthorized extra subsidy of up to 4,000 riel. This affects the canteen subsidy budget; at most one affected meal per student per day is relevant to this policy.

### Reproducibility
The faulty comparison is deterministic, but three pre-fix runs were not captured; reproducibility is therefore not reported as an observed `3 of 3`. The neighboring results below are evaluated from the pre-fix predicate. The existing 13:59:59 case is also retained in `SubsidyPolicyTest`.

| Input | Expected | Actual from pre-fix predicate | Verdict |
|-------|----------|-------------------------------|---------|
| 2027-03-15 13:59:59 | 4,000 | 4,000 | Pass |
| 2027-03-15 14:00:00 | 0 | 4,000 | Fail |
| 2027-03-15 14:00:01 | 0 | 0 | Pass |

The sampled failing region is the single 14:00:00 instant: the points one second to either side satisfy the rule. The 14:00:01 result is reasoned from the source, not from a retained test run.

Suspected fault (reconstructed; no pre-inspection note was saved): the lunch-end comparison may allow equality at 14:00.

## The fix
Smallest change in `SubsidyPolicy`:
```diff
-        if (t.isBefore(LUNCH_START) || t.isAfter(LUNCH_END)) return 0;
+        if (t.isBefore(LUNCH_START) || !t.isBefore(LUNCH_END)) return 0;
```
After the fix: `mvn verify` from `canteen-core` -> Tests run: 25, Failures: 0. The current Surefire report records 12 subsidy tests passing.

## Analysis (at most one page)
The suite contains 12 `SubsidyPolicyTest` tests. The intended pre-fix result is one failing test, `lunchEndIsExclusive`, at 14:00:00; its expected/actual values and neighboring outcomes are reconstructed from the old predicate because no pre-fix Surefire failure transcript or three-run record was retained. At one-second probe granularity, the defect is isolated to the exact boundary; 13:59:59 and 14:00:01 should pass, while 14:00:00 fails. The post-fix verification is observed: all 25 tests pass.

The examples expose the inclusive-start/exclusive-end distinction and show why checking only a typical lunch time would miss this defect. They do not exhaust calendar data: for example, a regression that incorrectly marks 2027-04-16 as a teaching day would pass all twelve tests, which check 2027-04-15 but not the last Khmer New Year holiday. Add `khmerNewYearLastDayIsNotATeachingDay` for a scholarship student's meal at 2027-04-16 12:00 and expect zero. Tests verify the selected examples and boundaries; they cannot establish correctness for untested dates or times, so requirements analysis and broader boundary coverage remain necessary.