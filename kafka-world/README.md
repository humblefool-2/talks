# Kafka for Everything

How we process payments with Kafka. No database on the hot path. No service calls between services.

- Slides: [`slide_personal.html`](./slide_personal.html) (source: [`slides.md`](./slides.md))
- Article: [How we built our Payment platform using Kafka](https://supertechnologies.substack.com/p/how-we-built-our-payment-platform)
- Length: about 24 minutes, 21 slides
- Next talk: [Reliability of our Streaming Pipelines](../cdc)

Build the deck:

```
npx -y @marp-team/marp-cli@latest slides.md --html -o slide_personal.html --allow-local-files
```
