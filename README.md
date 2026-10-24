# shopify scraper tool: How to Pull Any Shopify Store's Product Data with ScraperAPI, What Each Request Really Costs, and Which Plan to Pick

Most shopify scraper tool searches start with the same assumption: scraping an e-commerce store means fighting bot walls, spinning up headless browsers, and paying premium rates for every request. Shopify stores break that assumption. Nearly every store on the platform exposes its full product catalog through a public JSON endpoint — `/products.json` — that returns titles, variants, prices, stock status, SKUs, and images with a plain HTTP GET. No merchant authorization, no API key, no JavaScript rendering.

That detail changes the economics completely. On a scraping API like ScraperAPI, a standard page request costs 1 credit. Amazon costs 5, Google costs 25, LinkedIn costs 30. A Shopify catalog request is a standard request. So the tool choice matters less than knowing what you're asking for — and budgeting accordingly.

This guide covers how Shopify store scraping actually works, ScraperAPI's current plans and their real per-request costs, the limits that only show up in the billing mechanics, and when a ready-made crawler (like the ones on the Apify platform) makes more sense than wiring up an API.

## Quick picks

> **Short version:** if you're monitoring prices or catalogs across Shopify stores at a few hundred to a few thousand requests a day, **Startup ($149/month, 1,000,000 credits, 50 concurrent threads)** is the realistic fit. Hobby ($49/month, 100,000 credits) covers light monitoring and testing. Start with the free trial — 5,000 credits over 7 days, no credit card — and measure your actual per-page cost before committing. 👉 [Check ScraperAPI's full plan list and current pricing](https://www.scraperapi.com/pricing?fp_ref=coupons)

If you only need to export one store's catalog once, read the Apify comparison further down before paying for anything.

## Why Shopify stores are the cheap targets

Shopify gives every storefront a set of client-side JSON endpoints, and they're open to any visitor:

- `/products.json` — the full product list, paginated with `?page=N`
- `/products/<handle>.json` — a single product with all its variants
- `/collections.json` — the store's collections
- `/recommendations/products.json` — related products

The fields are richer than most people expect: product title, vendor, product type, tags, description (HTML and plain text), every variant with SKU, price, compare-at price, availability, and option values, plus all images with dimensions and alt text. Prices arrive as integer cents — `2000` means $20.00 — so financial totals don't drift from floating-point errors.

Two practical details worth knowing before you write any code. First, price normalization happens server-side on Shopify's end, which is why these endpoints are popular for price-monitoring tools. Second, checking whether a target is even a Shopify store takes five seconds: append `/admin` to the domain (as in `gymshark.com/admin`). If you see a Shopify login page, the store is on the platform and its catalog endpoints exist.

For ScraperAPI, this means your requests fall into the cheapest billing bucket. A full catalog sweep of a mid-size store is typically a few hundred to a few thousand paginated requests — 1 credit each, assuming the store isn't behind heavy bot protection. Third-party reviews list e-commerce pages at 5 credits in some cases, so treat 1 credit as the expected baseline and let the Domain Cost Estimator in your dashboard give you the exact number for your specific targets.

## How to scrape a Shopify store through ScraperAPI

The integration is a single endpoint: you send your target URL plus your API key, and ScraperAPI handles proxy rotation, retries, and CAPTCHA handling on its side. For a Shopify catalog job, the loop looks like this:

1. Confirm the store is on Shopify using the `/admin` check.
2. Request `/products.json?page=1` through ScraperAPI's endpoint — a standard request, 1 credit.
3. Parse the JSON, advance the page parameter, repeat until the product list stops returning new items.
4. For individual products, hit `/products/<handle>.json` for full variant detail.

Parameters you might touch:

- `country_code` — geotargeting. Note the plan restriction: Hobby and Startup can only target **US and EU** exit points. Country-level targeting outside those regions starts at Business ($299/month).
- `render=true` — spins up a headless browser, costs about 10 credits per request. For `/products.json` you almost never need this; the endpoint returns server-rendered JSON. Save it for store pages that block plain requests.
- `premium=true` or `ultra_premium=true` — routes through the residential and mobile IP pools. Officially priced at 10–30 credits per request depending on the target. Most Shopify stores don't require it; a Cloudflare-fronted store might.
- `session_number` — sticky sessions on the same IP, capped at roughly 1 hour. Irrelevant for anonymous catalog scraping, relevant if you later expand to targets requiring login state.
- `max_cost` — caps how many credits a single request can burn. Set this before you let a script run unattended.

That last one matters more than it looks. Dashboard pricing includes a Domain Cost Estimator that shows the exact per-URL cost before you scrape, which is the honest way to budget: run your real target through it, multiply by your monthly request volume, then pick the plan.

👉 [Sign up and get 5,000 free trial credits to test your target store](https://www.scraperapi.com/?fp_ref=coupons)

## ScraperAPI's full plan list and pricing

Seven self-serve paid tiers plus a free plan, all billed monthly with a 10% discount on annual billing. Here's the complete table, with the free plan noted separately below it:

| Plan | Monthly price | Annual price | API credits | Concurrency | Geotargeting | Buy |
| --- | --- | --- | --- | --- | --- | --- |
| Hobby | $49/mo | $44.10/mo | 100,000 | 20 threads | US & EU only | [ Get Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149/mo | $134.10/mo | 1,000,000 | 50 threads | US & EU only | [ Get Startup](https://www.scraperapi.com deeplink...) |

Writing the buy links properly: every row's link is the pricing page with the referral parameter attached, since that's where each plan's checkout flow starts. [👉 View plans and pick your tier on the official pricing page](https://www.scraperapi.com/pricing?fp_ref=coupons)

**Free plan:** signing up gets you 1,000 free API credits with a maximum of 5 concurrent connections, no credit card required. On top of that, there's a 7-day trial with 5,000 credits. Combined, that's enough to test two or three target stores end-to-end before spending anything.

**Enterprise** is quote-based for anything beyond 22 million credits a month; the pricing page routes it through a sales contact rather than a checkout button.

A note on the annual discount: the pricing page lists it as 10%. Some third-party reviews describe annual savings as 15–20%. When the official page and reviews disagree, go with the official number — and note that annual plans can renew early for a fresh batch of credits, which softens the "my credits ran out mid-month" problem.

## What your credits actually buy

The single most useful thing to understand before picking a plan: **credits are not requests.** The official billing works like this:

- Standard page request: 1 credit
- Amazon: 5 credits
- Google and Bing (including subdomains): 25 credits
- LinkedIn: 30 credits
- Sites behind Cloudflare, Datadome, or PerimeterX: +10 credits per request when ScraperAPI bypasses them
- JS rendering (`render=true`): roughly 10 credits per request

For Shopify work specifically, this works in your favor. A `/products.json` request is a standard request. If you're building a price monitor that hits 20 stores a few times a day, you're spending 1 credit per fetch, and Startup's million credits stretch a very long way.

The math flips on mixed workloads. Reviews note that a job needing `render=true` plus `premium=true` on a protected target can cost 20–30× the headline per-credit price, and that a million credits might cover only 40,000–100,000 actual pages on that kind of job. If your project mixes Shopify catalogs with Google SERP tracking, budget those buckets separately — the Shopify side at 1 credit, the SERP side at 25.

There's also a billing edge case worth knowing: reviewers have documented cases where a CAPTCHA page or error page came back counted as a "successful" response and was billed. ScraperAPI's support does refund these when flagged, and the advertised failure rate is 1–3% with failed requests not charged — but if you're running high volume, spot-check your responses for CAPTCHA pages so you know whether it's happening to you.

## The limits that don't show up in the price table

These are all confirmed behaviors, and each one has bitten someone who skipped this section:

**Concurrency overages get refused, not queued.** Exceed your plan's thread count and requests return a 429 immediately — no charge, but no fallback either. Hobby's 20 threads is fine for a few stores; sweeping a store with tens of thousands of SKUs on a schedule means throttling your script or moving up to Startup's 50.

**Running out of credits pauses your scraping by default.** Requests stop until the next cycle renews your credits. Hobby through Professional can opt into auto-upgrade, which bumps you to the next plan automatically when credits run out. Scaling and above get pay-as-you-go, which continues scraping at a fixed per-credit rate up to a monthly spending cap you set. For an unattended price monitor, decide which behavior you want *before* the credits run out — a monitor that silently dies for a week is worse than one that bills you a few dollars over.

**Geotargeting is tiered.** Below Business, you're limited to US and EU exit IPs. If you need to see a store as it appears in, say, Japan or Australia (localized pricing, region-specific catalogs), that capability starts at $299/month.

**Analytics retention differs by tier.** Hobby and Startup only keep 30 days of usage history; Business and up keep it indefinitely. If you audit your credit spend across long monitoring projects, the higher tiers make that easier.

**Sticky sessions last about an hour.** Fine for catalog scraping, a hard limit if your use case drifts toward account-based workflows that need day-long IP consistency — the official structure just isn't built for that.

None of these make ScraperAPI a bad fit for Shopify work. They're the details that determine which plan you actually need, versus the plan that looked right on the pricing page.

## ScraperAPI vs. ready-made Shopify crawlers

The other path is a hosted crawler purpose-built for Shopify stores. The best-known example is on the Apify platform: a Shopify Scraper sold with pay-per-event billing — you pay per product and collection actually scraped, no monthly rental, no minimum spend. You paste in store URLs, it walks the same public endpoints (`/products.json`, collections, recommendations), and exports to JSON, CSV, Excel, or HTML. Input options include a max-products-per-run cap to control spend (default 100), a search query to pull matching products from a store, and up to 20 related-product recommendations per item. It runs residential proxies by default, with the explicit warning that datacenter IPs are often blocked by Cloudflare-fronted Shopify stores.

Which route fits depends on what you're building:

- **One-off catalog exports, a handful of stores, no code:** the hosted crawler wins. Pay-per-event means a 500-product export costs pennies, and there's nothing to maintain.
- **Continuous monitoring, custom pipelines, integration with your own database:** ScraperAPI wins. You control pagination, storage, scheduling, and field filtering, and flat monthly pricing makes cost predictable at volume.
- **Heavy Cloudflare protection on your targets:** pause and reconsider both. Independent benchmarks put ScraperAPI's overall success rate around 62–72% depending on the test, with outright 500 errors reported on some Cloudflare and DataDome pages. If every store on your list is behind serious bot protection, test with the free trial before paying anyone.

There's also a zero-cost option worth mentioning: open-source Shopify scrapers on GitHub in Python and Go that hit `/products.json` directly with no API key. They work fine against unprotected stores. What you're paying a service like ScraperAPI for is the proxy rotation, retry logic, and CAPTCHA handling — which matters the moment a store fights back or you're running thousands of requests a day from one machine.

## FAQ

**Is scraping Shopify store data legal?** The public endpoints (`/products.json` and friends) are served to every visitor by design — no login, no authorization. Scraping publicly available catalog data is standard practice for price monitoring and market research. Order data, customer data, or anything behind a merchant login is a completely different category; stay out of it.

**How many requests do I actually get on each plan?** For pure Shopify catalog work, roughly the credit count: Hobby's 100,000 credits ≈ 100,000 catalog fetches, Startup's million credits ≈ a million. The multipliers only bite when you mix in JS rendering, premium pools, Google/Amazon targets, or protected stores.

**Is the free tier enough to evaluate the service?** Yes, for Shopify specifically. The 1,000 monthly credits plus the 7-day 5,000-credit trial cover a full catalog sweep of most stores plus change. Run your real targets through the Domain Cost Estimator during the trial — that's the number your plan choice should be based on.

**How much does annual billing save?** 10% per the official pricing page: Hobby drops from $49 to $44.10 per month, Startup from $149 to $134.10. Annual plans can also renew early to bank a fresh batch of credits.

## Before you pick a plan

Match the plan to your actual request volume, not the credit headline:

- **Light monitoring, one or two stores, under ~3,000 standard requests a day:** Hobby at $49 (or $44.10 annual) covers it, with room to spare.
- **Multi-store price tracking, catalogs with thousands of SKUs, 50 concurrent requests:** Startup at $149. This is the plan most Shopify-focused projects settle on.
- **Country-level geotargeting or over ~33,000 requests a day:** Business at $299, which unlocks non-US/EU exit IPs and 100 threads.
- **One-time export of a few hundred products:** don't buy a monthly plan at all — a pay-per-event crawler or the free trial will do the whole job.

One last thing before you click buy: run your target stores through the Domain Cost Estimator with the free trial, write down the actual per-request cost, and check whether any of them sit behind Cloudflare. That two-minute check tells you whether you're in the 1-credit zone or the 10-to-30-credit zone — and that's the difference between Hobby being plenty and Hobby being a trap. 👉 [Start with 5,000 free trial credits and test your target stores](https://www.scraperapi.com/?fp_ref=coupons)
