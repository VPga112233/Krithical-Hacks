Download the ZIP file and also download Node.js from ' https://nodejs.org/en/download/current ' 
Read RealREADME.md in zip file


# VouchAI

Evidence-backed professional discovery. Instead of guessing or hallucinating credentials, VouchAI searches the web in real time, pulls down live web pages, and checks every requirement against verbatim quotes from the source text.

---

## Why this exists

Most AI directories and aggregators make claims they can't back up—stating someone has 10 years of experience or a specific certification based on outdated or hallucinated data.

VouchAI flips this model: **no claim is marked as supported unless there is an exact quote from an independent public page to prove it.** If the page doesn't explicitly mention a credential, it's marked as *Not Stated*.

---

## Quick Start

The repository comes pre-configured with active API keys in `.env` and sample warm cache data in `cache.json`.

### 1-Click Launch (Windows)
Double-click **`start.bat`**. It checks for Node.js, installs dependencies if needed, launches `server.js`, and opens your browser to `http://localhost:3000`.

### Manual Launch (Windows / Mac / Linux)
```bash
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
npm install
node server.js
```
Then visit **http://localhost:3000** in your browser.

---

## Testing the App (2-Minute Test)

1. **Instant Verified Result**: Keep the default search (*Chartered Accountant*, *5+ years*, *India*, *Delhi*) and click **Find Providers**. Results return with green **Supported** badges showing the exact sentences extracted from the provider's live website.
2. **Live Streaming Pipeline**: Open *Advanced Settings* in the left sidebar, uncheck *Use local cache*, pick a profession (like *Lawyer* or *Mechanical Engineer*), and click **Find Providers**. You will see the live terminal logs stream events in real time as TinyFish queries the web and Groq evaluates candidates.
3. **Fallback Candidates**: If you search for an unusually strict combination where no candidate matches 100% of criteria, VouchAI automatically surfaces the top 2 closest public leads so you never get an empty dead-end screen.
4. **Export**: Click **Export CSV** or **Export JSON** to save findings.

---

## How It Works Under the Hood

The discovery pipeline runs through five stages on each search:

1. **Plan**: Based on the profession, location, experience, and certifications selected, the backend creates targeted search queries and evaluation criteria.
2. **Search (TinyFish Search API)**: Runs parallel search queries across public websites and LinkedIn profiles.
3. **Fetch (TinyFish Fetch API)**: Headless agents render each candidate's site and extract the raw text as clean Markdown.
4. **Judge (Groq)**: An LLM checks each criterion against the Markdown text. It can only assign a verdict if it quotes the exact snippet from the page. Any quote not physically present in the text is rejected.
5. **Corroborate**: If a candidate is found on LinkedIn, the engine runs an independent search on third-party sites to confirm their identity and practice.

---

## TinyFish AI Integration

We use **TinyFish AI** as the core web crawling and retrieval layer. Traditional scrapers (like Puppeteer or Cheerio) fail quickly in production because modern firm websites use dynamic JavaScript frameworks (React, Next.js), aggressive bot walls, or cookie banners.

TinyFish provides two APIs that solve this, integrated in `pipeline.js`:

### 1. Search API (`api.search.tinyfish.ai`)
Used in `findUrls()` to run parallel search queries targeted by profession, country, state, and certifications:

### 2. Fetch API (`api.fetch.tinyfish.ai`)
Used in `readPages()` to batch-load candidate websites and convert them to clean Markdown:


**Why Markdown output is essential:**
- Raw HTML is filled with navigation menus, scripts, CSS, and footers that waste LLM context window and cause hallucinations.
- TinyFish renders the JavaScript on the page and returns clean Markdown. This allows Groq's model to focus purely on the text of the candidate's bio, practice areas, and credentials.

### 3. LinkedIn Independent Cross-Check
When a candidate profile is discovered on LinkedIn, `corroborate()` queries TinyFish for non-LinkedIn websites mentioning their name and profession. TinyFish fetches the third-party page so Groq can confirm the person is an active practitioner outside of their own social profile.

---

## Tech Stack

- **Runtime**: Node.js v24 (ES Modules)
- **Server**: Express.js with Server-Sent Events (SSE) for real-time streaming
- **Web Crawler & Search**: TinyFish AI (`api.search.tinyfish.ai` & `api.fetch.tinyfish.ai`)
- **LLM Verification**: Groq Cloud (Llama 3.3 70B Versatile & Llama 3.1 8B Instant)
- **Frontend**: Clean vanilla JavaScript SPA (no build step, instant load, dark & light mode)
- **Offline Data**: ISO-3166 dataset covering 249 countries and 5,000+ subdivisions

---

You can check API connection status at any time by visiting `http://localhost:3000/api/health`.

---

## Project Structure

```
├── server.js             # Express server and SSE streaming endpoint
├── pipeline.js           # Core discovery pipeline (TinyFish + Groq + quote checks)
├── config.js             # Supported occupations, qualifications, and specialties
├── countries-data.json   # Offline ISO country and state dataset
├── start.bat             # 1-click launcher for Windows
├── cache.json            # Deterministic query cache
├── package.json          # Node.js dependencies
└── public/
    ├── index.html        # App layout and dashboard
    ├── style.css         # Dark/light theme styles
    ├── app.js            # Client-side SSE stream reader, UI logic, and exports
    └── logo.png          # VouchAI logo
```

---
