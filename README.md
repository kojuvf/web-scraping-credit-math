# web scraping node js: The Stack That Actually Works, Plus the Credit Math Most Tutorials Skip

Most "web scraping node js" tutorials stop at the same place: install Axios and Cheerio, parse a static page, done. That works until you hit a site that renders content in the browser, or one that rate-limits your IP after fifty requests. This article covers the working Node.js stack from the ground up, then gets specific about what it costs to run scrapers at scale, including the credit math that pricing pages rarely spell out for you.

**Quick pick:** if your Node.js scrapers need to survive JavaScript-rendered pages, CAPTCHAs, or bot protection at any real volume, a scraping API like **ScraperAPI** removes the proxy and browser-management layer entirely. Plans start at $49/month for 100,000 credits, there's a 7-day free trial with 5,000 credits and no credit card required, and annual billing takes 10% off every tier. 👉 [Check ScraperAPI's current plans and start the free trial](https://www.scraperapi.com/?fp_ref=coupons)

## The Node.js scraping stack: what you actually need

For static pages, the whole stack is a fetch call plus a parser. Node 18+ ships a built-in `fetch`, so the minimum viable scraper needs zero dependencies beyond Cheerio, which parses HTML with jQuery-style selectors.

The classic setup, following ScraperAPI's own step-by-step tutorial:

1. Install Node.js, verify with `node -v` and `npm -v`
2. Create a project folder and run `npm init -y`
3. Install the packages: `npm install axios cheerio` (add `puppeteer` only if you need browser automation — it downloads Chromium, so the install takes a while)
4. Fetch the page with Axios, load the response into Cheerio, extract your fields

For dynamic pages, Cheerio alone fails silently. If a page hydrates content client-side, the HTML string you pass to `cheerio.load()` is an empty shell and your selectors return nothing. Your options:

- **Puppeteer** — a Node.js library that drives Chromium. Click buttons, fill forms, take screenshots, scrape rendered data.
- **Playwright** — a modern automation library supporting cross-browser scraping. It's the most downloaded library in its category on NPM, per ScraperAPI's tutorial.

A practical pattern many scrapers converge on: try a plain fetch first, detect whether the HTML looks empty or shell-like, and only escalate to a headless browser when it does. That saves meaningful compute on sites that render server-side.

## Where a scraping API fits into this

The libraries above solve parsing. They don't solve the other half of the problem: getting blocked. Rotating proxies, CAPTCHA solving, and IP bans are infrastructure work, and maintaining your own proxy pool at scale is exactly the part that turns a weekend scraper into a second job.

ScraperAPI packages that layer into a single GET endpoint. You send your target URL (with parameters like `render=true` for JS rendering), it handles proxy rotation, retries, CAPTCHA and anti-bot detection, and returns the HTML. Your Node.js code keeps the same Axios-plus-Cheerio shape — you just point Axios at the ScraperAPI endpoint instead of the target site. Its official documentation ships Node examples alongside Python, PHP, Ruby, Java, and C#, and its tutorials for Redfin, Home Depot, and LinkedIn job listings all use this same combination. Under the hood there's a pool of roughly 40 million IPs mixing datacenter, residential, and mobile proxies, with automatic rotation and retries on failed requests.

Every paid plan includes JS rendering, premium proxies, JSON auto-parsing, rotating proxy pools, custom headers, custom sessions, desktop and mobile user agents, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee. Functionally, the tiers differ on volume, concurrency, and geotargeting, not on features.

## ScraperAPI plans and pricing: the full table

Seven paid tiers, all with the same feature set. Annual billing takes 10% off every plan.

| Plan | Price/mo (annual billed) | API Credits | Concurrent threads | Link |
| --- | --- | --- | --- | --- |
| Hobby | $49 ($44.10) | 100,000 | 20 | [ Hobby plan details](https://scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 ($134.10) | 1,000,000 | 50 | [ Startup plan details](https://scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 ($269.10) | 3,000,000 | 100 | [ Business plan details](https://scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | $475 ($427.50) | 5,000,000 | 200 | [ Scaling plan details](https://scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 ($877.50) | 10,500,000 | 300 | [ Professional plan details](https://scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 ($1,777.50) | 21,500,000 | 500 | [ Advanced plan details](https://scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | 22M+ | 500+ | [ Talk to sales](https://www.scraperapi.com/?fp_ref=coupons) |

Differences the table doesn't show:

- **Geotargeting**: Hobby and Startup are limited to US & EU. Country-level global targeting starts at Business and carries no extra credit cost there.
- **Analytics history**: Hobby and Startup keep 30 days; Business and up keep unlimited history.
- **Pay-as-you-go overage**: available only on Scaling and above. On Hobby, Startup, and Business, your options when credits run out are renewal or upgrading.
- **Support**: priority support starts at Professional; Enterprise adds a dedicated team and Slack channel.

## The credit math most articles skip

Here's the part that should drive your plan choice, and that most comparisons gloss over: the headline credit count is a ceiling, not a request count. A single request can cost anywhere from 1 to 75 credits.

The base rates, from ScraperAPI's documentation:

| Request type | Credits |
| --- | --- |
| Standard HTML request | 1 |
| JS rendering (`render=true`) | +10 |
| Premium proxy (`premium=true`) | +10 |
| Premium + rendering | 25 |
| Ultra premium (`ultra_premium=true`) | 30 |
| Ultra premium + rendering | 75 |
| Amazon, Walmart, eBay | 5 |
| Google, Bing (all subdomains) | 25 |
| LinkedIn | 30 |
| Cloudflare / Datadome / PerimeterX bypass | +10 |

Put concretely: the same 100,000-credit Hobby plan gets you anywhere from about 100,000 plain requests to roughly 1,333 ultra-premium rendered ones. If your target sites need `render=true`, your 100,000 credits are really 10,000 page loads. On protected domains there's an extra wrinkle — sites can switch their protection provider at any time, so a URL that cost 1 credit last month can quietly start costing 10 after a Cloudflare bypass kicks in.

Two tools keep this from becoming a guessing game. The dashboard's Domain Cost Estimator (also exposed as the `urlcost` API endpoint) returns the exact credit cost of any URL before you run a job, and the `sa-credit-cost` response header reports actual usage on every request. There's also a `max_cost` parameter to cap what any single scrape can consume — useful insurance when pointing at an unfamiliar domain.

The documentation also lists which parameters are free: `wait_for_selector`, `country_code`, `session_number`, `device_type`, `output_format`, `keep_headers`, and `autoparse` add nothing to the bill. Set those freely.

## Overages, concurrency limits, and the rules in the fine print

Three billing rules worth knowing before you commit, because they change how a plan behaves under load.

**Concurrency is a hard wall.** Exceeding your plan's thread limit returns a 429 and the request is turned away, not queued. There's no extra charge for going over, but there's also no waiting room. Twenty concurrent threads on Hobby means twenty simultaneous requests, full stop. For batch jobs this is often the real throughput bottleneck, sometimes more than the credit pool.

**Only successful requests are billed** — status codes 200 and 404 count, everything else doesn't. One exception: if you cancel a request yourself before ScraperAPI has had 70 seconds to finish, it's still charged.

**What happens when credits run out.** By default, scraping pauses until credits renew. On Hobby through Professional, you can opt into auto-upgrade, which bumps your account to the next plan automatically. On Scaling and above, pay-as-you-go keeps you scraping at a fixed per-credit rate up to a monthly cap you set. Annual plans can also renew early for a fresh batch of credits.

If you want to test your actual target URLs before committing to a tier, the 7-day trial ships 5,000 credits with no card required, and there's a permanent free account with 1,000 credits capped at 5 concurrent connections. 👉 [Start the free trial and run your first test scrapes](https://www.scraperapi.com/?fp_ref=coupons)

## How it stacks up against the alternatives

Against the libraries themselves, it's a layering question rather than a winner-take-all contest: Cheerio and Puppeteer handle parsing and rendering in your own code, while ScraperAPI handles delivery — proxies, CAPTCHAs, bans. Many production setups use both, exactly the combination ScraperAPI's own tutorials demonstrate.

Against comparable scraping-API services, the structural difference worth knowing is how geotargeting gets billed. Per ScraperAPI's own comparison page (worth re-checking against current competitor pricing before you decide), at least one well-known competitor — ScrapingBee — restricts geotargeting to premium proxies, which multiplies the credit cost to 10 per static request and 25 with rendering enabled, and doesn't include automatic CAPTCHA handling on its equivalent tier. ScraperAPI includes country-level geotargeting at no additional credit cost from the Business tier up. That gap can matter more than headline prices if your targets are spread across countries.

One honest caveat: ScraperAPI's own premium proxies also cost 10 credits per request, so its official advice is that standard proxies handle most targets and you should only pay the premium multiplier when a site actually has heavy anti-bot measures.

## Who this setup doesn't suit

Honest limits, because not every reader searching "web scraping node js" needs a paid API:

- **Your targets are static, low-volume, and friendly.** If you're scraping a handful of pages from sites without bot protection, plain `fetch` + Cheerio costs nothing and works fine. A scraping API is spending money to solve problems you don't have.
- **You need heavy Asia-Pacific or Latin America geotargeting on a budget.** Hobby and Startup only cover US & EU. Country-level targeting starts at Business ($299/mo), which is a real jump.
- **Your volume is unpredictable and you're on a lower tier.** Since pay-as-you-go overage only exists on Scaling and above, a Hobby account that suddenly needs 150,000 credits in a month has to upgrade rather than just pay for the extra. Worth thinking about before your project grows.
- **You want fully managed structured data with no code.** ScraperAPI returns HTML or parsed JSON for supported domains, but you still write your own parsing logic in Node.js. Turnkey data-feed products are a different category.

## FAQ

**Do I need Puppeteer if I'm using ScraperAPI?**
Usually no for rendering — `render=true` handles JS-rendered pages on the API side, at 10 credits per request. You'd still use Puppeteer if you need complex interaction flows (logins, multi-step clicks) that go beyond what parameters can express.

**How do I know a site's credit cost before scraping it?**
Use the Domain Cost Estimator in the dashboard, call the `urlcost` endpoint with your API key and target URL, or check the `sa-credit-cost` header on a test response. The API Playground lets you run a single request and see both the result and its cost.

**What happens if my scraper exceeds the concurrent request limit?**
You get a 429 response and the request is rejected rather than queued. No overage charge, but the request needs to be retried — worth building retry logic with backoff into your Node.js code regardless.

**Is the free trial actually usable?**
5,000 credits is enough to test real target URLs, including a few rendered or protected pages, and no credit card is required to start it.

## Final picks, by situation

Just experimenting or scraping a personal project: run the 7-day trial first, then Hobby at $49/month ($44.10 on annual billing) if it sticks — 100,000 credits covers a lot of standard requests, just budget for rendering. Small team shipping a production scraper against US/EU targets: Startup. Need global country-level targeting or predictable production volume: Business, where geotargeting and unlimited analytics unlock. Running jobs with spiky, hard-to-predict volume: Scaling, mainly for the pay-as-you-go overage so a busy month doesn't stall your pipeline. Beyond that, Professional and Advanced are straightforward per-credit scaling decisions.

One last check before you buy: run two or three of your real target URLs through the cost estimator with the exact parameters you plan to use. The difference between a 1-credit site and a 25-credit one is the difference between Hobby and Startup, and it takes two minutes to find out which kind you're dealing with. 👉 [Compare all ScraperAPI plans and pricing](https://scraperapi.com/pricing?fp_ref=coupons)
