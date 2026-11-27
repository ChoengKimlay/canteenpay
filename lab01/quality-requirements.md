# Lab-01 Task 2: quality requirements of CanteenPay

Pair: CHOENG Kimlay, HAI Tongmeng

Columns: internal or external (or process) · ISO/IEC 25010:2023 characteristic ·
dependability property (correctness, reliability, robustness, safety) or process
property (timeliness, visibility, improvability) · how it could be measured.

| Id  | Statement | Internal / external | ISO/IEC 25010 characteristic | Dependability or process property | How it could be measured |
|-----|-----------|---------------------|------------------------------|-----------------------------------|--------------------------|
| Q1  | `subsidy(...)` returns exactly the amount of rule SUB-1 for every purchase | external | functional suitability (functional correctness) | correctness | every clause of SUB-1 has a passing test; no open failure report on SUB-1 |
| Q2  | 999 of 1,000 kiosk purchases on a teaching day produce a correct receipt within 2 s | external | performance efficiency (time behavior) | correctness | count of passing receipts under 2 s out of 1,000 purchases in k6 load-test report and kiosk latency log |
| Q3  | When the network is down, a stall terminal keeps accepting purchases and sends them later | external | reliability (fault tolerance) | robustness | 100% of offline stall orders queued locally and flushed in terminal outbox sync log after a 15-minute network drop |
| Q4  | A card is never charged more than its balance, and never twice for one meal, whatever the kiosk and the app do at the same moment | external | reliability (fault tolerance) | safety | 0 negative-balance or duplicate-meal charges across 500 concurrent kiosk/app trials in concurrency test log |
| Q5  | A new holiday can be added to the academic calendar without changing `SubsidyPolicy` | internal | maintainability (modularity) | improvability | 0 lines modified in `SubsidyPolicy` across 5 added holiday entries in Git diff log |
| Q6  | `SubsidyPolicy` takes the time as a parameter instead of reading the system clock | internal | maintainability (testability) | correctness | 0 clock-API calls inside `SubsidyPolicy` and 100% deterministic unit tests in static-analysis and unit-test report |
| Q7  | The daily settlement balances to the riel: credited top-ups - purchases + refunds = change of all balances | external | functional suitability (functional correctness) | correctness | 0 KHR discrepancy across 30 daily runs in settlement reconciliation batch report |
| Q8  | The team can say at every build which rules have passing tests and how many failure reports are open | process | none: process | visibility | share of rules with at least one passing test, and the count of open reports, published per build |
| Q9  | A failure found at the pilot kiosk is traced to its root cause and a test is added so that it cannot come back | process | none: process | improvability | 100% of pilot failure tickets linked to a root-cause note and a new regression test in issue tracker and CI commit history |
| Q10 | The unit tests of a rule are written in the same iteration as the rule, not before the pilot release | process | none: test timing constraint not modeled as a product characteristic | timeliness | 100% of rule commits paired with unit tests in the same sprint branch in Git/sprint tracker log |
| Q11 | The third wrong PIN in a row blocks the card | external | security (accountability / authentication) | safety | 100% of cards blocked on the 3rd consecutive wrong PIN in 50 test runs in authentication test log |
| Q12 | A first-year student tops up with KHQR at the kiosk without help in under one minute | external | interaction capability (user engagement / operability) | reliability | at least 18 of 20 first-year pilot testers complete KHQR top-up in less than 60 s without assistance in usability observation log |

## The two hardest rows
The two hardest rows are Q4 and Q12. Checking Q4 requires reproducing race conditions between two independent clients under network delays and retries, so a quick manual test can miss rare double charges or overdrafts. Checking Q12 depends on real first-year students who may hesitate or misread prompts, meaning lab runs with experienced testers can give a false sense of ease. Anyone verifying Q4 or Q12 who cuts corners risks shipping financial bugs or confusing kiosk flows during the pilot.