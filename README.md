# Google Maps Business Contact Scraper MCP

Scrape business contact information from Google Maps — names, addresses, phone numbers, websites, ratings, and reviews. Built as an Apify MCP (Model Context Protocol) standby actor for AI agents and LLM integrations.

[Apify Store Listing](https://apify.com/red.cars/google-maps-business-contact-mcp) | [ScraperFleet data page](https://scraperfleet.com/actors/google-maps-business-contact-mcp/)

## What does this actor do?

Extracts **business contact data from Google Maps** using a headless browser. Search for businesses by query (e.g., "restaurants in Manhattan" or "plumbers near Times Square") and get back structured contact information including phone numbers, websites, ratings, and review counts.

The actor runs as an **MCP standby actor** — it stays idle until called via the MCP JSON-RPC protocol, then returns results directly. It's also usable as a standard batch Apify actor with JSON input.

## Why use this actor?

- **Lead generation** — Build prospect lists by category and location
- **B2B sales** — Find decision-maker contacts for companies in a territory
- **Market research** — Map competitor locations and contact details
- **Business discovery** — Find all businesses of a given type in a neighborhood
- **Real estate research** — Map all businesses at a given address or zip code

## Tools

| Tool | Price | Description |
|------|-------|-------------|
| `search_businesses` | $0.03 | Search Google Maps by query, returns name/address/rating/placeId |
| `get_business_details` | $0.05 | Get phone and website for a specific placeId |
| `search_businesses_with_emails` | $0.08 | Search + enrich each result with phone and website |

## How to use

### MCP Protocol (AI agents)

Send a JSON-RPC POST to `/mcp` with:

```json
{
  "method": "tools/call",
  "params": {
    "name": "search_businesses",
    "arguments": {
      "search_query": "restaurants in Manhattan",
      "max_results": 10
    }
  }
}
```

### Apify Batch Mode

```bash
apify call red.cars/google-maps-business-contact-mcp \
  --input '{"tool": "search_businesses", "params": {"search_query": "dentists in Brooklyn", "max_results": 5}}'
```

### JavaScript / TypeScript

```javascript
const { handleRequest } = await import('./dist/main.js');

const result = await handleRequest({
  toolName: 'search_businesses',
  arguments: { search_query: 'coffee shops in SoHo', max_results: 10 }
});
console.log(JSON.parse(result.content[0].text));
```

## Input

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `tool` | string | Yes | One of: `search_businesses`, `get_business_details`, `search_businesses_with_emails` |
| `search_query` | string | For search tools | Google Maps search query (e.g., "restaurants in NYC") |
| `place_id` | string | For details tool | Google Maps place ID from a search result |
| `max_results` | integer | No | Max results to return (1–50, default 10) |

### Batch input example

```json
{
  "tool": "search_businesses_with_emails",
  "params": {
    "search_query": "plumbers near Times Square",
    "max_results": 20
  }
}
```

## Output

Dataset items with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Business name |
| `address` | string | Full address |
| `phone` | string \| null | Phone number |
| `website` | string \| null | Website URL |
| `rating` | number \| null | Star rating (e.g., 4.5) |
| `reviews` | integer \| null | Number of reviews |
| `category` | string \| null | Business category |
| `placeId` | string \| null | Google Maps place ID (use with `get_business_details`) |

### Output example

```json
{
  "query": "pizza in Brooklyn",
  "count": 2,
  "businesses": [
    {
      "name": "Joe's Pizza",
      "address": "123 Smith St, Brooklyn, NY 11201",
      "phone": "+1-718-555-0100",
      "website": "https://joespizza.com",
      "rating": 4.7,
      "reviews": 892,
      "category": "Pizza delivery",
      "placeId": "ChIJ8fz2XcJZwok..."
    }
  ]
}
```

## Cost estimation

Pricing is **per tool call**, not per result:

| Tool | Price | Notes |
|------|-------|-------|
| `search_businesses` | $0.03 | Search-only, returns name/address/rating/placeId |
| `get_business_details` | $0.05 | Per placeId — phone and website |
| `search_businesses_with_emails` | $0.08 | Search + sequential enrichment |

**Actual platform cost per 1,000 results:** $9.48–$147.32 (median ~$22)
- Wide range reflects `max_results` (1–50) and enrichment depth
- `search_businesses` only runs: ~$3–$5 per 1,000 results
- `search_businesses_with_emails` with high `max_results`: up to $147/1K
- Sequential enrichment limits p99 cost — results are processed one at a time, so timeout doesn't mean wasted spend

**Tip:** Start with `search_businesses` (cheapest) to see data quality, then selectively enrich specific `placeId`s with `get_business_details` for $0.05 each.

## Tips and advanced options

- **Two-step enrichment** — Run `search_businesses` first ($0.03), then selectively call `get_business_details` ($0.05) per `placeId`. This gives granular control and lower per-result cost than `search_businesses_with_emails` when you only need a subset enriched.
- **Sequential processing** — `get_business_details` runs one page visit at a time, so timeout doesn't waste a full batch — you get partial results on timeouts.
- **Limit `max_results`** — Use `max_results: 5` for quick tests before scaling up to 50.
- **Rate limiting** — Google Maps may throttle rapid requests; add delays between calls in high-volume scenarios.
- **Proxy rotation** — Apify accounts include built-in proxy rotation for anti-bot protection.

## FAQ

**Is this legal to use?**
This scraper is for scraping publicly available business contact information from Google Maps. Ensure your use case complies with Google's Terms of Service and applicable laws. Do not use for spam, harassment, or unsolicited marketing.

**Why is the address sometimes empty?**
Google Maps doesn't always return a full street address in search results. Use the `placeId` with `get_business_details` for more complete data.

**Why does enrichment cost more?**
`search_businesses_with_emails` requires an additional page visit per business to extract phone and website. Use `get_business_details` individually for precise per-result cost control.

**What's the per-result cost?**
Platform-reported cost per 1,000 results averages ~$22.18 (range $9.48–$147.32). Search-only runs average $3–$5/1K. High-end runs hit enrichment on all results with `max_results=50`.

**How do I get a placeId?**
Run `search_businesses` first — each result includes a `placeId` field.

## Pair it with

**[Website Contact Scraper Pro MCP](https://apify.com/red.cars/website-contact-scraper-pro-mcp)** — pulls the emails, phones and social links from the websites of the businesses you find here, so a search-then-enrich pipeline goes Maps discovery to direct outreach without extra tooling.

## How It Compares

| | Google Maps Business Contact Scraper MCP | Website contact scrapers alone | Manual Google Maps searches |
|---|---|---|---|
| Finds new businesses by query | Yes, any local search | No, needs a URL list | Yes, one at a time |
| Contact fields returned | Names, addresses, phones, websites, ratings, reviews | Varies by site | Copy-paste by hand |
| Works from AI agents | Yes, MCP tools in Claude/Cursor/any MCP client | Some | No |
| Cost model | Pay per event, no subscription | Varies | Your time |

**Where this fits:** it is the discovery layer for local business outreach — search a city or category, get structured contacts back, then enrich or CRM-load them downstream.

## Support

For issues or feature requests: [Apify Issues](https://console.apify.com/actors/V2o1xu94Biqndv3Ru/issues)

For custom integrations: contact via Apify Store profile
