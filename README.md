# Talks

Slides from my engineering talks. Each talk has its own folder.

The talks follow one story. First, how we built a payment platform on Kafka. Then, how we proved that the data from our databases reached the other side.

## 1. Kafka for Everything

**Folder:** [`kafka-world/`](./kafka-world)

Two payments arrive for the same customer at the same millisecond. The classic answer is a row lock in a SQL database. The database then becomes the chokepoint.

We chose a different path. Kafka partitions serialize the work for each customer. Services keep their state in memory. They rebuild it from a state topic and a snapshot topic. No service calls another service directly.

Topics covered:

- The concurrency problem and why locks do not scale
- Kafka primer: partitions, offsets, consumer groups
- Gateway, reply topics, and tracing in the message
- In-memory state and the snapshot pattern
- Topic joins (N to 1) without a stream framework
- Derived data in OpenSearch and DynamoDB
- Why not Flink, and why not services plus a database
- Lessons and regrets

Read the written version: [How we built our Payment platform using Kafka](https://supertechnologies.substack.com/p/how-we-built-our-payment-platform).

## 2. Reliability of our Streaming Pipelines

**Folder:** [`cdc/`](./cdc)

Kafka is the source of truth for the platform. But a lot of data starts in MySQL, and CDC (change data capture) moves it into Kafka. This talk asks one question: did every change arrive?

Our CDC pipelines reported 92% reliability. Count-based checks gave a false picture. At-least-once delivery creates duplicates, and duplicates hide real misses. We switched to GTIDs, which identify each transaction. We checked identity, not counts. First-pass delivery reliability went to 99.78%.

Topics covered:

- The dual-write problem and a CDC primer
- Main pipeline and the old completeness setup
- Why counting lies
- GTID-based completeness checks
- Grafana dashboards, per pipeline and per transaction
- Results, scaling, and roadmap

## How to view the slides

Each folder has two files for the deck:

- `slides.md` is the source. It uses [Marp](https://marp.app).
- `slide_personal.html` is the rendered deck. Open it in a browser. Use the arrow keys to move.

To build a deck again, run this command in the talk folder:

```
npx -y @marp-team/marp-cli@latest slides.md --html -o slide_personal.html --allow-local-files
```

The two Mermaid diagrams in the CDC deck load from a CDN. Without internet, use the PNG backups in `cdc/`.

## About

I am Abhishek Jain, a software engineer. I like to find smarter ways to do monotonous jobs.

Memes in the decks belong to their original owners.
