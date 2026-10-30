# Reliability of our Streaming Pipelines

How we took Flink CDC completeness from 92% to 99.78%.

Count-based checks hid real misses. GTIDs caught what counts could not.

- Slides: [`slide_personal.html`](./slide_personal.html) (source: [`slides.md`](./slides.md))
- Length: about 26 to 28 minutes, 22 slides
- Previous talk: [Kafka for Everything](../kafka-world)

The 99.78% figure is the mean of per-pipeline first-pass delivery reliability. It covers about 30 pipelines, averaged over weekly values for 3+ months.

Two Mermaid diagrams load from a CDN. For offline use, see `mermaid-backup-*.png`.

Build the deck:

```
npx -y @marp-team/marp-cli@latest slides.md --html -o slide_personal.html --allow-local-files
```
