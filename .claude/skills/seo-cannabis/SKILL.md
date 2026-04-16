---
name: seo-cannabis
description: SEO strategy, schema markup, content patterns, and link-building tactics specific to hemp/CBD/THCA/cannabis e-commerce — a vertical where Google Ads, Meta Ads, and TikTok Ads are banned, making organic search THE primary growth channel. Use this skill whenever the user mentions SEO, search rankings, content marketing, schema.org, site structure, or organic growth for a cannabis-adjacent project. Also trigger for blog planning, keyword research, PDP optimization, or any marketing strategy for The PotFather. Critical: since paid ads are off-limits for this vertical, SEO is not a "nice to have" — it is the business's lifeline.
---

# SEO for Hemp / Cannabis E-commerce

Cannabis-adjacent brands are locked out of Google Ads, Meta Ads, and TikTok Ads. Organic search is your oxygen. This skill captures what actually works in the vertical.

## Channel Reality Check

| Channel | Status for hemp/THCA |
|---------|----------------------|
| Google Search (organic) | ✅ Primary growth engine |
| Google Ads | ❌ Banned |
| Meta Ads (FB/IG) | ❌ Banned |
| TikTok Ads | ❌ Banned |
| YouTube organic | ✅ Allowed, monetization limited |
| Email (Klaviyo/Postscript) | ✅ Primary retention |
| SMS (Postscript) | ✅ Allowed with compliance |
| Reddit organic | ✅ Community-driven |
| Affiliate / influencer | ✅ With FTC disclosure |
| Programmatic (Mantis, Fyllo) | ✅ Cannabis-specialized networks |

Implication: everything in SEO matters more than in a normal e-commerce vertical.

## Site Architecture for SEO

```
/                       → Homepage (brand terms + top category)
/strains/               → THCA strain category
/strains/[slug]/        → Individual strain/product (PDP)
/concentrates/          → Concentrates category
/concentrates/[slug]/   → PDP
/edibles/
/mystery-box/           → Subscription landing
/the-ledger/            → Blog (editorial name)
/the-ledger/[slug]/     → Blog posts
/ledger/category/[slug] → Content categories
/coa/                   → COA library (schema + compliance trust)
/coa/[batch]            → Per-batch lab reports
/faq/
/shipping-info/
/legal/                 → Farm Bill, state restrictions, privacy, tos
/contact/
```

Architectural rules:
- Flat URL hierarchy (max 3 levels deep)
- No trailing slashes OR consistent trailing slashes — pick one, stick with it
- 301-redirect old URLs on any restructure
- Breadcrumbs on every non-home page with schema
- HTML sitemap at `/sitemap` in addition to XML sitemap

## Core Technical SEO Checklist

- [ ] `robots.txt` allows crawling, disallows `/cart`, `/checkout`, `/account/*`, `/api/*`
- [ ] XML sitemap at `/sitemap.xml` submitted to Search Console
- [ ] Page speed: LCP <2.5s, INP <200ms, CLS <0.1 (measure in Search Console Core Web Vitals report)
- [ ] Mobile-first responsive
- [ ] HTTPS with HSTS
- [ ] Canonical tags on every page
- [ ] Meta title ≤60 chars, meta description ≤155 chars, unique per page
- [ ] Open Graph + Twitter Card metadata (for when pages are shared)
- [ ] Structured data validated in Rich Results Test
- [ ] 404 page helpful (search bar, category links)
- [ ] No orphan pages (every page reachable via nav or internal link)
- [ ] `hreflang` if multi-region (not initially)

## Schema.org Markup (Critical)

Hemp sites should implement these schema types with meticulous care — rich results dramatically boost CTR in a vertical where paid ads can't compete.

