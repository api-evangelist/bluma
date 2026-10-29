# Bluma

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Bluma is a San Francisco-based AI company (Y Combinator Fall 2025) building a short-form AI content engine that lets creators, agencies, and brands produce social media video ads at scale. The platform "de-edits" existing reference videos into their individual scenes, captions, and overlays, then uses AI asset generation and a node-based canvas editor to recreate and remix shots into new ads, UGC, and template-based videos optimized for TikTok, Reels, and Shorts. Bluma ships a public developer program: a REST API at api.getbluma.com/api/v1 for asynchronous video generation, templates, credits, API keys, webhooks and usage analytics; Mintlify-hosted documentation at docs.getbluma.com; first-party TypeScript and Python SDKs; a seven-event HMAC-signed webhook surface; published per-tier rate limits; and an undocumented but live hosted MCP server at api.getbluma.com/api/mcp gated by Clerk OAuth 2.1 with dynamic client registration. Founded by Alisa Wu (formerly Zoox/Amazon) and Stephen Ni.

**Areas:** Company · Artificial Intelligence · Video · Video Generation · Advertising · Marketing · Content Creation · Short-Form Video · Social Media · Generative AI · Creative Tools · Text-to-Speech

## Bluma on APIs.io

Everything profiled for Bluma is published on APIs.io rather than in this repository. Start at the provider page — **[apis.io/providers/bluma/](https://apis.io/providers/bluma/)** — or go straight to one of the APIs or properties below.

### APIs (2)

- [Bluma API](https://apis.io/apis/bluma/bluma-api/) — Asynchronous short-form video generation REST API. Submit a template id and a prompt to POST /v1/videos, receive a job id, and collect the finished render…
- [Bluma MCP Server](https://apis.io/apis/bluma/bluma-mcp-server/) — Hosted Model Context Protocol server on the Bluma API host. Discovered via RFC 9728 OAuth Protected Resource Metadata rather than from the documentation,…

### Published properties

Each property below is a page on APIs.io carrying the machine-readable artifact and what it says.

#### AsyncAPI (1)

Event-driven and webhook contracts.

- [Bluma Webhooks](https://apis.io/asyncapis/bluma/bluma-webhooks/)

#### MCP servers (1)

Model Context Protocol servers that expose the API to agents.

- [bluma-mcp.yml](https://apis.io/servers/bluma/bluma-mcp-yml/)

#### Security (2)

Authentication, trust centre, and vulnerability disclosure.

- [Bluma Authentication](https://apis.io/security/bluma/bluma-authentication/)
- [Bluma Domain Security](https://apis.io/security/bluma/bluma-domain-security/)

#### OAuth scopes (1)

The permissions the API asks for.

- [Bluma Scopes](https://apis.io/scopes/bluma/bluma-scopes/)

#### Plans and pricing (1)

Published plans, tiers and prices.

- [Bluma Plans Pricing](https://apis.io/plans/bluma/bluma-plans-pricing/)

#### Rate limits (1)

Published request limits.

- [Bluma Rate Limits](https://apis.io/rate-limits/bluma/bluma-rate-limits/)

## Where to find this API

Links to the company's own pages. This repository publishes no copy of their specifications — the canonical document is theirs and lives at the URLs below.

- [Website](https://www.getbluma.com)
- [Documentation](https://docs.getbluma.com/)
- [Getting started](https://docs.getbluma.com/quickstart)
- [API reference](https://docs.getbluma.com/api-reference/overview)
- [Pricing](https://docs.getbluma.com/concepts/credits)
- [Sign up](https://www.getbluma.com/signup)
- [Support](https://discord.gg/cZGXY4nrFZ)
- [Blog](https://www.getbluma.com/blog/)
- [Terms of service](https://www.getbluma.com/terms)
- [Privacy policy](https://www.getbluma.com/privacy)

## API Evangelist rating

**Kin Score 58.6** — strong · rubric 0.25.0 · scored 2026-10-23

| Facet | Score |
|---|---|
| Discoverability | 75.0 |
| Contract quality | 39.0 |
| Contract governance | 18.2 |
| Operational transparency | 55.3 |
| Developer ergonomics | 73.2 |
| Access clarity | 76.3 |

**Agent readiness 50.9** — agent-ready (gated down from agent-native)

| Agent-readiness dimension | Result |
|---|---|
| Spec Presence | yes |
| Agentic Access | — |
| Reversibility Documented | — |
| MCP Server | verified |
| Auth Clarity | served (partial credit) |
| Idempotency | — |
| Error Semantics | documented (partial credit) |
| OpenAPI Examples | — |
| Rate Limit Signal | documented (partial credit) |
| Event Surface Described | yes |
| Agent Skills | derived (partial credit) |
| Well Known Catalog | — |
| Consent Identity | — |
| Agent Card | — |
| Dry Run Mode | — |
| Delegated Identity | served (partial credit) |
| Protected Resource Metadata | verified |
| Dynamic Client Registration | yes |
| Agentic Commerce | — |
| Pagination Ratio | na |
| Operation Distinctiveness | na |

**Regulatory alignment 31.8** — AI & Biometrics

The rubric is published at [github.com/api-evangelist/kin-score](https://github.com/api-evangelist/kin-score). The per-check breakdown behind these numbers — which checks were earned, partial or missed — along with score history and how this provider compares to its peers, is available at [apis.io/providers/bluma/](https://apis.io/providers/bluma/).

## How this profile is made

API Evangelist profiles a company's public API surface the way a developer would find it: the website, developer portal and documentation, the specifications it publishes, its public repositories, and its pricing, status and changelog pages. From that it derives the properties above — contracts split per API, workflows, schemas, collections, governance rules, plans and rate limits — and scores them against the published [Kin Score rubric](https://github.com/api-evangelist/kin-score).

The artifacts are published on [APIs.io](https://apis.io), not in this repository. This repository is the public record for Bluma: the place to say something is wrong, missing or out of date, and to ask for a re-score or a removal.

## Open an issue

- **Something about this listing** — a wrong description, a missing API, a stale artifact or a question about the rating: [open an issue on this repository](https://github.com/api-evangelist/bluma/issues/new).
- **A new API, or one that is not in the catalog yet** — [submit it on APIs.io](https://apis.io/add/).
- **Anything else** — [the APIs.io inbox](https://github.com/api-search/inbox/issues/new) or [info@apievangelist.com](mailto:info@apievangelist.com).
