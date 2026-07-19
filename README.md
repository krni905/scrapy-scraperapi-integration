# How to Use ScraperAPI with Scrapy: The Complete Step-by-Step Integration Guide — API Endpoint, SDK, Proxy Mode, Custom Headers, and Settings Explained (Plus Full Plan Comparison and Free Trial Details)

If you've been building Scrapy spiders for a while, you know the drill. Things start beautifully — you write a clean spider, kick it off, and watch the data roll in. Then somewhere around request 200, the target site starts throwing 403s. You rotate a proxy. They block the proxy. You try a different user agent. They block that too. You stare at your terminal for a moment, suddenly aware that you've spent more time fighting anti-bot systems than actually getting data.

That's the exact problem ScraperAPI was built to solve. Instead of you managing a rotating proxy pool, handling CAPTCHAs, and tuning browser headers on your own, you just route your Scrapy requests through ScraperAPI's endpoint — and it deals with all of that on your behalf. This guide walks through exactly how to plug ScraperAPI into a Scrapy spider, from the simplest one-liner integration all the way to configuring your settings.py for production-scale scraping.

---

**Why Scrapy Needs a Managed Proxy Solution:**

Scrapy is genuinely excellent at the *scraping* part. It handles concurrency, retries, pipelines, and middlewares in a way that's hard to beat for structured data collection at scale. What it doesn't handle on its own is the part that websites actively fight back against: being identified as a bot.

The three things that get Scrapy spiders blocked fastest are predictable IP patterns, browser fingerprint inconsistencies, and failing CAPTCHA challenges. Maintaining your own rotating proxy pool for each of these is a project in itself. You need to source IPs, monitor their health, swap out burned proxies, and still handle the CAPTCHA side separately.

ScraperAPI short-circuits all of that. You make one request to their API, and they handle the routing through a pool of 40+ million IPs across 50+ countries, auto-rotate headers, solve CAPTCHAs, and retry failed requests before returning a clean response to your spider. You pay only for successful requests — a detail that matters a lot when you're sending volume.

👉 [Start a free ScraperAPI trial — 5,000 credits, no credit card required](https://www.scraperapi.com/?fp_ref=coupons)

---

**Method 1: API Endpoint Integration — The Quickest Route:**

This is the fastest way to get ScraperAPI working with an existing Scrapy spider. All you're doing is wrapping your target URL inside a ScraperAPI endpoint request before Scrapy sends it.

Here's a typical Scrapy spider before ScraperAPI:

python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'https://quotes.toscrape.com/page/1/',
            'https://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)


To route requests through ScraperAPI, you add a small helper function that reformats the URL as an API call:

python
import scrapy
from urllib.parse import urlencode

API_KEY = 'YOUR_API_KEY'

def get_scraperapi_url(url):
    payload = {'api_key': API_KEY, 'url': url}
    proxy_url = 'https://api.scraperapi.com/?' + urlencode(payload)
    return proxy_url

class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'https://quotes.toscrape.com/page/1/',
            'https://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=get_scraperapi_url(url), callback=self.parse)


That's really it. The spider's `parse` method doesn't change. The rest of your pipeline doesn't change. You just swapped out where the request goes, and ScraperAPI handles the rest.

---

**Method 2: Using the Python SDK — Cleaner and Less Boilerplate:**

If you'd rather not manage the helper function yourself, ScraperAPI ships a Python SDK that wraps it up cleanly. Install it first:

bash
pip install scraperapi-sdk


Then import and use it in your spider like this:

python
import scrapy
from scraper_api import ScraperAPIClient

client = ScraperAPIClient('YOUR_API_KEY')

class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'https://quotes.toscrape.com/page/1/',
            'https://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(client.scrapyGet(url=url), callback=self.parse)


The `client.scrapyGet()` method builds the formatted API URL for you — no payload dictionary, no manual URL encoding. For projects where you're integrating across multiple spiders or files, keeping a single `client` instance and calling `scrapyGet` wherever you need it is the tidier approach.

---

**Method 3: Proxy Mode — Drop-In Replacement for Existing Proxy Setups:**

If your spider already has proxy configuration baked in — maybe you've been running your own proxy pool and you just want to swap it out for ScraperAPI — the proxy mode integration requires the least code change.

Instead of restructuring your requests, you just point Scrapy's proxy `meta` parameter at ScraperAPI's proxy server address:

python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'https://quotes.toscrape.com/page/1/',
            'https://quotes.toscrape.com/page/2/',
        ]

        meta = {
            "proxy": "https://scraperapi:YOUR_API_KEY@proxy-server.scraperapi.com:8001"
        }

        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse, meta=meta)


