# MCP Audit: AskCruz, EOXS-TEAMS and hr

Date: 2026-10-08

## Executive summary

Fix three things before AskCruz touches a paying customer's data: raw SQL over the whole EOXS Odoo database, a write tool on the hr connector, and tier labels that nothing enforces per user. The connectors work well enough for an internal pilot. They are not safe for multi-user or customer-facing use yet.

| # | Risk | Severity | Est. fix |
| --- | --- | --- | --- |
| R1 | EOXS-TEAMS runs arbitrary SELECTs over 776 tables, including `res_users.password` and 15 stored RingCentral tokens | High | 3–5 days |
| R2 | The hr connector carries 33,965 client email threads, 2,275 Fireflies calls and the salary register, far beyond an HR need | High | 1 week |
| R3 | `update_asset` on hr can overwrite source documents that feed the wiki every 6 hours | Medium-High | 2 days |
| R4 | `access_tier` is a label; enforcement appears to be the endpoint split, not a per-user check | Medium | 2–3 weeks |
| R5 | Five thread-saving servers, two on the same URL, two with secrets inside the URL path, plus a test server left connected | Medium | 2 days |
| R6 | List and search tools take no limit or cursor, so results are unbounded | Medium | 1 week |
| R7 | Wiki answers are LLM summaries of 26–34k external emails with no injection screen | Medium | 3–4 weeks |

What is solid: no plaintext passwords (134 of 141 are PBKDF2, 7 are empty), the SQL tool is SELECT-only with a 30-second timeout and 1,000-row cap, both servers list the same 8 clients, and the EOXS DB was last written on 5 October 2026. Fix estimates are my judgment, not measured.

## Scope and method

This audit covered the Askcruz, hr and EOXS-TEAMS connectors using only read-only calls, at 100ft (staged) rigor. Each finding carries one of three labels.

- **Verified**: observed directly in a tool result, schema or count query.
- **Inferred**: follows from verified facts but was not tested.
- **Assumed**: plausible, flagged for you to confirm.

**Not done, by design:** no password, token or salary values were read, `update_asset` was never called, and no load testing or server-source review happened. Client Odoo instances (for example `sabre.eoxs.com`) were not queried, and I had no access to hosting, network or secrets-management configuration. Everything about infrastructure is therefore inferred from URLs and tool descriptions.

## 1. Access control and secrets

The biggest exposure is that one SQL tool reaches everything in the EOXS Odoo database, and the hr connector reaches far more than HR needs.

| ID | Finding | Evidence | Label |
| --- | --- | --- | --- |
| F1 | EOXS-TEAMS lists 776 tables and accepts any SELECT. `res_users` exposes `password` and `ringcentral_access_token`. 141 users: 134 PBKDF2, 7 empty, 0 plaintext; 15 hold a RingCentral token. Tables for mail servers (2 rows), IMAP fetch (1), payment acquirers (12), employees (340) and payslips (1) are in reach. | `list_tables`, `describe_table`, count queries | Verified (credential contents not read) |
| F2 | hr and Askcruz expose the same tool surface, but hr holds more data: 33,965 vs 26,831 email threads, 2,083 vs 1,086 wiki pages. HR users get client emails and call transcripts they do not need. | `get_index` on both | Verified |
| F3 | `hr__update_asset(slug, body, title)` replaces a source document. A scheduled job re-drafts the wiki page from it within 6 hours. The description says the connection "may be restricted to a specific slug". | Tool schema | Verified exists; restriction untested |
| F4 | Assets carry `access_tier` (all `tier2`, one `tier2_confidential_hr`). The salary register appears only via hr. Emails, calls and wiki pages show no tier. | `list_assets` on both | Inferred |
| F5 | `THREAD OV` and `Threads-OV` connect over legacy SSE with a long secret token inside the URL path. Paths end up in proxy and access logs. A test server (`auth testing`) is still attached to the account. | Connector URL list | Verified from URLs; log exposure assumed |

