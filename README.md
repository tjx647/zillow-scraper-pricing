# real estate scraper: how to get past Zillow's anti-bot, and what one rendered page actually costs you

Most "real estate scraper" searches end the same way: you find a GitHub repo or a no-code tool, run it once, and get a 403 from Zillow or Realtor.com within two seconds. That's not bad luck. Property portals sit behind some of the most aggressive bot protection on the web — Cloudflare, DataDome, device fingerprinting — and they load listings with JavaScript, so a plain HTTP request gets you an empty shell of a page.

This guide covers the three realistic routes (open-source repos, marketplace scrapers, and scraping APIs), what each costs, and the credit math that never shows up on a pricing page. If you want to skip the comparison, the short version:

> **Quick pick:** If you're scraping Zillow, Realtor.com, or Redfin at any real volume, a scraping API like ScraperAPI handles the proxies, JavaScript rendering, and CAPTCHAs for you. Plans start at **$49/month for 100,000 credits**, and there's a free trial with 5,000 credits that needs no credit card. 👉 [Check ScraperAPI's plans and start the free trial](https://www.scraperapi.com/?fp_ref=coupons)

## Why your real estate scraper keeps getting blocked

Because the data you want is behind JavaScript and anti-bot systems, and the page never sends it to plain HTTP clients.

Zillow loads its listings dynamically. In ScraperAPI's own tutorial, a naive `requests.get()` call returns markup without the property cards at all — the price, address, and bed/bath data only appear after a headless browser renders the page. On top of that, Zillow, Redfin, Realtor.com, and Rightmove all run bot detection that fingerprints your browser and IP. A datacenter IP without proper headers gets flagged fast.

Two things follow from this, and they matter more than which tool you pick:

1. **You need JavaScript rendering.** Any scraper that can't render JS will miss the listings entirely.
2. **You need clean IPs at volume.** Free proxies and your own connection die within a few pages.

One encouraging data point: independent benchmark tests that pitted multiple scraping APIs against hard targets have recorded a **100% success rate on Zillow specifically** for ScraperAPI (its results vary wildly by site — strong on e-commerce and real estate, weak elsewhere). Real estate happens to be one of the domains where it performs well.

## Three ways to build a real estate scraper

**Option 1: open-source repos on GitHub.** Free, and good for learning how listing pages are structured. The trade-off is that you own everything: proxies, rendering, rate limiting, and maintenance. Zillow's CSS classes include version numbers (the tutorial's selector for home details is `StyledPropertyCardHomeDetailsList-c11n-8-109-3__sc-1j0som5-0`), so when the site ships a frontend update, your selectors break and the repo — often a hobby project — may not get fixed. Several repos also note that Realtor.com requires US residential proxies, which you'd have to source and pay for separately.

**Option 2: marketplace scrapers like Apify actors.** Prebuilt scrapers for specific portals. Apify's Realtor.com scraper, for example, takes a city, ZIP, or search URL, pulls for-sale/for-rent/sold listings with prices, beds, baths, GPS coordinates, and agent info, and exports to CSV or Excel. Another actor adds price history, tax history, school data, flood risk, and agent contact enrichment, plus incremental mode that only returns new or changed listings. This is the fastest path if your target portal is covered. Costs run on Apify's platform credits (they give a $5 monthly allowance), and you're locked to whatever fields the actor author chose to extract.

**Option 3: a scraping API like ScraperAPI.** You send any URL to one endpoint; ScraperAPI routes it through a pool of 40M+ IPs across 50+ countries, renders JavaScript in headless Chrome, solves CAPTCHAs, and returns the full HTML. You bring your own parser (BeautifulSoup works fine) and export however you want. It's infrastructure rather than a finished scraper — you keep full control of fields and targets, at the cost of writing a few dozen lines of Python.