One note: Scrapy skips SSL verification by default, so you don't need to add any extra certificate configuration for this to work. It just drops in.

---

**Advanced Parameters: Getting More Out of ScraperAPI:**

The three methods above get you routing through ScraperAPI. Where things get more interesting is the optional parameters that let you customize what the API does with each request.

You pass these as additional keys in the payload dictionary when using the API endpoint method:

python
def get_scraperapi_url(url):
    payload = {
        'api_key': API_KEY,
        'url': url,
        'render': 'true',          # Execute JavaScript before returning HTML
        'country_code': 'us',      # Use proxies from a specific country
        'premium': 'true',         # Route through residential/mobile IPs
        'session_number': '123',   # Reuse the same proxy across requests
        'keep_headers': 'true',    # Pass your own custom headers through
        'device_type': 'mobile',   # Spoof mobile user agent
    }
    proxy_url = 'https://api.scraperapi.com/?' + urlencode(payload)
    return proxy_url


A few of these deserve a closer look:

- **`render=true`** is essential for JavaScript-heavy pages — single-page applications, sites that load content via API calls after the initial page load, infinite scroll. Without rendering, Scrapy would just get the bare HTML skeleton.
- **`premium=true`** routes your requests through residential and mobile IPs instead of datacenter IPs. These are significantly harder to detect as scrapers, but they cost 10 additional credits per request — so use them selectively.
- **`country_code`** lets you target specific geographic proxy pools. Useful for scraping localized content or pricing that varies by region. Worth noting: this is only available globally on Business plans and above; Hobby and Startup are limited to US and EU proxies.
- **`session_number`** lets you pin a request sequence to the same IP — useful when the target site expects session continuity across multiple pages.

If you're sending your own custom headers, combine `keep_headers=true` with your normal Scrapy `headers` parameter:

python
headers = {
    "X-MyHeader": "123",
    "Accept-Language": "en-US"
}

yield scrapy.Request(
    url=get_scraperapi_url(url),
    headers=headers,
    callback=self.parse
)


---

**Configuring settings.py for Production Use:**

Here's something that trips people up: you can have the integration working perfectly in your spider code, but still be leaving performance on the table because your `settings.py` isn't tuned to match your ScraperAPI plan.

The three settings that matter most are concurrency, retries, and robots.txt compliance.

**Concurrency** is the big one. Each ScraperAPI plan comes with a thread limit, and Scrapy's default `CONCURRENT_REQUESTS` is set conservatively — lower than what most paid ScraperAPI plans allow. If you're on a Startup plan with 50 concurrent threads and Scrapy is only sending 16 requests at a time (its default), you're paying for capacity you're not using.

Set this to match your plan:

python
# settings.py

ROBOTSTXT_OBEY = False  # Required — prevents request interference with the API

CONCURRENT_REQUESTS = 5    # Free Plan
# CONCURRENT_REQUESTS = 20   # Hobby Plan
# CONCURRENT_REQUESTS = 50   # Startup Plan
# CONCURRENT_REQUESTS = 100  # Business Plan
# CONCURRENT_REQUESTS = 200  # Scaling Plan

RETRY_TIMES = 3  # ScraperAPI returns 500 on failures and doesn't charge for them

# Disable these — they reduce concurrency and aren't needed with ScraperAPI
# DOWNLOAD_DELAY = 1
# RANDOMIZE_DOWNLOAD_DELAY = True


The `ROBOTSTXT_OBEY = False` line is required when using the API endpoint method. Without it, Scrapy will try to fetch the robots.txt from ScraperAPI's endpoint URL rather than the target site, which causes issues.

`RETRY_TIMES = 3` is worth setting because ScraperAPI returns HTTP 500 on failed requests and doesn't charge you for those failures. Scrapy's built-in retry middleware catches 500s and retries automatically — and over 97% of requests succeed on the first try, with virtually all of the rest succeeding within 3 attempts.

---

**The Scrapy Middleware Option:**

Beyond the three main integration methods, ScraperAPI also maintains an official Scrapy downloader middleware package for teams that prefer to configure the integration at the middleware level rather than in spider code:

python
# settings.py with scrapy-scraperapi-proxy middleware

SCRAPERAPI_ENABLED = True
SCRAPERAPI_URL = 'api.scraperapi.com'
SCRAPERAPI_KEY = 'your_api_key'

DOWNLOADER_MIDDLEWARES = {
    'scrapy_scraperapi.ScraperApiProxy': 610,
}


