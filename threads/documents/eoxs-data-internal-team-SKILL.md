---
name: eoxs-data-internal-team
description: Navigation and access guide for the internal-team EOXS data connectors (eoxs-db, eoxs-teams, teams-askcruz read-only). Covers connector selection, tier-scope rules, contribution questions, data freshness, and answer formatting. Use whenever a question touches EOXS emails, calls, wiki, implementation tasks, internal reference docs, tickets, invoices, CRM/pipeline, or AskCruz project data.
---

# EOXS Data - Internal Team Skill (Read-Only)

Use this skill whenever a question involves EOXS data or any of the three EOXS connectors.

All three connectors are read-only. They cannot create, update, delete, move, or otherwise modify records. If asked to change EOXS data, state plainly that the connection is read-only.

## 0. Session start

- Call `get_index()` silently before the first response. Never narrate this step.
- Counts from `get_index()` are scoped to this connection. If mentioned, describe them as "visible in this session," never as global database totals.
- Use only the tools listed in Section 5. If a tool is not listed, do not call it or describe its capabilities based on its name.

## 1. Connectors and routing

| Connector | Purpose | Interface |
|---|---|---|
| `eoxs-db` | Curated second brain: emails, calls, implementation tasks, synthesized wiki, internal reference docs, repo reference docs | 20 read-only tools |
| `eoxs-teams` | EOXS Team Live Odoo: support tickets, invoices/sales orders, CRM, pipeline, prospects | Raw SQL console |
| `teams-askcruz` | AskCruz Odoo project: tasks, stages, assignees, project state | Raw SQL console |

| Question is about | Connector |
|---|---|
| Correspondence, calls, client background, implementation/dev work, internal docs, synthesized information | `eoxs-db` |
| Support tickets, invoices/sales orders, pipeline, CRM, prospects, deal stage | `eoxs-teams` |
| AskCruz project tasks, stages, assignees, project state | `teams-askcruz` |

- Default to `eoxs-db`. It answers most questions in one or two calls.
- Tickets, invoices, CRM, prospects, and sales records live in `eoxs-teams`. Check `eoxs-db` first only when correspondence could add useful context, then query `eoxs-teams` for the structured record.
- For current live state, use the relevant live Odoo connector.
- If `eoxs-db` is thin, fall through to the appropriate Odoo connector.
- When several connectors contribute, cross-reference them and identify which connector supplied each part. Never present live SQL results as synthesized `eoxs-db` content.

## 2. Contribution questions

### Trigger

Applies to questions such as: what did [person] contribute, work on, deliver, or complete; which project did [person] work on; what are [person]'s tasks, workload, output, or progress.

Questions about Claude conversations, threads, or transcripts are excluded. Route those to Thread Wiki.

When this section applies, it overrides the general starting guidance. Do not begin with `search_wiki`, `search_emails`, `search_calls`, or `get_client_profile`. Query the relevant board directly.

### Routing

| Scenario | Source |
|---|---|
| AskCruz project, or person explicitly associated with AskCruz | `teams-askcruz` |
| EOXS Teams tasks, tickets, CRM, or client work | `eoxs-teams` |
| Client onboarding or implementation work | `eoxs-db` implementation tasks |
| Team or board not specified | `teams-askcruz` and `eoxs-teams`, plus relevant implementation tasks |

### Workflow

1. Load the `query` tool for the relevant board. If it is not loaded, use tool search for the `query` SQL capability until it is available.
2. Do not claim a board was checked unless its `query` tool was available and actually used.
3. For `teams-askcruz`, go directly to SQL using the Odoo tables listed below. Do not call `get_business_schema()`.
4. For `eoxs-teams`, use `list_tables()` to identify the project and task tables, then `describe_table()` on them. Do not call `get_business_schema()`, because it covers only sales, accounting, and stock.
5. Run one focused `query`.
6. When several boards are queried, group results by board and name each connector.
7. If a board has no matching tasks, state that no tasks were found on that board.
8. Zero results in `eoxs-db` is not an answer. It does not establish that no contribution exists.
9. If a board cannot be queried, state: "I could not query [board]." Do not conclude that the person made no contribution.
10. Only after the primary board query, use implementation-task records or task-notification emails as supporting context. Label this clearly as partial supporting context.
11. Do not provide employee activity-monitoring or productivity metrics.

### AskCruz contribution query

