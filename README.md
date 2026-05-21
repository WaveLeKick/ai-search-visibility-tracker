[AI Search Visibility Tracker](https://apify.com/highbrow_fame/ai-search-visibility-tracker?fpr=data)

# AI Search Visibility Tracker — ChatGPT, Perplexity, Gemini, Google AIO

Track if your brand or domain gets cited in **ChatGPT, Perplexity, Gemini, and Google AI Overviews** — across 24 languages, with daily diff alerts and a stability score across multiple samples. **Bring-your-own-key** model — start at $0/month with Gemini's free tier.

![Multilingual edge — same intent in 4 languages, real cloud run](https://images.apifyusercontent.com/StwA5uKGXYjo4ZJgu4JfaR6CWCsx-m981E6uqv2p338/w:1800/cb:1/aHR0cHM6Ly9hcGkuYXBpZnkuY29tL3YyL2tleS12YWx1ZS1zdG9yZXMvRHhydVNDOHBRbHZPZXZBVFEvcmVjb3Jkcy9tdWx0aWxpbmd1YWwtZWRnZS5wbmc.webp)

> ⭐ **If this actor saves you time, please rate it on Apify.** Reviews raise its Store visibility and help the next person find it.

## Why this actor

The Generative Engine Optimization (GEO) tracker market in 2026 is dominated by enterprise tools — Otterly ($29–489/mo), Profound ($99–2k+/mo, no free trial), Peec AI (€89–199/mo), Semrush AI Toolkit ($99–549/mo), Ahrefs Brand Radar ($328+/mo). They share three blind spots:

1. **Pricing locks out indie SEOs and small agencies** — the cheapest viable tier is $29/mo for 15 prompts.
2. **Multilingual coverage is shallow** — Semrush admits "US English only," Profound and Otterly are English-first.
3. **No bring-your-own-key option** — you pay them, they pay the LLM vendors, and you can't see the underlying call.

This actor flips all three:

- **$0/month entry point** — supply your own free Gemini API key (250 queries/day on the free tier) and run end-to-end at zero marginal cost.
- **24 languages** — Hungarian, German, French, Spanish, Italian, Polish, Czech, Romanian, Turkish, Japanese, Korean, Russian, Ukrainian, and 11 more.
- **BYOK transparency** — your OpenAI / Perplexity / Anthropic keys, your bills, your data. The actor never proxies your traffic through our account.

## What it does

Given a list of queries (e.g. *"best CRM for small business"*, *"top fogorvosok Budapesten"*) and one or more domains you care about, the actor:

1. Fans out each query to every selected engine — Gemini, ChatGPT (gpt-4o-mini-search-preview), Perplexity Sonar, Anthropic Claude with web_search.
2. Captures the answer text **plus** structured citations (URL + title + snippet).
3. Detects whether each tracked domain appears — via citation OR via brand mention in the answer text.
4. Computes citation share-of-voice across your domains and competitor domains.
5. Optionally runs each query N times and reports a **stability score** (how consistently each source is cited — useful for filtering one-shot hallucinations).
6. Optionally **diffs against a previous run** and surfaces gained/lost citations per query (delta mode).

## Quick start ($0)

1. Get a free Gemini key: [https://aistudio.google.com/apikey](https://aistudio.google.com/apikey) (Google account, no credit card).
2. Run the actor with this minimal input:

```
{
    "queries": ["best CRM for small business", "top project management tools"],
    "brandDomains": ["yourcompany.com"],
    "engines": ["gemini"],
    "geminiApiKey": "AIza..."
}
```

That's it. Free tier covers 250 queries/day on Gemini 2.5 Flash — enough to track ~250 prompts daily for $0.

## Going further (BYOK paid engines)

Add any combination of these to the input:

| Engine | Where to get a key | Approx cost / query |
| --- | --- | --- |
| OpenAI / ChatGPT | [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys) | ~$0.025 |
| Perplexity Sonar | [https://www.perplexity.ai/settings/api](https://www.perplexity.ai/settings/api) | ~$0.005–0.015 |
| Anthropic Claude | [https://console.anthropic.com/](https://console.anthropic.com/) | ~$0.01–0.02 |

A single query across **all four** engines costs you ~$0.05–0.08 in third-party API fees. You pay them directly.

## Output

![A single citation record — real cloud run output](https://images.apifyusercontent.com/xGdBdygNGLUFow3ZZACTgMZ8fkiF6ime46e3-L47eEk/w:1800/cb:1/aHR0cHM6Ly9hcGkuYXBpZnkuY29tL3YyL2tleS12YWx1ZS1zdG9yZXMvRHhydVNDOHBRbHZPZXZBVFEvcmVjb3Jkcy9jaXRhdGlvbi1kZXRhaWwucG5n.webp)

Each engine call writes one record to the dataset:

```
{
    "type": "citation",
    "engine": "gemini",
    "model": "gemini-2.5-flash",
    "query": "best CRM for small business",
    "language": "en",
    "answerText": "...",
    "citationCount": 7,
    "citations": [
        {"url": "https://...", "host": "hubspot.com", "registrable": "hubspot.com", "title": "..."}
    ],
    "brandMatches": {
        "yourcompany.com": {"cited": true, "viaCitation": true, "viaText": false, "matchedUrls": ["https://yourcompany.com/pricing"]}
    },
    "competitorMatches": {
        "competitor.com": {"cited": false, "viaCitation": false, "viaText": false, "matchedUrls": []}
    },
    "citationShare": {
        "yourcompany.com": {"hits": 1, "sharePct": 14.3},
        "competitor.com": {"hits": 0, "sharePct": 0}
    },
    "latencyMs": 2104
}
```

When `samplesPerQuery > 1` the actor also writes one `stability` record per (query, engine):

```
{
    "type": "stability",
    "engine": "gemini",
    "query": "best CRM for small business",
    "samples": 3,
    "stability": {
        "hubspot.com":   {"occurrences": 3, "samples": 3, "stability": 1.0},
        "salesforce.com":{"occurrences": 2, "samples": 3, "stability": 0.67},
        "pipedrive.com": {"occurrences": 1, "samples": 3, "stability": 0.33}
    }
}
```

When `previousRunDatasetId` is set, the actor writes a `delta` record showing per-query gained/lost citations versus the prior run.

A final `summary` record closes every run with stats and finishedAt.

## Recipes

### Daily monitoring with weekly diff

Schedule the actor daily. Use `Apify schedules` to run it every morning. Pass last week's `defaultDatasetId` as `previousRunDatasetId` to get a weekly diff alongside today's snapshot.

### Multilingual brand share-of-voice

Set `language: "hu"` (or `de`, `fr`, …) and Gemini will reply in that language and prefer same-language sources. Especially useful for European B2B SaaS, agencies running multi-country pipelines, or local SEO consultants.

### Stability filter for hallucinations

Set `samplesPerQuery: 3` and ignore citations whose `stability < 0.5` — these are likely one-shot hallucinations rather than reliable AI rankings.

## Local development

```
# install deps
npm install

# unit tests on pure-JS utils (no API keys, no network)
npm test

# end-to-end smoke test against real engines
GEMINI_API_KEY=AIza... npm run smoke
```

`scripts/test-utils.js` validates the citation parser, brand-detector, share-of-voice math, stability computation and delta diff with hand-crafted fixtures — runs in under 100ms with no network.

`scripts/smoke.js` calls each engine for which a key is set and prints a compact citation report. Useful for verifying the adapters still work after API surface changes.

## Architecture

```
.
├── .actor/                 Apify actor metadata (actor.json, input_schema.json)
├── src/
│   ├── main.js             entry point — reads input, fans out, writes dataset
│   ├── utils.js            pure helpers (no I/O): canonicalHost, citation parsing, share-of-voice, delta diff
│   └── engines/
│       ├── gemini.js       Gemini 2.5 Flash + google_search grounding (free tier)
│       ├── openai.js       Responses API + web_search_preview tool
│       ├── perplexity.js   Chat completions + sonar model
│       └── anthropic.js    Messages API + web_search tool
├── scripts/
│   ├── test-utils.js       offline assertions for utils.js
│   └── smoke.js            live-network smoke test for the engine adapters
└── INPUT_SCHEMA.json       legacy top-level schema (also at .actor/input_schema.json)
```

The four engine adapters share the same return shape so `main.js` doesn't care which engine produced a record. Adding a new engine = one file under `src/engines/` plus one entry in `engineAdapters` in `main.js`.

## Limitations / honest disclosure

- **AI answers are non-deterministic.** A single sample of a single query can miss citations that appear most of the time — that's why we expose `samplesPerQuery` and a stability score. Don't make business decisions on a `samplesPerQuery: 1` snapshot; use 3+ for monitoring you trust.
- **Free Gemini tier has rate limits.** 10 requests/minute, 250 requests/day on `gemini-2.5-flash`. The actor's `maxConcurrency` defaults to 4 to stay well under the per-minute cap. For higher volume, upgrade to Gemini's paid tier (still cheaper than any GEO SaaS).
- **Google AI Overviews are not yet covered** in this v0.1 build. Gemini grounded answers are a strong proxy (same underlying LLM and search index) but the literal SERP AIO block is a separate feature. v0.2 will add it via Apify's `apify/google-search-scraper` actor.
- **Engines may shift their APIs.** OpenAI's `web_search_preview` and Anthropic's `web_search_20250305` are explicitly versioned tools. We pin sensible defaults; if a vendor breaks compatibility, the relevant adapter file is the only file that needs to change.

## Roadmap

- v0.2 — Google AI Overview capture via the existing `apify/google-search-scraper` actor (chained, BYOK Apify proxy).
- v0.3 — Slack / webhook alerting on citation gains/losses.
- v0.4 — White-label CSV export with agency branding.
- v0.5 — Public weekly leaderboards (e.g. "Top 50 brands cited in ChatGPT for project management software, EN/DE/HU") generated from aggregated runs of consenting users — distribution / SEO play.

## License

ISC.