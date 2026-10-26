# System Design Gym — Product & Technical Spec

> Working name. A LeetCode-style practice app for system design: you build an
> architecture by connecting components on a canvas, the app throws traffic and
> failures at it, and a test suite tells you whether the design holds.

Status: **Draft v0.2** · Owner: Arsène · Last updated: 2026-10-26

---

## 1. Problem & goals

System design is usually learned passively (videos, books, mock interviews).
There is no equivalent of "write code → run tests → see red → fix → green".
You can't *feel* why a single database falls over at 5k RPS, or why a cache
flush takes your site down.

**Goals**

1. **Practice, not reading.** Each concept is learned by building something that
   has to pass tests.
2. **Visible failure.** A bad design should visibly break: overloaded nodes turn
   red, queues fill up, requests get dropped, latency charts spike.
3. **Progressive difficulty.** Exercises go from "connect a client to a server"
   up to "design a news feed for 50M users", the way LeetCode goes from Two Sum
   to hard DP.
4. **Requirement-driven reasoning.** Every exercise starts from functional and
   non-functional requirements (QPS, latency, availability, budget), so you
   practice going from numbers to architecture.
5. **Practise the process, not just the architecture.** Every exercise follows
   the 4-step interview framework (§3): scope → blueprint → deep dive →
   wrap-up.
6. **Deterministic and explainable.** Same design + same scenario = same
   result. Every failing test explains *why* it failed.

**Non-goals (for v1)**

- Byte-accurate emulation of real systems. The simulator is a teaching model,
  roughly right in shape, not a benchmark.
- Writing or running real code inside components.
- Multiplayer, social features, or a backend with accounts (deferred to v3).

---

## 2. Core concepts

| Concept | What it is |
|---|---|
| **Component** | A node type on the canvas: Client, Load Balancer, App Server, Cache, SQL DB, Queue, etc. Each has capacity, latency, cost, config and failure modes. |
| **Connection** | A directed edge between two components. Traffic flows along edges. An edge can be restricted to certain request types (e.g. `writes only`). |
| **Design** | The graph the user builds: components + connections + per-component config. |
| **Workload** | What traffic looks like: request types, read/write ratio, RPS over time, key distribution, payload size. |
| **Scenario** | A workload plus a timeline of events (traffic spike, kill a node, flush a cache, add network latency). |
| **Test** | A scenario plus assertions on the metrics it produces (p99 latency, error rate, cost, backlog, data loss…). |
| **Exercise** | A problem split into the 4 stages of the framework (§3): scope questions, blueprint (API, palette, budget, baseline tests), deep-dive focus areas, wrap-up. Plus hints and an editorial. |
| **Stage** | One of the 4 steps: ① Scope, ② Blueprint, ③ Deep dive, ④ Wrap-up. Each has its own gate and score. |
| **Focus area** | A deep-dive topic within an exercise (e.g. "ID generation") with its own tests. |

---

## 3. Guiding framework: the 4-step method

The app's structure follows the 4-step framework from Alex Xu's *System Design
Interview* (ByteByteGo, chapter "A Framework for System Design Interviews").
It is the backbone of the exercise flow, the curriculum and the grading, so
practising in the app means rehearsing the same process you'd follow in a real
45-minute interview.

| Step | Interview (≈45 min) | What it becomes in the app |
|---|---|---|
| **1. Understand the problem & establish design scope** | 3–10 min. Ask clarifying questions: features, users, scale now and in 6–12 months, read/write patterns, existing stack. Don't jump to a solution. | **Scope stage.** You don't get the requirements handed to you. You pick clarifying questions to "ask the interviewer" and write a requirements sheet. Then back-of-the-envelope estimates. |
| **2. Propose a high-level design & get buy-in** | 10–15 min. Draw the blueprint with boxes (clients, APIs, servers, data stores, cache, CDN, queue). Sanity-check it against the estimates. Walk through concrete use cases. Optionally API and data model. | **Blueprint stage.** Define the API, then build the graph on the canvas. "Buy-in" = trace each use case through the graph and pass the baseline-load tests. |
| **3. Design deep dive** | 10–25 min. Agree with the interviewer on which components matter most and go deep on them: bottlenecks, trade-offs, specific algorithms. Manage your time. | **Deep-dive stage.** The exercise names 1–3 focus areas (e.g. "cache stampede", "hot shard"). Each has its own chaos/scale tests and component config unlocks. |
| **4. Wrap up** | 3–5 min. Identify bottlenecks, recap, discuss error cases, operations (monitoring, rollout), and the next scale curve (e.g. 1M → 10M users). | **Wrap-up stage.** Predict which node breaks first at 10×, then run the 10× test to check. Failure-mode tests. A short written recap. |

### 3.1 Dos and don'ts as in-app nudges

The chapter's dos and don'ts become gentle, non-blocking prompts:

| Framework advice | Nudge in the app |
|---|---|
| Don't jump into a solution without clarifying | Canvas is locked until the Scope stage is submitted (except in Sandbox) |
| Ask for clarification; don't assume | Requirements you never asked about still apply in the hidden tests. You find out the hard way, and the diagnostic says which question would have revealed it. |
| Don't go too deep on one component early | In the Blueprint stage, advanced config is collapsed. A hint appears if you tune one node a lot before the graph is complete. |
| Design the most critical components first | Deep-dive focus areas are ordered. The first one is worth the most. |
| Suggest multiple approaches; there's no single right answer | Tests check outcomes, never graph shape. Editorials show 2+ valid designs with their trade-offs. You can save design variants and compare runs side by side. |
| Communicate your thinking | Each stage has an optional "why" note field. In Interview mode this is required and reviewed. |
| The interview isn't over once you have a design | Wrap-up stage is part of passing in capstones. |
| Keep an eye on time | Optional timer with per-stage budgets (mandatory in Interview mode). |

### 3.2 Exercise types (like LeetCode tags)

Not every exercise needs all four stages. Early exercises pre-fill some stages
so you focus on one skill. Short "drills" train a single step, much like
targeted LeetCode problems:

| Type | Stages you do | Example |
|---|---|---|
| **Build** | 2 (+3), scope given | "Read-heavy profile page": requirements handed over, build and pass |
| **Scope drill** | 1 only | Given a vague prompt ("design Instagram"), pick the 5 questions that matter most, out of 15 |
| **Estimation drill** | 1 (numbers only) | "10M DAU, 2 posts/day, 1 MB each: storage per year?" |
| **Bottleneck drill** | 4 only | Pre-built design: click the node that fails first at 5× traffic, then run to check |
| **Fix-it** | 3 only | Broken design that fails one chaos test: make it pass with minimal cost |
| **Full design** | 1 → 4 | Capstones (URL shortener, news feed, chat…) |

---

## 4. User experience

### 4.1 Modes

1. **Learn** — one page per fundamental. Each has a short explanation, 1–3
   curated YouTube links, and a 2-minute "lab": a tiny pre-built design with one
   slider to play with (e.g. cache size → watch hit rate and DB load change).
2. **Practice** — the exercise catalog (the core of the app).
3. **Sandbox** — empty canvas, every component unlocked, custom workload. For
   experimenting.
4. *(later)* **Interview** — a full 4-step run against a 45-minute timer with
   per-stage budgets (Scope 3–10, Blueprint 10–15, Deep dive 10–25, Wrap-up
   3–5). The prompt is open-ended and the "interviewer" is an LLM that answers
   your clarifying questions and picks the deep-dive topics. At the end you get
   a rubric review: scoping, high-level design, depth, trade-offs,
   communication.

### 4.2 Exercise screen (the main screen)

```
┌───────────────────────────────────────────────────────────────────────────┐
│ ◀ Catalog   #04 Read-heavy Profile Page   ● Medium   Budget $800/mo       │
│  ① Scope ✓ ─── ② Blueprint ● ─── ③ Deep dive ○ ─── ④ Wrap-up ○  ⏱ 14:32 │
├───────────────┬───────────────────────────────────────────┬───────────────┤
│ PROBLEM       │                CANVAS                     │ INSPECTOR     │
│ Story         │                                           │ (selected     │
│ Functional    │   [Client]──►[LB]──►[App x3]──►[Cache]    │  node config) │
│ Non-functional│                          │                │ instances: 3  │
│  • 20k RPS    │                          └────►[SQL DB]   │ cpu: medium   │
│  • 95% reads  │                                           │ cost: $210/mo │
│  • p99<150ms  │                                           │               │
│ Estimation ✎  ├───────────────────────────────────────────┤               │
│               │ PALETTE: Client LB App Cache SQL Queue …  │               │
│ Hints (0/3)   ├───────────────────────────────────────────┴───────────────┤
│               │ ▶ Run   ⏩ 10x   Scenario: [steady ▾]   Tests 3/5 ✅       │
│               │ ─ charts: RPS · p99 latency · error rate · cost ──────────│
│               │ ─ results: ✅ steady  ✅ budget  ❌ spike  ❌ db-failover   │
└───────────────┴───────────────────────────────────────────────────────────┘
```

The stage bar at the top is always visible. You can go back to an earlier stage
at any time. Moving forward requires that stage's gate to pass.

**Flow for a full exercise (4 stages)**

