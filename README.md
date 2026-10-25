<p align="center"><a href="https://apify.com/DevilScrapes/google-ads-transparency"><img src="assets/banner.svg" alt="Google Ads Transparency Scraper — Advertiser Ad Library Export — JavaScript example" width="100%"></a></p>

# Google Ads Transparency Scraper for Node.js: ad library data in your backend

<p align="center">
<a href="https://apify.com/DevilScrapes/google-ads-transparency"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="JavaScript" src="https://img.shields.io/badge/Node.js-20%2B-339933?logo=nodedotjs&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/google-ads-transparency-scraper-javascript/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/google-ads-transparency-scraper-javascript/actions/workflows/ci.yml/badge.svg"></a>
</p>

This repository shows how to call the **[Google Ads Transparency Scraper](https://apify.com/DevilScrapes/google-ads-transparency)** from Node.js with the official `apify-client` package. A service of yours sends a list of competitor domains or advertiser IDs; Apify fetches the matching creatives from Google's Ads Transparency Center and hands back plain JSON objects. Each object identifies the advertiser and the creative, the landing domain, the dates Google showed the ad and preview links where Google offers them. It is a plain ES module, so the core call drops into an Express route or a serverless handler as is.

> **Actor:** [Google Ads Transparency Scraper — Advertiser Ad Library Export](https://apify.com/DevilScrapes/google-ads-transparency) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in Python:** [google-ads-transparency-scraper-python](https://github.com/devilscrapes/google-ads-transparency-scraper-python)

## 🔥 What you get

- **Two ways to name a target**: a landing domain such as `nike.com`, or an advertiser ID starting with `AR` when you already know exactly which account you want. Both lists can go into the same run.
- **A creative appears once per run**: when two of your targets surface the same `creative_id`, the repeat is dropped before it is written or billed.
- **Fetching stops at your budget**: with `maxResults` set to 10, the Actor requests pages only until ten creatives are in hand rather than paging to the end and discarding the rest.
- **Screening mode for long domain lists**: set `screeningMode` to `true` and each target collapses into a single summary row with an ad count, the earliest and latest dates seen and one sample creative link.
- **Base fee only after Google replies**: the $0.20 start event is charged once a search gets an answer, even an answer with zero ads. A run that never gets one ends FAILED and costs nothing.
- **Failures stay local**: a refused first page is retried with backoff on a new proxy session, a failure on a later page keeps the creatives already collected, and a target that still fails is skipped while the rest finish.

## 🎯 Use cases

### Power an ad intelligence page in your product

Expose an endpoint that accepts a competitor's domain, starts the Actor and returns the creatives to the browser. Each item already holds a link into the Transparency Center, so a front end can render a gallery with a 'view on Google' button and no extra lookups.

### Post new competitor ads to a team channel

Run `node index.js` on a schedule, keep the previous export and compare `creative_id` values. Anything new goes to Slack or email with the advertiser, landing domain and first shown date, which gives a paid search team a daily heads-up without anyone refreshing a web page.

### Qualify leads by whether they buy Google ads

Sales teams selling to advertisers can feed a prospect list through `screeningMode` and write `has_ads_detected` and `ad_count` back onto each CRM record. One summary row per domain keeps the cost proportional to the number of prospects instead of the number of ads.

### Handle large batches with webhooks

Serverless functions should not sit waiting on a big run. Start it with `start()`, register an Apify webhook for the succeeded event, and let a second function read the dataset when the webhook fires.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/google-ads-transparency-scraper-javascript.git
cd google-ads-transparency-scraper-javascript
npm install
export APIFY_TOKEN=your_token_here
node index.js                       # uses input.json
node index.js my-input.json --out data --max-charge 2
```

The script starts the [Google Ads Transparency Scraper — Advertiser Ad Library Export](https://apify.com/DevilScrapes/google-ads-transparency), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

The full integration fits in four statements. `call()` resolves once the run is finished, and `listItems()` reads the creatives out of the run's default dataset.

```javascript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: 'YOUR_APIFY_TOKEN' });
const run = await client.actor('DevilScrapes/google-ads-transparency').call({"searchDomains": ["nike.com"], "maxResults": 10, "maxPages": 1, "proxyConfiguration": {"useApifyProxy": true}});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items);
```

## 📥 Input

`index.js` sends `input.json` to the Actor unchanged. The example looks up one brand domain, limits the run to a single results page and ten creatives, which keeps a smoke test fast and cheap. Supply `advertiserIds` alongside or instead of domains when you already know the exact account.

```json
{
  "searchDomains": [
    "nike.com"
  ],
  "maxResults": 10,
  "maxPages": 1,
  "proxyConfiguration": {
    "useApifyProxy": true
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `searchDomains` | array | no | `["nike.com"]` | — |
| `advertiserIds` | array | no | `[]` | — |
| `region` | string | no | `"anywhere"` | `anywhere`, `AE`, `AR`, `AT`, `AU`, `BE`, `BG`, `BR`, `CA`, `CH`, `CL`, `CO`, `CZ`, `DE`, `DK`, `EE`, `EG`, `ES`, `FI`, `FR`, `GB`, `GR`, `HU`, `ID`, `IE`, `IL`, `IN`, `IT`, `JP`, `KR`, `LT`, `LV`, `MX`, `MY`, `NL`, `NO`, `NZ`, `PH`, `PL`, `PT`, `RO`, `SA`, `SE`, `SG`, `SK`, `TH`, `TR`, `UA`, `US`, `VN`, `ZA` |
| `dateFrom` | string | no | — | — |
| `dateTo` | string | no | — | — |
| `maxResults` | integer | no | `1000` | — |
| `maxPages` | integer | no | `25` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true}` | — |
| `screeningMode` | boolean | no | `false` | — |

## 📤 Output

Default runs yield one object per creative. Timestamps are integers in seconds since the epoch, so multiply by 1,000 before passing them to `Date`. Any field Google leaves out comes back as `null`, which makes optional chaining a sensible habit on `impressions` and the two preview URLs.

| Field | Type | Always present |
|---|---|---|
| `advertiser_id` | string | no |
| `advertiser_name` | string | no |
| `creative_id` | string | no |
| `creative_url` | string | no |
| `landing_domain` | string | no |
| `format_type` | integer or null | no |
| `first_shown_ts` | integer or null | no |
| `last_shown_ts` | integer or null | no |
| `impressions` | integer or null | no |
| `preview_image_url` | string or null | no |
| `preview_content_js_url` | string or null | no |
| `region` | string | no |
| `scraped_at` | string | no |
| `domain` | string or null | no |
| `has_ads_detected` | boolean or null | no |
| `ad_count` | integer or null | no |
| `first_seen_ts` | integer or null | no |
| `last_seen_ts` | integer or null | no |
| `sample_creative_url` | string or null | no |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Ad creative scraped | $0.003 |

Prices as configured on 2026-10-16. The [Actor page](https://apify.com/DevilScrapes/google-ads-transparency) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in JavaScript

This script compares today's `results.json` with a `previous.json` you saved last time and lists creatives that were not there before, newest first. It relies only on Node.js built-ins and treats a missing previous file as an empty baseline.

```javascript
import { readFile } from 'node:fs/promises';

const load = async (path) => {
    try {
        return JSON.parse(await readFile(path, 'utf8'));
    } catch {
        return [];
    }
};

const today = await load('results.json');
const previous = await load('previous.json');
const known = new Set(previous.map((ad) => ad.creative_id));
const isoDay = (seconds) => (seconds ? new Date(seconds * 1000).toISOString().slice(0, 10) : 'unknown');

const fresh = today
    .filter((ad) => !known.has(ad.creative_id))
    .sort((a, b) => (b.first_shown_ts ?? 0) - (a.first_shown_ts ?? 0))
    .map((ad) => ({
        advertiser: ad.advertiser_name,
        domain: ad.landing_domain,
        firstShown: isoDay(ad.first_shown_ts),
        link: ad.creative_url,
    }));

console.log(`${fresh.length} creatives not seen in the previous export`);
console.table(fresh.slice(0, 25));
```

## ❓ FAQ

### Will this work from TypeScript?

Yes. `apify-client` includes its own type definitions, so the run object and dataset calls are typed. Declaring a small interface for a creative row gives you autocompletion on `creative_id`, `landing_domain` and the timestamp fields.

### How do I keep a broken input from costing money?

Inputs are checked before anything is charged. A run with no domains and no advertiser IDs, or an unknown region code, is rejected up front without a start fee, so a bug in your request builder shows up as a failed run rather than a bill.

### What does my code see when a domain has no ads?

In the default mode the run succeeds and that domain contributes nothing. In screening mode it gets an explicit row with `has_ads_detected` set to `false`. A domain whose search failed gets no row at all, so a missing row should be read as 'not checked'.

### How do I cap spending from a backend job?

Pass a limit with `--max-charge`, or set `maxTotalChargeUsd` in the `call()` options. When the cap is reached, the Actor stops fetching instead of collecting creatives it would not be allowed to charge for.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/google-ads-transparency) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [Google Ads Transparency Scraper — Advertiser Ad Library Export on Apify Store](https://apify.com/DevilScrapes/google-ads-transparency)
- [Python version of this example](https://github.com/devilscrapes/google-ads-transparency-scraper-python)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Meta Ad Library Scraper — Ad Search & Export](https://apify.com/DevilScrapes/meta-ad-library-scraper) — the same competitor question asked of Facebook and Instagram ads.
- [TikTok Ad Library Scraper](https://apify.com/DevilScrapes/tiktok-ad-library-scraper) — ads from TikTok's commercial content library.
- [Apple Ad Repository Scraper](https://apify.com/DevilScrapes/apple-ad-repository-scraper) — App Store search ads from Apple's ad repository.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/google-ads-transparency) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with Google or Alphabet.