Why F1 matters beyond "read-only": any agent that reads an inbound email or call transcript can be talked into running a query. Hashes can be cracked offline, and RingCentral tokens may allow calls and call logs. Read-only does not mean safe.

**Fixes**

1. **F1:** Give the tool its own read-only Postgres role. Grant `SELECT` only on masked views over the 11 business tables (`sale_order`, `account_move`, `res_partner`, and so on). Revoke on `res_users`, `ir_*`, `hr_*`, `payment_*`, `fetchmail_server` and `ir_mail_server`. Add a SQL parser (for example sqlglot) with a table and column allowlist ahead of execution. Rotate the 15 RingCentral tokens and move token storage out of a queryable column.
2. **F2:** Split by purpose. hr serves HR documents and candidate data. Remove the client email, call and wiki tools from it.
3. **F3:** Move writes to a separate writer server with a hard slug allowlist, version history, a diff, a named approver, and human approval before re-ingestion.
4. **F4:** Enforce the tier in the data layer, for example `WHERE tier <= token.tier`, on every table. Tag emails, calls and wiki pages with a tier at ingest. This fits the per-server OAuth work you already have in flight.
5. **F5:** Move auth to an `Authorization: Bearer` header on streamable HTTP, rotate the URL tokens now, and remove the test server.

## 2. Data integrity and freshness

The same question can get a different answer depending on which endpoint you ask, and "customer" means something unreliable in the EOXS database.

**D1. The two knowledge servers have diverged.** Both list the same 8 clients, but hr holds up to twice as many wiki pages. Cause unknown: tier filtering, separate ingestion, or lag.

| Measure | Askcruz | hr |
| --- | --- | --- |
| Wiki pages | 1,086 | 2,083 |
| Entity pages | 295 | 676 |
| Analysis pages | 537 | 880 |
| Prospect pages | 25 | 37 |
| Email threads | 26,831 | 33,965 |
| Fireflies calls | 1,986 | 2,275 |
| Fathom calls | 117 | 151 |
| Implementation tasks | 777 | 828 |

**D2. No connector reaches client invoices or tickets.** The Askcruz tool says invoices and tickets are "not in this system" and to query EOXS Teams Odoo. But EOXS-TEAMS is EOXS's own database; clients run separate instances (for example `sabre.eoxs.com`). Brannon Steel and RW Conklin Steel have no `odoo_base_url` at all. Retention and churn questions, which matter for your AskCruz goal, cannot be answered from data.

**D3. "Customer" counts in the EOXS DB will mislead.** 7,041 of 76,432 partners have `customer_rank > 0`, yet there are only 194 sale orders (68 confirmed, 17 draft), 14 vendors and 0 purchase orders. That is about 36 customers per sale order, so the rank is likely set by CRM activity rather than revenue. 10,834 partners (14%) have no email.

**D4. Naming hides freshness.** The database is called `Eoxteams_12Feb24`, but sale orders, invoices and stock moves were last written on 5 October 2026. It is live or a fresh replica; I could not tell which. Odoo 13 is also past vendor support to my knowledge, and the tool already needs a note that the column is `type`, not `move_type`.

**Fixes**

1. **D1:** Make one corpus the source of truth with the tier applied at read time. Add `last_ingested_at` and a `corpus_version` to `get_index`, and run a nightly job that diffs counts by page type and alerts on drift above 2%.
2. **D2:** Register the missing Odoo URLs, then add a read-only gateway per client instance with the same masked-view pattern as F1, so invoice and ticket data becomes queryable.
3. **D3:** Define a customer as a partner with at least one confirmed or done sale order, or one posted `out_invoice`. Ship it as a `v_customers` view and document it in the tool's business context.
4. **D4:** Return `data_as_of` with every query result, confirm replica lag, and rename the database label in the context text.

## 3. MCP and tool design

The tools were built for a human with a SQL console, not for a model with a finite context window and untrusted inputs.

