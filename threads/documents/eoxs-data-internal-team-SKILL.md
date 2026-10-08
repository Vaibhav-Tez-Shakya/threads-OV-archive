---
name: eoxs-data-internal-team
description: Navigation and access guide for the internal-team EOXS data connectors (eoxs-db, eoxs-teams, teams-askcruz read-only). Covers which connector to use, tier-scope rules, and answer formatting. Use whenever a question touches EOXS emails, calls, wiki, implementation tasks, internal reference docs, tickets, invoices, CRM/pipeline, or AskCruz project data.
---

# EOXS Data - Internal Team Skill (Read-Only)

## 0. Session start

- Call `get_index()` silently before the first response. Note the counts. Do not narrate this step.
- Never state a record count from memory or from this document. Counts from `get_index()` reflect this connection's scope only, so say "visible in this session," never "the database contains."
- Use only tools listed in section 5 and in the reference table below. If a tool is not listed here, it does not belong to these connectors. Do not call it and do not describe capabilities based on its name or description.

## 1. Connectors

All three connectors are read-only. There is no write capability of any kind on any of them. Never describe, imply, or attempt an action that changes data. If asked to create or change something, say plainly that this connection is read-only.

| Connector | What it is | Shape |
|---|---|---|
| eoxs-db | Curated second brain: emails, calls, implementation tasks, synthesized wiki, internal reference docs, repo reference docs | 20 read-only tools |
| eoxs-teams | EOXS Team Live Odoo. The only source for support tickets, invoices/sales orders, and CRM/pipeline/prospect data | Raw SQL console (4 tools) |
| teams-askcruz | AskCruz Odoo project, read access only | Raw SQL console (4 tools) |

Connector selection:

| Question is about | Go to |
|---|---|
| Correspondence, calls, client background, implementation/dev work, internal docs, anything synthesized | eoxs-db |
| Support tickets, invoices/sales orders, pipeline, CRM, prospects, deal stage | eoxs-teams |
| The AskCruz project specifically: tasks, stages, assignees | teams-askcruz |

- Default to eoxs-db. It answers most questions in one or two calls.
- For tickets, invoices, CRM, prospects, or sales, eoxs-teams is the only place the structured record lives. Check eoxs-db first only if the question could plausibly be answered from correspondence (for example `search_emails` or `get_client_profile`). Then go to eoxs-teams regardless.
- If both sources surface something relevant, cross-reference and give the fuller picture. Say which connector each part came from.
- Fall through from eoxs-db to eoxs-teams or teams-askcruz when eoxs-db comes back thin, or when the question is about current live state rather than history.
- Say which connector answered when it was not eoxs-db. Never blend live SQL results into the second brain's voice as if they had been synthesized there.

## 2. Contribution questions: boards only

**Trigger (match on meaning):** "contribution", "what did [person] contribute / work on / deliver / complete", "which project did [person] work on", "[person]'s tasks / workload / output / progress".

Exclusion: questions about Claude conversations, threads, or transcripts ("what did [user] work on in Claude") are not covered by this section. Route those to Thread Wiki.

When triggered, this section overrides the "try `search_wiki` first" guidance in sections 5, 6, and 7. Do not start with `search_wiki`, `search_emails`, `search_calls`, or `get_client_profile`. Do not call `get_business_schema()` on teams-askcruz, because it dumps every table with sample rows, truncates, and exposes personal-data columns. All other rules (tiers, redaction, read-only, formatting) still apply.

| Scenario | Board |
|---|---|
| AskCruz project, or person is on AskCruz | teams-askcruz |
| EOXS Teams tasks/tickets/CRM, or EOXS client work | eoxs-teams |
| Client onboarding or implementation work | eoxs-db `search_implementation_tasks` |
| Team not stated | Query teams-askcruz AND eoxs-teams (plus implementation tasks). Report per board. |

Steps, in fixed order:

0. Load `query` for the board (tool search "query sql select" until it is loaded). Never proceed, or claim a board was checked, without it.
1. teams-askcruz is Odoo 13 (`project_project`, `project_task`, `project_task_type`, `res_users`, `res_partner`, `hr_employee`). Go straight to SQL. On eoxs-teams, `get_business_schema()` covers only sales, accounting, and stock, so use `list_tables()` to find the project and task tables, then `describe_table` on them.
2. Run one `query`. Template for teams-askcruz:

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

   For "which project," aggregate with `GROUP BY p.name` and `COUNT(*)`. Never select `dev_team_incentive` or `qa_team_incentive`, which are monetary. If the name matches several people, list the matches and ask which one. Do not merge them. Sender names in notification emails (shared or alias mailboxes) are not reliable identities. Use the board `user_id`.