### Product schema (on every PDP)
```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Zkittlez THCA Flower - Indoor",
  "image": ["https://thepotfather.com/images/zkittlez-1.jpg"],
  "description": "Indoor-grown Zkittlez THCA flower, 27.4% total THC...",
  "sku": "TPF-ZKT-001",
  "brand": { "@type": "Brand", "name": "The PotFather" },
  "offers": {
    "@type": "Offer",
    "url": "https://thepotfather.com/strains/zkittlez-thca",
    "priceCurrency": "USD",
    "price": "45.00",
    "availability": "https://schema.org/InStock",
    "itemCondition": "https://schema.org/NewCondition"
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.7",
    "reviewCount": "134"
  }
}
```

### BreadcrumbList on category + PDP
### Organization schema on homepage with `sameAs` links to all social profiles
### FAQPage schema on FAQ page AND on long-tail content pages
### Article schema on blog posts with author, datePublished, image
### LocalBusiness (if you add physical retail later)

## Keyword Strategy

### Tiers of intent

**Tier 1 — Transactional (highest value)**
- `buy thca flower online`
- `thca flower near me` (yes, even for online — people search this way)
- `best delta 8 gummies`
- `thca pre rolls`
- `moonrock for sale online`

**Tier 2 — Commercial Investigation**
- `thca vs delta 9`
- `is thca legal in [state]`
- `best thca strains 2026`
- `live rosin vs badder`

**Tier 3 — Informational (top of funnel, long-term)**
- `what is thca`
- `how does delta 8 make you feel`
- `terpene guide`
- `how to dab concentrates`

### Keyword tool workflow

1. Seed terms from product catalog (every strain name, every concentrate type)
2. Expand with Ahrefs/SEMrush (or free: Ubersuggest, Google autocomplete, AlsoAsked)
3. Cluster by intent: transactional → category/PDP; investigation → comparison content; informational → blog
4. Prioritize by: volume × (1 - KD) × intent-weight. For hemp, lower volume keywords often convert better (less crowded).

### Local + "near me" tactic

Even for online-only hemp, optimize for "[product] near me" style queries. These have massive volume and Google serves regular organic results when no local pack exists. Geo-targeted landing pages (`/thca-flower-california`, `/thca-flower-texas`) with state-specific legality copy rank well and convert.

**BUT**: don't create pages for blocked states. Do not rank for "buy thca in Idaho" — that's a legal exposure.

## Content Engine (The Ledger)

The blog should be called something on-brand. For PotFather, "The Ledger" or "La Cosa" fits the Godfather theme.

### Content pillars
1. **Education**: "What is THCA?", "THCA vs Delta-9", terpene guides, extraction methods
2. **Strain reviews**: each strain gets a deep review post linking to PDP
3. **State-by-state legality**: one post per allowed state, updated quarterly
4. **Ritual / lifestyle**: pairing concentrates with evening rituals, etc.
5. **Science**: how cannabinoids work, entourage effect, terpenes

### Format that ranks

- Target length 1,500-2,500 words for pillar content, 800-1,200 for reviews
- H1 matches search intent EXACTLY, H2s hit related subtopics
- Include: FAQ section at end (FAQPage schema), comparison table, a short video embed (YouTube), internal links to 3-5 related products
- Author byline with E-E-A-T signals (real person, bio, credentials)
- Updated-at timestamp visible (Google weights freshness for this vertical)

### Topical authority strategy

Google trusts sites that demonstrate depth. Pick 3-5 core topic clusters, write 8-12 posts in each cluster with tight internal linking, THEN expand. Scattered posting on unrelated topics dilutes authority.

## On-Page SEO Per Template

### Homepage
- H1: Brand + 1 primary keyword ("The PotFather — Premium THCA & Hemp Concentrates")
- Above-fold: value prop + CTA to category
- Category grid (internal linking)
- Featured products (internal linking to PDPs)
- Trust signals (COA emphasis, lab partners, shipping policy)
- Internal link to The Ledger
- FAQ block with FAQPage schema

### Category (PLP)
- H1: "THCA Flower" (match the category term)
- Intro paragraph (150-250 words) with primary + LSI keywords before the product grid
- Filters usable (cannabinoid %, effect, price) — make filter URLs crawlable via canonicalization strategy
- Pagination with `rel="next"` / `rel="prev"` OR load-more (not both)