| ID | Finding | Label |
| --- | --- | --- |
| T1 | Askcruz and hr define the same 23 or more tools under different servers, and hr adds `update_asset`. Two copies of one codebase will drift, and identical names invite the model to call the wrong server. | Verified |
| T2 | `list_implementation_tasks` accepts only `client` and `stage`; `search_implementation_tasks` only `query` and `client`. No limit, cursor or date filter, against 777–828 tasks and 27–34k email threads. | Verified from schemas |
| T3 | `get_client_profile` bundles contacts, tasks, emails, calls and wiki pages in one call, and its description tells the model to prefer it. No size cap is stated. | Verified; size unknown |
| T4 | `get_business_schema` returns up to 3 sample rows for 11 tables including `res_partner` and `account_move`, so real names, emails and amounts land in model context by design. | Verified from description |
| T5 | Every EOXS-TEAMS response repeats a six-line business-context block, a token tax on every call that belongs in server instructions once. | Verified |
| T6 | 12 MCP servers are attached. Five persist threads or chats: `Shubham-OV` and `thread saving` share one URL; `THREAD OV`, `Threads-OV` and `threads-OV_vaibhav` are separate. | Verified from URLs |
| T7 | The EOXS-TEAMS and 3GM-Odoo gateways sit on a `lancerdevops.me` domain. Who operates it and holds the DB credentials is not visible from here. | Assumed risk |

**Fixes**

1. **T1:** One codebase, one deployment per trust zone, tool lists generated from the caller's token scope. Prefix tools by purpose, for example `intern_search_emails` vs `hr_search_candidates`.
2. **T2, T3:** Add `limit` (default 20), `cursor`, `since` and `until`, and a `fields` projection. Always return `total_count` and `truncated`. Cap `get_client_profile` per section and add a `sections` parameter.
3. **T4:** Replace samples with synthetic rows plus column documentation.
4. **T5:** Move the business context into the server's `instructions` field once.
5. **T6:** Keep one thread server. Delete duplicates and the `auth testing` server. Standardize on streamable HTTP with header auth.
6. **T7:** Confirm ownership, contract and secrets handling for the `lancerdevops.me` hosts. If it is a vendor, require audit logs and a data-processing agreement.

## 4. Reliability and observability

You cannot currently answer "who queried what, and when" from the evidence I could see.

| ID | Finding | Label |
| --- | --- | --- |
| O1 | `ir_logging` has 0 rows in the EOXS DB, and no tool exposes an audit trail of MCP calls. | Verified for DB; MCP logs unknown |
| O2 | The SQL tool has a 30s timeout and 1,000-row cap but no visible per-user quota or concurrency limit. | Verified caps; quotas assumed absent |
| O3 | Wiki ingestion runs every 6 hours, but the model gets no per-document freshness. | Verified from tool description |
| O4 | Thread persistence runs on `onrender.com` hosts. If one is down or cold-starting, transcripts are lost silently. | Assumed |
| O5 | 30 active Odoo crons and 8 monitor configs exist, but no health tool surfaces their state. | Verified counts |

**Fixes**

1. **O1:** Log every tool call as structured JSON: caller, tool, argument hash, rows returned, latency and error code. Ship it to one store and alert on unusual volume per user.
2. **O2:** Add per-token rate limits, a concurrency cap, and a query cost guard such as `EXPLAIN` row estimates over a threshold.
3. **O3:** Return `ingested_at` with every wiki result and warn when a source is older than its page.
4. **O4:** Add a health endpoint per server, retry with backoff, and a local write-ahead queue so a failed `save_message` is retried rather than dropped.
5. **O5:** Expose a read-only `get_health` tool that reports cron last-run, ingestion lag and error counts.

## 5. Retrieval quality and knowledge-base poisoning

Most of the wiki is machine-written from external text, so one bad input can quietly reach every user's answers.