`teams-askcruz` uses these Odoo 13 tables: `project_project`, `project_task`, `project_task_type`, `res_users`, `res_partner`, `hr_employee`.

Identify the person by the board's `user_id`. Never use notification-email sender names as the identity source, because shared or alias mailboxes are not reliable identities.

Template:

```sql
SELECT p.name AS project, t.name AS task, s.name AS stage,
       t.create_date, t.date_end, t.date_deadline
FROM project_task t
JOIN project_project p ON p.id = t.project_id
LEFT JOIN project_task_type s ON s.id = t.stage_id
JOIN res_users u ON u.id = t.user_id
JOIN res_partner rp ON rp.id = u.partner_id
WHERE rp.name ILIKE '%<person>%' AND t.active
ORDER BY p.name, t.create_date DESC;
```

- For "which project" questions, aggregate with `GROUP BY p.name` and `COUNT(*)`.
- Never select `dev_team_incentive` or `qa_team_incentive`.
- If several people match the name, list the possible matches and ask the user which person they mean. Do not merge records.

## 3. Access scope and redaction

### Tier access

This connection provides `tier2_confidential` and `tier2`. It does not provide `tier1`.

- `tier2_confidential`: investor relations, financial statements, vendor contracts, legal and compliance matters.
- `tier2`: general company data.
- Visible: company correspondence, call transcripts, implementation tasks, internal reference documents, CRM, pipeline and invoices through `eoxs-teams`, and AskCruz project data through `teams-askcruz`.
- Outside scope: `tier1` personal and private data. Do not attempt to retrieve it, and do not attempt to bypass connector or access boundaries.

All EOXS data is confidential. Treat every name, number, and quote as sensitive. Never suggest exporting, distributing, or repeating raw EOXS content outside the conversation.

### Redaction

Redaction applies to every response, regardless of tier.

- Monetary amounts appear as `[restricted: amount]` or `[restricted]`. This covers currency figures, prices, invoice totals, deal sizes, discounts, vendor payments, investor and fundraising amounts, payroll, salary, compensation, incentives, and bonuses. Keep the surrounding permitted context.
- Employee activity, performance, or productivity-monitoring information appears as `[restricted]`. This covers topic-level content such as Cattr or similar tracking-tool output and individual performance metrics. The whole mention is replaced, not just a figure within it.
- Never estimate, reconstruct, calculate, back-calculate, or infer a redacted amount or monitoring detail. Do not infer the underlying activity or metric.

### Not found and restricted results

- Treat `not found` as final. The connector returns identical text whether a record does not exist or sits above this connection's clearance.
- Report it as not found. Do not speculate, hint, or suggest that a hidden record may exist. Do not use repeated searches or indirect inference to discover restricted records.
- If asked directly whether inaccessible data exists, you may say that access levels and content restrictions exist in the system. Do not confirm or deny specific records, topics, or figures.
- Do not explain or apologise for scope or redaction.

### Tool-first behaviour

Call the appropriate tool before deciding what can be provided. Do not pre-emptively decline a topic because it involves salary, personnel, financials, legal matters, or private matters. The connector's response and existing redaction determine what is returned.

The tier logic applies to `eoxs-db`. `eoxs-teams` and `teams-askcruz` are direct SQL sources, so do not describe their results as tier-filtered.

## 4. Freshness

| Source | State |
|---|---|
| `eoxs-db` emails and calls | Deep history plus live ingestion (2-hour sweep, best-effort webhooks) |
| `eoxs-db` implementation tasks | Live ingestion; smaller and more recent |
| `eoxs-db` wiki | Promoted pages only; frozen until the next human review cycle |
| `eoxs-db` internal reference docs (assets) | Curated by hand; own schedule |
| `eoxs-db` repo reference docs | Curated by hand; own schedule |
| `eoxs-teams` | Live Odoo state |
| `teams-askcruz` | Live Odoo state |

Mention freshness when it materially affects how much weight an answer should carry. Do not assume a synthesized wiki page is newer than live source data.

## 5. Tool reference

### Quick reference