### PDP
- H1: product name
- Title tag: `{Product} — {Type} | The PotFather` (59 chars)
- Meta description: first 150 chars of optimized copy ending with CTA
- Product schema complete
- Reviews visible + schema
- Related products (internal linking, 4-8 items)
- Sticky "Add to Collection" on mobile
- COA link prominent (trust signal, E-E-A-T)
- "Pairs well with" cross-sell section

### Blog post
- H1 matches query intent
- TOC with jump links (improves dwell time)
- Internal links: 2-3 to relevant products, 2-3 to other posts
- External links: 1-2 authority sources (science journals, NCBI, Farm Bill text)
- Image with descriptive alt text
- Author bio at bottom
- Comment section OR "join the list" CTA

## Link Building for Cannabis

Hard vertical because mainstream outlets avoid linking. Tactics that work:

1. **Industry directories**: Leafly, Weedmaps, Wikileaf, AllBud — get listed even if hemp-only
2. **Cannabis media**: High Times, Marijuana Moment, Benzinga Cannabis — pitch data-driven stories
3. **Podcast guesting**: cannabis/wellness podcasts — each episode earns a backlink in show notes
4. **HARO / Qwoted / Featured**: respond to journalist queries with hemp expertise
5. **Strain databases**: submit strains with links to your PDP to AllBud, SeedFinder, Leafly
6. **Reddit**: organic value-first posting in r/THCA, r/cbd (strict rules, follow them)
7. **Scholarship / data study links**: publish an annual "State of THCA" report with original data, pitch to cannabis press
8. **Partnerships**: guest posts on complementary brands (glass, lighters, accessories)

## Google My Business

Even if online-only, claim your GMB with service-area business. Helps for "near me" queries from allowed states. DO NOT list products with federal trademark / prohibited terms — keep the listing on brand/services level.

## Email + SEO Loop

- Newsletter signups → build owned audience (the audience you can reach when Google updates)
- Promote top-performing blog posts in email to drive repeat visits + time-on-site signals
- Use email content to test which topics resonate before investing in full SEO posts

## Measurement

### Monitor weekly:
- Organic sessions (GA4 / Plausible)
- Top 20 keyword rankings (Ahrefs / SE Ranking)
- CTR per ranking page (Search Console)
- Core Web Vitals (Search Console)
- New backlinks (Ahrefs alerts)

### Monitor monthly:
- Conversion rate by landing page
- Revenue per organic visitor
- Topic cluster coverage (are any pillars missing content?)
- Competitor movement (who's eating your SERP?)

## Hemp-specific SEO Gotchas

1. **Google can sandbox new hemp domains** — expect 3-6 months before ranking meaningfully. Start SEO BEFORE you launch.
2. **Manual actions happen** — if a reviewer flags prohibited claims, you get a manual penalty. Audit content for FDA/medical claims monthly.
3. **Affiliate pages can hurt** if low quality — thin content on "best THCA gummies" pages pinging up with paid reviews gets de-indexed in updates.
4. **YMYL territory**: Google treats cannabis as "Your Money or Your Life" content, demanding higher E-E-A-T. Show author credentials, medical reviewer if possible, cite sources.
5. **Disambiguation**: "The PotFather" competes with parody/joke results. Own the SERP by claiming Wikipedia stub (if notable), LinkedIn, press releases, branded YouTube, Reddit, Quora, and dominating the first page for brand queries.

## Claude's behavior with this skill

- When writing any product/category/blog copy, apply on-page best practices automatically (title, meta, headings, schema hint).
- When asked for content ideas, cluster them by pillar and intent tier, don't just list.
- When generating technical SEO code, default to App Router metadata API patterns and JSON-LD in a `<script type="application/ld+json">`.
- Never suggest Google Ads, FB Ads, TikTok Ads as growth tactics for this vertical.
- Audit generated marketing content against the prohibited-claims list from the compliance skill before returning.