Install it first with `python setup.py install` after cloning the official repo. This approach is useful when you want all requests across a project to route through ScraperAPI by default, without modifying individual spider files.

---

**Understanding Credit Costs Before You Scale:**

One thing that catches people off guard: not every request costs 1 credit. The base rate applies to standard, unprotected pages — but the actual cost per request scales with the target site and any parameters you add.

| Target / Parameter | Credit Cost |
|---|---|
| Standard page (basic HTML) | 1 credit |
| Amazon | 5 credits |
| Google / Bing (and subdomains) | 25 credits |
| LinkedIn | 30 credits |
| Cloudflare / Datadome bypass | +10 credits |
| `render=true` (JS rendering) | +10 credits |
| `premium=true` (residential IPs) | +10 credits |
| `screenshot=true` | +10 credits |
| `ultra_premium=true` | +30 credits |
| `premium=true` + `render=true` | 25 credits total |
| `ultra_premium=true` + `render=true` | 75 credits total |

The practical implication: if you're scraping Amazon with JavaScript rendering enabled, that's 5 (domain) + 10 (render) = 15 credits per request. A 100,000-credit Hobby plan gets you roughly 6,600 Amazon scrapes, not 100,000. The ScraperAPI dashboard has a URL Cost Estimator tool that lets you check the exact credit cost for any target URL before you run jobs at scale. Use it.

On the upside: you're **only charged for successful requests**. If ScraperAPI can't get a clean response, it returns a 500 and doesn't burn your credits. This matters a lot at volume.

---

**Full ScraperAPI Plan Comparison:**

ScraperAPI runs across several tiers from a free trial entry point up to enterprise contracts. All paid plans include JS rendering, rotating proxy pools, CAPTCHA bypass, custom headers, session control, automatic retries, unlimited bandwidth, and a 99.9% uptime SLA. The differences across tiers are volume, concurrency, and geotargeting scope.

