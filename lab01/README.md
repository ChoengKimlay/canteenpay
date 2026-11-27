# canteenpay

## Lab-01: Software quality in a nutshell

Pair: Choeng Kimlay (GitHub: ChoengKimlay), HAI Tongmeng (GitHub: TongmengHAI)

## Files changed in this lab
| File | New or changed | What and why |
|------|----------------|--------------|
| `canteen-core/pom.xml` | supplied, unchanged | Maven build, JUnit 6, AssertJ, and JaCoCo configuration |
| `canteen-core/src/main/java/edu/itc/canteen/core/*.java` | supplied | Core domain and policy implementation |
| `canteen-core/src/main/java/edu/itc/canteen/core/SubsidyPolicy.java` | changed | Fix exclusive 14:00 lunch-end check; see `lab01/analysis.md` |
| `canteen-core/src/test/java/edu/itc/canteen/core/ProjectSmokeTest.java` | new | Build and test-framework smoke check |
| `canteen-core/src/test/java/edu/itc/canteen/core/TopUpPolicyTest.java` | new | Top-up fee and validation examples |
| `canteen-core/src/test/java/edu/itc/canteen/core/WalletTest.java` | new | Wallet charge, credit, validation, and transfer behavior |
| `canteen-core/src/test/java/edu/itc/canteen/core/SubsidyPolicyTest.java` | new | Twelve SUB-1 examples, including the lunch-end regression |
| `lab01/quality-requirements.md` | new | Quality requirements and measures |
| `lab01/tools.md` | new | Pair environment details |
| `lab01/vv-mapping.md` | new | Verification and validation activity mapping |
| `lab01/analysis.md` | new | Failure report, fix, and test analysis |
| `README.md` | changed | Lab file inventory and contribution record |

## Reused from earlier labs
None; Lab-01 is the first lab. The supplied `pom.xml`, core Java sources, and initial `ProjectSmokeTest` were used as provided, except for the one-line `SubsidyPolicy` fix.

## Change log
| Date | Change | Tests after the change |
|------|--------|------------------------|
| 2026-11-27 | Lab-01 setup, quality activities, Task 4 tests, and Task 5 subsidy regression test/fix | `mvn verify`: 25 tests, 0 failures |

## Who did what
| Task | Student | Commit(s) |
|------|---------|-----------|
| Setup, Task 1, Task 3, Task 5 tests and fix | A: Choeng Kimlay | `32f6b80`, `19da006`, `223ce19` |
| Task 2, Task 4, Task 5 analysis and README | B: HAI Tongmeng | `7f7350f`, `e400f48`, this commit |
| Challenge and review | A and B | Pair activity; README review should be confirmed by both partners |