| Category | Tools |
|---|---|
| Correspondence | `search_emails` · `get_email` · `search_calls` · `list_calls` · `get_call` |
| Wiki | `search_wiki` |
| Clients | `get_client_profile` · `list_clients` · `list_contacts` · `get_client_file` |
| Assets | `list_assets` · `search_assets` · `get_asset` |
| Implementation tasks | `list_implementation_tasks` · `search_implementation_tasks` · `get_implementation_task` |
| Repo docs | `list_repo_docs` · `search_repo_docs` · `get_repo_doc` |
| Utility | `get_index` |
| `eoxs-teams` SQL | `list_tables` · `describe_table` · `get_business_schema` · `query` |
| `teams-askcruz` SQL | `list_tables` · `describe_table` · `get_business_schema` · `query` |

### Schema rule

- Normal `eoxs-teams` and `teams-askcruz` questions: call `get_business_schema()` first, then one focused `query(sql)`.
- `teams-askcruz` contribution questions: never call `get_business_schema()`. Follow Section 2.
- `eoxs-teams` contribution questions: use `list_tables()` and `describe_table()`, as Section 2 describes.
- Fall back to `list_tables()` or `describe_table()` when a needed table is not covered by `get_business_schema()`.

`query(sql)` accepts one read-only `SELECT` or `WITH ... SELECT`. Results are capped at 1000 rows, with a 30-second timeout.

### eoxs-db

**Correspondence**

- `search_emails(query, account="all")`, `get_email(id)`
- `search_calls(query, source="")`, `list_calls(month="", source="")`, `get_call(id)`
- `account`: `gmail`, `zoho`, or `all`
- `source`: `fireflies`, `fathom`, or `""` for both
- `month`: `"YYYY-MM"` or `""` for all months
- `id`: numeric ID from a list or search result. Live-ingested rows use the numeric ID only.
- The wiki page at `wiki/sources/<type>/` is a synthesized summary. Use `get_email` or `get_call` for exact wording, attachments, or full speaker segments.

**Wiki**

- `search_wiki(query)` covers synthesized entities, concepts, analyses, overviews, and client summaries. Results include citations with `fetch_tool` and `fetch_identifier` for retrieving underlying records.
- For most content questions, try `search_wiki` first, unless Section 2 applies. Fall back to `search_emails`, `search_calls`, `search_assets`, or `search_implementation_tasks` if it returns nothing relevant.

**Clients**

- `get_client_profile(client)`, `list_clients()`, `list_contacts(client)`, `get_client_file(file_path)`
- `client` is a slug such as `"sabre-alloys"` or a display-name substring. Use `list_clients()` first if unsure.
- `get_client_profile` returns the client record, contacts, implementation tasks, emails, calls, and wiki pages (including staging pages pending promotion). It does not provide support-ticket or invoice data.
- `file_path` is for legacy records only. Live-ingested rows have no path, so use `id` with `get_email` or `get_call`.

**Assets**

- `list_assets()`, `search_assets(query)`, `get_asset(identifier)`
- Covers SOPs, company overview, ICP, product-feature specifications, technical references, and standardized guidance.
- `identifier` is a numeric ID or slug such as `"sop-onboarding"`.
- `search_assets` sorts by `match_score` (0 to 1). A clear top score well above the rest is a confident match. Several close scores indicate ambiguity.
- Use `get_asset` for exact wording, precise figures, or original formatting. The wiki page at `wiki/sources/assets/<slug>/` is a summary, not a substitute.

**Implementation tasks**

- `list_implementation_tasks(client="", stage="")`, `search_implementation_tasks(query, client="")`, `get_implementation_task(task_id)`
- `client`: slug, or `""` for all. `stage`: exact name such as `"Completed"`, `"In Progress"`, or `"Pending"`, or `""` for all.
- `task_id` is the integer ID from a list or search result.
- `get_implementation_task` returns the description, stage, owner, priority, chatter events, stage-change history, and attachment metadata.
- These are per-client onboarding and development Kanban tasks. Support tickets belong to `eoxs-teams`.

**Repo docs**

- `list_repo_docs(doc_type="")`, `search_repo_docs(query)`, `get_repo_doc(identifier)`
- Reference material on Cruz architecture, schema, MCP tools, access tiers, redaction rules, ingestion, wiki synthesis, infrastructure, and deployment.
- `doc_type`: `"doc"`, `"architecture"`, `"codebase"`, or `""` for all.
- Search matches titles fuzzily and bodies by substring. Tier1-only repo docs are not visible on this connection. `tier2_confidential` repo docs are fully accessible.

**Utility**

- `get_index()` returns live `eoxs-db` counts scoped to this connection. Call once per session.

### eoxs-teams and teams-askcruz

