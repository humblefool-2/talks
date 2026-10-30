# Talks

Slides from my engineering talks. Each talk has its own folder with a README.

| # | Talk | One line |
|---|------|----------|
| 1 | [Kafka for Everything](./kafka-world) | How we process payments with Kafka. No locks, no database on the hot path. [Article](https://supertechnologies.substack.com/p/how-we-built-our-payment-platform) |
| 2 | [Reliability of our Streaming Pipelines](./cdc) | Kafka is the source of truth, but did all the data get there? CDC completeness from 92% to 99.78%. |

Each deck is Marp. Open `slide_personal.html` in a browser, or rebuild from `slides.md`:

```
npx -y @marp-team/marp-cli@latest slides.md --html -o slide_personal.html --allow-local-files
```

Memes belong to their original owners.
