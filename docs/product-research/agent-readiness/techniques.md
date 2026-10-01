---
tags:
  - agent-readiness
---

# Techniques and evidence

The agent readiness techniques the [checkers](checkers.md) test for, what the evidence says about each, and what adopting them would mean for Wagtail.

## How to read the evidence column

The strongest evidence available comes from the [ora.ai research lab](https://vercel.com/kb/guide/make-your-site-readable-by-ai-agents), which traced **1,033 agent runs across 25 sites** plus **190 controlled fetch probes across 19 site configurations**, published August 2026 with a public dataset. The [AgentReady specification](https://www.agentready.org/) grades practices by that evidence: `MUST` for changes that broke an agent, `SHOULD` for practices that measurably shaped agent behavior, `MAY` for emerging conventions.

The [Website Specification](https://specification.website/spec/agent-readiness/) independently rates the same techniques as required, recommended, or optional. Where the two agree, confidence is high.

### What the study actually measured

The dataset is [published](https://github.com/agentready-org/standard/blob/main/data/traces.csv), and reading it changes how much weight some headline numbers can carry. Every figure below reproduces the article exactly before breaking it down further.

The 1,033 runs are **two vendors, four models, and two harnesses**:

|              |                                                                                                      |
| ------------ | ---------------------------------------------------------------------------------------------------- |
| Models       | `claude-haiku-4-5` (31.7%), `claude-sonnet-4-6` (22.8%), `gpt-5.4` (22.8%), `claude-fable-5` (22.7%) |
| Vendor split | 77.2% Anthropic, 22.8% OpenAI                                                                        |
| Harnesses    | `claude-agent-sdk` (52.4%), `eve` (47.6%)                                                            |
| Sites        | 25, all commercial SaaS or developer-tool vendors                                                    |

Three consequences worth carrying into any decision.

**The Markdown headline is a property of the harness, not of agents.** Broken down:

| harness            | runs | web fetches | format specified | markdown requested |
| ------------------ | ---- | ----------- | ---------------- | ------------------ |
| `claude-agent-sdk` | 541  | 517         | **0**            | **0**              |
| `eve`              | 492  | 2,953       | 2,361            | 2,259 (76.5%)      |

One harness requested Markdown on three quarters of its fetches. The other requested it **zero times, because its fetch tool exposes no format parameter at all**. The published "65.1% of fetches requested Markdown" averages a harness that can with one that cannot, and the "95.7% when a format was specified" figure is definitionally restricted to the harness that can.

**Model variance is roughly twentyfold.** Within the harness that supports it: `gpt-5.4` 96.5%, `claude-haiku-4-5` 84.9%, `claude-fable-5` 4.3%.

**No site in the sample resembles this one.** All 25 are commercial vendors with products, pricing, and mostly APIs. There is no documentation-only site, no open-source project documentation, and no non-commercial site in the study.

!!! warning "This does not make Markdown endpoints a bad idea"

    It changes the claim from "agents want Markdown" to "some agent harnesses can request Markdown, and where they can, most models use it heavily." That is still a good reason to serve it — the cost is low and the upside is real for a meaningful share of clients. It is not a reason to expect a measurable effect across all agents.

    Note also that the two mechanisms appear to substitute for each other: the harness that cannot request Markdown reached `llms.txt` **more** often (36.0% of runs versus 27.2%). Serving both covers both kinds of client.

### Usage rates when a resource is present

The same lab published [observed usage rates](https://ora.ai/blog/what-agents-actually-reach) — how often each resource is _actually used_ once it exists. Docs pages (88%) and homepages (84%) sit in one tier; everything built specifically for machines falls off a cliff below them. Arrival order follows the same shape: homepage at turn 1.0, docs at 1.7, `llms.txt` at 3.0, `llms-full.txt` at 6.7.

Three of those numbers bear directly on decisions taken here:

| Resource                 | Used when present | Why it matters to us                                                                                                                                         |
| ------------------------ | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `llms.txt`               | 46%               | The best-performing machine-readable file by some distance, and the guide's is complete                                                                      |
| robots.txt / sitemap.xml | **9%**            | Low direct value to agents. The `Sitemap:` directive is worth having as crawler hygiene, but it should not be counted as an agent readiness gain             |
| `llms-full.txt`          | **5%**            | The least-used resource measured. The guide's is 172 KB and generated automatically, so it costs little to keep — but it does not warrant further investment |

**Reachability, not value, is what holds the rest back.** On the one site in the study that shipped the full stack _and linked it properly_, the same files inverted — `openapi.json` 17% → 72%, `.well-known/*` 34% → 63%. The lab's own conclusion is that discovery is the bottleneck, which is the single most transferable finding in this research.

## Tier 1: the techniques with real evidence behind them

### Server-rendered content

**Evidence: strongest available.** In the controlled probes, a fetch-only client _could not retrieve an answer available only through JavaScript_. This is one of only two changes that broke an agent outright.

Guide status: passing. Fully server-rendered, no JavaScript dependency.

### Unblocked crawler access

**Evidence: strongest available.** The second of the two breaking changes — _neither client could retrieve the answer when the server returned 403_.

Guide status: passing, with a caveat. No AI crawler is blocked, and 23 user agents receive identical content. But the `robots.txt` `Disallow` rules omit the locale prefix, so they cover paths that only redirect rather than the canonical ones agents actually reach. A compliant crawler is blocked from `/search/`, and allowed `/en/search/` — the URL every page links from its search form — along with `/en/search_json/?query=`, which returns page data as JSON.

### Markdown source endpoints

**Evidence: strong.** **65.1% of agent fetches requested Markdown** (2,259 of 3,470). When a fetch specified any format at all, it chose Markdown **95.7% of the time** (2,259 of 2,361).

Rated `Recommended` by the Website Specification, which names two mechanisms: a `.md` suffix on the canonical URL, or content negotiation.

Guide status: implemented, but at a `/markdown/` path segment rather than either named mechanism — which two checkers correctly flagged, and which cost detection in a third. Content negotiation is absent; six `Accept` header variants all return HTML, and `Vary` does not include `Accept`.

Because agents probe conventional shapes as well as following links, the non-standard path costs twice: tools miss it, and habitual probing misses it too.

### Linking your machine-readable files

**Evidence: strong, and it reframes the others.** **86–97% of attributed fetches came through links rather than guessed paths**, depending on file type.

That does not mean agents never guess. They probe conventional paths from habit — `/pricing` and `/integrations` get hit early, and `/api` was guessed in ~11% of runs while existing on only ~2% of sites. The accurate reading is narrower: **machine-readable files are reached overwhelmingly via links, and separately, agents do probe conventional URL shapes.** Both are worth serving.

Guide status: partial. The `rel="alternate"` element is present on all 45 pages and correct — three of the five checkers followed it successfully. But there are **no HTTP `Link:` headers anywhere**, so any client that does not parse the HTML body cannot discover anything.

### llms.txt

**Evidence: moderate and specific.** Agents reached `llms.txt` through links in **86%** of attributed fetches (356 of 416). Of the 329 runs that reached it, **36% later grounded the final answer in its content**. It works — when it is linked.

Guide status: implemented well. 45 entries with 1:1 sitemap parity, spec-conformant structure, `llms-full.txt` alongside it. Referenced from all 45 HTML pages but from neither `robots.txt` nor the sitemap. The `rel="describedby"` discovery link introduced by the llms.txt v2 convention is absent.

### First-party documentation pages

**Evidence: strong.** Agents reached docs in **83% of runs** (855 of 1,033), fetching an average of 3.4 docs pages. **47% of grounded answers traced to a docs page.**

Relevant because it validates the premise: for a documentation site, agent readiness is not a side concern.

## Tier 2: worth doing, weaker evidence

### Structured data

**Evidence: explicitly inconclusive.** The study _"did not measure an improvement in agent accuracy or task completion from JSON-LD"_, and advises repeating critical facts in visible text because some converters drop it.

Guide status: minimal. All 45 pages carry a bare `WebPage` with `headline`, `datePublished`, `dateModified` and nothing else. No `BreadcrumbList`, no `author`, no `publisher`, no `inLanguage`, no `license`.

The one genuinely useful addition would be entity disambiguation. "Wagtail" is also a bird genus, and nothing on the site asserts which entity it documents. Wikidata `Q25206006` is the correct target.

!!! note "Be honest about this one"

    Structured data is cheap and probably helpful, but the best available study did not measure a benefit. It should not be presented as evidence-backed.

### `.well-known` discovery files

**Evidence: moderate.** Agents reached a `.well-known` path in **22.7% of runs** (234 of 1,033).

Guide status: good. Agent Skills discovery with a verifying sha256 digest, plus an AI catalog. One defect: the catalog declares `did:web:guide.wagtail.org` but `/.well-known/did.json` returns 404, so the identity claim does not resolve.

### Content Signals

**Evidence: none measured.** Rated `Optional` by the Website Specification. The syntax rests on the IAB Tech Lab specification and validator convention — the IETF binding draft expired in May 2026.

Guide status: shipped during this research. Declares `ai-train=yes, search=yes, ai-input=yes`, matching the guide's CC0 licensing.

Worth doing because it is one line and makes an existing intention explicit, not because agents are known to read it.

### Stable URLs

**Evidence: not measured directly, but the only `Required` item** in the Website Specification's agent readiness category.

Guide status: failing. Retired paths from an information architecture migration return 404 with no redirect, including one still linked from a live page. The migration never propagated to the Icelandic tree, leaving 44 pages on the old scheme and orphaned from the sitemap.

## Tier 3: not applicable here

The checkers penalized the guide for all of these. The Website Specification rates every one **optional**, and none applies to a documentation site with no API, no login, and no checkout.

- MCP server cards, WebMCP, A2A agent cards
- OAuth and OIDC discovery, OAuth Protected Resource metadata, `auth.md`
- OpenAPI specifications, API catalogs, developer portals
- DNS-AID, Web Bot Auth, NLWeb, Schemamap
- x402, MPP, UCP, ACP and other agentic commerce protocols

Publishing stubs to satisfy a scanner would advertise capabilities that do not exist. Cloudflare's 25/100 on its API and MCP category is best read as "this site is not an API", not as a defect.
