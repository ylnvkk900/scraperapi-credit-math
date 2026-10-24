# web crawling in python: Build a Crawler That Survives Real Sites, With ScraperAPI Pricing and Credit Math Explained

Search for **web crawling in python** and you'll mostly find the same tutorial: install `requests` and BeautifulSoup, write a loop that follows links, done. That tutorial works. The problem is what happens after it — your crawler hits page 200, the target site starts serving CAPTCHAs, your IP gets rate-limited, or every "page" comes back as an empty JavaScript shell.

This article covers both halves of the problem. First, what a DIY Python crawler can genuinely handle and where it stops. Then, what it actually costs to fix those failures — using ScraperAPI's current plans, the credit math that determines your real per-page cost (which most reviews gloss over), and the situations where a scraping API is the wrong tool entirely.

> **Quick take:** For crawling standard sites in Python, ScraperAPI's Hobby plan at **$49/month for 100,000 credits** covers most hobbyist workloads, and there's a **free 7-day trial with 5,000 credits — no credit card required** — so you can test it against your actual target before paying anything. 👉 [Start the free 7-day trial and point your Python crawler at a real site](https://www.scraperapi.com/?fp_ref=coupons)

## A minimal Python crawler: what's genuinely easy

The core of a web crawler in Python is small. You keep a queue of URLs to visit, a set of URLs you've already seen, a rule that keeps you on one domain, a depth cap, and a polite delay between requests. `requests` fetches the page, BeautifulSoup extracts the links, and you write results out as JSON Lines so a crashed run doesn't lose everything. Checking `robots.txt` with the built-in `RobotFileParser` takes about five more lines and keeps you on reasonable terms with the site.

For static HTML — documentation sites, blogs, quote aggregators, forums — this is genuinely all you need. The tutorials ranking for this keyword are right about that part. Roughly 40 lines of Python and you have a working crawler with deduplication, scope control and depth limits.

The trouble is that "static HTML" is a shrinking slice of the web.

## Three places a DIY crawler breaks down

The first wall is JavaScript. A lot of modern sites load their content client-side, so `requests` returns a clean-looking HTML skeleton containing none of the data you came for. The standard fix is Playwright or Selenium, which works but turns your lightweight script into a headless browser farm that eats memory and slows every request to seconds.

The second wall is rate limiting. Crawl more than a handful of pages per minute on most commercial sites and your IP gets throttled or banned. Rotating residential proxies solve this, but building a reliable proxy rotation layer is its own infrastructure project — one that has nothing to do with the data you actually want.

The third wall is the hard one: anti-bot systems. Cloudflare, DataDome, PerimeterX and Akamai fingerprint things at the TLS and protocol level, below where your code operates. A plain `requests` call fails on these sites no matter how good your proxy rotation is, because the detection happens before your HTTP request is even fully processed. One reviewer who tested proxy-based scrapers against 40 protected sites put it bluntly: the detection "operates at the TLS and protocol level," which proxy rotation and headless browsers can't fake their way past.

Each of these three problems is solvable alone. Solving all three, reliably, at volume, is a full-time infrastructure job. That's the gap a scraping API fills: you send a URL, the service routes it through its proxy pool, renders JavaScript if asked, handles CAPTCHAs, and returns HTML. ScraperAPI is one of the better-known options — it's been around since 2018, runs a proxy pool its docs put at over 40 million IPs, and bills by credit rather than by proxy hours.

## ScraperAPI: every plan on the official pricing page

Here's the complete lineup from ScraperAPI's pricing page. Annual billing takes 10% off every plan.

| Plan | Price (monthly / annual) | Credits/mo | Concurrency | Geo-targeting | Get started |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | 1,000 | 5 | US & EU | [ Start free](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | $49 / $44.10 | 100,000 | 20 | US & EU only | [ Get Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 / $134.10 | 1,000,000 | 50 | US & EU only | [ Get Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 / $269.10 | 3,000,000 | 100 | Country-level | [ Get Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | $475 / $427.50 | 5,000,000 | 200 | Country-level | [ Get Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 / $877.50 | 10,500,000 | 300 | Country-level | [ Get Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 / $1,777.50 | 21,500,000 | 500 | Country-level | [ Get Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | 22,000,000+ | 500+ | Country-level | [ Contact sales](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Every paid plan includes the same feature set: JS rendering, premium residential and mobile IPs, CAPTCHA and anti-bot handling, JSON auto-parsing, custom sessions, automatic retries, unlimited bandwidth and a 99.9% uptime guarantee. Plans differ by credits, concurrency and geo-targeting granularity, not by features — so there's no situation where you pay $49 and discover rendering is "locked behind Business."

## The credit math that determines your real cost

Here's the part most pricing summaries skip: **a credit is not a request.** ScraperAPI's own documentation prices different domains differently — a standard page costs 1 credit, Amazon costs 5, Google and Bing cost 25, LinkedIn costs 30, and sites behind Cloudflare, DataDome or PerimeterX add 10 credits per request when the service bypasses them.

Feature flags stack on top of that. Per the docs: `render=true` (JavaScript rendering) costs 10 credits, `premium=true` (premium proxies) costs 10, both together cost 25, `ultra_premium` costs 30, and ultra-premium with rendering runs 75.

Run the numbers and the headline credit counts shrink fast. That Hobby plan's 100,000 credits:

- Plain static pages: 100,000 real requests
- Every request with JS rendering: about 10,000 real requests
- Every request with rendering plus premium proxies (25 credits): about 4,000 real requests

This is why the second most common developer complaint about ScraperAPI — after anti-bot performance — is that "real costs climb fast." A 10.5-million-credit Professional plan sounds enormous, but if your workload renders JavaScript on every request, your effective budget is one-tenth of that: 1.05 million requests.

Two tools help you stay in control. The Domain Cost Estimator in the dashboard shows the exact credit cost for any URL before you crawl it, and you can set a `max_cost` per request so a single expensive page can't silently eat your balance. Checking both before committing to a plan is the single most useful thing you can do during the trial. 👉 [Check your target domains' credit costs on ScraperAPI](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Hidden billing details worth knowing before you buy

Several things that affect your cost only show up in the docs and FAQ, not on the pricing page.

**Concurrency overruns are free.** If you exceed your plan's concurrent thread limit, ScraperAPI returns a 429 and rejects the request without charging a credit — it doesn't queue and it doesn't bill. Keep an eye on your thread count in code, though; rejected requests still waste your time.

**Running out of credits pauses you by default.** Scraping stops until your credits renew. To keep going you have options, and they differ by tier: auto-upgrade (opt-in) moves you to the next plan automatically when credits run out, available on Hobby through Professional; pay-as-you-go continues at a fixed per-credit rate up to a monthly spending cap you set, available on Scaling and above. Annual plans can renew early for a fresh batch of credits.

**Failed requests aren't billed — officially.** The docs state that only successful requests consume credits, and internal retries don't count against you. One caveat from third-party reviews: some users report CAPTCHA or error pages occasionally being returned as "successes" and billed. If you see that, support is reportedly responsive about refunds — but it's worth sampling your returned HTML to make sure you're getting what you paid for.

**Lower tiers have geographic and analytics limits.** Hobby and Startup are restricted to US and EU geo-targeting; the full 70-country list unlocks at Business. The analytics dashboard on lower tiers only shows the last 30 days; Business and up get unlimited history. Neither matters for a weekend project, but if you need to target, say, Japan or Brazil specifically, Hobby won't do it regardless of credits.

## The uncomfortable part: performance on protected sites

ScraperAPI advertises automatic CAPTCHA handling and anti-bot bypassing, and its marketing materials imply compatibility with Cloudflare and friends. The independent data says otherwise, and you should see it before spending money.

One detailed 2026 review ran premium proxies plus JS rendering against 40 protected sites, 5,000 requests each, using the service's own recommended settings. Results: Akamai Bot Manager 4.2% success, DataDome 11.7%, Kasada 1.8%, PerimeterX 8.3%, Cloudflare Turnstile 22.4%. The Kasada case is the starkest illustration: at roughly 1,389 credits consumed per successful request, you'd burn through an entire Business plan's 3 million credits to successfully scrape about 2,160 pages — $299 for 2,160 pages, or $0.14 per page.

Broader benchmarks are kinder but still middling: one measured 72.57% success, another put the overall average around 62.8–63.7%, only slightly above the industry average of roughly 58–59%. Users also report outright 500 errors on some Cloudflare and DataDome pages.

To be fair, the same review that produced those brutal numbers concluded that for workloads that are 90% basic sites, the Business plan at $299/month is "genuinely good value," the API is simple enough that a zero-experience developer can integrate it in minutes, and support is responsive. The failure mode is specific: it's the protected targets, not the product overall.

If your crawl list is dominated by Cloudflare or DataDome sites, dedicated "unlocker" products do better — Bright Data's Web Unlocker is independently measured around 95–98% success on that same class of targets, though at a higher price and with more setup friction. That's the honest trade-off: ScraperAPI is the cheap, simple option for sites that don't fight back, and an expensive lesson on sites that do.

## Wiring ScraperAPI into your Python crawler

The integration is deliberately minimal. Instead of calling `requests.get(target_url)`, you call ScraperAPI's endpoint with your target URL as a parameter and your API key in the request. That one substitution gets you proxy rotation, retries and CAPTCHA handling on every call.

From there, the useful parameters map directly onto crawler problems: `render=true` turns on JavaScript rendering when a page comes back empty; `country_code` pins requests to a specific country; `session_number` gives you a sticky session that reuses the same IP for up to 15 minutes after last use — useful for login flows or paginated sequences; `autoparse=true` returns structured JSON instead of raw HTML for supported domains. There are also dedicated structured endpoints for Amazon search, Google search, Google Shopping, Walmart and eBay that return parsed JSON, which means you stop maintaining CSS selectors against sites that redesign constantly.

For actual crawling at volume, the async endpoint lets you submit batches of URLs and get results via webhook, and the DataPipeline feature provides a low-code UI for scheduling recurring jobs. Official client libraries exist for Python, Node, PHP, Ruby, Java and cURL, and there's a LangChain integration if you're feeding crawled pages into agent workflows.

If you're building this on the free tier: the signup plan includes 1,000 credits with 5 concurrent connections, and the 7-day trial starts you with 5,000 — enough to crawl a few hundred rendered pages and see whether the output quality holds up on your targets. 👉 [Sign up for ScraperAPI and test the API with your own Python code](https://www.scraperapi.com/?fp_ref=coupons)

## Which plan, by situation

- **Learning or evaluating:** Free plan (1,000 credits) plus the 7-day trial (5,000 credits). Costs nothing, no card required.
- **Personal projects, standard sites, no JS-heavy targets:** Hobby at $49/month. If your pages are static, 100,000 requests is a lot of crawling.
- **Real product with JS rendering on most pages:** Startup at $149/month. Remember the 10x multiplier — 1M credits is about 100K rendered requests.
- **Business volume, need non-US/EU geo-targeting:** Business at $299/month is the cheapest tier with country-level targeting and unlimited analytics history.
- **Heavily protected targets:** think twice before scaling up. The math above shows per-success costs on Kasada or Akamai sites can run $0.02–$0.14 per page; a dedicated unlocker will likely cost less at that point even at a higher sticker price.

Two things to confirm before checkout: check your highest-volume domains in the Domain Cost Estimator so your credit math is based on your actual targets, and make sure your concurrency needs fit the plan — 429s on a 5-thread Free plan arrive quickly if your crawler is parallel by design. 👉 [Compare all plans and pick your tier on ScraperAPI](https://www.scraperapi.com/pricing?fp_ref=coupons)
