# Cemstone EPD Data Collection Solution — Source of Truth

**Last Updated:** Case Competition Prep  
**Purpose:** Full context document for any new chat session. Captures all decisions, rationale, open questions, and implementation details for the Cemstone EPD data collection solution.

---

## 1. Company & Problem Background

**Cemstone** is a Minnesota-based ready-mix concrete producer with **80 concrete production facilities**. Concrete accounts for approximately 90% of their carbon footprint. Their EPD work is handled by engineers and specialists as a non-primary responsibility — there is no single dedicated owner.

**Environmental Product Declarations (EPDs)** are independently third-party verified documents, required under Minnesota's **Buy Clean Buy Fair Act**, that disclose the environmental impact of a product across its lifecycle. They are a competitive advantage for Cemstone when bidding on government and private projects, as buyers increasingly require them.

Cemstone's EPD partner is **Climate Earth EPD**, which handles the actual EPD generation and verification. Cemstone's role is entirely **data collection and submission** — they gather the required data and send it to Climate Earth.

### EPD Lifecycle Stages in Scope (A1–A3 only)

Cemstone reports **cradle-to-gate** EPDs, meaning only stages A1 through A3. They are a materials supplier, not a builder, so downstream stages are out of scope.

- **A1 — Raw Materials:** Environmental data for each ingredient in the concrete mix (cement, fly ash, slag, aggregates, admixtures, etc.), sourced from 20–30 suppliers per facility. This includes both the emissions from creating/sourcing the ingredient AND delivery emissions to the supplier's facility.
- **A2 — Delivery to Cemstone:** Emissions from transporting raw materials from each supplier to the Cemstone facility. Calculated from: distance × transport mode × fuel/emission factor.
- **A3 — Manufacturing at Cemstone:** Energy usage at the Cemstone facility during concrete production — electricity, natural gas, diesel, water, waste disposal, and recycling efforts.

### Current Process Pain Points

| Stage | Core Problems |
|---|---|
| A1 | 20–30 supplier contacts per facility; suppliers don't understand data requirements; no urgency/priority on their end; back-and-forth delays; takes 3–6 months average |
| A2 | Unknown supplier starting locations; inconsistent or unknown transport modes; no systematic way to calculate distances; same info re-collected each time |
| A3 | Manual "detective work" — visiting multiple utility provider websites individually to download billing records; data spread across multiple portals; no centralized source |

**Current Timeline:** 3 months average, 6 months maximum per facility  
**Certification target:** 3–5 new facilities per year, plus ongoing re-certifications for 9 already-certified facilities

---

## 2. Solution Overview

The solution has two major layers:

**Layer 1 — Process Optimization (Searching Phase):** Redesign how data is collected for A1, A2, and A3 to reduce manual effort, reduce communication delays, and produce cleaner output data that is easy to feed into AI.

**Layer 2 — AI Recording Phase:** A scoped, rules-based AI system (similar in concept to Claude Cowork — no-code, easy for employees to use and audit) that lives in a shared folder. It reads all collected data files and populates a pre-existing Excel template. A review agent verifies accuracy. Final human review checks Excel against source files before upload to Climate Earth.

The goal is not to eliminate human involvement — it is to **shift human effort away from data entry and administrative burden toward judgment, oversight, and relationship management**, while producing a more consistent and accurate output.

---

## 3. Layer 1 — Process Optimization (Searching Phase)

### A1 — Supplier Environmental Data Collection

#### Problem
- Suppliers do not understand what data is being requested
- Suppliers do not prioritize Cemstone's requests — no urgency signal
- Communication is unstructured (emails, phone calls, back-and-forth clarification)
- Same supplier data is re-collected from scratch for each new facility certification, even when the supplier and material have not changed

#### Solution

