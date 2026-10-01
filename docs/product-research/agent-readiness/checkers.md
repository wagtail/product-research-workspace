---
tags:
  - agent-readiness
---

# The checkers

Five checkers were run against [guide.wagtail.org](https://guide.wagtail.org/) on 29 September 2026. This page describes what each one is and how far to trust it. Results are in [findings](findings.md).

## Summary

| Checker          | Score    | Type                          | Weighting                              |
| ---------------- | -------- | ----------------------------- | -------------------------------------- |
| Cloudflare       | 33 / 100 | Vendor tool                   | Protocol and commerce endpoints        |
| ora.ai           | 55 / 100 | Research-lab scanner          | Discovery, access, usability, payments |
| agent-ready.dev  | 59 / 100 | Developer-led project         | Content readability                    |
| is-agentic.com   | 61 / 100 | Vercel, rendering an Ora scan | Ora's scan, re-scored by site type     |
| Claude SEO skill | 65 / 100 | Open-source agent suite       | Configurable                           |

## Cloudflare "Is your site agent ready"

Checks `robots.txt`, sitemap, `Link` headers, DNS-AID, Markdown content negotiation, Web Bot Auth, Content Signals, Agent Skills, Agentic Resource Discovery, API catalog, OAuth and OIDC discovery, MCP server cards, WebMCP, and four agentic commerce protocols.

Heavily weighted toward machine-to-machine plumbing: eight of fifteen scored checks concern APIs, authentication, or MCP, and four more concern commerce. It produced the lowest score of the five by a wide margin, almost entirely from categories that do not apply to a documentation site.

## ora.ai

The research lab behind the measured agent behavior cited in Vercel's guide. Runs **125 checks** grouped into four layers: discovery, access, usability, payments.

The most transparent of the five. Every check returns a status, a score, a specification URL, a maturity rating (`verified` or `emerging`), and a tier (`required`, `recommended`, `emerging`). Results are available as JSON, which is why this is the only checker whose raw output we could store in full.

## is-agentic.com

Published by **Vercel Inc.**. Infers a site category — it correctly identified the guide as "Docs & content" — and re-bands the results into essential checks, checks recommended for that category, and bonus signals.

The site-type inference is a genuinely good idea that does not go far enough: seven of the nine reported failures are API checks (OpenAPI spec, developer portal, JSON error responses, function calling compatibility) against a site it had just classified as documentation.

!!! warning "One scan, two scores — 55 versus 61"

    These are not independent results. `is-agentic.com` renders an **ora.ai scan**: its embedded result object carries `"source":"ora.ai"`, points at `https://ora.ai/guide.wagtail.org`.

    What differs is the scoring model. `ora.ai` groups into discovery, access, usability, and payments. `is-agentic.com` infers a site category and re-buckets into essential, recommended-for-that-category, and bonus signals — then reports 61 where ora.ai reports 55.

    So the six-point gap is **the same underlying scan, weighted two ways**. That is the sharpest illustration in this research that a score is a reporting choice, not a measurement.

## agent-ready.dev

Developer-led project.

Reports three separate scores rather than one: the Vercel Agent Readability rubric (59), llmstxt.org compliance (94), and an accessibility tree audit (88). Splitting them is more useful than a composite, because the three measure genuinely different things.

The only checker that audited the accessibility tree, which is a reasonable proxy for how cleanly text extracts.

## Claude SEO skill

An open-source Claude Code plugin ([AgriciDaniel/claude-seo](https://github.com/AgriciDaniel/claude-seo), MIT licensed). Not a checker in the same sense as the others: it is an agent suite that runs live measurements and reasons about them.

More flexible than a fixed rubric — it can be pointed at a specific question and will investigate. Correspondingly less reproducible.

## Semrush, and why it is not one of the five

[Semrush](https://www.semrush.com/) is a general SEO site audit tool, not an agent readiness checker, so it is cited in [findings](findings.md) without being scored here. It earned the citations for two reasons.

**It crawled far wider.** 100 pages against the checkers' 1 to 25, and unlike any of them it crawled the translated trees. That is why it found five empty meta descriptions the others could not see, and a broken English link — `/en/reference/page-status-definitions/`, still referenced from a live page — that **no agent readiness checker caught**.

**It corroborated the descriptions finding by a completely different method.** Semrush reported 16 missing meta descriptions; the 11 English ones match the Claude SEO result page for page. Two unrelated tools arriving at the same list is stronger evidence than either alone.

!!! note "Mainstream SEO tooling has started shipping agent checks"

    Semrush's audit now includes `Llms.txt not found` and `Llms.txt has formatting issues` as first-class checks. Both passed. That is a reasonable signal that some of this is moving from novelty into routine site auditing — worth watching, because it changes who is likely to raise these issues on a client project.

    Its remaining coverage is conventional SEO: hreflang conflicts, structured data validity, sitemap and robots.txt format. Nothing on Markdown endpoints, content negotiation, `.well-known`, Agent Skills, or MCP.

## The AgentReady specification

Not a checker. [AgentReady](https://www.agentready.org/) is an open specification from Ora, Vercel, Mintlify, and Auth0, and it is the best-grounded document we reviewed.

Three things set it apart:

- **Stable identifiers** for each practice (`AR-FIND-01` and so on), so findings can be cited precisely.
- **Strength levels tied to evidence**: `MUST` for changes that actually broke an agent in the studies, `SHOULD` for practices that measurably shaped where agents went, `MAY` for emerging conventions.
- **Explicit provenance per claim**: `STUDIES` marks the 1,033 traced agent runs and controlled experiment, whose dataset is public; `ORA SCANNER` marks live reachability data. Untagged statements are labeled as recommendations rather than findings.

No other source we reviewed distinguishes measured behavior from opinion. It also restates ora.ai's ecosystem baseline, which is the only way to tell whether a given score is good — see [context for the grade](README.md#headline-findings). The live figures live at [ora.ai/research](https://ora.ai/research) and drift slightly from the restatement.

!!! warning "Two of the five checkers inherit this study's sample"

    `ora.ai` built the study, and `is-agentic.com` renders its scans. Their rubrics therefore carry its sampling: two model vendors, four models, two harnesses, and 25 sites that are all commercial SaaS or developer-tool vendors. Several headline numbers also turn out to be properties of one harness rather than of agents generally — see [what the study actually measured](techniques.md#what-the-study-actually-measured).

    This is not a reason to discount them. They are the best-evidenced checkers of the five. It is a reason not to read their scores as measurements of how *all* agents behave, particularly for a non-commercial documentation site, which is a shape the study never tested.

Its own summary is the most useful sentence in the field:

> Two changes broke an agent: hiding the answer behind JavaScript, and blocking access. Everything else is find, then read, then act.

## How to use checkers

- **Run more than one.** The overlap is the trustworthy set. Anything reported by a single tool needs verifying by hand.
- **Verify every failure against the live response.** `curl` settles most of them in seconds. One of our five reported a `<link>` element absent that is present on every page — and several other failures turned out to be accurate reports of a convention we had not followed, which is a different problem needing a different fix.
- **Establish that a technique applies before implementing it.** Publishing a stub OAuth document to satisfy a scanner advertises a capability you do not have.
- **Do not treat a score as a target.** Several rubrics reward techniques that the available evidence does not show agents using.
