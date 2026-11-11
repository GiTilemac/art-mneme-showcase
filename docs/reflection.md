# Reflection

What went wrong, what the agents were good and bad at, and the limits of this project.

[Back to overview](../README.md)

## Failures and fixes

### Caught by the process

| What happened | How it was caught | What changed |
|---|---|---|
| Tests could run against the wrong database if its address was spelled differently | QA agent | The check now compares what the server reports about the two databases, with tests |
| A request body that was not valid UTF-8 got the framework's error instead of the app's error format | QA agent | A reproducing test first, then one handler for the whole app |
| Two unusual inputs caused server errors that no acceptance criterion covered | Developer agent reported them on the issue instead of fixing them | I decided the behaviour; reproducing tests were written, then the fix |
| 32 new tests failed because the tests themselves were wrong, not the code | Full suite run before committing | Tests corrected; suite passed |
| The generated frontend's service layer was itself a mock, had no tests and no longer matched the API contract | A check I asked for before connecting the backend | Layer reshaped to the contract, with tests for the mock |

### Found in deployment and operation

| What happened | Fix |
|---|---|
| The first automated deploy was refused by AWS | GitHub's identity token used a different subject format than the role trusted; the trust rule was corrected |
| The alert query missed failures that happened right after a deploy | The rule now compares the counter with its value 15 minutes earlier instead of using a rate function |
| The first event after a deploy was not counted on the dashboard | Counters are now reported as zero at start-up |
| Dashboard bars counted an event twice | Query fixed after checking it against real events |
| Alert notifications had nowhere to go | Found while going live: the monitoring service's default notification route had no channel; corrected in its settings |
| The on-call agent could not confirm its diagnosis on prod | Added a read-only prod status script and allowed the agent to run it |
| The frontend generator ran out of credits mid-project | The remaining frontend work was done by the agent team from a self-contained brief |

## What AI accelerated

- **Breadth.** Backend, frontend, infrastructure templates, pipelines, telemetry and dashboards in one short project, including areas outside my usual work.
- **Edge-case coverage.** Agents wrote tests for cases I would not have listed by hand, such as concurrent creation of the same record.
- **Options analysis.** Costed comparisons of hosting and stack choices in minutes, which made the decisions faster and better informed.
- **Documentation that stays current.** Contract, schema and architecture documents were updated in the same change as the code.

## Where AI needed a human

- **Deciding.** Left alone, agents fill gaps with plausible assumptions. The stop-and-ask rule exists because undecided questions kept appearing: thresholds, error messages, what to do with an old photo, what to postpone.
- **Knowing what matters.** Which workflow deserves the alert, what an acceptable cost is, and what can wait are product judgements.
- **External systems.** Cloud accounts, permissions and service settings had to be set up and corrected by me.
- **Self-checking.** The agent that writes code is a poor judge of it. A separate QA agent with no ability to change code found defects the developer agent's own tests had passed.
- **Staying in scope.** Without explicit limits, agents refactor and extend. The rule "every changed line traces to the request" was needed.

## Limits of this project

- **No external users.** It is a single-owner project. There are no adoption or customer-impact figures.
- **Deployed and drilled, not operated.** The system ran on AWS and was exercised with planned drills. It has not handled an unplanned incident, and the deployment is no longer running.
- **The on-call agent has not fixed a real bug.** In the drill the fault was injected, and the correct outcome was to change nothing.
- **Verification by criteria, not by code review.** I approved acceptance criteria and relied on tests and independent QA. I did not read every line, and parts of the stack are outside my usual languages.
- **No measurement of the agents themselves.** I did not track how often QA failed a task or how many iterations each took.
- **The team pattern is not mine.** The PM, developer and QA split comes from the AI Dev Tools Zoomcamp. The product, decisions, rules and operating setup are my own.
- **One server.** The database shares a disk with the app; daily snapshots are the only protection.

## What I would do next

1. Run an on-call drill with a real seeded bug, to see the agent propose a fix end to end.
2. Measure the agent team: QA failure rate per task and iterations to pass.
3. Move the on-call agent to a short-lived cloud job so it does not depend on a running laptop.
4. Add a second alert for latency on the main workflows.
5. Separate the database from the app server before any wider release.
