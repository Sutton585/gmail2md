# gmail2md: Email Knowledge Accumulator

gmail2md is a specialized Sandman module designed to transform your inbox into a structured, chronological knowledge base. It targets specific Gmail labels, extracts high-signal content, and generates Markdown notes optimized for Obsidian vaults and AI-driven RAG (Retrieval-Augmented Generation) pipelines.

This module is part of the **Sandman Suite** and adheres strictly to the [Unified Module Blueprint](../unified_module_blueprint.md).

---

## 1. Core Mission & Use Case

Unlike traditional email archiving, gmail2md is built for **active knowledge management**. Its primary mission is to:
- **Target Signal:** Monitor specific labels (e.g., `Alerts/Jobs`, `Newsletters`, `Research`) rather than the whole inbox.
- **Extract Links & Entities:** Automatically identify and queue external URLs (like Job Postings) for secondary scraping modules.
- **Maintain Chronology:** Append new replies or updates to existing notes, preserving the full context of a thread.

### The Job Alert Pipeline (MVP Goal)
In its MVP state, this module is optimized to handle **Job Alert Emails** (LinkedIn, Indeed, etc.).
1. **gmail2md** scrapes the alert email.
2. It identifies the "Job Alert" label and triggers the `Processor` to extract job titles and URLs.
3. It creates an abbreviated "Job Page" for each link, establishing a relational link between the **Source Email** and the **Target Job**.

---

## 2. Key Dependencies

- **imaplib & email (Standard Library):** Used for connecting to Gmail via the IMAP protocol and parsing raw MIME email content. This was chosen to maintain zero-dependency reliability for the core connection.
- **Google Client Library (Optional):** Supports the official Gmail API for high-volume users who require OAuth-based authentication.
- **BeautifulSoup (bs4):** Used by the Processor to parse HTML email bodies and extract embedded job board URLs.

---

## 3. Implementation: The 5 Buckets

Following the Sandman Blueprint, the internal architecture is divided into:

1. **Config (Settings):** Manages `label` targeting, `max_post_age` (to avoid historical bloat), and `auth_directory` paths.
2. **Client (Network):** Supports **Google OAuth (Gmail API)** for high-volume users and **IMAP** for users wanting a simpler, protocol-based connection.
3. **Processor (Translation):**
    - Maps `Message-ID` to `post_id`.
    - Sanitizes HTML body into clean Markdown.
    - **Entity Extraction:** Specifically looks for patterns matching job board URLs to populate the `jobs` metadata.
4. **DatabaseManager (State):** Tracks which `Message-ID` has been processed to ensure zero duplicates.
5. **Orchestrator (Loop):** Iterates through labels, handles the "Job Scraper" handoff, and executes the State Reconciliation Flow.

---

## 3. Platform Specifics & Extended Schema

### Unique Front-Matter Fields
- `sender`: The "From" address.
- `label`: The Gmail label being targeted.
- `thread_id`: The Gmail thread identifier (used for appending replies).
- `extracted_links`: A list of URLs found in the body.
- `jobs_scraped`: (Boolean/Int) Status of the secondary job-extraction pass.

### Behavioral Toggles
- `max_post_age_days`: (Default: 5) Only process emails newer than X days.
- `capture_unread_only`: (Boolean) If true, only process emails not yet marked as read.
- `mark_as_read`: (Boolean) Whether to mark the email as read after a successful scrape.

---

## 4. Addressing the Architectural Questions

### Q1: Universal Scraping & Noise Removal
**The Problem:** Re-writing scrapers for every site is inefficient.
**The Solution:** Leverage "AI-Native" scraping engines.
- **Recommendation:** Use **Trafilatura** or **Crawl4AI**.
    - **Trafilatura** is the "Gold Standard" for clean text extraction (noise removal). It excels at finding the "meat" of an article while ignoring headers/footers.
    - **Crawl4AI** is a modern, open-source crawler specifically for RAG. It handles JavaScript (essential for LinkedIn/Indeed) and outputs "Fit Markdown."
- **Strategy:** Instead of a "Universal Scraper" module, build a **`web2md`** utility module that uses `Crawl4AI`. Other modules (like Gmail) simply pass a URL to this utility.

### Q2: Orchestration Layer (Sandman vs. Others)
**The Problem:** Building a custom UI/logic layer is time-consuming.
**The Solution:** Use **n8n** for "Plumbing," keep **Sandman** for "Logic."
- **Recommendation:** **n8n** is the most mature self-hosted automation tool. It has built-in Gmail, Discord, and Telegram nodes.
- **Sandman's Role:** Don't build a complex UI for Sandman. Use Sandman as a **Suite of CLI Tools**. 
    - Use **n8n** to *trigger* Sandman (via shell commands or Docker). 
    - Sandman provides the "Living Notes" logic and "Standard Schema" that generic tools like n8n or Paperclip lack.
- **Paperclip AI:** Better for "Autonomous Agents" with budgets. For a structured knowledge pipeline, Sandman's deterministic "Job Model" is safer.

### Q3: Monitoring Job Sites (APIs vs. Scrapers)
**The Problem:** Job sites are aggressive and APIs are restricted.
**The Solution:** Use **JobSpy**.
- **Recommendation:** **JobSpy** is a Python library that aggregates LinkedIn, Indeed, Glassdoor, and ZipRecruiter into a single Pandas DataFrame. It handles the "API-like" querying without needing official (and often impossible-to-get) API keys.
- **Aggregator Strategy:** Focus 50% on "High-Signal" sources (JobSpy/LinkedIn/Indeed) and 50% on "Conventional" sources (Direct RSS feeds from niche boards, company career pages).

### Q4: Organizing the Next Steps
**The Strategy:** A Three-Tiered Module Architecture.
1.  **Tier 1: Source Scrapers (e.g., `gmail2md`, `reddit2md`)**
    - Responsibility: Identify URLs and high-level context.
2.  **Tier 2: Entity Extractors (e.g., `job2md`)**
    - Responsibility: Take a URL (from Gmail or Manual Input) and turn it into a "Job Note" using **JobSpy** or **Crawl4AI**.
3.  **Tier 3: Utility Modules (e.g., `web2md`)**
    - Responsibility: General HTML -> Markdown conversion for any URL not handled by a specific Entity Extor.

**Manual Input:** Support this via a simple **CLI command** or **Webhook**.
Example: `python sandman.py --module job2md --url [LINK]`. 
This allows your Apple Shortcut to simply send an HTTP POST to your server, which then triggers the Job Scraper.

---

## 5. Next Steps for gmail2md
1.  **Architecture:** Draft the `docs/architecture.md` using `imaplib` for the Client bucket (fastest path to MVP).
2.  **Job Scraper:** Create a sibling directory `modules/job2md/` and initialize it with **JobSpy**.
3.  **Integration:** Define the "Handoff Schema" where gmail2md writes a list of `extracted_links` that `job2md` picks up in the next cycle.
