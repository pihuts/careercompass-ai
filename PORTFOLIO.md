# CareerCompass AI - Portfolio Blurb

## One-liner

An n8n automation that scrapes Hacker News job postings, uses GPT-5.6 Sol to score each job against my profile, drafts a personalized cover letter, tracks strong matches in Google Sheets, and emails me a daily digest.

## Problem

Job hunting on HN's "Who is hiring?" thread means reading hundreds of unstructured posts, manually judging fit, and writing a tailored cover letter for each promising role. That is hours of repetitive work per month, and promising roles get missed.

## Solution

I built an n8n workflow that handles the full pipeline:

1. Query the Hacker News Algolia API for the latest "Who is hiring?" thread.
2. Fetch every job comment from the HN item API.
3. Clean the raw HTML and pass each job to GPT-5.6 Sol with a strict JSON schema.
4. The model extracts structured fields and scores the role 0-100 against my real profile.
5. Strong matches (>= 75) are upserted to Google Sheets by HN item ID, so daily runs never duplicate rows.
6. A daily email digest arrives with match reasons, missing skills, a suggested email subject, and a ready-to-review cover letter draft.

## Design decisions

- **Structured output, not free text.** The LLM must return a validated JSON schema, which keeps the data pipeline predictable.
- **Human in the loop.** The automation prepares the work; I review and send applications. No spam, no burned reputation.
- **Right-sized model.** `gpt-5.6-sol` handles scoring and cover-letter quality; a 10-job limit keeps costs low.
- **No duplicate rows.** Upsert keyed on HN item ID means the workflow is safe to run daily.
- **Configurable profile.** The candidate profile lives in one node and can be overridden with an environment variable.

## Stack

- n8n
- Hacker News Algolia API + HN Firebase API
- OpenAI GPT-5.6 Sol with structured output parsing
- Google Sheets API
- Gmail API

## Skills demonstrated

- API integration and pagination handling
- Data cleaning and transformation
- LLM prompt engineering with strict JSON output
- Workflow orchestration (scheduling, branching, error resilience)
- Persistence and idempotency
- Product thinking: automation that augments a human instead of replacing judgment

## Result

Every morning I open one email that already contains the jobs worth my time, why they fit, what I am missing, and a cover letter draft. What used to be a multi-hour monthly scan is now a 10-minute review.