| Plan | Monthly Price | Annual Price | API Credits/Month | Concurrent Threads | Geotargeting | Get Started |
|---|---|---|---|---|---|---|
| **Free Trial** | $0 (7 days) | — | 5,000 one-time | 5 | Basic |  [Start Free Trial](https://www.scraperapi.com/?fp_ref=coupons) |
| **Hobby** | $49/mo | $44.10/mo | 100,000 | 20 | US & EU only |  [Get Hobby Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Startup** | $149/mo | $134.10/mo | 1,000,000 | 50 | US & EU only |  [Get Startup Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Business** | $299/mo | $269.10/mo | 3,000,000 | 100 | Global |  [Get Business Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Scaling** | $475/mo | $427.50/mo | 5,000,000 | 200 | Global |  [Get Scaling Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Professional** | $975/mo | $877.50/mo | 10,500,000 | 300 | Global |  [Get Professional Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Advanced** | $1,975/mo | $1,777.50/mo | 21,500,000 | 500 | Global |  [Get Advanced Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| **Enterprise** | Custom | Custom | 22M+ | 500+ | Global |  [Contact Sales](https://www.scraperapi.com/?fp_ref=coupons) |

A few specifics worth flagging:

- **Geotargeting is gated.** Hobby and Startup plans are limited to US and EU proxy pools only. If your scraping targets require country-specific IPs outside those regions, you need Business or above.
- **Pay-as-you-go overflow** is only available from Scaling upward. On Hobby, Startup, and Business, hitting your monthly credit limit means either upgrading or pausing until renewal.
- **Analytics history** is capped at 30 days for Hobby and Startup; Business and above get unlimited dashboard history.
- **Annual billing saves 10%** across all plans — it's applied automatically at checkout, no coupon code needed.
- **Credits don't roll over.** They reset at each billing cycle, so it's worth sizing your plan to actual monthly usage rather than buying headroom that goes unused.

👉 [Compare all plans and start your free trial here](https://www.scraperapi.com/?fp_ref=coupons)

---

**Which Plan Makes Sense for Scrapy Users:**

Here's a quick framework for thinking about plan selection based on what you're actually building:

If you're running a **personal project or prototype** — testing a scraper against a handful of product pages, building a price tracker for a side project, or just learning how Scrapy works in practice — the **free trial** gets you 5,000 credits with no card required. Once you've confirmed your credit burn rate per request, the **Hobby plan at $49/month** covers 100,000 standard page credits with 20 concurrent threads. For plain HTML pages without heavy anti-bot systems, that's a meaningful monthly volume.

If you're building something that other people depend on — **a small SaaS, an internal data pipeline, an agency workflow** — the **Startup plan at $149/month** is where sustained scraping starts making economic sense. One million credits with 50 concurrent threads is a real production-grade setup, though you're still on US/EU proxies only.

If you need **global proxy coverage** or you're scraping sites that require geolocation flexibility, the **Business plan at $299/month** is the first tier that unlocks global geotargeting. It also enables unlimited analytics history in your dashboard, which matters once you're diagnosing performance across long-running jobs.

The **Scaling plan and above** are built around teams that have outgrown "which plan should I pick" and are instead managing predictable high-volume workloads — they add PAYG overflow billing so you're never hard-stopped mid-month, and priority support starts at the Professional tier.

---

**What Developers Actually Say About Using ScraperAPI with Scrapy:**

The consistent feedback across Trustpilot (around 4.5/5), G2 (around 4.4/5), and developer forums is that ScraperAPI's documentation is genuinely good — unusual enough to be worth mentioning — and that the integration path for Scrapy specifically is straightforward. Most developers report being able to get a working integration in under an hour, with the main learning curve being the credit cost multipliers for different target domains rather than anything technical.

The most common complaint isn't reliability — it's the gap between the headline credit number and the real effective page count once you factor in domain-based multipliers. Developers scraping Amazon or Google who don't check the credit cost per request before choosing a plan tend to be surprised. Running the dashboard's cost estimator before committing to a plan is the piece of advice that shows up consistently across independent reviews.

---

**Common Integration Mistakes and How to Avoid Them:**

A few things that trip people up when first connecting Scrapy to ScraperAPI:

**Forgetting to disable `ROBOTSTXT_OBEY`** — when you use the API endpoint method, Scrapy's robots.txt fetcher will try to pull `robots.txt` from `api.scraperapi.com` instead of your actual target site. That causes unexpected behavior. Set `ROBOTSTXT_OBEY = False` in `settings.py`.

**Leaving `DOWNLOAD_DELAY` active** — Scrapy's download delay is designed to be polite to target servers and is a good default for direct scraping. But when you're routing through ScraperAPI, the API handles rate limiting and request spacing on its end. Having `DOWNLOAD_DELAY` enabled as well just slows you down unnecessarily without adding any benefit.

**Not matching `CONCURRENT_REQUESTS` to your plan limit** — ScraperAPI is built for concurrency. If you're on a Business plan with 100 threads available but Scrapy is only sending 16 concurrent requests (the default), you're getting less than a fifth of what you're paying for. Check your plan's thread limit and set `CONCURRENT_REQUESTS` to match.

**Not setting `RETRY_TIMES`** — ScraperAPI's own infrastructure handles a lot of retries before sending back a 500, but occasionally some do come through. Scrapy's retry middleware handles these cleanly, and since failed requests don't cost credits, retrying is essentially free.

---

**A Practical Starting Template:**

Putting everything together, here's what a clean starting Scrapy spider looks like with ScraperAPI integrated via the API endpoint method, with production settings:

python
# spider.py
import scrapy
from urllib.parse import urlencode

API_KEY = 'YOUR_API_KEY'

def get_scraperapi_url(url, **kwargs):
    payload = {'api_key': API_KEY, 'url': url, **kwargs}
    return 'https://api.scraperapi.com/?' + urlencode(payload)

class MySpider(scrapy.Spider):
    name = "my_spider"

    def start_requests(self):
        urls = [
            'https://example.com/page/1',
            'https://example.com/page/2',
        ]
        for url in urls:
            yield scrapy.Request(
                url=get_scraperapi_url(url, render='true'),
                callback=self.parse
            )

    def parse(self, response):
        # Your parsing logic here
        pass


python
# settings.py
ROBOTSTXT_OBEY = False
CONCURRENT_REQUESTS = 20   # Adjust to match your plan
RETRY_TIMES = 3
# DOWNLOAD_DELAY = 0       # Leave commented out
# RANDOMIZE_DOWNLOAD_DELAY = False


That's a clean, production-ready starting point. Adjust `CONCURRENT_REQUESTS` and the optional API parameters based on your plan and your targets.

---

**Getting Started:**

The fastest path from reading this to running a working integration is to grab a free API key first. ScraperAPI's 7-day trial gives you 5,000 credits with no credit card required — enough to point your spider at real targets and see exactly how many credits each request type costs before you pick a paid plan.

👉 [Start your free ScraperAPI trial — 5,000 credits, no card required](https://www.scraperapi.com/?fp_ref=coupons)

Once you have your API key, you can drop it into any of the three integration methods above and have requests routing through ScraperAPI in minutes. The rest is just tuning your `settings.py` to match your plan and target profile — and actually getting the data you came for.
