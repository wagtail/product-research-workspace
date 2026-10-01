---
tags:
  - agent-readiness
---

# Findings

What the five [checkers](checkers.md) reported on [guide.wagtail.org](https://guide.wagtail.org/), cross-referenced against direct measurement.

## What the guide already does well

These are worth stating first, because no single checker score reflects them. They group into three layers, roughly in the order an agent encounters them.

### Access and rendering

The two changes the AgentReady studies found actually broke task completion — hiding content behind JavaScript, and blocking access — are both absent. The guide is fully server-rendered, so a fetch-only client receives the complete article text without executing JavaScript, and `robots.txt` is a single permissive rule with no AI crawler blocked.

Nor does what a crawler receives differ from what a person receives. The same page fetched under 20 crawler user agents, a browser, a bare `curl`, and no user agent at all returned identical HTML across all 23 once the per-request CSP nonce and CSRF token were normalized — and the Markdown endpoint was **byte-identical across all 23 with no normalization at all**.

### Machine-readable content

All 45 sitemap pages have a working `/markdown/` endpoint, and `llms.txt` and `llms-full.txt` are complete and spec-conformant. The Markdown is a small fraction of the HTML it replaces: the largest page measured is 65,586 bytes as HTML against 15,807 as Markdown.

Sitemap `lastmod` matches Markdown frontmatter `last_modified` on 45 of 45 pages, with no drift, providing a consistent freshness signal across the two representations.

### Discovery and supporting content

The guide also exposes machine-readable discovery resources: an AI catalog is published alongside the Agent Skills implementation, and the SKILL.md SHA256 digest matches the value declared in the discovery index.

Images are also well represented in the Markdown: all 133 images have alt text, with a median of 13 words, and the descriptions are preserved through the conversion.

## Detection is a separate problem from implementation

The guide has served per-page Markdown for some time, advertised with a `<link rel="alternate" type="text/markdown">` element on all 45 pages. How the five handled it:

| Checker         | Result                                                                                                                                    |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| ora.ai          | **Detected** — _"Markdown alternate advertised and verified: `.../en/markdown/` serves markdown"_                                         |
| is-agentic.com  | **Detected** — the same check, since it renders ora.ai's scan                                                                             |
| Claude SEO      | **Detected** — recorded as _"present on 45/45 HTML pages"_                                                                                |
| Cloudflare      | **Did not check.** Its only Content check is Markdown negotiation, which is genuinely absent — so its report is correct as far as it goes |
| agent-ready.dev | **Missed it**                                                                                                                             |

Three of five found it by following the link. The failures split into two different kinds, and only one is a scanner error.

**A convention mismatch, reported correctly.** agent-ready.dev's P15 — _"No .md or .mdx version of this page found"_ — and ora.ai's failing "Markdown URL fallback" check are both **true**. The guide serves Markdown at a `/markdown/` path segment, and [llmstxt.org](https://llmstxt.org/) specifies a `.md` suffix:

> provide a clean markdown version of those pages at the same URL as the original page, either with `.md` appended (`page.html.md`) or with the extension replaced by `.md` (`page.md`)

That is the guide failing a convention it chose not to follow, not a tool getting it wrong.

**One genuine false negative.** agent-ready.dev's P17 reports _"No `<link rel="alternate" type="text/markdown">` element or equivalent Link header found"_. The element exists on all 45 pages, and three other checkers found it by following exactly that link. This is the only verified false negative across the five.

**One internal contradiction.** ora.ai passes "Markdown alternate link" but returns "not applicable" on "Markdown frontmatter metadata", explaining _"No served markdown found"_ — because that check looks only for a root `.md` file or content negotiation. The same tool found the Markdown in one check and declared it absent in another. `is-agentic.com` inherits both results.

!!! note "What this does and does not show"

    It does not show that checkers routinely miss working implementations — most found this one. It shows something narrower: **a non-standard URL shape costs you detection even when the capability is real**, and tools disagree internally about what counts as "served Markdown".

    Agents themselves mostly reach these files by following links rather than guessing paths, so the `/markdown/` convention likely costs more in tooling and reporting than in agent behavior. Both are worth fixing, for different reasons.

## Issues found

The actionable list, using ora.ai as the baseline because it has the broadest check set and the only evidence base behind it. Filtered to checks that apply to a documentation site — API, OAuth, MCP, payments, and commerce are excluded, see [not applicable here](techniques.md#tier-3-not-applicable-here).

Tiers are **ora.ai's own**, taken from its JSON.

### Required

| Finding                                                                                                                | ora's suggested fix                                                                   | Also reported by                                       |
| ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| **No when-to-use guidance.** `llms.txt` describes what the guide _is_ but never says when an agent should reach for it | A "when to use this" section in `llms.txt` naming best-fit use cases                  | —                                                      |
| **JSON-LD has no identity type.** One bare `WebPage` block, so an agent cannot tell what the site _is_                 | An identity type with `name`, `description`, `url` — `TechArticle` fits documentation | agent-ready.dev (P11), Claude SEO, direct measurement  |
| **`og:image` not detected.** Verified cause: the tag uses `name="og:image"` where Open Graph requires `property=`      | Change `name=` to `property=`. One word                                               | —                                                      |
| **Trust anchor pages incomplete.** `/en/about/` resolves; `/en/contact/` and `/en/privacy/` both 404                   | Publish `/contact` and `/privacy` — but see the note below                            | —                                                      |
| **Homepage content ratio below target** — 3.5% against a 5% floor                                                      | At least 500 characters of real prose in the raw HTML, with a clear `h1`              | agent-ready.dev (P13, 1.7%), direct measurement (2.2%) |
| **No HTTP `Link:` headers** on any of 45 pages, RFC 8288                                                               | `Link: </sitemap.xml>; rel="sitemap"`, plus the Markdown alternate                    | Cloudflare, direct measurement                         |

### Recommended

| Finding                                                                                                                            | ora's suggested fix                                                           | Also reported by |
| ---------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | ---------------- |
| **No `sameAs` entity linking** in JSON-LD, so "Wagtail" the CMS cannot be distinguished from the bird genus                        | `sameAs` to Wikidata, Wikipedia, and the GitHub org                           | Claude SEO       |
| **No `Organization` schema** and no extended schema types beyond the bare `WebPage`                                                | Add `Organization`, and broaden the type set — but see the note below         | —                |
| **404s carry no Markdown error body.** Status code is correct; a request with `Accept: text/markdown` gets no Markdown explanation | A short Markdown body on 404 linking back to the docs, sitemap, or `llms.txt` | —                |
| **No Markdown content negotiation** — `Accept: text/markdown` returns HTML, `Vary` omits `Accept`                                  | Serve Markdown on `Accept: text/markdown`, add `Vary: Accept`                 | **all five**     |
| **The guide is absent from Wikipedia and Wikidata** — see the correction below                                                     | Reference the guide from both, and correct the stale `P856`                   | —                |

### Emerging

| Finding                                                                                                                      | ora's suggested fix                                                                                                  | Also reported by      |
| ---------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | --------------------- |
| **No Markdown representation for a cold-arriving agent.** Neither the homepage nor any well-known root path returns Markdown | Serve Markdown at the site root, or support negotiation on the homepage                                              | —                     |
| **No `.md` suffix fallback.** `/index.md` and per-page `.md` both 404; Markdown lives at a `/markdown/` path segment instead | Alias `.md` alongside the existing `/markdown/` route                                                                | agent-ready.dev (P15) |
| **No section-level `llms.txt`** for `/how-to-guides/`, `/concepts/`, `/reference/`                                           | A per-section `llms.txt` under each top-level path                                                                   | —                     |
| **No bot-UA Markdown serving**                                                                                               | ora suggests detecting AI user agents and serving them Markdown. **We would not** — that is cloaking by another name | —                     |

!!! warning "Three of ora's fixes do not transfer to a non-commercial docs site"

    Its rubric assumes a company selling something, and the recommendations inherit that.

    - **Organization schema**: ora asks for `contactPoint` with a phone number and a `PostalAddress`, to "verify your business legitimacy". The guide is a CC0 documentation subdomain with no business address. Inventing one would be worse than omitting the schema.
    - **Trust anchor pages**: `/contact` and `/privacy` exist on `wagtail.org`, not on the docs subdomain. Duplicating them here to satisfy a check would be cargo-culting.
    - **Schema type breadth**: ora suggests adding `FAQPage`. Google retired FAQ rich results, and the Claude SEO skill's own quality gates forbid recommending it. Take `BreadcrumbList` from that recommendation and leave the rest.

!!! note "ora tiers Markdown negotiation lower than every other checker"

    ora splits it across three checks at three tiers — "Markdown content negotiation" (recommended), "Markdown agent docs" (emerging), and "Bot-UA markdown serving" (emerging) — and scores two of the three as bonus items that do not count toward the total. So none of them weighs much.

    Every other checker treats it as central. It is **the whole of Cloudflare's Content category**, scoring zero, and agent-ready.dev fails it at P19. Given the measured demand for Markdown, the cross-tool signal is a better guide here than ora's own weighting.

### Reported outside ora's check set

Not ora checks, so they carry no tier. Included because more than one source reproduced them, or because they feed something ora does check.

| Finding                                                                                                                                                                                              | Reported by                                                                   |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| **11 English pages have an empty meta description**, propagating into `llms.txt`, the Markdown frontmatter, and the Open Graph tags                                                                  | Semrush (16, including 5 in translated trees), Claude SEO, direct measurement |
| **No `rel="describedby"` link** to `llms.txt` — the v2 discovery mechanism                                                                                                                           | agent-ready.dev (P24)                                                         |
| **`AGENTS.md` is not served from the site.** It exists in the repository — ora found it there and passed its separate "agent platform configs" check — but `guide.wagtail.org/AGENTS.md` returns 404 | agent-ready.dev (S12, S13)                                                    |

!!! note "The when-to-use content half exists already"

    The published `SKILL.md` opens with _"A professional support helper for Wagtail CMS users. Use when answering Wagtail CMS user questions with the Wagtail Guide as authoritative documentation."_ That is when-to-use guidance — it just sits in the Agent Skills manifest rather than in `llms.txt`, where ora looks for it. Closer to a placement problem than a missing one.

### Already fixed during this research

`Content-Signal` directives and a `Sitemap:` declaration in `robots.txt`, and YAML frontmatter on the Markdown endpoints.

!!! warning "One of these is ora.ai's error, not the guide's"

    ora.ai reports _"No Wikipedia article or Wikidata entity found for 'wagtail'"_. Both exist: [Wagtail (software)](https://en.wikipedia.org/wiki/Wagtail_(software)) returns HTTP 200, and Wikidata [Q25206006](https://www.wikidata.org/wiki/Q25206006) is labelled "Wagtail — website content management system".

    The underlying concern survives the error and is worth acting on. Neither entity references `guide.wagtail.org` — the Wikipedia article has 26 external links and none point here — and Wikidata's `P856` (official website) still points at **`wagtail.io`**, a domain the project has moved away from. The entity exists, but it is stale and does not know the guide exists.

## Problems the checkers missed

Found by direct measurement or by Semrush, not by any agent readiness checker.

**`robots.txt` rules cover redirects, not the paths agents reach.** All content lives under locale prefixes, but the `Disallow` rules are un-prefixed. Tested against the live file with a compliant parser:

```text title="May a crawler fetch this?"
NO    /search/                        ← what robots.txt blocks; only ever redirects
YES   /en/search/                     ← the real page, linked from every page's search form
YES   /en/search_json/?query=publish  ← JSON API, returns page data, unbounded queries
NO    /admin/
YES   /en/admin/                      ← 404s anyway, so no exposure
```

The redirect does not rescue the rule. A crawler obeying `Disallow: /search/` never fetches it, so it never follows the redirect — and the canonical path it does discover is not covered. Every checker reported `robots.txt` as passing, because they validate syntax and AI-bot directives rather than whether the rules match live URLs.

**Translated URLs serve English content.** `/fr/`, `/is/`, and `/pt-br/` return HTTP 200 for pages that are not translated, serving English text. The Markdown differs from English by exactly two lines — the `last_modified` date and the `Page URL:` line — with the entire body identical. The HTML is worse: `/fr/reference/account-settings/` serves `<html lang="fr">` over English content, which is a false claim that misleads language detection in any ingestion pipeline.

**`/de/` is a phantom locale.** An accepted URL prefix appearing in no hreflang set and no language switcher, serving a complete duplicate of the English corpus — byte-identical including `last_modified`. `/zz/` is rejected, so the prefix list is deliberate.

**35% of hreflang alternates are dead.** 38 of 108, measured serially with a retry on each failure, zero recovered. Arabic is 87.5% broken, Icelandic 38.6%. No page has a self-referencing hreflang tag and none has `x-default`. The root cause is an English information architecture migration that never propagated to `/is/` at all — 44 pages still on the old URL scheme, orphaned from the sitemap.

**Retired URLs return 404 with no redirect.** Including `/en/reference/page-status-definitions/`, which is still linked from a live page. Stable URLs is the only practice the [Website Specification](https://specification.website/spec/agent-readiness/stable-urls/) rates **required** in its entire agent readiness category.

**An advertised identity does not resolve.** `ai-catalog.json` declares `did:web:guide.wagtail.org`, which must resolve at `/.well-known/did.json`. That returns 404.

**The Markdown corpus is islands, not a graph.** `llms.txt` itself is fine — 45 absolute links, all pointing at `/markdown/` URLs. The problem is one level down, **inside the Markdown page bodies**: of 145 internal links across them, **zero point at a `/markdown/` URL**. 134 are root-relative paths to HTML and 11 are absolute paths to HTML. So an agent that follows `llms.txt` correctly, fetches a Markdown page, then follows a link in that page is ejected into 12x larger HTML on its first hop. The root-relative ones are also unresolvable once `llms-full.txt` is read out of context, which is how that file is designed to be consumed.

**No ordered lists anywhere.** Zero `<ol>` elements across all 45 pages in **both** HTML and Markdown, so this is an authoring pattern rather than a conversion loss. Procedures are written as prose: _"First… Second… Last…"_. Section length is heavily left-shifted, with a median of 53 words and only 3.8% in the 134–167 word range that extracts cleanly.

## Suggested priorities

Grouped by theme rather than listed flat, because several findings share one fix. Ordered by evidence strength and effort.

### 1. Complete the Markdown delivery layer

The highest-evidence technique, and the guide is most of the way there already. Four findings collapse into one piece of work:

- `.md` suffix aliases alongside the existing `/markdown/` route
- `Accept: text/markdown` negotiation, with `Vary: Accept` — including on `/` and `/en/`, so an agent arriving cold from a search result can get Markdown without first parsing the HTML to find the `rel="alternate"` link
- A Markdown body on 404s when Markdown was requested

### 2. Make the Markdown layer a graph, not islands

The links **inside the Markdown page bodies** point at HTML, so an agent that arrives via `llms.txt` leaves the Markdown layer on its first hop. Rewriting them to `/markdown/` targets, and making them absolute, is one change to the same link renderer and fixes both that and the 134 unresolvable links in `llms-full.txt`.

### 3. Advertise what already exists

The guide publishes more than anything discovers. All of these are header or `<link>` changes over content that is already served:

- HTTP `Link:` headers for the sitemap, the Markdown alternate, and `llms.txt`
- `rel="describedby"` pointing at `llms.txt`
- Serve `AGENTS.md`, which exists in the repository but 404s on the site

### 4. Enrich the structured data

One template, several findings. Add to the JSON-LD an identity type (`TechArticle` fits documentation), `description`, `url`, `inLanguage`, `BreadcrumbList`, and `sameAs` pointing at Wikidata `Q25206006` and the Wikipedia article. Skip ora's `Organization` and `FAQPage` suggestions — see the caveats in [findings](#issues-found).

While there: `og:image` uses `name=` where Open Graph requires `property=`.

### 5. Repair the URL and locale contract

The only practice rated **required** by the Website Specification is the one currently failing.

- Restore the retired URLs from the information architecture migration, using `wagtail.contrib.redirects`
- Stop emitting `hreflang` for locales where the specific page does not exist — 38 of 108 alternates are dead
- Stop serving English under a translated `lang` attribute, and resolve the `/de/` prefix, which duplicates the entire English corpus

### 6. Fill the content gaps

- The 11 empty `search_description` fields, which feed the meta tag, Open Graph, the `llms.txt` entry, and the Markdown frontmatter from one place
- A "when to use this" section in `llms.txt`
- Reference the guide from the Wikipedia article and Wikidata, and correct `P856`, which still points at `wagtail.io`
