[AI Search Visibility Tracker](https://apify.com/khadinakbar/ai-search-visibility-tracker?fpr=data)

# AI Search Visibility Tracker — AEO & Citation Audit for Perplexity, ChatGPT, Claude & Gemini

**Find out if AI search engines cite your website as a source — and which topics they don't.** Submit any keyword or topic, and this actor queries Perplexity, ChatGPT, Claude, and Gemini to check whether your domain appears in their citations. No API keys needed. Results in minutes.

## What This Actor Does

When someone asks Perplexity "how do I do keyword research?", the AI cites 5–8 source URLs. This actor finds out if **your domain is one of them** — and if not, flags it as an **AEO content gap** you can act on.

Tracks four signals per check:

- **Citation presence** — is your domain cited at all?
- **Citation rank** — position 1 = your site is the primary source (lower = better)
- **Content gap** — AI cited competitors but not you → high-priority content opportunity
- **Competing domains** — see who else gets cited for your topics

Runs real-time web search on every query. No cached or static data.

## Who Uses This

| Buyer | Use case |
| --- | --- |
| **SEO teams** | AEO (Answer Engine Optimization) — measure whether content updates translate to AI citation gains |
| **Content marketers** | Find topics where AI answers with competitor pages instead of yours |
| **Digital agencies** | Report AI search visibility scores to clients; track month-over-month |
| **Website owners** | Understand which pages AI platforms treat as authoritative sources |

## Output Data Table

| Field | Type | Description |
| --- | --- | --- |
| `domain_cited` | Boolean | True if any page from your domain was cited |
| `citation_rank` | Integer or null | 1-based position in citation list (1 = primary source) |
| `domain_citation_count` | Integer | How many of your pages were cited |
| `cited_page_url` | String or null | First page from your domain that was cited |
| `cited_pages` | Array | All cited pages from your domain |
| `content_gap` | Boolean | AI cited others but not you — actionable gap |
| `ai_coverage_ratio` | Float | Your pages / total sources (0.25 = 25% of sources are yours) |
| `competing_domains` | Array | Other domains cited for the same topic |
| `keyword` | String | The keyword/topic checked |
| `platform` | String | perplexity, chatgpt, claude, or gemini |
| `ai_answer_excerpt` | String | First 300–600 chars of the AI's answer |
| `visibility_score` | Integer | 0–100 composite across all keywords and platforms |
| `checked_at` | ISO datetime | When the check ran |

## Run Summary (Key-Value Store)

Every run saves `LAST_RUN_SUMMARY` to the key-value store:

- `visibility_score` — 0–100 composite (citation rate × rank quality × topic coverage × page bonus)
- `citation_rate` — fraction of checks where your domain was cited
- `avg_citation_rank` — where your domain typically appears in source lists
- `topic_coverage_rate` — percentage of keywords cited at least once
- `content_gaps` — list of uncited keywords (highest-priority AEO opportunities)
- `top_cited_pages` — your most-cited pages with citation counts
- `top_competing_domains` — competitor domains ranked by how often AI cites them for your topics
- `recommendations` — 3–5 actionable steps based on your data

## AI Search Visibility vs Brand Monitoring

|  | AI Search Visibility Tracker | AI Search Brand Monitor |
| --- | --- | --- |
| **Tracks** | Domain / page citations in source lists | Brand name mentions in AI answers |
| **Input** | Keywords / topics | Brand name |
| **Key metric** | Citation rank position | Mention rate + sentiment |
| **Content gaps** | ✅ Finds uncited topics | ❌ |
| **Page-level detail** | ✅ Which exact page was cited | ❌ |
| **Best for** | SEO / content teams | Brand / PR / marketing teams |

Run both for a complete AI search presence picture.

## Pricing

**$0.09 per keyword × query × platform check** (plus standard Apify actor-start fee based on memory).

| Scale | Config | Total checks | Approx. cost |
| --- | --- | --- | --- |
| Quick audit | 5 keywords × 1 query × 2 platforms | 10 | ~$0.90 |
| Standard | 10 keywords × 3 queries × 4 platforms | 120 | ~$10.80 |
| Deep audit | 20 keywords × 5 queries × 4 platforms | 400 | ~$36.00 |

All AI costs (Perplexity Sonar, GPT-4o Search Preview, Claude `web_search`, Gemini Google Search grounding) are included — no API keys required.

## How to Use

### Quick start (minimum input)

```
{
  "targetDomain": "yoursite.com",
  "keywords": ["your topic 1", "your topic 2", "your topic 3"]
}
```

### Full input with competitors and custom queries

```
{
  "targetDomain": "ahrefs.com",
  "keywords": ["SEO", "keyword research", "backlink analysis", "site audit"],
  "competitorDomains": ["semrush.com", "moz.com", "backlinko.com"],
  "platforms": ["perplexity", "chatgpt", "claude", "gemini"],
  "queryTemplates": ["topic_authority", "how_to", "what_is", "comparison"],
  "maxQueriesPerKeyword": 3,
  "webhookUrl": "https://hooks.zapier.com/your-hook",
  "responseFormat": "detailed"
}
```

### Python

```
import apify_client

client = apify_client.ApifyClient("YOUR_API_TOKEN")
run = client.actor("khadinakbar/ai-search-visibility-tracker").call(run_input={
    "targetDomain": "yoursite.com",
    "keywords": ["your topic 1", "your topic 2"],
    "platforms": ["perplexity", "chatgpt"],
    "maxQueriesPerKeyword": 2,
})

for item in client.dataset(run["defaultDatasetId"]).iterate_items():
    if item["content_gap"]:
        print(f"Gap: {item['keyword']} on {item['platform']}")
    if item["domain_cited"]:
        print(f"Cited rank {item['citation_rank']}: {item['cited_page_url']}")
```

### JavaScript / Node.js

```
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: 'YOUR_API_TOKEN' });
const run = await client.actor('khadinakbar/ai-search-visibility-tracker').call({
    targetDomain: 'yoursite.com',
    keywords: ['your topic 1', 'your topic 2'],
    platforms: ['perplexity', 'chatgpt', 'claude', 'gemini'],
    maxQueriesPerKeyword: 3,
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
const gaps = [...new Set(items.filter(i => i.content_gap).map(i => i.keyword))];
console.log('Content gaps:', gaps);
```

## MCP / AI Agent Usage

Exposed as `apify--ai-search-visibility-tracker` in the Apify MCP server. Claude, ChatGPT, and other AI agents can call it directly:

```
Use apify--ai-search-visibility-tracker to check whether ahrefs.com is cited 
for "SEO" and "keyword research" on Perplexity and ChatGPT.
```

Accepts AI-friendly input aliases:

- `targetDomain` → `domain`, `website`, `site`, `url`
- `keywords` → `topics`, `keyword`, `searchTerms`, `terms`
- `competitorDomains` → `competitors`, `competingSites`

## Query Templates

| Template | What it asks | Example query |
| --- | --- | --- |
| `topic_authority` | Best resources / authoritative sites for X | "What are the best resources for learning SEO?" |
| `how_to` | Step-by-step guides | "How do I do keyword research? Step-by-step." |
| `what_is` | Definitions and explanations | "What is backlink analysis? Explain with examples." |
| `comparison` | Top tools compared | "Best tools for site audit — compare options in 2026." |
| `expert_picks` | Professional recommendations | "What do SEO experts recommend for rank tracking?" |

## AI Platform Coverage

| Platform | Model | Search method | Citation source |
| --- | --- | --- | --- |
| Perplexity | sonar | Native Sonar search | `citations[]` array |
| ChatGPT | gpt-4o-search-preview | Native OpenAI web search | Inline markdown + annotations |
| Claude | claude-sonnet-4.5 | `web_search_20250305` tool | `message.annotations[]` url_citations |
| Gemini | gemini-2.5-flash | Native Google Search grounding | `groundingMetadata.groundingChunks[]` |

## Scheduling for AEO Trend Tracking

Run weekly to track visibility score changes over time. Each run stores `run_id` in every record — compare citation rate and citation rank across weeks without extra setup.

**Recommended schedule:** Monday morning, same keyword set each week.

## Webhook Integration

Set `webhookUrl` to receive the full run summary on completion:

- **Slack** — post weekly visibility score to `#seo` channel
- **Make.com / Zapier / n8n** — alert content team when new gaps appear
- **Google Sheets** — append weekly score rows for trend charts

## FAQ

**Does this track brand name mentions?**
No — it tracks domain citations in source lists. For brand name tracking use [`ai-search-brand-monitor`](https://apify.com/khadinakbar/ai-search-brand-monitor).

**Which AI crawl bots should I allow in robots.txt?**
Allow `PerplexityBot`, `GPTBot`, `ClaudeBot`, `Google-Extended`, `Googlebot`. Blocking these prevents AI indexing and citation. Also add an `llms.txt` file at your domain root listing key pages.

**What is a content gap?**
`content_gap: true` means the AI cited other sites for that topic but not yours. Create more comprehensive, up-to-date content on that topic to earn citations.

**How is citation_rank measured?**
1-based index of the first URL from your domain in the AI's citation list. Rank 1 = your site is the most prominently cited source. Lower numbers are better.

**Why is my citation rate low despite good Google rankings?**
AI citation and search ranking are correlated but not identical. Common causes: robots.txt blocking AI crawlers, thin page content, lack of structured data (FAQ/HowTo schema), or pages not cited by other high-authority sources.

**What is AEO?**
Answer Engine Optimization — optimizing your content to be cited by AI search platforms (Perplexity, ChatGPT, Claude, Gemini) when they answer queries. AEO is complementary to traditional SEO and increasingly important as AI search gains share.

## Legal Notice

This actor queries publicly accessible AI search APIs. It does not scrape any website. Intended for legitimate SEO research, content strategy, and website analytics. Review each AI platform's terms of service before commercial use.