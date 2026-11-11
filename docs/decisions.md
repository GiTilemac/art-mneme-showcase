# Decisions

The problem, and the engineering decisions with the alternatives considered.

[Back to overview](../README.md)

## The problem

People who visit museums and galleries want to remember what they saw and find it again. Photo galleries, notes apps and spreadsheets store the items but do not help with rediscovery: nothing connects one work to others by the same artist, on the same subject, from the same movement or in the same museum.

ArtMneme is built around one order of priority: collection first, exploration second, community third. The first version covers a private collection, a watchlist of works still to see, and relationship exploration within the user's own artworks.

Why AI agents for the build: the scope was a complete system, from specification to an on-call process, in a short time and partly in a stack I do not normally work in. That is only feasible with agents doing the implementation, and only safe with tight rules on what they may decide.

## Decision records

Each decision below was mine. For most, an AI assistant laid out the options and trade-offs first.

### Backend stack

| | |
|---|---|
| **Decision** | Python and FastAPI |
| **Alternatives** | TypeScript backend (Fastify or NestJS); Django with Django REST Framework; Next.js full-stack; a backend-as-a-service |
| **Reasoning** | I can read Python business rules directly. FastAPI validates requests and generates the API documentation from the contract, so rules can be checked without writing code. Next.js and backend-as-a-service were rejected because they blur the line between frontend, API and backend, which the contract-first process depends on. |

### API contract before code

| | |
|---|---|
| **Decision** | Define every endpoint in OpenAPI before any backend task starts |
| **Alternative** | Let the contract emerge from the implementation |
| **Reasoning** | The frontend was generated separately. A fixed contract let the frontend and backend be built by different agents and still fit. It also gave QA something exact to check against. |

### Hosting

| | |
|---|---|
| **Decision** | One EC2 server running Docker Compose, created with CloudFormation |
| **Alternatives** | Four AWS options were compared with their costs |
| **Reasoning** | Lowest cost and fewest moving parts for a single-user first version, while still reproducible from a template. The database on the server's disk is the main trade-off, covered by daily snapshots. |

### Photo storage

| | |
|---|---|
| **Decision** | Private S3 bucket, photos served through the backend |
| **Alternative** | Cloudflare R2, which was the original choice |
| **Reasoning** | Once hosting moved to AWS, a second provider only added credentials to manage. With S3 the server uses its role and no keys are stored. Serving through the backend keeps every photo private to its owner. |

### Environments

| | |
|---|---|
| **Decision** | Dev runs locally; the AWS stack is prod; deploy to prod is a manual step |
| **Alternative** | A cloud dev environment that receives every push, with manual promotion to prod |
| **Reasoning** | A second cloud stack doubles the cost for one developer. Every push is still fully tested, and promotion to prod stays a deliberate act. GitHub's protected environments were not available on the plan in use, so the deploy role is restricted to the main branch instead. |

### Observability stack

| | |
|---|---|
| **Decision** | Grafana Cloud free tier, shared by dev and prod |
| **Alternative** | Self-hosted Prometheus, Loki, Tempo and Grafana on a second server |
| **Reasoning** | No server to run or pay for, and monitoring stays up when the app server is down. The self-hosted stack is kept as a local option for offline work. |

### Alert threshold

| | |
|---|---|
| **Decision** | Fire on one server-side creation failure within 15 minutes |
| **Alternative** | Three failures within ten minutes |
| **Reasoning** | With little traffic, a blocked user may try once or twice and leave. A higher threshold would miss exactly the case the alert exists for. |

### On-call agent authority

| | |
|---|---|
| **Decision** | The agent may commit a fix on its own branch; a human reviews, merges and deploys |
| **Alternatives** | Investigate and report only; or allow the agent to deploy |
| **Reasoning** | A proposed fix with passing tests saves real time. Pushing and deploying stay with a human because an agent acting on a wrong diagnosis in production costs more than a slower fix. |

### Where the on-call agent runs

| | |
|---|---|
| **Decision** | Locally, started by one command |
| **Alternative** | A cloud version: alert to a queue, a short-lived container job running the agent |
| **Reasoning** | No extra cost or infrastructure for a first version. The trade-off is that it only works while the script is running. The role file and limits carry over unchanged to a cloud version. |

### Scope

| | |
|---|---|
| **Decision** | Postpone community features, duplicate warnings, detail photos and shared-tag exploration |
| **Alternative** | Build the full specification |
| **Reasoning** | The budget of AI credits was limited. I chose a complete, deployed and monitored core over a wider feature set. Already-built parts were left in place, and the contract and documents were updated so they still describe what exists. |