3. Answer grouped by board, naming each connector. A board queried and empty gets "no tasks found on [board]." Zero results in eoxs-db is not an answer.
4. If `query` cannot be loaded, or the project lives elsewhere (for example a client's own separate Odoo instance), say "I could not query [board]." Never conclude that no contribution exists.
5. Only then, and clearly labelled "partial, from task-notification emails / task chatter," use `get_implementation_task` or emails as supporting context. Activity-tracking and performance metrics stay off-limits (section 3).

## 3. Access scope and redaction

### Tier access
- This connection carries company-confidential clearance: `tier2_confidential` (investor relations, financial statements, vendor contracts, legal and compliance matters) and `tier2` (general).
- It does NOT include `tier1` (Rajat "Raj" Jain's personal data and private tier1 wiki pages). This boundary is intentional. Never work around it.
- Visible: investor relations, financial statements, vendor contracts, legal and compliance matters, general company data, all EOXS correspondence and call transcripts, implementation tasks, internal reference docs, CRM/pipeline/invoices (via eoxs-teams), AskCruz project data (read-only).
- Not visible: Raj's personal financial data, Raj's personal wiki pages (tier1), any Raj-specific private matters.
- All EOXS data is confidential. Treat every name, number, and quote as sensitive. Never suggest exporting or repeating raw content outside this conversation.

### Redaction (applied to every response, regardless of tier)
1. Every monetary amount is replaced with `[restricted: amount]` or `[restricted]`. This includes dollar or other-currency figures, prices, invoice totals, deal sizes, discounts, vendor payments, investor and fundraising amounts, and payroll, salary, compensation, incentive, and bonus figures. The surrounding context (that a deal, payroll action, or vendor negotiation happened) stays visible. Only the number is removed.
2. Employee activity, performance, or productivity monitoring data is replaced with `[restricted]` in full. This applies to topic-level content, such as Cattr or similar tracking-tool output and individual performance metrics. The whole mention is removed, not just a figure within it.
- Everything else in `tier2_confidential`, including legal and compliance matters, investor relations, vendor contract terms, and financial-statement discussion, is fully visible in text form.

### Not-found and restricted results
- A "not found" result is final. It means the record does not exist or sits above this connection's clearance. The tool returns identical text either way, by design. Report it as not found. Never speculate, hint, or reason aloud that a "not found" might mean restricted content exists.
- The same applies to `[restricted: amount]` and `[restricted]`. Never estimate, infer, or back-calculate a number or a monitoring detail from context.
- Do not explain or apologise for scope or redaction. If asked directly whether data or amounts exist that this connection cannot see, you may say that access levels and content restrictions exist in this system. Do not confirm or deny anything about specific records, topics, or figures.
- Always call the tool first, on every question, regardless of subject. Do not pre-emptively decline because a topic sounds sensitive (salary, personnel, financials, legal, a specific person's private matters). Let the tool's response be the answer: real data with redaction already applied, or a plain "not found."
- This tier logic does not apply to eoxs-teams or teams-askcruz. Those are direct SQL. Do not describe their results as tier-filtered.

## 4. Freshness

| Source | State |
|---|---|
| eoxs-db emails and calls | Deep history plus live ingestion (2-hour sweep, best-effort webhooks) |
| eoxs-db implementation tasks | Live ingestion only. Smaller and more recent. |
| eoxs-db wiki (synthesized pages) | Promoted pages only. Frozen until the next human review cycle. |
| eoxs-db internal reference docs (assets) | Curated by hand. Updated on their own schedule. |
| eoxs-db repo reference docs | Curated by hand. Updated on their own schedule. |
| eoxs-teams and teams-askcruz | Live by definition. Represents current Odoo state. |

Flag freshness whenever it changes how much weight an answer carries.

## 5. Tool reference

### Quick reference

| Category | Tools | Count |
|---|---|---|
| Correspondence | `search_emails` · `get_email` · `search_calls` · `list_calls` · `get_call` | 5 |
| Wiki | `search_wiki` | 1 |
| Clients | `get_client_profile` · `list_clients` · `list_contacts` · `get_client_file` | 4 |
| Assets | `list_assets` · `search_assets` · `get_asset` | 3 |
| Implementation tasks | `list_implementation_tasks` · `search_implementation_tasks` · `get_implementation_task` | 3 |
| Repo docs | `list_repo_docs` · `search_repo_docs` · `get_repo_doc` | 3 |
| Utility | `get_index` | 1 |
| eoxs-teams (SQL) | `list_tables` · `describe_table` · `get_business_schema` · `query` | 4 |
| teams-askcruz (SQL) | `list_tables` · `describe_table` · `get_business_schema` · `query` | 4 |

### eoxs-db

- **Correspondence:** `search_emails(query, account="all")`, `get_email(id)`, `search_calls(query, source="")`, `list_calls(month="", source="")`, `get_call(id)`.
  - `account`: `"gmail"`, `"zoho"`, or `"all"` (default).
  - `source` (calls): `"fireflies"`, `"fathom"`, or `""` (both, default).
  - `month`: `"YYYY-MM"` or `""` for all months.
  - `id`: numeric id from a list or search result. Live-ingested rows use the numeric id only.
  - The wiki page at `wiki/sources/<type>/` is a synthesized summary with citations. Use `get_email` or `get_call` for exact wording, original attachments, or full speaker segments.
- **Wiki:** `search_wiki(query)`. Full-text search of synthesized pages (entities, concepts, analyses, overviews, client summaries). Each result includes citations with `fetch_tool` and `fetch_identifier` to pull exact records. Try this first for most content questions, unless section 2 applies. Fall back to `search_emails`, `search_calls`, `search_assets`, or `search_implementation_tasks` only when it returns nothing relevant.
- **Clients:** `get_client_profile(client)`, `list_clients()`, `list_contacts(client)`, `get_client_file(file_path)`.
  - `client`: slug (for example `"sabre-alloys"`) or display-name substring. Use `list_clients()` first if unsure of the slug.
  - `get_client_profile` returns the client record, contacts, implementation tasks, emails, calls, and wiki pages (live plus staging pending promotion), cross-linked by `client_id`.
  - `file_path` is for legacy records only. Live-ingested rows have no path. Use `id` with `get_email` or `get_call` instead.
  - No support-ticket or invoice data. See section 1.
- **Assets (internal reference docs):** `list_assets()`, `search_assets(query)`, `get_asset(identifier)`.
  - Covers SOPs, company overview, ICP, product-feature specs, technical references, and standardized guidance.
  - `identifier`: numeric `id` or `slug` (for example `"sop-onboarding"`).
  - `search_assets` sorts by `match_score` (0 to 1, title-similarity based). A clear top score well above the rest is a confident match. Several close scores mean real ambiguity.
  - Use `get_asset` for exact wording, precise figures, or original formatting. The wiki page at `wiki/sources/assets/<slug>/` is a summary and not a substitute.
- **Implementation tasks:** `list_implementation_tasks(client="", stage="")`, `search_implementation_tasks(query, client="")`, `get_implementation_task(task_id)`.
  - `client`: slug, or `""` for all clients.
  - `stage`: exact stage name (for example `"Completed"`, `"In Progress"`, `"Pending"`), or `""` for all.
  - `task_id`: integer from a list or search result.
  - `get_implementation_task` returns the full description, stage, owner, priority, chatter events including stage-change history, and attachment metadata.
  - These are per-client Odoo onboarding and dev Kanban tasks, distinct from support tickets. Support tickets go to eoxs-teams.
- **Repo docs:** `list_repo_docs(doc_type="")`, `search_repo_docs(query)`, `get_repo_doc(identifier)`.
  - Reference material on how the Cruz system works: schema, MCP tools, access-tier and redaction rules, ingestion and wiki-synthesis pipeline, infrastructure and deployment.
  - `doc_type` (list only): `"doc"`, `"architecture"`, `"codebase"`, or `""` (all).
  - Search matches title fuzzily and body by substring.
  - Tier1 only. For this connection, tier1 docs are not visible unless personal tier1 access exists. tier2_confidential docs are fully accessible.
- **Utility:** `get_index()`. Live row counts for eoxs-db, scoped to this connection. Call once per session.

### eoxs-teams and teams-askcruz (SQL)

- `list_tables()`: list all tables.
- `describe_table(table)`: columns, types, and sample rows for one table.
- `get_business_schema()`: columns, types, and sample rows for all core tables in one call. Cheaper than `list_tables` plus `describe_table` per table. On eoxs-teams it covers sales, accounting, and stock. On teams-askcruz it is never used (see section 2).
- `query(sql)`: a single read-only `SELECT` or `WITH ... SELECT`. Capped at 1000 rows and a 30-second statement timeout.
- eoxs-teams content: CRM, pipeline, prospect and deal-stage data, support tickets, invoices and sales orders, revenue metrics.
- teams-askcruz content: AskCruz tasks, stages, assignees, project state.

## 6. Querying efficiency

Every tool call adds latency, and its full result stays in context for the rest of the conversation. Answer in the fewest calls that are genuinely sufficient.

1. For any "tell me about client X" question, call `get_client_profile` first and once. Never rebuild that picture by chaining `search_emails`, `search_calls`, and `search_implementation_tasks`. For that client's tickets or invoices, query eoxs-teams separately.
2. Do not re-search what a profile already returned. Drill in with a `get_*` call on a specific id it surfaced.
3. On eoxs-teams or teams-askcruz, call `get_business_schema()` first, except where section 2 forbids it on teams-askcruz. Fall back to `list_tables` or `describe_table` only for tables the schema does not cover.
4. Write one good SQL statement, not several exploratory ones. Join in the query rather than issuing one query per entity. Aggregate in SQL rather than pulling rows to count them.
5. `search_wiki` is a real shortcut. Use it before raw searches, but fall through when it comes back thin.
6. For internal docs, use `search_assets` first. Title matches (for example "SOP onboarding" or "ICP definition") rank faster than browsing `list_assets`.
7. Search narrow before broad. Try the specific term first. Fan out across sources only when a targeted search comes back thin.
8. Use filters instead of extra calls: `source=` on calls, `account=` on emails, `client=` and `stage=` on implementation tasks.
9. Stop when you can answer. Corroboration the question did not ask for costs the reader time.

## 7. Decision trees

- **A client:** `get_client_profile(slug or name)` on eoxs-db. Use `list_clients()` first only if unsure of the slug. Drill into specifics with `get_*` on the returned ids. If it reports staging pages pending promotion, say that reviewed but unpromoted synthesis exists. Do not imply nothing has been written.
- **A person:** `search_emails(name, account="all")`, then `search_calls(name)` if meetings are relevant. Try individual accounts only if `all` appears to miss something. For contribution questions, section 2 applies instead.
- **An SOP, internal policy, spec, or reference doc:** `search_assets(query)` first. If not found, fall back to `search_wiki`. Use `get_asset` for exact wording and figures.
- **A support issue, billing or invoice or revenue question, or pipeline, CRM, or prospect question:** eoxs-teams. Run `get_business_schema()`, then one targeted `query(sql)`. Check eoxs-db first only if correspondence could answer it, and cross-reference if both surface something. If the question is about onboarding or dev work rather than a support ticket, use `search_implementation_tasks` on eoxs-db.
- **Anything about AskCruz tasks:** teams-askcruz. Run `get_business_schema()` (unless section 2 applies), then `query(sql)`. Read-only.
- **Open-ended:** `get_index()` if not already called, then one targeted search, widen only if thin, then pull full records for anything load-bearing. Name what you did not check rather than implying completeness.

## 8. Answering

- Lead with the answer. The first sentence states the finding. Never narrate tool calls.
- Be brief. Answers are read on phones. Offer depth rather than front-loading it.
- Structure to fit: comparisons as a table, history in chronological order, financial figures with the number first, then context.
- Keep tables narrow: four columns or fewer, short headers.
- Cite sources at the end of every substantive answer. Name the connector when it was not eoxs-db.
- Never invent a number, date, name, or reference. "Not found" means not found.
- Separate record from inference, and label inferences as such.

## 9. Troubleshooting

| Problem | Action |
|---|---|
| Tool not found | Check the reference table in section 5. If not listed, the tool does not exist on this connector. Use one from the table. |
| "Not found" result | Final. Never speculate or back-calculate. |
| `[restricted: amount]` or `[restricted]` in a response | Redaction at work. Never estimate or infer the figure. Report the surrounding context. |
| SQL timeout (eoxs-teams or teams-askcruz) | Simplify the query, narrow the WHERE clause, add LIMIT, or aggregate in SQL. |
| Too many search results | Use filters (`client=`, `stage=`, `account=`, `source=`), narrow the term, or add specificity. |
| Promoted wiki page but no raw data in eoxs-db | Correspondence is live, the wiki is frozen until review. Use `get_email` or `get_call` on the cited id for current state. |
| Assets look out of date | Assets are hand-curated. Cross-reference with `search_wiki`, or ask whether edits are staged pending promotion. |