If your target is Zillow or Redfin and you want structured data without writing selectors at all, ScraperAPI also runs dedicated real estate data endpoints, including a Zillow scraper solution page. 👉 [See ScraperAPI's Zillow scraper for structured listing data](https://www.scraperapi.com/solutions/real-estate-data-collection/zillow-scraper/?fp_ref=coupons)

## ScraperAPI pricing: every plan on the official pricing page

| Plan | Monthly price | Credits/month | Concurrent threads | Geotargeting | Get it |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | 1,000 (5,000 with 7-day trial) | 5 | None | [Sign up free, no card needed](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 ($44.10 annual) | 100,000 | 20 | US & EU | [Start Hobby at $49/mo](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 ($134.10 annual) | 1,000,000 | 50 | US & EU | [Start Startup at $149/mo](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 ($269.10 annual) | 3,000,000 | 100 | 50+ countries | [Start Business at $299/mo](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 ($427.50 annual) | 5,000,000 | 200 | 50+ countries | [Start Scaling at $475/mo](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 ($877.50 annual) | 10,500,000 | 300 | 50+ countries | [Start Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 ($1,777.50 annual) | 21,500,000 | 500 | 50+ countries | [Start Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | 22,000,000+ | 500+ | 50+ countries | [Contact sales for Enterprise](https://www.scraperapi.com/?fp_ref=coupons) |

Every paid plan includes JavaScript rendering, premium residential and mobile IPs, automatic CAPTCHA handling, structured data APIs, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee. Annual billing takes 10% off automatically — no promo code — and the 7-day free trial upgrades you from 1,000 to 5,000 credits without asking for a card.

## The credit math nobody shows you: what one Zillow page really costs

This is where most pricing comparisons stop at "$49 for 100,000 credits" and leave you to find out the hard way that you won't get 100,000 pages.

ScraperAPI bills by credits, and the base rate is simple: a standard page costs 1 credit. But feature flags stack on top, and — this is the part buried in the technical docs — **the stacking is non-linear**:

| Feature flag | Extra credits |
| --- | --- |
| `render=true` (JavaScript rendering) | +10 |
| `premium=true` (residential proxies) | +10 |
| `ultra_premium=true` | +30 |
| Anti-bot bypass (Cloudflare, DataDome, PerimeterX) | +10 each |
| `premium=true` + `render=true` combined | **+25** (not +20) |
| `ultra_premium=true` + `render=true` combined | **+75** (not +40) |

For real estate work, the practical case looks like this: Zillow pages bill at the base 1 credit (they sit outside the 5-credit e-commerce tier for Amazon or the 25-credit search-engine tier for Google), but you realistically need JavaScript rendering, so a rendered property page costs around **11 credits**. That converts to real money like this:

| Plan | Base rate | Cost per rendered Zillow page (~11 credits) | Per 1,000 pages |
| --- | --- | --- | --- |
| Hobby ($49, 100K credits) | ~$0.00049/credit | ~$0.005 | ~$5.40 |
| Startup ($149, 1M credits) | ~$0.00015/credit | ~$0.0016 | ~$1.64 |
| Business ($299, 3M credits) | ~$0.0001/credit | ~$0.0011 | ~$1.10 |

So on Business, a 5,000-listing monthly pull across a few metros runs you roughly $5–6 in credits — the plan price buys far more headroom than most people assume once they do this math.

The other trap: Hobby's 100,000 credits can mean ~9,000 rendered Zillow pages — or, if you flip on `ultra_premium` for Cloudflare-protected pages, as few as ~1,333 requests. Same plan, same price, wildly different output. The dashboard has a Domain Cost Estimator that tells you the exact credit cost for any URL before you run it, and you can set a `max_cost` per request as a cap.

Two more limits worth knowing before you commit. Credits don't roll over — unused credits expire at the end of each billing cycle. And if you run out mid-month, Hobby through Professional pause by default; only Scaling ($475/month) and above get pay-as-you-go overflow at a fixed per-credit rate. If your listing volume spikes unpredictably, that alone can decide which tier you need.

## How to scrape Zillow with ScraperAPI in practice

The setup takes four Python libraries and about twenty lines: `requests` to call ScraperAPI, `beautifulsoup4` to parse, `pandas` to export CSV. Install with `pip install requests beautifulsoup4 pandas`.

The core pattern from ScraperAPI's official Zillow tutorial:

1. **Pass your API key in the header** (`x-sapi-api_key`) and enable rendering with `x-sapi-render: true`. This is what makes the listings appear.
2. **Use Zillow's URL structure for pagination.** City pages follow `zillow.com/houston-tx/`, and page 2, 3, 4 are `/houston-tx/2_p/`, `/3_p/`, and so on. Loop over the pages you need, with a `time.sleep()` delay between requests.
3. **Use a render instruction set** to mimic human behavior: wait for the price selector to appear, scroll to the bottom of the page, then wait a few seconds. Without this, you can render the page but snapshot it before the cards populate. Useful detail here: instruction parameters like `wait_for_selector`, `country_code`, and `device_type` don't add credits — only rendering, premium proxies, and anti-bot bypass do.
4. **Extract with data-test attributes, not classes.** Price lives in `span[data-test="property-card-price"]` and address in `address[data-test="property-card-addr"]`. These are more stable than the versioned class names like `c11n-8-109-3` that change when Zillow ships a frontend update.

The tutorial's demo pulls price, address, beds, baths, and square footage from the first five pages of Houston listings into a CSV. It's a working skeleton for market tracking, comp analysis, agent lead generation, or investment screening — the use cases ScraperAPI itself lists for real estate data.

If you'd rather validate the whole approach before paying anything, the free trial covers roughly 450 rendered Zillow pages — enough for a real test on one metro area, not just a toy request. 👉 [Open a free ScraperAPI account and run your first metro test](https://www.scraperapi.com/?fp_ref=coupons)

## Which plan fits your listing volume

Quick decision rules based on the credit math above:

- **Just testing, or scraping one city occasionally:** the free trial. Don't pay yet.
- **A few hundred to ~1,000 rendered pages per month** (a personal investment watchlist, one metro): Hobby at $49/month. That's ~9,000 rendered Zillow pages.
- **A few thousand pages monthly** (agent lead gen for a team, multi-city monitoring): Startup at $149/month gets you ~90,000 rendered pages — a 10x jump for 3x the price, the best per-page rate below Business.
- **Anything serious, or you need geotargeting beyond US/EU:** Business at $299/month. It's the first tier with 50+ country geotargeting, and at roughly $1.10 per thousand rendered pages the math starts making sense for ongoing pipelines.
- **Unpredictable spikes or 24/7 monitoring:** Scaling at $475/month or higher, because it's the first tier with pay-as-you-go overflow instead of a hard stop when credits run out.

One honest caveat about fit: ScraperAPI is general-purpose infrastructure with no click-to-run interface, so non-technical users will feel the friction. If you only need Realtor.com listings and want a prebuilt, no-code solution with incremental change tracking, an Apify actor does that with less setup, and its $5 monthly credit makes small runs nearly free. ScraperAPI's advantage shows up when you need any portal (not just the ones an actor covers), custom fields, your own parser, and room to scale — and Zillow is one of the targets where its benchmarks are genuinely strong.

## FAQ

**Is scraping Zillow or Realtor.com legal?** Publicly available listing data is generally scrapeable, but these sites' terms of service prohibit it, and enforcement varies. Stick to public pages, keep request rates reasonable, and never scrape behind a login — and for commercial projects, get proper legal counsel.

**Do I need proxies on top of ScraperAPI?** No. Premium residential and mobile IPs are included in every plan, including the free trial. That's one of the main reasons to use an API instead of building your own stack, where residential proxies would be a separate monthly bill.

**What happens if I exceed my credits or concurrent threads?** Exceeding your concurrent limit returns a 429 response — the request gets rejected without charge, and nothing queues. Running out of credits pauses scraping until renewal unless you've enabled auto-upgrade (Hobby through Professional) or pay-as-you-go (Scaling and up).

**Can I scrape Redfin and Realtor.com the same way as Zillow?** Yes — ScraperAPI works on any URL. Redfin also has dedicated structured data endpoints (search, agent data, and more) on higher plans, so you can skip HTML parsing entirely there.

## Before you check out

Match the plan to your real monthly page count, not your aspirational one — the cost table above tells you exactly what each tier buys in rendered Zillow pages, and the trial makes the first number free to verify.

Three things to confirm before paying: run the Domain Cost Estimator on one of your target URLs so you know the true per-page credit cost, check whether your volume fits inside the plan's credits or needs pay-as-you-go overflow (only Scaling and up have it), and remember credits expire each cycle, so don't pre-buy a year of a tier you'll outgrow in a month. If the numbers check out, 👉 [lock in annual billing on ScraperAPI and save 10%](https://www.scraperapi.com/pricing/?fp_ref=coupons) — the discount applies automatically, and the free trial is still yours to burn through first.