**① Scope — understand the problem.**
1. Read a deliberately vague story ("Design a URL shortener").
2. **Ask clarifying questions.** Pick from a list of ~15 candidate questions
   ("How many URLs per day?", "Can links expire?", "Custom aliases?", "Which
   tech stack does the team know?"). Each answer the interviewer gives fills in
   a requirement. Some questions are irrelevant. They cost nothing but are
   flagged afterwards, so you learn which ones matter.
3. **Write the requirements sheet**: tick the functional requirements in scope,
   and confirm the non-functional numbers you learned (DAU, read/write ratio,
   latency, availability, retention).
4. **Back-of-the-envelope estimates**: peak QPS, storage over N years,
   bandwidth, cache memory. Each is checked within a tolerance (e.g. ±30%) and
   comes with a worked solution.
5. *Gate:* the requirements sheet is submitted. Missed key questions are
   allowed, and their requirements still apply in the hidden tests.

**② Blueprint — high-level design & buy-in.**
1. *(capstones)* **API**: pick or define endpoints, e.g.
   `POST /shorten`, `GET /{code}`. Each endpoint becomes a request class in
   the workload. *(optional)* Sketch the data model: entities plus the main
   access pattern.
2. **Build the graph**: drag components from the palette, connect and configure
   them. Advanced settings are collapsed at this stage.
3. **Validate** (instant, static): "DB is not reachable from any server",
   "Client must connect to an entry point", "over budget by $120".
4. **Walk through use cases**: for each endpoint, click ▶ trace and watch a
   single request travel its path, with per-hop latency.
5. **Run** the baseline scenario: watch the animation and live charts.
6. *Gate ("buy-in"):* baseline-load tests and the budget test pass.

**③ Deep dive — go deep where it matters.**
1. The exercise names 1–3 **focus areas**, ordered by importance (e.g.
   "Unique ID generation", "Hot links: caching", "Analytics without slowing
   redirects"). Each comes with a short explanation, the component settings it
   unlocks, and its own tests (scale, chaos, consistency).
2. Iterate: change the design, run, read diagnostics.
3. *Gate:* the tests for every focus area pass. Optional areas add stars.

**④ Wrap-up — bottlenecks, failures, next scale.**
1. **Predict the bottleneck**: before running, click the node you think fails
   first at 10× traffic. Then run the 10× scenario and see if you were right.
2. **Error cases**: the hidden chaos suite runs (node loss, network partition,
   cache flush). Results are graded.
3. **Operations** checklist: did you add monitoring or alerting, health checks,
   and a rollout strategy? Lightweight checkboxes that unlock editorial
   sections.
4. **Recap**: three sentences on your design and its main trade-off. Optional,
   but saved with the solution so you can re-read it before an interview.
5. *Done:* the full test results, the efficiency score, and the **editorial**
   with 2+ reference designs and their trade-offs. The editorial also unlocks
   after 3 failed submissions.

### 4.3 Visual language (how "breaking" looks)

| State | Visual |
|---|---|
| Utilization < 70% | Node green, smooth particle flow |
| 70–90% | Node amber, particles slow down |
| > 90% / queue growing | Node red, pulsing, queue bar fills under the node |
| Dropping requests | Red particles "fall off" the edge, error counter ticks up |
| Node down (chaos) | Node greyed out with ✕, edges dashed |
| Cache | Badge with live hit rate (e.g. `hit 87%`) |
| DB replica | Badge with replication lag |
| Queue | Badge with backlog size and drain ETA |

Edge thickness scales with RPS. Hovering a node shows a small sparkline of its
load and latency. A timeline scrubber lets you replay a run and stop at the
moment things went wrong.

### 4.4 Failure diagnostics (the main teaching tool)

Every failed assertion produces a human-readable explanation built from the
metrics, e.g.:

> ❌ **p99 latency < 150 ms** — measured **2,340 ms** at t=45s.
> `sql-db-1` hit **340% utilization**: 6,800 reads/s arrived, capacity is
> 2,000 reads/s. Requests queued and timed out.
> 💡 Over 95% of traffic is reads. What could serve repeated reads without
> touching the database?

Diagnostics come from a rule set (bottleneck detection, SPOF detection,
retry amplification, cache stampede, hot shard…), not free text, so they're
deterministic and testable. Hint text is written per exercise.

---

## 5. Component catalog

Each component has: **capacity** (per instance), **base latency**, **cost**
($/month per instance), **config options**, **failure modes**. The numbers are
deliberately round so they're easy to reason about in your head.

| Component | Key config | Capacity (per instance) | Base latency | Cost | Teaches |
|---|---|---|---|---|---|
| **Client** (traffic source) | workload is set by the scenario | — | — | — | entry point |
| **DNS** | geo routing on/off | ∞ | 5 ms (cached) | $1 | entry, geo routing |
| **CDN** | cache static %, TTL | 100k RPS | 10 ms hit | $50 + per-GB | edge caching |
| **Load Balancer** | algorithm (round-robin, least-conn, hash), health checks on/off | 50k RPS | 1 ms | $20 | horizontal scaling, SPOF |
| **API Gateway / Rate limiter** | limit per client, burst | 30k RPS | 2 ms | $40 | protecting backends |
| **App Server** | instances, size (S/M/L), autoscaling min/max, stateful sessions on/off, timeout, retries | S: 500 / M: 1k / L: 2k RPS | 10 ms | $30 / $60 / $120 | stateless scaling, timeouts, retries |
| **Cache (Redis)** | memory GB, TTL, eviction (LRU/LFU), cluster nodes | 50k ops/s | 1 ms | $40/GB-tier | hit rate, stampede, invalidation |
| **SQL DB** | size, read replicas, sharding (key, shard count), sync/async replication | M: 2k reads/s + 500 writes/s | 5 ms read / 10 ms write | $200 | replicas, failover, sharding |
| **NoSQL / KV store** | partitions, replication factor, consistency (ONE/QUORUM/ALL) | 10k ops/s per partition | 3 ms | $150 | partitioning, tunable consistency |
| **Message Queue** | partitions, retention | 20k msg/s | 2 ms enqueue | $60 | async work, buffering spikes |
| **Worker** | instances, processing time per job | depends on job cost | per job | $30 | consumers, backlog |
| **Object Storage (S3)** | — | ∞ | 50 ms | per-GB | blobs, offloading |
| **Search Index** | shards, refresh interval | 5k queries/s | 20 ms | $250 | secondary access patterns |
| **WebSocket Gateway** *(v3)* | connections per node | 50k conns | 5 ms | $80 | real-time, fan-out |

Components are unlocked progressively through exercises so the palette doesn't
overwhelm beginners.

---

## 6. Simulation engine (the heart of the app)

### 6.1 Requirements

- Runs **in the browser**, in a Web Worker, so the UI never freezes.
- A 120-second simulated scenario finishes in **< 1 s** wall-clock for designs up
  to ~50 nodes (test mode), or plays at 1×/10× for animation (run mode).
- **Deterministic**: seeded PRNG; same inputs → identical outputs. Required
  for tests to be fair and for the engine itself to be unit-testable.
- Pure TypeScript with no UI dependencies (`packages/engine`), so it can also
  run in Node for CI-checking every exercise's reference solution.

### 6.2 Model: fluid flow + sampled requests

Simulating 50k individual requests per second would be slow. Instead there are
two layers:

**Layer 1 — fluid flow (load and capacity).** Time advances in ticks
(`dt = 100 ms`). Each tick, for every request class (e.g. `read:profile`,
`write:post`), the engine walks the graph in topological order and computes the
arrival rate λ at each node:

- **Routing**: a node splits its outgoing flow across its downstream edges
  according to its type. A load balancer splits evenly across *healthy* targets
  (or by hash). An app server sends reads to the cache if one is connected and
  misses to the DB, sends writes to the DB or queue. An edge can override this
  (`reads only`, `writes only`, `async`).
- **Capacity & queueing**: each node has service rate μ = capacity × instances.
  If λ > μ the node's queue grows by (λ − μ)·dt. When the queue passes its
  limit, the excess is dropped (counted as errors). Queue wait ≈ queue / μ
  (Little's law).
- **Utilization** ρ = λ / μ drives the node color and the latency model:
  latency ≈ base × (1 + ρ/(1−ρ)) for ρ < 1 (M/M/1-like curve, capped), so
  latency climbs steeply as you approach saturation — the "hockey stick"
  everyone should internalize.
- **Timeouts & retries**: if a node's latency exceeds the caller's timeout,
  those requests count as failures, and if the caller has retries enabled they
  come back as *extra load* next tick. This reproduces retry storms.

**Layer 2 — sampled requests (latency distribution + animation).** Each tick
the engine samples ~200 synthetic requests per class and routes them through the
graph using the probabilities from layer 1 (cache hit chance, which replica,
drop chance), summing per-hop latencies. That gives a real latency distribution
(p50/p95/p99) and the particles you see on the canvas.

### 6.3 Component behaviors worth modelling precisely

| Behavior | Model |
|---|---|
| **Cache hit rate** | Keys follow a Zipf distribution (skew `s` set by the workload). Hit rate ≈ fraction of request mass covered by the top-K keys, where K = cache capacity / avg item size. Starts cold and warms up over time. |
| **Cache flush / stampede** | Chaos event sets hit rate to 0; all reads go to the DB at once. With no request coalescing, the DB overloads. Config option `coalesce misses` fixes it. |
| **Write-through vs cache-aside** | Affects write path latency and the stale-read metric. |
| **Read replicas** | Add read capacity. Async replication → replication lag grows with write load → `staleReadRatio` metric. Sync → higher write latency. |
| **Failover** | Primary dies: with a replica + auto-failover, writes fail for `failoverTime` (e.g. 10 s) then recover. Without a replica, writes fail until the end of the scenario. |
| **Sharding** | Write and read capacity multiply by shard count. Bad shard key (e.g. `country`, `celebrity user_id`) yields a skewed distribution → hot shard saturates while others idle. |
| **Queue + workers** | Queue absorbs bursts; backlog = ∫(λ_in − μ_workers). Metrics: backlog size, oldest-message age. No drops unless retention is exceeded. |
| **Autoscaling** | App servers scale between min/max when ρ > 70% for N seconds, with a provisioning delay (e.g. 30 s). Teaches that autoscaling doesn't save you from a sudden spike. |
| **Stateful servers** | If sessions are stored in-memory and a server dies, its users' requests fail (`sessionLoss` metric). Fix: external session store. |
| **Rate limiter** | Excess traffic is rejected with 429 (counted separately from 5xx, so tests can require "shed load, don't crash"). |
| **Network** | Each edge adds latency; cross-region edges add 80 ms. Partition event cuts an edge. |

### 6.4 Metrics produced (what tests can assert on)

Global, per request class, and per node, over time:

`rps`, `successRate`, `errorRate` (5xx), `rejectedRate` (429),
`latency.p50/p95/p99`, `utilization`, `queueDepth`, `cacheHitRate`,
`replicationLagMs`, `staleReadRatio`, `queueBacklog`, `oldestMessageAgeS`,
`dataLoss` (acknowledged writes not persisted), `sessionLoss`,
`monthlyCost`, `availability` (fraction of seconds with success ≥ 99%).

### 6.5 Static analysis (before running)

Cheap graph checks that run on every edit:

- unreachable nodes / dangling edges / cycles that aren't allowed
- client not connected to an entry point
- writes have no path to a durable store
- **SPOF detection**: any node whose removal disconnects client from a durable
  store (shown as a ⚠ badge, many exercises have a hidden test for it)
- budget exceeded
- disallowed component for this exercise

---

## 7. Tests & grading

### 7.1 Test definition

A test = scenario + assertions + failure hint:

```yaml
id: survives-db-primary-failure
hidden: true
scenario:
  workload: default
  durationS: 120
  events:
    - at: 40s
      kill: { componentType: sql-db, role: primary }
assert:
  - metric: availability
    op: ">="
    value: 0.95
  - metric: dataLoss
    op: "=="
    value: 0
hint: What happens to writes when the only database disappears?
```

### 7.2 Standard test families

Exercises compose their tests from a shared library so they feel consistent:

| Family | Example assertion |
|---|---|
| **Steady load** | p99 < X ms and errors < 0.1% at target RPS |
| **Peak / spike** | survives 3× traffic for 30 s with errors < 1% |
| **Budget** | monthly cost ≤ budget |
| **Chaos: single node** | kill any one node (each in turn) → availability ≥ 99% |
| **Chaos: dependency** | cache flush / DB primary loss / AZ outage |
| **Durability** | `dataLoss == 0` under failure |
| **Consistency** | stale reads ≤ 1% (or "read-your-writes" required) |
| **Async** | backlog drains within 60 s after a spike |
| **Overload protection** | under 10× abuse traffic, legit users still get ≥ 99% success |

### 7.3 Scoring (per stage)

Each test belongs to a stage, so results map onto the framework and show you
*which step* of your process is weak:

| Stage | Graded on |
|---|---|
| ① Scope | key questions asked (e.g. 4/5), requirements sheet correct, estimates within tolerance |
| ② Blueprint | static validation, baseline-load tests, budget, API covers the functional requirements |
| ③ Deep dive | focus-area tests (scale, chaos, consistency) |
| ④ Wrap-up | bottleneck prediction correct, chaos suite, 10× test (bonus), operations checklist |

- **Pass** = all required tests green in stages ②–④. Scope is scored, not
  gated, because missing a question already shows up as failing hidden tests.
- **Efficiency score** (optional, shown after passing): cost and p99 compared
  with the reference solution, displayed as stars. Gives a reason to come back
  and optimize, like LeetCode's runtime percentile.
- **Skill radar** on the profile page: your average score per stage across all
  exercises, e.g. "strong at Blueprint, weak at Scope". This tells you which
  drills to do next.
- Attempts and time per stage are recorded locally.

---

## 8. Curriculum (exercise progression)

Each exercise introduces **one** new idea and reuses earlier ones. Numbers are
indicative. The tracks also add stages of the 4-step framework gradually, so
the process gets built up the same way the components do:

| Track | Stages you do yourself | Stages pre-filled |
|---|---|---|
| 1 — Building blocks | ② Blueprint | ① requirements given, no ③/④ |
| 2 — Reliability & scale | ② + ③ (one focus area) + ④ bottleneck prediction | ① requirements given |
| 3 — Data at scale | ① estimation + ② + ③ + ④ | ① clarifying questions given |
| 4 — Capstones | ① → ④, all of it | nothing |

Each track also has a set of **drills** (§3.2): 5-minute single-stage problems
to repeat until they're automatic, the way you'd grind easy LeetCode problems.
For example, Track 2 has bottleneck drills, Track 3 estimation drills, Track 4
scope drills.

### Track 0 — Fundamentals (Learn mode)
Client/server & HTTP · latency numbers every programmer should know · DNS ·
load balancers · horizontal vs vertical scaling · caching · SQL vs NoSQL ·
replication · sharding · queues · CDN · CAP & consistency · rate limiting.
Each = explanation + video links + mini-lab.

### Track 1 — Building blocks (Easy)
| # | Exercise | New idea | Breaks if… |
|---|---|---|---|
| 1 | **Hello, World** — serve 100 RPS | client → server → DB | nothing connected to DB |
| 2 | **Black Friday** — 4k RPS | horizontal scaling + LB | one server (cap 1k) melts |
| 3 | **Sticky problem** — logins | stateless servers, shared session store | server killed → users logged out |
| 4 | **Read-heavy profile page** — 20k RPS, 95% reads | cache-aside | DB at 1000% utilization |
| 5 | **Static assets** — images & JS | CDN + object storage | app servers waste capacity on images, budget exceeded |

### Track 2 — Reliability & scale (Medium)
| # | Exercise | New idea | Breaks if… |
|---|---|---|---|
| 6 | **No single point of failure** | redundancy everywhere | hidden test kills each node in turn |
| 7 | **Read replicas** | replica scaling, replication lag | stale reads exceed the consistency limit |
| 8 | **Database failover** | primary/replica, durability | writes lost when primary dies |
| 9 | **Send welcome emails** | queue + workers, async | synchronous email call makes p99 explode |
| 10 | **Traffic spike** | queue as buffer, autoscaling delay | autoscaling alone is too slow |
| 11 | **Protect the API** | rate limiting, 429 vs 5xx | abusive client takes everyone down |
| 12 | **Cache stampede** | request coalescing, TTL jitter | cache flush at t=60 s kills the DB |
| 13 | **Retry storm** | timeouts, backoff, circuit breaker | retries triple the load on a struggling DB |

### Track 3 — Data at scale (Hard)
| # | Exercise | New idea | Breaks if… |
|---|---|---|---|
| 14 | **Write-heavy metrics ingestion** — 50k writes/s | sharding | single primary caps at 500 writes/s |
| 15 | **Pick a shard key** | key skew, hot partitions | sharding by `country` → hot shard |
| 16 | **Search** | secondary index, eventual consistency | full scans on the primary DB |
| 17 | **Go global** | multi-region, geo DNS, cross-region latency | EU users at 300 ms p99 |

### Track 4 — Classic interview systems (Capstones)
Full 4-step flow: vague prompt, clarifying questions, API, blueprint, 2–3
deep-dive focus areas, wrap-up.

| # | System | Key scope question | Deep-dive focus areas |
|---|---|---|---|
| 18 | **URL shortener** | "How many new URLs/day? Read/write ratio?" | ID generation, caching hot links, redirect latency |
| 19 | **Pastebin** | "Max paste size? Expiry?" | object storage vs DB, expiry cleanup |
| 20 | **Rate limiter as a service** | "Per user or per IP? Hard or soft limits?" | distributed counters, limiter as a SPOF |
| 21 | **News feed** | "Max friends? Ranked or chronological?" | fan-out on write vs read, celebrity problem |
| 22 | **Chat (WhatsApp-lite)** | "1:1 only or groups? Online presence?" | WebSocket gateway scaling, presence, ordering |
| 23 | **Video upload (YouTube-lite)** | "Upload size? Which resolutions?" | async transcoding pipeline, CDN cost |
| 24 | **Ride matching (Uber-lite)** | "Location update frequency?" | geo-indexing, high write rate of locations |

---

## 9. Exercise file format (content as code)

Exercises live in the repo as files, validated with Zod at build time. Adding an
exercise = adding a file + a reference solution; CI runs the solution against
the tests to prove the exercise is solvable and that a naive design fails.

The file is organised by the four stages. A stage can be marked `given`, in
which case the app shows it pre-filled and read-only. That's how Track 1–3
exercises skip stages.

```yaml
# content/exercises/18-url-shortener.yaml
id: url-shortener
number: 18
title: URL Shortener
type: full-design            # build | scope-drill | estimation-drill | bottleneck-drill | fix-it | full-design
difficulty: medium
concepts: [caching, id-generation, read-heavy, base62]
prerequisites: [read-heavy-profile, no-single-point-of-failure]
timeBudgetMin: { scope: 8, blueprint: 12, deepDive: 20, wrapUp: 5 }
story: Design a URL shortening service like TinyURL.

# ① Scope ------------------------------------------------------------------
scope:
  questions:
    - id: volume
      ask: How many URLs are shortened per day?
      answer: 100 million new URLs per day.
      reveals: { writeRps: 1160 }
      key: true                # counted in the "key questions asked" score
    - id: ratio
      ask: What's the read/write ratio?
      answer: About 10 redirects for every URL created.
      reveals: { readRatio: 0.91 }
      key: true
    - id: retention
      ask: How long do we keep URLs?
      answer: 10 years.
      reveals: { retentionYears: 10 }
      key: true
    - id: custom-alias
      ask: Can users pick a custom alias?
      answer: Not for now.
      key: false
    - id: ui-color
      ask: What color should the button be?
      answer: That's not something we need to decide here.
      irrelevant: true         # flagged in feedback, no penalty
  functional:                  # the user ticks which ones are in scope
    inScope: [shorten-url, redirect]
    outOfScope: [custom-alias, analytics-dashboard]
  estimation:
    - id: write-qps
      question: Average write QPS?
      answer: 1160
      tolerance: 0.3
      workedSolution: 100M / 24h / 3600s ≈ 1160
    - id: storage
      question: Storage needed over 10 years (avg record 100 bytes)?
      answer: 36.5TB
      tolerance: 0.3

# ② Blueprint ---------------------------------------------------------------
blueprint:
  api:                         # each endpoint becomes a request class
    - { id: shorten, method: POST, path: /api/v1/shorten }
    - { id: redirect, method: GET, path: "/{code}" }
  palette: [client, load-balancer, app-server, cache, sql-db, nosql]
  budgetPerMonth: 2500
  workload:
    classes:
      - { endpoint: redirect, ratio: 0.91, keyspace: 365_000_000_000, zipf: 1.2 }
      - { endpoint: shorten, ratio: 0.09 }
    rps: { shape: diurnal, peak: 23000 }
  tests:
    - id: baseline
      assert:
        - { metric: latency.p99, class: redirect, op: "<", value: 100 }
        - { metric: errorRate, op: "<", value: 0.001 }
    - id: budget
      assert: [{ metric: monthlyCost, op: "<=", value: 2500 }]

# ③ Deep dive ---------------------------------------------------------------
deepDive:
  focusAreas:                  # ordered by importance
    - id: id-generation
      title: Unique, short IDs at 1,160 writes/s
      unlocks: [app-server.idStrategy]   # hash+collision check | counter | snowflake
      tests:
        - id: no-collisions
          assert: [{ metric: idCollisionRate, op: "==", value: 0 }]
    - id: hot-links
      title: A viral link gets 50k redirects/s
      tests:
        - id: viral-link
          scenario: { events: [{ at: 20s, hotKey: { class: redirect, share: 0.4, rps: 50000 } }] }
          assert: [{ metric: latency.p99, class: redirect, op: "<", value: 150 }]
      optional: false

# ④ Wrap-up -----------------------------------------------------------------
wrapUp:
  bottleneckPrediction: { scenario: 10x-traffic }    # the answer is computed by the engine
  tests:
    - id: survives-any-single-node
      hidden: true
      scenario: { chaos: kill-each-node }
      assert: [{ metric: availability, op: ">=", value: 0.99 }]
  operations: [monitoring, health-checks, rate-limiting]

hints:
  scope: ["You can't size anything without knowing traffic and retention."]
  blueprint: ["Redirects are 91% of traffic. Where should they be served from?"]
  deepDive: ["Hashing the long URL can collide. What guarantees uniqueness?"]
editorial: ./18-url-shortener.md
referenceSolutions: [./18-url-shortener.cache-sql.json, ./18-url-shortener.kv.json]
naiveSolution: ./18-url-shortener.naive.json   # CI checks that this one fails
```

---

## 10. Technical architecture

### 10.1 Stack

| Concern | Choice | Why |
|---|---|---|
| Language | TypeScript (strict) everywhere | one language for engine, UI, content validation |
| UI | React + Vite | fast dev loop, huge ecosystem |
| Canvas | **React Flow (@xyflow/react)** | node/edge editor with drag, zoom, custom nodes out of the box |
| Particles/animation | SVG for edges + a `<canvas>` overlay for particles | thousands of particles without React re-renders |
| State | Zustand | simple, works well with React Flow |
| Charts | uPlot or Recharts | uPlot for live time series performance |
| Styling | Tailwind CSS | speed |
| Engine | pure TS package, runs in a Web Worker (Comlink) | determinism, testability, no UI jank |
| Content | YAML + Markdown, validated with Zod | exercises reviewable in PRs |
| Persistence (v1) | IndexedDB (via `idb-keyval`) | no backend needed: progress, saved designs |
| Testing | Vitest (engine + content), Playwright (e2e) | |
| Hosting | static site (Cloudflare Pages / Vercel / GitHub Pages) | free, no server |
| *(v3)* Backend | Supabase (auth + Postgres) | accounts, sync, leaderboards |

### 10.2 Repo layout (monorepo, pnpm workspaces)

```
apps/web/              React app (canvas, panels, routing)
packages/engine/       simulation engine (pure TS, no DOM)
packages/catalog/      component definitions (capacity, cost, config schema)
packages/content/      exercise schema (Zod) + loader
content/exercises/     YAML + editorial markdown + reference solutions
content/learn/         fundamentals pages
docs/                  this spec, ADRs
```

### 10.3 Key interfaces (sketch)

```ts
type ComponentType = 'client' | 'load-balancer' | 'app-server' | 'cache' | 'sql-db' | 'queue' | ...;

interface DesignNode { id: string; type: ComponentType; config: Record<string, unknown>; position: {x:number;y:number} }
interface DesignEdge { id: string; from: string; to: string; filter?: 'reads' | 'writes' | 'all'; mode?: 'sync' | 'async' }
interface Design     { nodes: DesignNode[]; edges: DesignEdge[] }

interface ComponentDef {
  type: ComponentType;
  configSchema: ZodSchema;             // drives the inspector form
  cost(config): number;                // $/month
  capacity(config): Capacity;          // per request class/op
  route(ctx: RouteCtx): FlowSplit[];   // how incoming flow leaves the node
  onEvent?(event: ChaosEvent, state): void;
}

interface SimResult {
  timeline: Tick[];                    // per-tick metrics, global + per node + per class
  samples: RequestTrace[];             // for animation/replay
  summary: Metrics;                    // aggregates tests assert on
}

function simulate(design: Design, scenario: Scenario, seed: number): SimResult;
function runTests(design: Design, exercise: Exercise): TestResult[];  // includes diagnostics
```

Adding a new component = one `ComponentDef` + one React node renderer. The engine
core never special-cases component types.

### 10.4 Quality bar for the engine

- Unit tests per component behavior (e.g. "cache with K items under Zipf(1.1)
  has hit rate within ±2% of analytic value").
- **Golden tests per exercise**: the reference solution passes all tests; a
  "naive" solution (also stored) fails the expected ones. Runs in CI.
- Property tests: adding capacity never increases error rate; removing a node
  never improves availability.

---

## 11. Roadmap

| Phase | Scope | Exit criterion |
|---|---|---|
| **0. Prototype** (1–2 weekends) | Canvas with 4 components (client, LB, server, DB), fluid engine only, node colors by utilization, one hard-coded exercise (Blueprint stage only) | You can watch a single server turn red and fix it with an LB |
| **1. MVP** | Engine layers 1+2, cache + queue + worker, test runner + diagnostics, **stage bar with stages ② and ④ (bottleneck prediction)**, exercises 1–6, bottleneck drills, Learn pages with videos, IndexedDB progress | You'd use it yourself for a week |
| **2. Depth** | Chaos events, replicas/failover, rate limiter, retries/timeouts, autoscaling, replay scrubber, hints & editorials, **stage ③ focus areas**, **stage ① (clarifying questions, estimation) + scope/estimation drills**, exercises 7–13 | Every Track 2 exercise has a visible "aha" failure |
| **3. Scale** | Sharding, NoSQL, multi-region, CDN, API definition in stage ②, capstones 18–24, per-stage scoring + skill radar, timer, efficiency score, exercise editor | Can practice a full 4-step interview problem end to end |
| **4. Interview & community** | Interview mode (LLM interviewer via the Claude API answers open questions and reviews against the 4-step rubric), accounts, cloud sync, leaderboards, user-submitted exercises | Other people use it |

---

## 12. Open questions

1. **Routing rules: implicit or explicit?** Implicit defaults (servers send
   reads to cache first) are beginner-friendly but hide decisions. Proposal:
   implicit by default, edge filters available from Track 2.
2. **How much config per component?** Too many knobs = a config UI, not a
   design exercise. Proposal: ≤ 4 options per component, more unlocked later.
3. **Clarifying questions: multiple choice or free text?** Picking from a list
   is deterministic and easy to grade, but it hints at what to ask. Free text
   is closer to a real interview but needs an LLM to match questions to answers.
   Proposal: a list in Tracks 3–4, free text only in Interview mode.
4. **Should stage ① gate the canvas?** Locking the canvas enforces "don't jump
   to the solution" but can feel heavy when you just want to build. Proposal:
   gated in capstones, a skippable reminder elsewhere.
5. **Open-ended grading** for capstones: multiple valid architectures must
   pass, so tests must assert on outcomes (latency, cost, availability), never
   on graph shape. Keep to this rule everywhere.
6. **Name** of the app.

---

## 13. Success metrics (personal project)

- You can explain *why* each design choice in a capstone is needed, citing the
  failure you saw without it.
- Time to solve a Track 2 exercise drops below 15 minutes.
- You finish a capstone inside the 45-minute stage budgets, with no stage
  scoring below 70% on the skill radar.
- (If shared) Day-7 retention of other users and number of exercises completed
  per user.