| ID | Finding | Label |
| --- | --- | --- |
| Q1 | The tool descriptions call wiki pages "synthesized" summaries, distinct from raw sources. On Askcruz, 537 of 1,086 pages are analyses and 295 are entities, so most answers rest on summaries of summaries. | Verified |
| Q2 | 26,831–33,965 email threads and 2,000+ call transcripts from outside parties feed ingestion. Text in them can carry instructions aimed at an agent. | Inferred |
| Q3 | Search is full-text on emails, and fuzzy title plus substring on assets. I saw no evidence of semantic ranking, reranking or an evaluation set. | Inferred from tool descriptions |
| Q4 | Two call vendors (Fireflies 1,986, Fathom 117) may record the same meeting twice. | Assumed |

**Fixes**

1. **Q1:** Require every wiki claim to cite raw source ids (`sources_raw` already exists), and store `last_verified`. For numbers, SOP steps and salaries, answer from the raw asset, never the summary.
2. **Q2:** Treat all ingested external text as untrusted data. Scan it at ingest for instruction-like content and quarantine hits. Never let retrieved text trigger a tool call, and require confirmation for any write.
3. **Q3:** Add hybrid retrieval (keyword plus embeddings) with a reranker. Build an evaluation set of 50 real questions with known source documents and track recall@10 on every change.
4. **Q4:** Deduplicate calls by participants, start time and duration before ingestion.

## 6. Roadmap against the AskCruz goal

Security first, because one leaked credential or poisoned wiki page costs more than a month of feature work, and your first outside customers will ask about it. Estimates assume one engineer working part-time alongside normal support duties.

| Phase | Work | Gate to move on |
| --- | --- | --- |
| Days 1–7 | Rotate RingCentral and URL-path tokens; remove the `auth testing` server and duplicate thread servers; revoke `res_users` and `ir_*` reads; pull `update_asset` off hr | Credential and write exposure closed; a SELECT on `res_users` is refused |
| Days 8–30 | Masked read-only views for EOXS-TEAMS; split hr from client data; add limit and cursor to list and search tools; structured audit log | Every tool call logged; no unbounded result |
| Days 31–60 | Server-side tier enforcement and ingest tagging; one corpus with a nightly drift check; `v_customers` view; per-client Odoo gateways for Brannon and Conklin | Same question returns the same answer on both servers; tier test suite passes |
| Days 61–90 | Injection screening at ingest; cited-source wiki; hybrid retrieval with a 50-question evaluation set | recall@10 baseline recorded; injection test set blocked |

Top-two priorities if time is tight: F1 and the hr split. They are the only items that can cause an irreversible incident.

## Appendix: verification log

Checks run on 8 October 2026, all read-only.

| Check | Result |
| --- | --- |
| `get_index` on Askcruz and hr | Counts in section 2, D1 |
| `list_clients` on both | Identical 8 clients; Brannon and RW Conklin have null `odoo_base_url` |
| `list_assets` on both | 17 assets on Askcruz, 18 on hr; extra on hr is `eoxs-salary-details` (`tier2_confidential_hr`) |
| EOXS-TEAMS `list_tables` | 776 tables |
| `describe_table res_users` | `password` and `ringcentral_access_token` columns present |
| Count query on `res_users` | 141 users; 134 PBKDF2; 7 empty or null; 0 other; 15 with a RingCentral token |
| Freshness query | Sale orders, invoices and stock moves last written 5 Oct 2026 |
| Credential-table counts | `ir_mail_server` 2, `fetchmail_server` 1, `payment_acquirer` 12; contents not read |
| HR data counts | `hr_employee` 340, `hr_payslip` 1 |
| Business counts | 76,432 partners; 7,041 with `customer_rank > 0`; 14 vendors; 194 sale orders (68 confirmed, 17 draft); 752 account moves; 124 pickings; 0 purchase orders |
| Ops counts | `ir_logging` 0; active crons 30; monitor configs 8 |
| Tool schemas | Pagination parameters absent on task list and search tools |
