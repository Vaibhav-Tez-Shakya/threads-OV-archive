# MCP audit: AskCruz, EOXS, HR – problems & solutions

- Thread ID: 16167aea-612b-4eec-8433-94fb21591c72
- Created: 2026-10-08T13:05:57.178Z
- Updated: 2026-10-08T13:09:45.800Z

---

## User





Suppose you are a senior AI Engineer with 12+ Years of experience check the connected MCP of Askcruz EOXS and hr what problems can you flag out and also provide me a solution for those problems

---

## Claude




Thread is set up. This is an audit task, so I need your rigor level before I start pulling from the AskCruz, EOXS-TEAMS and hr connectors. [Presented rigor options: 5ft deep (Quick), 10ft deep (Verified), 50ft deep (Deep), 100ft deep (Staged).]

---

## User



Q: Which rigor level for the MCP audit (AskCruz, EOXS-TEAMS, hr)?
A: 100ft: Staged, section by section

Q: Which craft level for the final write-up?
A: Sketch

---

## Claude


Staged audit: outline, then Section 1 (access and secrets).

I'm treating the connectors as a pre-launch security and architecture review. Everything below is read-only, and I did not read any secret values or the salary document.

---

## Outline


1. Access control and secrets exposure (below)
2. Data integrity and freshness (AskCruz vs hr drift, EOXS-TEAMS coverage gaps)
3. MCP and tool design (duplicated servers, write paths, schema sprawl)
4. Reliability and observability (logging, limits, failure modes)
5. Retrieval quality (synthesized wiki vs raw sources, stale ingestion)
6. Prioritized fix roadmap tied to the AskCruz $250–500k goal

---

## Section 1: access control and secrets


F1 (High, verified by schema and count queries): EOXS-TEAMS gives raw SQL over the entire Odoo DB (776 tables). res_users.password is queryable (141 users: 134 PBKDF2-hashed, 7 empty, none plaintext). 15 users have a RingCentral access token stored and readable. Tables for mail servers, IMAP fetch, payment acquirers, payslips and 340 employee records also exist in scope.

F2 (High, verified from indexes): The hr server has the same corpus as AskCruz. hr holds 33,965 email threads, 2,275 Fireflies calls and 2,083 wiki pages (AskCruz: 26,831, 1,986 and 1,086). An HR purpose does not need client emails or call transcripts.

F3 (Medium-High, tool exists; scope unverified): hr__update_asset is a write tool. It replaces SOP/reference documents, which are re-ingested into the wiki every 6 hours. One prompt-injected agent call could poison the knowledge base for every user. The tool description says it "may be restricted" to a slug, but I haven't tested that.

F4 (Medium, inferred, not tested): Tiering looks like a label, not enforcement. Assets carry access_tier, and the salary register (tier2_confidential_hr) appears only on the hr endpoint, which suggests separation by endpoint rather than per-user checks in the query layer. Emails, calls and wiki pages show no tier at all.

Assumptions not checked: the credential columns in the mail-server, fetchmail and payment tables are populated (I only confirmed the tables exist); email and call content on the AskCruz endpoint contains sensitive material.

Fixes
- F1: Create a dedicated read-only Postgres role. Grant SELECT only through views on the 11 business tables (sale_order, account_move, res_partner, etc.) with PII columns removed. Revoke on res_users, ir_*, hr_*, payment_*, fetchmail_server and ir_mail_server. Put a SQL parser (e.g. sqlglot) in front of the tool with a table and column allowlist, and keep the existing 30s timeout and 1,000-row cap. Rotate the 15 RingCentral tokens and revoke token storage in a readable column.
- F2: Split servers by purpose. The hr server should serve HR documents and candidate data only. Remove the client email, call and wiki tools from it.
- F3: Move write tools to a separate writer server with a hard slug allowlist. Keep version history and require human approval before re-ingestion. Log the caller, the diff and the timestamp.
- F4: Enforce the tier server-side (e.g. WHERE tier <= token.tier) on every table. Tag emails, calls and wiki pages with a tier at ingest. This ties into your per-server OAuth work.

Next, Section 2: I'll compare AskCruz and hr data drift and test whether the EOXS-TEAMS DB covers what client questions need. Two of the eight registered clients (Brannon, RW Conklin) have no Odoo URL.

Shall I proceed to Section 2?

---

## User

now create a comissioned one

---
