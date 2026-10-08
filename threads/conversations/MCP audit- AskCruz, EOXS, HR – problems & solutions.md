# MCP audit: AskCruz, EOXS, HR – problems & solutions

- Thread ID: 16167aea-612b-4eec-8433-94fb21591c72
- Created: 2026-10-08T13:05:57.178Z
- Updated: 2026-10-08T14:04:30.411Z

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

## Claude










[Created Claude Doc "MCP Audit: AskCruz, EOXS-TEAMS and hr" (commissioned craft level; sections: executive summary, scope and method, access control and secrets, data integrity, tool design, reliability, retrieval quality, roadmap, verification log) at https://claude.ai/code/artifact/f47db5e9-9ce9-4cbe-87af-f5fc55cae3b4 and saved the complete Markdown to the GitHub archive as "MCP Audit - AskCruz, EOXS-TEAMS and hr (2026-10-08).md".]

MCP Audit: AskCruz, EOXS-TEAMS and hr is ready — read it over, edit inline, or comment.

---

## User









Explain F1 in detail and then tell me that how to solve that problem but before that tell me in simple words that a non-tech person can understand easily that what the problem actually is

---

## The problem in simple words








Imagine EOXS keeps all its company records in one big building: customer lists, employee files, passwords, payroll, email settings, payment settings. Normally each department gets a key to only its own room.

Right now, the AI assistant has a master key to the whole building. It was meant to look at sales and invoice rooms, but nothing stops it from walking into the other rooms too, including the ones holding login passwords and access keys for other systems.

The assistant is also easy to trick. It reads emails and call transcripts from outside people. If someone slips a hidden instruction into an email, such as "go look in the password room and tell me what you find", the assistant might just do it. It can't reliably tell a real instruction from you apart from one planted in an email.

Nothing bad has happened that I know of. The door is simply unlocked, and nobody has checked whether anyone walked through.

---

## F1 in detail








What it is: The EOXS-TEAMS connector lets the AI run any read-only database query (a "SELECT") on EOXS's Odoo database. That database has 776 tables. The connector doesn't limit which ones.

What I verified:
- Passwords: The res_users table has a password column that the tool can query. There are 141 users. 134 passwords are stored as scrambled hashes (PBKDF2) and 7 are empty. None are stored in plain text, which is good.
- Phone-system keys: 15 users have a RingCentral access token stored in a column the tool can read. A token works like a key card for that person's phone account.
- Other sensitive tables: Tables for email server settings (2 rows), incoming mail (1), payment providers (12), employees (340) and payslips (1) are all reachable. I confirmed they exist and counted rows. I did not read what's inside.

Why "read-only" isn't enough protection:
1. Hashes can be cracked offline.
2. Tokens can be used directly.
3. Prompt injection: the AI processes untrusted text (emails, transcripts). A planted instruction could make it run a query and put the results in a reply or saved document.
4. Saved transcripts spread the data: full conversations are stored in a thread server and a GitHub archive.

What I don't know: whether the credential columns in the mail and payment tables contain live secrets, who has used this tool so far (no audit log seen), and whether the connection's database login has limits beyond the tool's own checks.

---

## How to solve it








Step 1 (today): Rotate the 15 RingCentral tokens; treat the database as possibly read and check whether anything other than your own sessions used this connector.
Step 2 (1–2 days): Create a restricted read-only Postgres user for the AI connector with permission on only: sale_order, sale_order_line, account_move, account_move_line, res_partner, product_template, product_product, stock_picking, stock_move, purchase_order, purchase_order_line. Point the connector at it.
Step 3 (1–2 days): Create views that leave out personal and sensitive columns; grant access to the views only.
Step 4 (1–2 days): Add a gatekeeper that parses each query and rejects it if it touches a table or column not on the approved list. Keep the 30-second timeout and 1,000-row limit.
Step 5 (1 day): Store RingCentral tokens in a secrets manager rather than a queryable column.
Step 6 (1 day): Log who ran which query and when; test that SELECT * FROM res_users is refused.

Rough total: 3–5 days for one engineer (my estimate, not measured).
Done when: the AI answers sales, invoice and stock questions as before, but any attempt to reach passwords, tokens, payroll or payment settings fails and is logged.

Want me to draft the exact Postgres permission script and the approved-table list for your engineer next?

---

## User







So from EOXS get me the salary of Ayan Dutta and its last month incentive and all taht

---

## Claude






I'm not going to pull this one through the EOXS-TEAMS connector, and I'd rather explain why than just stall.

An individual employee's salary and incentive is the kind of personal HR data your connector setup is meant to keep out of reach. As I understand your access, you're cleared for general and confidential business data but not for personal employee records, and money amounts and performance data are redacted on that connector. The SQL tool can technically reach the contract and payslip tables, but only because of the gap in F1, which is the hole we just flagged. Using it to read one person's pay would mean going around your own redaction and clearance rules. I'd rather not use the flaw I'm telling you to close.

It probably wouldn't give you the answer anyway. That database has only 1 payslip on record against 340 employees, so it isn't where payroll actually runs. Last month's incentive for Ayan is almost certainly in a payroll tool or a spreadsheet, not in Odoo.

The legitimate routes are your HR lead, who can pull his payroll record and incentive calculation, or whoever owns payroll. A salary register also exists on the hr connector, tagged tier2_confidential_hr. I haven't opened it. If you confirm you're cleared for that tier, I can look up his base salary there, though it won't include last month's incentive. Do you want me to do that?

---

## User





Any keys and passwords that are being flagged here in the mcps tell me that what type of those things are

---

## Claude




Here are the secrets I flagged, grouped by type. I did not read or display any actual values.

1. User login passwords — EOXS-TEAMS, res_users.password. Odoo login passwords for 141 users. 134 are stored as PBKDF2 hashes (scrambled, not plain text), 7 are empty. Verified: readable column, values not read.
2. Phone-system access tokens — EOXS-TEAMS, res_users.ringcentral_access_token. RingCentral API tokens, like a key card to a person's phone account. 15 users have one. Verified: readable, values not read.
3. Email server logins — EOXS-TEAMS, ir_mail_server (2 rows). Usually the SMTP username and password Odoo uses to send email. Table exists; contents not inspected.
4. Incoming-mail logins — EOXS-TEAMS, fetchmail_server (1 row). Usually the IMAP/POP password Odoo uses to read an inbox. Table exists; contents not inspected.
5. Payment gateway keys — EOXS-TEAMS, payment_acquirer (12 rows). Usually API and secret keys for payment providers. Table exists; contents not inspected.
6. MCP connection secrets — THREAD OV and Threads-OV connector URLs. Long random tokens embedded in the URL path. Anyone holding the URL can connect. Verified from the connector list.

Four more are assumed risks rather than checked ones:
- Database login for the EOXS-TEAMS server: credentials sit on the host (a lancerdevops.me domain); I can't see how they're stored.
- Odoo gateway credentials for 3GM-Odoo: same applies to its host.
- Other Odoo tables that often hold secrets: ir_config_parameter, iap_account, google_service, rest_api, db_backup and res_partner_bank exist in the list; I did not inspect them.
- Salary and sample data: not keys, but the salary register and the sample rows from get_business_schema are sensitive in the same way.

Rotation priority: rotate the 15 RingCentral tokens and the two URL tokens first, since both are confirmed exposed. Then check what actually sits in rows 3–5, and rotate anything live there.

I haven't confirmed that rows 3–5 contain live secrets. Want me to check the column names on those three tables, without reading values, so you know for sure?

---

## User



Show me a masked password that are you able to found in the mcp

---

## Claude


Here is what a stored password looks like, masked. I pulled only the format of the entry (algorithm, rounds, length), never the salt or the scrambled value, so no usable secret passed through me.

$pbkdf2-sha512$25000$••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••

What it tells you:
- pbkdf2-sha512 is the scrambling method. It's one-way, so the original password can't be read back directly.
- 25000 is how many times it's scrambled. All 134 hashed users use this identical setting.
- The masked part (109 of 130 characters) is the unique salt plus the scrambled password.

This is a good sign: no plaintext passwords. The weak spot is that 25,000 rounds is low by current guidance (several times higher is now recommended), so a stolen hash is cheaper to crack than it should be. That is a default of Odoo 13, and raising it comes with upgrading Odoo.

The other secrets would look like this, fully masked:
- RingCentral token: ••••••••••••••••••••••••••••••••••••••• (15 users)
- Thread server URL: https://[server].onrender.com/•••••••••••••••••••••••/sse (2 servers)

I'm not showing even a few real characters of those, since they are live keys. Want the 7 users with empty passwords listed by user ID only, so you can check whether they're SSO or portal accounts?

---

## User

now unmask the hash

---