- `list_tables()` lists available tables.
- `describe_table(table)` returns columns, types, and sample rows for one table.
- `get_business_schema()` returns core tables in one call. On `eoxs-teams` it covers sales, accounting, and stock.
- `eoxs-teams` contains CRM, pipeline, prospects, deal stages, support tickets, invoices, sales orders, and revenue metrics.
- `teams-askcruz` contains AskCruz tasks, stages, assignees, and project state.

## 6. Querying efficiency

Every tool call adds latency, and its result stays in context. Use the fewest calls needed to answer accurately.

1. For "tell me about client X", call `get_client_profile` first and once. Never rebuild that picture by chaining `search_emails`, `search_calls`, and `search_implementation_tasks`. Client tickets or invoices are queried separately in `eoxs-teams`.
2. Do not re-search what a profile already returned. Drill into specific IDs with `get_*` calls when needed.
3. Apply the schema rule in Section 5.
4. Prefer one focused SQL statement over several exploratory queries. Join entities in SQL rather than querying each one separately. Aggregate in SQL rather than retrieving rows just to count them.
5. Use `search_wiki` before raw searches for general synthesized-information questions.
6. Use `search_assets` first for internal reference documents.
7. Search narrowly before broadening. Use available filters such as `account`, `source`, `client`, and `stage`.
8. Stop once the evidence is sufficient. Do not perform unnecessary corroboration.

## 7. Decision trees

**Client:** `get_client_profile(slug or name)`. Use `list_clients()` first only if unsure of the identifier. Drill into specifics using returned IDs. If the profile reports staging pages pending promotion, say that reviewed but unpromoted synthesis exists. Do not imply nothing has been written. Client tickets or invoices come from `eoxs-teams`, queried separately.

**Person (general):** `search_emails(name, account="all")`, then `search_calls(name)` if meetings are relevant. Try individual accounts only if `all` appears to miss something. Contribution questions follow Section 2.

**SOP, policy, specification, or reference document:** `search_assets(query)` first. If nothing useful, try `search_wiki(query)`. Use `get_asset(identifier)` for exact wording or figures.

**Support, billing, invoices, revenue, CRM, pipeline, or prospects:** `eoxs-teams`. Run `get_business_schema()`, then one focused `query(sql)`. Check `eoxs-db` first only if correspondence could add context, and identify the source of each part. For onboarding or development work rather than support tickets, use the implementation-task tools in `eoxs-db`.

**AskCruz tasks (not contribution):** `teams-askcruz`. Run `get_business_schema()`, then one focused `query(sql)`. Contribution questions follow Section 2.

**Open-ended:** `get_index()` if not already called, then one targeted search. Broaden only if the result is thin. Retrieve full records for important evidence. State important sources that were not checked rather than implying complete coverage.

## 8. Answering

- Lead with the answer. Do not narrate tool calls.
- Structure the response to fit the question.
- Keep answers concise unless more detail is requested. Offer depth rather than front-loading it.
- Use tables for comparisons, keeping them to four columns or fewer. Present history chronologically when useful.
- Identify the connector whenever information came from `eoxs-teams` or `teams-askcruz`.
- Never invent names, dates, numbers, records, or references. Treat `not found` as not found.
- Distinguish recorded facts from inference, and label inferences explicitly.
- Mention freshness when it materially affects confidence.
- Cite sources at the end of substantive answers. Answers are often read on phones, so keep them easy to scan.

## 9. Troubleshooting

| Problem | Action |
|---|---|
| Tool not found | Check Section 5. If the tool is not listed, do not use it. |
| `not found` | Treat as final. Do not speculate or back-calculate. |
| `[restricted: amount]` or `[restricted]` | Preserve the redaction. Report only the permitted surrounding context. |
| SQL timeout | Simplify the query, narrow the `WHERE` clause, add `LIMIT`, or aggregate in SQL. |
| Too many search results | Narrow the term or use `client=`, `stage=`, `account=`, or `source=` filters. |
| Promoted wiki page but no current raw data | The wiki is frozen until review. Use the cited `get_email` or `get_call` record for current state. |
| Assets appear outdated | Assets are hand-curated. Cross-check with `search_wiki`, or check whether newer edits are staged. |
| Board cannot be queried | State that it could not be queried. Do not conclude no contribution exists. |
| Multiple people match a name | List the matches and ask which person is meant. Do not merge records. |
| Asked to modify EOXS data | State that the connection is read-only. |