**1. NetSuite Online Supplier Form**  
At the start of any EPD data collection cycle for a facility, Cemstone sends each relevant supplier a link to a **NetSuite native online form** (NetSuite's external-facing form capability, sometimes called SuiteForms or Online Customer Forms). Suppliers do not need a NetSuite login to complete this form.

The form is designed around the exact data fields required by the EPD PCR (Product Category Rules), so suppliers receive a structured, specific ask rather than a vague email. Form design:

- **Required fields:** Structured data inputs for each specific environmental metric needed (e.g., GWP in kg CO₂e per unit, energy source type, declared unit)
- **File upload option:** Suppliers who prefer to submit their own EPD document, LCA report, or data sheet can upload a file instead of entering data manually. This accommodates suppliers of varying sophistication.
- **Catchall/notes field:** Free text for any additional context, caveats, or supporting information the supplier wants to include.
- **Clear deadline language:** The form includes explicit language stating the requested response timeline (e.g., one week).

Form submissions flow directly into NetSuite as structured records, automatically associated with the relevant supplier and facility.

**2. Automated Follow-Up Emails**  
If a supplier has not responded within a set interval (e.g., one week), an automated follow-up email is triggered. This is managed via **NetSuite's workflow engine** integrated with Cemstone's existing email provider.

The email is a standard template, sent as a reply to the original outreach:  
*"We sent you a form for supplier information input one week ago and have not received a response. This information is critical to our operations and a timely response is greatly appreciated. Please contact [name/email/phone] if you need any assistance."*

The follow-up logic requires a **response status tracker** (see Section 5 — Data Architecture).

**3. Supplier Data Library (Reuse Across Facilities)**  
Once a supplier's environmental data has been collected and verified for one facility, it is **stored in NetSuite as a supplier record**. When a new facility certification begins, the system checks whether current data already exists for each required supplier before sending a new form.

- If data exists and is within its validity window → no outreach needed, data is reused
- If data is expired, or if the supplier has indicated a change in material or process → new form is sent

This eliminates redundant outreach for suppliers already in the system, which is significant given Cemstone's growing portfolio of certified facilities. This is not a complex integration — it is a lookup against records already being stored in NetSuite from form submissions.

**Output of A1:** A combination of:
- Structured JSON records from NetSuite (for suppliers who completed the form)
- Uploaded files (PDFs, EPD documents, data sheets) for suppliers who chose file upload
- Both are placed into the AI input folder for a given facility

---

### A2 — Transport Distance Calculation

#### Problem
- Supplier shipping origin locations are unknown or inconsistently recorded
- Transport modes vary per supplier and are not systematically documented
- No existing process for calculating actual route-based distances
- Distances are re-calculated (or estimated) from scratch each certification cycle

#### Solution

**1. Supplier Location Collection at Relationship Start**  
When Cemstone begins a new supplier relationship, a **one-time onboarding form** is sent (also a NetSuite online form) that collects:
- Supplier shipping origin address (mandatory field)
- Primary transport mode(s) used to deliver to Cemstone facilities (truck, rail, barge, or combination)
- Any known secondary/handoff points if multimodal

This data is stored in NetSuite as part of the supplier's vendor record and is a one-time collection — it does not need to be re-collected unless the relationship changes.

**2. Distance Calculation via APIs**  
Using the stored supplier address and the known Cemstone facility address, distances are calculated automatically via API based on transport mode:

- **Truck (road):** Google Routes API — returns route-based driving distance in miles/km, not straight-line. Supports batch requests (multiple supplier origins against one facility destination in a single API call). Rail/transit modes are also supported within this API.
- **Rail:** Google Routes API (transit mode with RAIL preference) — **TBD: confirm whether freight rail routes are accurately covered, as Google's transit routing is primarily built for passenger rail. This remains an open question.**
- **Barge / Inland Waterway:** Searoutes Routing API (Sea + Inland product) — returns accurate inland waterway route distances considering traffic separation, canals, and waterway-specific routing. Also returns SECA/ECA zone distances separately, which is relevant for emission factor differentiation. Searoutes also offers a dedicated CO2 API that calculates emissions directly from route data — worth evaluating as a potential addition.
- **Multimodal/Combination:** Distance is calculated per leg using the appropriate API for each leg, then summed.

**3. Distance Storage in NetSuite**  
Calculated distances are stored in NetSuite as records associated with the supplier-facility pair. Distance is not recalculated on every certification cycle — it is only recalculated if the supplier's shipping origin or transport mode changes.

**Master Supplier + Distance Data Export for AI:**  
A **master supplier file** (JSON or CSV) is maintained and exported from NetSuite. This file contains:
- All suppliers, their addresses, and their transport modes (master-level, facility-agnostic)
- A **per-facility distance file** that maps each supplier to its calculated distance for a specific Cemstone facility

Both files are placed in the AI input folder at the start of each EPD cycle. The AI reads these files and pulls the relevant supplier-distance data for each ingredient when populating the Excel template. The employee exports these files from NetSuite — this is a human-initiated export, not an automated pull, to ensure the employee is deliberately selecting the correct and current data.

**Output of A2:** A structured JSON or CSV file containing supplier names, transport modes, and calculated distances for the facility being certified. Placed in the AI input folder.

---

### A3 — Facility Energy & Manufacturing Data

#### Problem
- Energy usage data (electricity, natural gas, diesel, water, waste) is spread across multiple utility provider portals and internal billing records
- An employee must manually visit each portal, log in, locate billing history, and download records
- Data may also exist in NetSuite via accounts payable, but extraction is inconsistent
- Utility providers vary significantly in their digital capabilities — no single standard applies universally

#### Decision
After evaluating multiple approaches (Green Button standard, utility API aggregation platforms, RPA portal scraping, email-based capture), the team chose to **keep A3 as a human-driven process** with the goal of **centralizing and organizing the collection**, rather than fully automating it. This avoids the reliability and coverage risks of automated scraping, ensures an employee is actively verifying source accuracy, and keeps the solution scope manageable.

#### Solution

**Centralized A3 Email Inbox**  
Cemstone establishes a **dedicated email address** for all utility and billing-related correspondence (e.g., `facilitydata@cemstone.com` or a facility-specific variant). This inbox becomes the single destination for:
- Utility billing statements (wherever utilities send email copies)
- Any billing notifications or account summaries received via email

**Organized Folder Structure**  
Within the inbox, **sender-based folders** are configured to automatically sort incoming emails by utility provider. This eliminates the need to hunt across multiple inboxes or portals — an employee can go to one place and find all billing data organized by source.

**Human Download & Deposit**  
An employee (the designated EPD process owner — see Section 7) visits each utility portal as needed, downloads the relevant billing period records, and deposits them into the AI input folder for that facility. The employee may also forward any relevant emails from the centralized inbox into the folder.

The key shift from the current process: rather than hunting across accounting files and portals with no clear ownership, **one employee owns this task with a clear destination** (the centralized inbox and the AI input folder). The detective work is reduced to a known, bounded set of sources.

**Output of A3:** Billing PDFs, downloaded statements, and/or email content — placed in the AI input folder. These are unstructured or semi-structured files. The AI handles extraction.

---

## 4. Layer 2 — AI Recording Phase

### Concept
A scoped, rules-based AI system that operates on a shared folder. The intent is a **no-code, easy-to-use system** that any employee can operate and audit — conceptually similar to Claude Cowork. It is not a general-purpose AI agent; it has narrow, well-defined instructions tied to Cemstone's specific EPD data template and calculation rules.

### Input Folder Contents
At the start of each facility's EPD cycle, the designated employee assembles the following into the AI input folder:

| File | Source | Format |
|---|---|---|
| A1 supplier data (form responses) | NetSuite export | JSON |
| A1 supplier files (EPD docs, data sheets) | Supplier uploads | PDF, image, or other |
| A2 master supplier + distance file | NetSuite export | JSON or CSV |
| A3 billing records | Employee download/deposit | PDF, email text, image |
| Excel EPD template | Cemstone standard template | .xlsx |

Files should ideally be organized/named in a way that corresponds to the order of the Excel template, to make the human review step easier.

### Processing Agent
The AI agent reads all files in the folder and populates the Excel template. Its behavior is:
- **Rules-based:** It knows exactly which data fields to extract and where they map in the Excel
- **Calculation-aware:** It understands specific EPD calculations (e.g., distance × emission factor, unit conversions) and performs these rather than just transcribing raw numbers. Uses hardcoded tools/skills for calculations to avoid nondeterministic errors, and encourage adherence to actual processes.
- **Format-agnostic:** It can extract data from structured JSON, PDFs, images, or email text — whichever format each file takes
- **Non-autonomous:** It does not make judgment calls or fill in gaps with assumptions. If a required field cannot be found in the source files, it flags it rather than estimating.

### Review Agent
A second AI agent reviews the populated Excel against the source files. Its behavior:
- Verifies that each value in the Excel matches its corresponding source file
- If a discrepancy is found, it returns that specific field to the processing agent for re-extraction
- This loop continues until all fields are verified
- **The review agent does not interact with a human during this loop** — it is fully automated

### Output
A completed, AI-verified Excel file. This file is the output of the recording phase and requires no human involvement to produce.

---

## 5. Upload & Review Phase (Human)

### Process
1. The designated employee opens the AI input folder (now containing both source files and the output Excel)
2. Source files should be sorted/named in Excel order to facilitate efficient review
3. The employee goes through each source file and verifies that the corresponding Excel entry matches — the **only question being checked is: does the Excel match the source file?**
4. The employee is not re-doing calculations or re-interpreting data — they are purely confirming transcription accuracy
5. Once satisfied, the employee uploads the Excel to Climate Earth EPD for processing and verification

---

## 6. Response Status Tracker (NetSuite)

To support automated follow-up logic for A1, a lightweight **custom record type in NetSuite** tracks supplier form status per facility per EPD cycle. Each record contains:

| Field | Description |
|---|---|
| Supplier | Link to vendor record |
| Facility | Which Cemstone facility this cycle is for |
| EPD Cycle / Year | Identifier for the certification cycle |
| Form Sent Date | When the form link was sent |
| Form URL | Link to the specific form sent |
| Response Received | Boolean (yes/no) |
| Response Date | Date form was completed or file uploaded |
| Follow-Up Sent | Boolean — has the automated follow-up triggered? |
| Follow-Up Date | When follow-up was sent |
| Notes | Manual notes field for the process owner |

NetSuite's **workflow engine** reads this record. The trigger condition is:
- `Response Received = No` AND
- `Days since Form Sent Date >= 7` (or configured interval) AND
- `Follow-Up Sent = No`

When all conditions are met, the workflow sends the standard follow-up email via Cemstone's connected email provider and sets `Follow-Up Sent = Yes`.

This is not considered overcomplicating the solution — the tracker is a necessary component for the follow-up automation to function, and it also gives the process owner a single dashboard view of outstanding responses across all suppliers for a given facility cycle.

---

## 7. Process Ownership

A **single designated employee** takes ownership of the EPD data collection process. This person is responsible for:
- Sending supplier form links at the start of each certification cycle
- Monitoring the response status tracker in NetSuite
- Assembling the AI input folder (depositing A3 billing files, exporting A1/A2 data from NetSuite)
- Initiating the AI recording phase
- Performing the final human review
- Uploading the completed Excel to Climate Earth

The efficiencies created by this solution (eliminating manual data entry, automating follow-ups, reusing supplier data across facilities) are expected to free up enough time within the existing team that this can become a defined part of one employee's role, rather than being distributed across multiple engineers and specialists as it currently is.

---

## 8. Data Architecture Summary

| Data Type | Storage Location | Format | Who Creates It |
|---|---|---|---|
| Supplier environmental data (A1) | NetSuite custom record | Structured record (from form) | Supplier (via form) |
| Supplier uploaded files (A1) | NetSuite file attachment | PDF / image / doc | Supplier (via form) |
| Supplier address & transport mode | NetSuite vendor record | Structured record | Supplier (via onboarding form) |
| Calculated distances (A2) | NetSuite record (supplier-facility pair) | Structured record | Calculated via API, stored by employee/system |
| Master supplier + distance export | AI input folder | JSON or CSV | Employee exports from NetSuite |
| A3 billing records | AI input folder | PDF / email / image | Employee downloads and deposits |
| Response status tracker | NetSuite custom record | Structured record | System (auto-updated by workflows and form submissions) |
| Output Excel | AI input folder | .xlsx | AI recording agent |

---

## 9. Services, APIs & Integrations Used

This section is intended to support a teammate doing financial/cost analysis. Pricing should be researched independently based on usage volume.

### Google Routes API (formerly Distance Matrix API)
- **Purpose:** Calculate road driving distances between supplier shipping origins and Cemstone facilities (A2 — truck transport). Also supports transit modes including rail.
- **Pricing model:** Pay-per-use, billed per element (origin-destination pair). Research current pricing at [Google Maps Platform Pricing](https://mapsplatform.google.com/pricing/)
- **Notes:** Supports batch requests (multiple supplier origins against one facility in a single call). Rail support via transit mode — freight rail coverage should be verified as a separate open question (see Section 10).

### Searoutes Routing API (Sea + Inland product)
- **Purpose:** Calculate barge and inland waterway distances between supplier origins and Cemstone facilities (A2 — barge/water transport)
- **URL:** https://searoutes.com/routing-api/
- **Pricing model:** Subscription-based. Research current pricing at Searoutes website.
- **Notable capability:** Returns SECA/ECA zone distances separately (relevant if emission factors differ within vs. outside these zones). Also offers a dedicated **CO2 API** that calculates emissions directly from routes — worth evaluating as an optional addition.

### NetSuite (Cemstone's existing ERP)
- **Purpose (multiple uses):**
  - Host supplier intake forms (Online Forms / SuiteForms — external-facing, no supplier login required)
  - Store supplier environmental data records (A1)
  - Store supplier address and transport mode records (A2)
  - Store calculated distance records (A2)
  - Response status tracker (custom record type)
  - Workflow engine for automated follow-up email triggers
  - Data export for AI input files
- **Pricing:** Cemstone already uses NetSuite. Evaluate whether additional modules or custom record types require license upgrades.
- **Notes:** NetSuite's Online Customer Forms are accessible via a publishable external URL — no supplier login required. They are tied to the CRM module and require both **Customer Relationship Management and Sales Force Automation features** to be enabled in NetSuite. For supplier-specific data collection (as opposed to customer/lead capture), a **custom record type** ("Supplier EPD Submission") should be created in NetSuite with a corresponding external form. This avoids any record-type mismatch and keeps supplier submissions clearly separated from customer data. Custom record types and external forms are available within standard NetSuite SuiteCloud capabilities — this does not require the Vendor Portal or Procurement add-on. **Whether Cemstone's current license has CRM/SFA enabled must be confirmed.** If not, enabling it would be required, and may involve a module add-on cost.

### AI Recording System (Cowork-style)
- **Purpose:** Process the AI input folder, populate the Excel EPD template, run review loop
- **Concept:** No-code, rules-based system with a narrow scope tied to Cemstone's specific EPD workflow. Employees interact via a simple folder-based interface — no technical skill required.
- **Comparable product:** Claude Cowork (Anthropic) — a desktop automation tool for non-developers
- **Pricing/licensing:** Dependent on platform selected. Research current Claude Cowork or equivalent pricing.
- **Notes:** The AI system has two agents: a processing agent (extracts data and populates Excel) and a review agent (verifies accuracy, loops back to processing agent if issues found). Both operate without human interaction. Final output is a verified Excel file.

### Cemstone's Existing Email Provider
- **Purpose:** Sending automated follow-up emails triggered by NetSuite workflows
- **Integration:** NetSuite has native email sending capabilities that connect to standard email providers. No separate integration likely needed.

---

## 10. Open Questions & TBD Items

| Item | Status | Notes |
|---|---|---|
| Rail distance API coverage | Open | Google Routes API supports transit rail, but this is primarily passenger rail. Freight rail routing coverage needs verification. If not adequate, a freight-specific rail distance API should be identified. |
| NetSuite Online Forms feature availability | Confirm | Verify that Cemstone's current NetSuite license includes the Online Forms / SuitePortal capability needed to host external supplier forms. |
| Distance API trigger | Resolved | API call triggers when an EPD cycle is initiated for a facility. When the process owner creates a new EPD Cycle record in NetSuite for a given facility, the system checks each required supplier for that facility. If a stored distance already exists for that supplier-facility pair, it is reused. If no distance exists yet (new supplier or new facility pairing), the API call is made, the result is stored in NetSuite, and no repeat call is needed for future cycles unless the supplier address or transport mode changes. |
| Searoutes CO2 API | Optional | Searoutes offers direct emissions calculation from route data — could replace a manual calculation step in A2. Evaluate whether this simplifies the AI recording step. |
| EPD validity and recertification trigger | Open | EPDs are valid for ~5 years, but early recertification may be beneficial (e.g., if a supplier reduces their emissions). No automated flagging mechanism is currently planned — this is a future consideration. |
| Green Button Connect | Ruled out | Not universally adopted across all utility providers serving Cemstone's facilities. Not included in solution. |
| Utility bill management services (Urjanet, etc.) | Ruled out | Too complex for current scope. Not included in solution. |
| Public EPD database pre-population | Ruled out | Too complex in terms of setup and integrations. Not included in solution. |

---

## 11. What This Solution Does NOT Include

To maintain clarity for judges and keep scope manageable:
- **No automation of A3 data retrieval** — A3 remains a human-driven collection process, just more organized and centralized
- **No scraping of utility portals** — brittle and unreliable
- **No Green Button / utility API integration** — not universally available
- **No public EPD database integration** — adds setup complexity
- **No full end-to-end automation** — human review is a deliberate and permanent step, not a temporary gap
- **No changes to Cemstone's relationship with Climate Earth EPD** — the solution ends at uploading the completed Excel to Climate Earth

---

## 12. NetSuite Licensing — Tier Analysis & Cemstone Assessment

### NetSuite Tier Structure (Current as of 2025–2026)

NetSuite has four service tiers based on user count, monthly transaction volume, and file storage:

| Tier | Users | Monthly Transaction Lines | File Storage | Notes |
|---|---|---|---|---|
| **Standard** | Up to 100 | Up to 200,000 | 100 GB | No additional tier fee beyond base license |
| **Premium** | Up to 1,000 | Higher volume | 1,000 GB | Additional fee |
| **Enterprise** | Up to 2,000 | High volume | 2,000 GB | Additional fee |
| **Ultimate** | Up to 4,000 | Very high volume | 4,000 GB | Additional fee |

Base platform license starts at approximately **$999/month**. Full user licenses run approximately **$99–199/month per user**. Advanced modules are add-ons priced at approximately **$300–$1,500+/month** each depending on complexity.

### What Cemstone Likely Has

Cemstone is a mid-size Minnesota concrete producer with 80 facilities and an estimated employee count in the hundreds. They use NetSuite for accounting and billing purposes (referenced in case materials as the source of billing records). Based on this profile:

- **Edition:** Mid-Market Edition — most likely. They have multiple facilities (effectively multiple locations), more than 10 users, and operational complexity that exceeds the Starter tier. They are not large enough to need Enterprise.
- **Service Tier:** Almost certainly **Standard** — 80 facilities with a finance/operations team of reasonable size would stay well within 100 users and 200,000 monthly transaction lines for a concrete producer's accounting operations. No additional tier fee expected.
- **Modules likely active:** Core ERP (GL, AP, AR, purchasing, basic inventory), possibly basic reporting. A concrete producer's primary NetSuite use case is financial management and procurement — not CRM or marketing.
- **Modules likely NOT active:** CRM / Sales Force Automation (not core to a B2B concrete producer's operations), Procurement add-on / Vendor Portal (possible but not certain), advanced workflow add-ons.

### Implication for This Solution

The critical dependency is the **CRM / Sales Force Automation feature**, which is required to enable NetSuite's external-facing Online Forms. If Cemstone does not currently have this enabled, it would need to be added. This is a feature toggle within NetSuite — it may be included in their current license at no additional cost (many Mid-Market licenses include CRM), or it may require a module add-on.

**If CRM is not currently active:** Cemstone has two options:
1. Enable CRM/SFA within NetSuite (confirm whether this is included or requires a paid module)
2. Use a lightweight external form tool instead (e.g., a simple web form or Google Form that emails submissions to the centralized inbox) — simpler but loses the native NetSuite integration benefit

All other NetSuite capabilities required by this solution — custom record types, workflow engine, vendor records, saved search exports, email sending — are standard features included in the base Mid-Market license.

---

## 13. Cost Items for Financial Analysis

This section lists all cost items associated with the solution. Actual pricing should be researched and confirmed by the teammate doing financial modeling. Ranges and notes are provided as starting context only.

### One-Time / Setup Costs

| Cost Item | Description | Estimated Range | Notes |
|---|---|---|---|
| NetSuite customization (development) | Creating custom record types (Supplier EPD Submission, EPD Cycle, Response Tracker), external form configuration, workflow setup, saved search/export configuration | $5,000–$20,000 | Depends on whether Cemstone uses an internal NetSuite admin or an external NetSuite partner. Simpler if Cemstone already has an internal admin. |
| Distance API integration setup | Building the logic that calls Google Routes API and Searoutes API when a new EPD Cycle record is created, stores results in NetSuite | Included in NetSuite customization above, or $2,000–$8,000 separately | Depends on implementation approach |
| AI recording system setup | Configuring the AI agent with Cemstone's EPD Excel template, field mapping rules, calculation logic, and review agent | TBD — dependent on platform selected | Anthropic Claude Cowork or equivalent. One-time configuration effort. |
| Process documentation & training | Documenting the new workflow, training the designated EPD process owner | 20–40 hrs of internal employee time | Not a cash cost but a time cost |

### Recurring / Annual Costs

| Cost Item | Description | Pricing Model | Estimated Range | Notes |
|---|---|---|---|---|
| Google Routes API | Distance calculations for truck/rail transport (A2). Triggered per new supplier-facility pair only — not per certification cycle if data is reused. | Pay-per-use (per element/pair) | Very low — likely <$100/year | At Cemstone's scale, total number of unique supplier-facility pairs is bounded. Most calls happen during initial onboarding. Research current pricing at Google Maps Platform. |
| Searoutes API | Distance calculations for barge/inland waterway transport (A2). Same reuse logic — called once per supplier-facility pair. | Subscription | ~€1,500/year (approx. $1,600–1,700 USD) | From Searoutes published pricing. Confirm current rate. Consider whether CO2 API adds-on is worth including. |
| AI recording system | Ongoing subscription for the Cowork-style AI tool used for the recording phase | Subscription | TBD | Research Claude Cowork pricing or equivalent platform. |
| NetSuite CRM/SFA module (if not already active) | Required to enable external Online Forms | Add-on module fee | $300–$1,000/month | **Only applicable if not already in Cemstone's license. Must be confirmed.** This is the highest-risk cost item. If CRM is already included, this line item is $0. |
| NetSuite base license (existing) | Already paid by Cemstone — not a new cost | — | $0 incremental | Standard tier, no upgrade expected |

### Cost Items That Are Explicitly Not New Costs

| Item | Reason |
|---|---|
| NetSuite platform subscription | Already in use |
| Employee time for EPD process | Redirected from existing engineers — net reduction in total time, not a new headcount |
| Climate Earth EPD subscription | Pre-existing relationship — not affected by this solution |

---

## 14. Solution Narrative for Judges

The current EPD data collection process at Cemstone is slow, labor-intensive, and bottlenecked at three distinct points: suppliers who don't respond, distances that aren't systematically tracked, and energy data that's scattered across multiple portals. Cemstone's engineers spend months doing work that is fundamentally administrative.

Our solution does not try to solve everything with AI. Instead, it redesigns the process so that the right work is done by the right tool: humans maintain judgment and relationships, structured systems handle storage and communication, and a narrow AI handles the tedious task of turning a folder of mixed files into a clean Excel. The result is a process that is faster, more consistent, cheaper to run, and owned by a single person rather than distributed across an engineering team.
