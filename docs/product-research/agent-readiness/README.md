---
tags:
  - agent-readiness
---

# Agent readiness

Research into how agent readiness checkers work, what they report, and which of it is worth acting on. [guide.wagtail.org](https://guide.wagtail.org/) was used as the test site.

## Why this research

Agent readiness has no ratified standard. The conventions in circulation — `llms.txt`, Markdown source endpoints, Content Signals, Agent Skills, MCP — come from different sources and are at very different stages of adoption. Tooling has appeared faster than agreement on what agent readiness should actually mean or measure.

That makes "run a checker and fix what it says" an unreliable strategy, which is what this research set out to test.

## What we did

Five checkers were run against the guide in September 2026, and their results cross-referenced against each other and against direct measurement. We also used Semrush for supplementary technical checks where these overlapped with agent-readiness concerns. Full detail in [checkers](checkers.md) and [findings](findings.md).

| Checker                                                        | Score    |
| -------------------------------------------------------------- | -------- |
| [Cloudflare's isitagentready.com](https://isitagentready.com/) | 33 / 100 |
| [ora.ai](https://ora.ai/)                                      | 55 / 100 |
| [agent-ready.dev](https://agent-ready.dev/)                    | 59 / 100 |
| [is-agentic.com](https://is-agentic.com/)                      | 61 / 100 |
| [Claude SEO skill](https://github.com/AgriciDaniel/claude-seo) | 65 / 100 |

We also reviewed the specifications and research those checkers are built on, which turned out to matter more than any of the scores. See [sources](#sources).

## Headline findings

**Scores are not comparable, and should not be treated as targets.** The five results span 33 to 65 for the same site on the same day. Most starkly, `is-agentic.com` **renders an ora.ai scan** — its result object carries `"source":"ora.ai"` — and reports 61 where ora.ai reports 55. One scan, two scores, differing only in how each tool buckets and weights it.

**Most of the gap is category weighting, not disagreement about facts.** Checkers penalize documentation sites for lacking OpenAPI specs, OAuth discovery, MCP servers, and commerce protocols. Seven of `is-agentic`'s nine failures are API checks against a site with no API — even though it correctly inferred the site type as "Docs & content".

**A non-standard URL shape costs you detection.** The guide already served per-page Markdown. Three of five checkers found it, one never checked, and one reported the advertising `<link>` element absent when it is present on all 45 pages. Two separately — and correctly — flagged that the Markdown is not at the `.md` suffix the conventions specify. See [findings](findings.md#detection-is-a-separate-problem-from-implementation).

**The things that actually break agents are basic, and the guide already does them.** The AgentReady spec summarizes its own study data bluntly: the only two changes that broke an agent were hiding content behind JavaScript and blocking access.

**Discovery is the bottleneck, not the formats.** Observed usage rates show docs pages at 88% and homepages at 84%, then a cliff: `llms.txt` 46%, `.well-known/*` 34%, `llms-full.txt` 5%. But on the one studied site that shipped the full stack _and linked it_, those same files jumped to 63–72%. The machine-readable formats work; they lose on reachability.

**Context for the grade.** [ora.ai's scored population](https://ora.ai/research) — roughly 99,000 sites as of September 2026 — averages 40 with a median of 39. Grade D alone holds 40% of sites, 65% sit at D or F, and only 0.375% reach A or above. A score in the mid-50s is above average, not failing.

## Scope

In scope: whether machines can find, fetch, and use the content. Technical changes made in code.

Out of scope: search rankings, backlinks, Core Web Vitals, and mobile rendering. These came up in the audits and were set aside. We also did not attempt to quantify any improvement in visibility, only whether a technique is detected and used.

## Pages

- [Checkers](checkers.md) — what each tool is, what it measures, how far to trust it.
- [Findings](findings.md) — what they reported on the guide, cross-referenced against direct measurement.
- [Techniques](techniques.md) — the techniques themselves, the evidence for each, and what they would mean for Wagtail.

## Sources

Reference material reviewed alongside the scans, and what each contributed.

| Source                                                                                                                   | What we took from it                                                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [AgentReady specification](https://www.agentready.org/)                                                                  | Practice identifiers and strength levels tied to evidence (`MUST` / `SHOULD` / `MAY`), and a restatement of ora.ai's ecosystem baseline                                                 |
| [Make your site readable by AI agents](https://vercel.com/kb/guide/make-your-site-readable-by-ai-agents) (Vercel)        | Per-practice study claims with sample sizes, and the prescribed `Content-Type` for Markdown responses                                                                                   |
| [ora research](https://ora.ai/research) and [What agents actually reach](https://ora.ai/blog/what-agents-actually-reach) | The score distribution across ~99,000 sites, plus observed usage rates per resource, arrival order by turn, and the reachability inversion — the most directly useful evidence we found |
| [`traces.csv` and `fetchability.csv`](https://github.com/agentready-org/standard/tree/main/data)                         | The published study dataset. Analyzing it directly is what surfaced the harness and sampling caveats                                                                                    |
| [Website Specification, agent readiness](https://specification.website/spec/agent-readiness/)                            | An independent required / recommended / optional rating per technique, and the only source rating stable URLs as required                                                               |
| [llmstxt.org](https://llmstxt.org/)                                                                                      | The `.md` suffix convention and the `rel="describedby"` discovery link, quoted directly                                                                                                 |

Two notes on provenance. The AgentReady specification, the Vercel guide, the ora research post, and the dataset all come from the same collaboration (Ora with Vercel, Mintlify, and Auth0), so they are not independent of each other. The Website Specification and llmstxt.org sit outside that group, which is why agreement between them carries more weight.
