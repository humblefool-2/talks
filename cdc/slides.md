---
marp: true
theme: default
paginate: true
style: |
  section {
    background: linear-gradient(135deg, #14171a 0%, #1c2126 60%, #16302b 100%);
    color: #e2e8f0;
    font-size: 1.35rem;
    padding: 2rem 3rem;
    color-scheme: dark;
  }
  section.lead {
    background: linear-gradient(135deg, #0d0f10 0%, #12181a 50%, #0f2b23 100%);
    text-align: center;
    justify-content: center;
  }
  h1, h2 {
    color: #baff29;
    font-weight: 700;
  }
  h1 {
    font-size: 2.6rem;
  }
  h2 {
    font-size: 1.8rem;
    border-bottom: 2px solid rgba(186, 255, 41, 0.3);
    padding-bottom: 0.3rem;
    margin-bottom: 1rem;
  }
  strong {
    color: #56ccf2;
  }
  code {
    background: rgba(255, 255, 255, 0.1);
    color: #9be8a1;
    padding: 2px 6px;
    border-radius: 4px;
    font-size: 0.95em;
  }
  pre:not(.mermaid) {
    background: rgba(0, 0, 0, 0.35) !important;
    border-left: 3px solid #baff29;
    border-radius: 0 8px 8px 0;
    padding: 1rem 1.2rem !important;
  }
  pre:not(.mermaid) code {
    background: transparent;
    color: #e2e8f0;
    padding: 0;
  }
  blockquote {
    border-left: 4px solid #a3e635;
    background: rgba(163, 230, 53, 0.1);
    padding: 0.6rem 1rem;
    border-radius: 0 8px 8px 0;
    color: #d4ff7a;
    font-style: italic;
    margin: 0.8rem 0;
  }
  table {
    font-size: 1.05rem;
    border-collapse: collapse;
    width: 100%;
  }
  table tr {
    background: rgba(255, 255, 255, 0.03) !important;
    border-top: 1px solid rgba(186, 255, 41, 0.2) !important;
  }
  table tr:nth-child(2n) {
    background: rgba(86, 204, 242, 0.07) !important;
  }
  th {
    background: rgba(186, 255, 41, 0.18) !important;
    color: #baff29 !important;
    padding: 0.5rem 0.8rem;
    border-bottom: 2px solid rgba(186, 255, 41, 0.4) !important;
  }
  td {
    padding: 0.4rem 0.8rem;
    border-color: rgba(186, 255, 41, 0.15) !important;
    color: #e2e8f0 !important;
  }
  section::after {
    color: rgba(255, 255, 255, 0.4);
    font-size: 0.8rem;
  }
  li {
    margin: 0.3rem 0;
  }
  ul ul li {
    font-size: 0.95em;
    color: #cbd5e1;
  }
  pre.mermaid, div.mermaid {
    background: rgba(255, 255, 255, 0.05) !important;
    border: 1px solid rgba(186, 255, 41, 0.2) !important;
    border-radius: 12px;
    padding: 1rem !important;
    text-align: center;
  }
  pre.mermaid svg, div.mermaid svg {
    max-width: 100%;
    max-height: 62vh;
  }
  .stat {
    font-size: 5rem;
    font-weight: 800;
    color: #baff29;
    text-align: center;
    line-height: 1.1;
  }
  .stat-sub {
    font-size: 1.4rem;
    color: #56ccf2;
    text-align: center;
  }
  .todo {
    display: block;
    margin-top: 0.8rem;
    padding: 0.5rem 0.9rem;
    border: 1px dashed #ef4444;
    border-radius: 6px;
    background: rgba(239, 68, 68, 0.1);
    color: #fca5a5;
    font-size: 0.9rem;
    font-style: italic;
  }
---

<script>
  // Marp/bespoke hides every non-active slide with display:none, and Mermaid
  // measures text width at render time. Measuring hidden (display:none)
  // elements returns 0, so node boxes come out too narrow and labels get
  // clipped ("half-cut" text). Fix: temporarily swap the hidden slides to
  // visibility:hidden (still measurable, just not seen) before Mermaid's
  // auto-render pass runs, then let bespoke resume its normal display:none
  // behavior once rendering has settled. Runs before mermaid.min.js loads so
  // the override is in place before Mermaid's DOMContentLoaded render fires.
  var _mermaidFix = document.createElement('style');
  _mermaidFix.textContent = '.bespoke-marp-inactive,.bespoke-marp-active-ready{display:block!important;visibility:hidden!important;position:absolute!important;top:0;left:0;}';
  document.head.appendChild(_mermaidFix);
  setTimeout(function () { _mermaidFix.remove(); }, 4000);
</script>
<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
<script>mermaid.initialize({ startOnLoad: true, theme: 'dark' });</script>

## About Me

<img src="abhishek.jpg" style="float:right; width:220px; margin: 0 0 1rem 2rem; border-radius: 12px;">

### Abhishek Jain

*"Lazy engineer, always trying to find smarter ways to do monotonous jobs"*

<br/>

<img src="QR.png" alt="LinkedIn QR code" style="width:240px; display:block; margin-top:0.8rem; border-radius: 8px;">

---

<!-- _class: lead invert -->

# Reliability of our Streaming Pipelines

### How we took Streaming Pipelines Reliability
### from 92% to 99.78%

---

## The Pillars We Think About

When we build any system, we reason about these individually:

- **Scalability**: can it handle more load without falling over?
- **Availability**: is it up when something calls on it?
- **Consistency / Reliability**: is the data correct and complete?
- **Latency & Throughput**: how fast, and how much?

<br/>

> We did too. But for our clients, one of these mattered more than the rest.

---

## Why We Needed Streaming Pipelines

<img src="this_is_fine.gif" style="float:right; width:260px; margin: 0 0 1rem 2rem; border-radius: 8px;">

Your service writes to two places: the database, and something downstream, Kafka, a search index, a cache.

Two separate writes, two separate chances to fail:
- DB write succeeds, downstream write fails → **silent drift**
- Downstream write succeeds, DB write fails → **phantom data**

No single atomic transaction covers both writes, and neither write tells you the other one's outcome. If it fails, you get silence, not a signal.

<br/>

**This is the problem Change Data Capture (CDC) exists to solve.**

---

## What is CDC?

**Change Data Capture**: capture every row-level change in a database as it happens, and stream it out to whatever needs to react to it. The pieces below are how we do that on top of MySQL:

- **Binlog**: MySQL's own append-only log of every row change; already the source of truth for replication
- **GTID** (Global Transaction Identifier): a unique ID MySQL stamps on every committed transaction; lets you ask any replica "have you seen this one?" It's the building block completeness checking is built on. Needs `gtid_mode=ON`, which MySQL replication already uses, so it comes for free
- **GTID set**: one or more single GTIDs or ranges of GTIDs, e.g. `3E11FA47-71CA-11E1-9E33-C80AA9429562:1-3:11:47-49`. This is what gets compared, not individual GTIDs one by one
- **Debezium**: reads the binlog, turns raw replication events into structured change records
- **Flink**: the streaming engine running the CDC job: consumes the binlog stream, applies logic, writes out
- **Sink**: where changes land: Kafka, S3, or others, depending on the consumer

<br/>

**Quick vocabulary check:** database → table → row. **DDL** changes structure (`CREATE`, `ALTER`), **DML** changes data (`INSERT`, `UPDATE`, `DELETE`). CDC has to handle both.

---

## Current Architecture: Main Pipeline

<img src="main-pipeline-architecture.png" style="display:block; max-width:100%; max-height:38vh; margin: 0.5rem auto; border-radius: 8px;">

One job per MySQL instance, configurable to capture **every table** or a **specific subset**. Reads from binlog replicas, processes events through operators (`MySqlSource → Filter → CDCEventProcessor`), writes to DDL/DML/DLQ sinks feeding Kafka. Sidecar (`cdc-buddy`) runs alongside the job.

GitOps deployment end to end: desired state lives in **Git**, **ArgoCD** reconciles it, the **Flink Operator** turns the `FlinkDeployment` CRD into a running job on Kubernetes.

---

## Current Architecture: Completeness Setup

<pre class="mermaid">
flowchart LR
    DB[(MySQL)] --> BR[Binlog Reader]
    BR --> EP[Events Producer]
    EP --> K[(Kafka)]
    DB -.->|count| SMC[Source Metrics Collector]
    K -.->|count| SKMC[Sink Metrics Collector]
    SMC --> MS[(Metrics Storage)]
    SKMC --> MS
</pre>

How we measured reliability before: compare **message counts** on the source side vs the sink side.

<img src="task_failed.jpg" style="float:right; width:260px; margin: 0 0 1rem 2rem; border-radius: 8px;">


---

## The Reliability Problem

<img src="fire-mad-men.gif" style="float:right; width:220px; margin: 0 0 1rem 2rem; border-radius: 8px;">

Two separate problems, easy to conflate, and we did:

- **Detection was broken.** At-least-once delivery means duplicates exist. Comparing total message counts (source vs sink) can still "match" even when real transactions are missing; a duplicate elsewhere silently offsets a real miss. The count check wasn't measuring completeness at all; it just felt like it was.
- **Recovery was time-boxed.** Binlog / topic retention: **3 days** (longer retention costs storage and adds maintenance overhead). Even once a miss *is* caught, if it's outside that window, the data behind it is already gone: unfixable, not just delayed.

**~92% reliability.** Not good enough for Fintech-grade consumers.

<br/>

**Retention didn't cause the misses; it's why, once counting hid them past the window, they became permanent.**

---

## The Goal

Solve completeness **end to end**: not just "did we read everything from MySQL," but "did everything we read actually make it to the sink."

- **Source-side completeness**: every transaction MySQL accepted was consumed
- **Sink-side completeness**: every transaction consumed was correctly published

<br/>

Both sides needed a real answer, not a spot check. And it's not just an internal metric: it's what lets us tell our customers, with evidence, that their data made it through.

---

## New Architecture: End-to-End

<style scoped>
section { padding: 1.2rem 3rem; }
pre.mermaid svg, div.mermaid svg { max-height: 38vh !important; max-width: 98% !important; }
section p { font-size: 0.8rem; margin-top: 0.4rem; }
</style>

<pre class="mermaid">
%%{init: {"flowchart": {"nodeSpacing": 8, "rankSpacing": 18, "subGraphTitleMargin": {"top": 8, "bottom": 6}}, "themeVariables": {"fontSize": "12px"}}}%%
flowchart TB
    MYSQL[("MySQL replica<br/>binlog")]
    GTIDCOL["GTID collector"]
    MYSQL --> GTIDCOL

    subgraph JOB["CDC-to-Kafka"]
      direction LR
      RB["Read binlog"] --> RM["RecordMakers.java"] --> DOTS["..."] --> KS0["KafkaSink"]
      GG1["GTIDGatherer1"] --> KS1["KafkaSink"]
      GG2["GTIDGatherer2"] --> KS2["KafkaSink"]
      RB -.-> GG1
      DOTS -.-> GG2
    end
    MYSQL -.-> RB

    GTIDCOL --> T_EXEC[["GTID_EXECUTED<br/>topic"]]
    KS1 --> T_OBS[["GTID_OBSERVED<br/>topic"]]
    KS2 --> T_DEBEZ[["DEBEZ_GTID_OBSERVED<br/>topic"]]
    KS0 --> T_MSG[["CDC_MESSAGES<br/>topic"]]

    subgraph GTIDC["gtid-completeness"]
      direction TB
      GES["GTID_EXECUTED Source<br/>(diffForTimeRange)"]
      GOS["GTID_OBSERVED Source"]
      VAL1["Validate"]
      GMS["GTID_MetricSink"]
      GES --> VAL1
      GOS --> VAL1
      VAL1 --> GMS
    end

    subgraph CDCC["cdc-completeness"]
      direction TB
      DGS["DEBEZ_GTID_OBSERVED Source"]
      CMS_S["CDC_MESSAGES Source"]
      VAL2["Validate"]
      CMS["CDC_MetricSink"]
      DGS --> VAL2
      CMS_S --> VAL2
      VAL2 --> CMS
    end

    T_EXEC --> GES
    T_OBS --> GOS
    T_DEBEZ --> DGS
    T_MSG --> CMS_S

    GMS --> RESULT["CDC_COMPLETENESS_RESULT<br/>sink"]
    CMS --> RESULT

    classDef job fill:#ffc966,stroke:#cc8800,color:#1a1a1a
    classDef topic fill:#c6e86b,stroke:#7a9b2e,color:#1a1a1a
    classDef inner fill:#fff4d6,stroke:#cc8800,color:#1a1a1a
    classDef source fill:#cfe8ff,stroke:#4a90d9,color:#1a1a1a
    classDef sink fill:#d9c6f5,stroke:#7a4fae,color:#1a1a1a
    class JOB,GTIDC,CDCC,GTIDCOL job
    class T_EXEC,T_OBS,T_DEBEZ,T_MSG topic
    class RESULT sink
    class RB,RM,DOTS,KS0,GG1,KS1,GG2,KS2,GES,GOS,VAL1,GMS,DGS,CMS_S,VAL2,CMS inner
    class MYSQL source
</pre>

Two new `GTIDGatherer` branches tap the CDC job: **GTIDGatherer1** off `Read binlog` → **GTID_OBSERVED**; **GTIDGatherer2** off the later `[...]` stage → **DEBEZ_GTID_OBSERVED**. A standalone **GTID collector** polls MySQL for **GTID_EXECUTED**. `gtid-completeness` is built; `cdc-completeness` is still ahead. `diffForTimeRange` diffs GTID_EXECUTED against GTID_OBSERVED within a rolling time window instead of all history, keeping the check fast.

---

## Incremental Strategy

<img src="drake_meme.jpg" style="float:right; width:220px; margin: 0 0 1rem 2rem; border-radius: 8px;">

Build the whole thing, source *and* sink completeness, up front? Or start smaller and learn?

- Started with **source-side (GTID) completeness only**
- Reasoning: it would surface unknowns before committing to the full design, and might already be enough on its own to tell us how reliable the platform really is
- Feedback loop first, full scope later: sink-side (CDC-completeness) comes next, informed by what we learn here

---

## New Architecture: Main Job, Updated

<img src="main-job-updated-with-gtid-branch.png" style="display:block; max-width:100%; max-height:46vh; margin: 0.5rem auto; border-radius: 8px;">

The CDC job now also emits **Observed GTID**, every transaction it read from the binlog, down its own Kafka sink branch.

---

## New Architecture: Completeness Job

<img src="completeness-job-gtidsetmatcher.png" style="display:block; max-width:100%; max-height:46vh; margin: 0.5rem auto; border-radius: 8px;">

A separate Flink job consumes both **GTIDExecuted** (source of truth, from MySQL) and **GTIDObserved** (from the main job's new branch). The **`GtidSetMatcher`** operator compares the two sets and writes anything missing to the **Completeness Result sink**.

This is the half we shipped: just source-side (GTID) completeness, the **left side** of the end-to-end picture. `cdc-completeness`, validating the actual Kafka messages and not just GTIDs, is the incremental step still ahead.

---

## Implementation Details

<img src="completeness-result-sink-schema.png" style="display:block; max-width:100%; max-height:34vh; margin: 0.3rem auto; border-radius: 8px;">

Results land in a **MySQL sink** (`CdcGtidCompletenessResult`), deliberately simple, and it plugs straight into Grafana's MySQL datasource.

---

## Displaying on Grafana

One panel set per completeness pipeline, per MySQL schema. Alerts fire on:
- Missed transactions **> 0** within a **3-hour** window (gives lagging transactions time to catch up)
- Matched events **= 0** within a **30-minute** window (pipeline down, Kafka not getting events, or can't reach MySQL). Short on purpose: a false alarm is better than a late one

<img src="grafana-dashboard-overview.png" style="display:block; max-width:100%; max-height:32vh; margin: 0.5rem auto 0; border-radius: 8px;">

---

## Grafana: Per-Transaction Detail

<img src="grafana-dashboard-detail.png" style="width:100%; border-radius: 8px;">

Binlog timestamp, matched/missed GTID sets per minute; drill-down from the summary panels before.

---

## Results

<div class="stat">92% → 99.78%</div>
<div class="stat-sub">raw source-side completeness, gaps caught and reconciled to 100% within the week</div>
<div class="stat-sub" style="font-size:1rem; color:#cbd5e1; margin-top:0.4rem;">mean across ~30 pipelines (~25 schemas), averaged weekly over 3+ months</div>

<br/>

Good enough for Fintech-grade consumers to build on.

---

## Scaling & High Availability

- **Kubernetes autoscaling**: pods scale automatically with load, already running in production
- Migrated to a **K8s IOPS (Input/Output Operations Per Second) cluster** for better throughput
- **Kafka partition increases** to relieve both producer and consumer bottlenecks
- **TaskManager scale-up per pipeline**: still a manual step today
- **High availability**: **JobManager HA via ZooKeeper**, for both main and completeness pipelines. If a cluster fails, a standby JobManager starts in another cluster and brings up the TaskManager pods, so no single cluster failure takes completeness checking down with it

---

## What Went Well, Learnings & What Needs to Improve

<img src="success_kid.jpg" style="float:right; width:200px; margin: 0 0 1rem 2rem; border-radius: 8px;">

**Went well:**
- Starting with source-side (GTID) completeness alone already moved the needle: we didn't need the full solution to get real signal
- MySQL sink + Grafana was the right call for the completeness results: simple, no new tooling to operate
- High availability paid off: a cluster or AWS AZ going down didn't need manual recovery, and didn't cause completeness misses

**Learnings:**
- **Check identity, not counts.** Counts hide misses when duplicates exist. GTIDs do not.
- **Detection alone does not raise reliability.** It shows you where to fix. Scaling and HA were part of the fixing.

**Needs to improve:**
- One completeness pipeline per main pipeline doesn't scale: every new main pipeline still needs its own completeness setup built and maintained

---

## Roadmap

<img src="brain.jpg" style="float:right; width:240px; margin: 0 0 1rem 2rem; border-radius: 8px;">

<style scoped>
ul { margin-bottom: 0 !important; }
table { margin-top: -40px !important; }
</style>

- **POC in progress: merge all completeness checks into one pipeline**, one job consuming multiple schemas/tables/topics and reporting across all of them, instead of a new pipeline per schema.

| | Separate pipelines (today) | Merged pipeline (POC) |
|---|---|---|
| Ops overhead | New pipeline per schema | One pipeline, config-driven |
| If it goes down | Only that schema shows 0% | Every schema it covers shows 0% |
| Data actually lost | None | None (3-day retention, always replayable) |

- **Per-transaction counting** (via Debezium's transaction metadata): checks that *every query inside* a transaction was processed, not just the transaction as a whole. Next lever: **99.78% → 99.9%**.

---

<!-- _class: lead invert -->

# Questions?
