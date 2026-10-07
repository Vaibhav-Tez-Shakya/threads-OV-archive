# 3GM Client Deep Dive - AskCruz

- Thread ID: 23a728fa-83f8-4ad0-b9a2-31b5f6241f9d
- Created: 2026-10-07T16:44:33.242Z
- Updated: 2026-10-07T16:45:45.189Z

---

## User


give me all the information about the 3gm client of askcruz. dig deep into it and provide me with all the possible information

---

## Claude

## 3GM Steel — Comprehensive AskCruz Client Profile

### **Client Basics**
- **Company:** 3GM Steel
- **Slug:** 3gm-steel
- **Domain:** 3gmsteel.com
- **ERP Instance:** Odoo 13 at https://3gm.eoxs.com (database: threegmsteels)
- **Status:** Active, long-term EOXS customer (original ERP implementation 2022)
- **Industry:** Steel manufacturing and distribution

---

### **Key Contacts**
1. **Travis Lane** — travis@3gmsteel.com (CTO/Technical lead; primary decision-maker for AskCruz deal)
2. **Jessica Worley** — jessica@3gmsteel.com (Implementation contact; heavily involved in onboarding)
3. **Leslie Countryman** — leslie@3gmsteel.com (Additional operational contact)
4. **Matt Inman** — (Access rights elevated to match Jessica Worley's permissions, Aug 2026)

---

### **AskCruz Engagement Status**

#### **Deal Status (Oct 2026)**
- **Product:** AskCruz (Company Brain) — AI-powered knowledge platform
- **Implementation Stage:** Final invoicing + licensing commenced
- **Contract Terms:** 
  - Initial scope: Reduced to 2-user (down from broader scope; Travis negotiated narrower initial commitment)
  - Shorter initial term requested (vs. standard longer commitment)
  - Final implementation invoice sent; first licensing payment received
  - Weekly usage check-ins proposed

#### **Deal Timeline**
- **Aug 12, 2026:** AskCruz Proposal call conducted
- **Aug 2026:** Travis confirms deal at reduced 2-user scope; requests shorter initial term
- **Sep 4, 2026:** Azure AD admin-consent misconfiguration blocking Outlook connection; resolved via admin approval call
- **Oct 2026:** Implementation invoicing complete; weekly check-ins now active

#### **Current Status (As of Oct 6, 2026)**
- ✅ Implementation complete
- ✅ Licensing payments initiated
- ✅ First billing cycle in progress
- 🔄 Weekly usage monitoring established

---

### **Implementation History & Completed Work**

**28 total implementation tasks logged** (dating back to April 26, 2022 — original EOXS onboarding)

#### **Recent/Active AskCruz-Specific Issues Resolved:**
- Outlook integration blocked by Azure AD admin consent → Fixed via admin call (Sep 4, 2026)
- Successful data connection to Odoo 13 ERP database established
- User access (2-seat license) provisioned and tested

#### **EOXS Original Implementation Tasks (2022, now Completed):**
- Kick Off Calls: Access Rights, Company Details, Reports, Ports Information ✅
- Discovery Calls: Travis, Jessica, Leslie, Donnie, Rachel, Adam Buck, etc. ✅
- Master Data Import: Customer Data ✅, Vendor Data ✅, Product Master ✅
- Payment terms: Kick off payment processed ✅

---

### **Communication Volume & Engagement**

**Email Activity:**
- **Total emails:** 964 messages in AskCruz support queue
- **Recent pattern:** Heavy invoicing/payment discussion threads (INV/2026/2368, INV/2026/2403, INV/2026/2434)
- **Active accounts:** support_zoho (primary), ron_gmail, raj_gmail
- **Thread volume:** Ongoing dialogue around product creation, invoice reconciliation, PO/tag tagging issues

**Call Activity:**
- **Total recorded calls:** 13 Fireflies transcripts
- **Most recent:** Aug 12, 2026 — AskCruz Proposal
- **Pattern:** Mix of technical issue reviews, ticket resolution, and strategic discussions
  - Jul 29, 2026: Travis Lane call
  - Jan 10, 2025: Issue review
  - Dec 10, 2024: Ticket Review (S05462)
  - Mar 22, 2024: Meeting
  - Jan 22, 2024: Server Migration discussion

---

### **Known Issues & Operational Context**

**3GM-Specific Odoo Problems (from Wiki/Recent Activity):**
1. **Coil Availability Tagging Bug** (CNJ/OUT/00074, Jun–Sep 2026)
   - Search/availability function failures on tonnage card
   - Ongoing intermittent issues

2. **Invoice Discrepancies (High-Priority Unresolved)**
   - **INV/2026/2403:** Still unresolved after second escalation (Oct 6, 2026)
   - **INV/2026/2368:** Confirmed correct after investigation
   - **INV/2026/2434:** Total amount due mismatch vs. calculated weight × CWT rate (Sep 30, 2026)
   - Pattern: Weight-based pricing calculation errors or field mapping issues

3. **Purchase Order State Bug**
   - Four POs (P04612–P04615) stuck in "Confirmed but Can't Receive" state (Sep 28, 2026)
   - Three also require PO unlock to fix customer field before processing

4. **Coil Inventory/Availability Issues**
   - 12 coils invoiced under INV/2026/2246 still showing as available, distorting inventory/AR reports (Sep 21, 2026)
   - Suggests invoice-to-inventory reconciliation gap

5. **Product Creation Requests**
   - Recurring requests for new color variants (PRM:28 17 variants, PRM:PBR variants in Burgundy & Crimson Red)
   - New coating line product additions (Sep 4, 2026)
   - Hot roll oxide-coated line width corrections (Sep 8–9, 2026)

6. **Coil Upload Errors**
   - Recurring coil-upload errors into purchase orders across ND/IN/00015 and ND/IN/00016 (Sep 22–23, 2026)

7. **PO/Lot-Serial Price Mismatches**
   - Related-price correction requests (e.g., SCP/OUT/02587, Sep 3–4, 2026)

8. **Historical Open Issues**
   - SRTI./OUT/03994: Coil not found when adding to transfer (Mar 2026, still open)
   - S11105 Tonnage card pricing error (MAC Metal Sales – KY, Aug 2026)

9. **Suspected Duplicate Payment**
   - Investigation of duplicate payment claim for Reliable Metal Buildings LLC / INV/2026/2146 (Sep 4, 2026)

---

### **Wiki Coverage & Documentation** 
**40 live wiki pages** (fully indexed and searchable):

**Key Categories:**
- **Implementation:** Kickoff, onboarding, discovery calls, data integration (2022)
- **Operational:** Support tickets, invoicing, billing history, server/platform maintenance
- **Technical:** Coil tagging fixes, bug investigations, product creation requests, data quality issues
- **Strategic:** AskCruz proposal, implementation details, customer story (draft, Oct 5, 2026)
- **Infrastructure:** IRIS AI historical data preload (Aug 2026)

**Pages marked as "restricted":** 3 (redacted for security/sensitivity)

**Most Recent Wiki Activity:**
- Oct 6, 2026: INV/2026/2403 unresolved escalation analysis
- Oct 5, 2026: AskCruz customer story draft submitted to you for review
- Sep 30, 2026: Invoice total mismatch investigation
- Sep 28, 2026: PO state bug root cause analysis
- Sep 21, 2026: Inventory/AR reconciliation issue documentation

---

### **Strategic Insights for AskCruz Growth**

1. **Proof Point:** 3GM is now a **reference customer** for AskCruz — reduced-scope deal but actively engaged with weekly check-ins; positioning for customer story marketing.

2. **Use Case:** Knowledge platform for a complex ERP environment (Odoo 13 with heavy customizations for steel operations) — demonstrates AskCruz's value in manufacturing/distribution sectors.

3. **Operational Issues:** 3GM's ongoing support load is **high** — 964 emails in 4 months, persistent invoice/coil-tagging/PO state bugs suggest either:
   - Opportunity for AskCruz to resolve knowledge gaps (runbooks, troubleshooting guides, root causes)
   - Risk if AskCruz doesn't deliver quick ROI; support burden could overwhelm 2-user license value

4. **Expansion Risk/Opportunity:** Currently licensed for 2 users (Travis, 1 other). Growth into broader team adoption hinges on:
   - Demonstrating concrete time-savings on support ticket resolution
   - Showing impact on operational bug diagnosis and RCA velocity
   - Word-of-mouth to Jessica, Leslie, Matt Inman for broader buy-in

5. **Payment Health:** Invoicing friction (INV/2026/2403, others) is **active** — monitor closely to ensure licensing payments don't stall due to frustration with unresolved operational issues.

---

### **Action Items & Monitoring Points**

| Area | Status | Action |
|---|---|---|
| **Weekly Check-Ins** | Scheduled | Track adoption metrics and support reduction |
| **INV/2026/2403** | 🔴 Open (2nd escalation) | Resolve urgently to maintain payment momentum |
| **AskCruz Customer Story** | 🟡 Pending | Your review (draft Oct 5) — critical for GTM |
| **Product Variant Requests** | Ongoing | Map these to AskCruz's ability to reduce manual ticket creation |
| **Coil Availability Bug** | 🔴 Ongoing | High-impact — use as case study if AskCruz provides workaround documentation |

---
