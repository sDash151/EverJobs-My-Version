# HackerNews Jobs Integration Guide

## Overview
- **Platform:** HackerNews (YC startup jobs)
- **Method:** Firebase REST API
- **Authentication:** None (public API)
- **Difficulty:** Easy

## API Endpoints

### 1. Get Job Story IDs
```
https://hacker-news.firebaseio.com/v0/jobstories.json
```

### 2. Get Job Details
```
https://hacker-news.firebaseio.com/v0/item/{id}.json
```

## How It Works

### Step 1: Fetch Job IDs
Get array of all job story IDs:

**Request:**
```typescript
GET https://hacker-news.firebaseio.com/v0/jobstories.json
```

**Response:**
```json
[
  37234567,
  37234568,
  37234569,
  ...
]
```
Returns array of ~100-200 job IDs.

### Step 2: Fetch Each Job
For each ID, get full job details:

**Request:**
```typescript
GET https://hacker-news.firebaseio.com/v0/item/37234567.json
```

**Response:**
```json
{
  "by": "username",
  "id": 37234567,
  "score": 1,
  "time": 1705920000,
  "title": "YC Startup (W24) is hiring engineers",
  "text": "<p>We're a YC-backed startup...<p>Apply: https://...",
  "type": "job",
  "url": "https://jobs.example.com"
}
```

**Fields:**
- `id` - Unique job ID
- `title` - Job title
- `text` - Job description (HTML)
- `url` - External job URL (optional)
- `time` - Unix timestamp
- `by` - HN username of poster

## Key Features

### Data Processing

**HTML Parsing:**
```typescript
// Extract job info from HTML text
const html = job.text;
const $ = cheerio.load(html);

// Common patterns:
// - Company name in title or text
// - Email addresses for applying
// - Location in text
```

**Date Conversion:**
```typescript
// Unix timestamp to ISO date
postedDate = new Date(job.time * 1000).toISOString().split('T')[0];
// Example: 1705920000 → "2026-01-15"
```

**Search Filtering:**
```typescript
// Filter by search term
if (searchTerm) {
  const titleMatch = title.toLowerCase().includes(searchTerm.toLowerCase());
  const textMatch = text.toLowerCase().includes(searchTerm.toLowerCase());
  if (!titleMatch && !textMatch) continue;
}
```

### Rate Limiting
- Fetch job IDs: Once
- Fetch individual jobs: In batches
- Batch size: 15 jobs at a time
- Max IDs to process: 100
- No delay needed (Firebase is fast)

### Data Extracted
- Job ID
- Job title
- Company name (parsed from title/text)
- Job description (HTML or plain text)
- Application URL (from `url` field or parsed from text)
- Posted date
- Contact emails (extracted from description)

## Implementation Details

### Job ID
```typescript
id: `hackernews-${job.id}`
```

### Company Name Extraction
```typescript
// Try to extract from title
// Pattern: "Company Name (YC W24) is hiring..."
const companyMatch = title.match(/^([^(]+)/);
companyName = companyMatch?.[1]?.trim() || null;
```

### Description Format
```typescript
if (descriptionFormat === 'plain') {
  description = htmlToPlainText(job.text);
} else if (descriptionFormat === 'markdown') {
  description = markdownConverter(job.text);
} else {
  description = job.text; // Keep as HTML
}
```

### Email Extraction
```typescript
// Extract emails from description
emails = extractEmails(description);
```

### Job URL Priority
```typescript
jobUrl = job.url || `https://news.ycombinator.com/item?id=${job.id}`;
```

## Usage Example

```typescript
const input = {
  searchTerm: 'frontend engineer',
  resultsWanted: 25,
  descriptionFormat: 'markdown'
};

const response = await hackerNewsService.scrape(input);
// Returns JobResponseDto with jobs array
```

## Important Notes

1. **Free API:** No authentication or rate limits
2. **Limited Jobs:** Only ~100-200 active jobs at any time
3. **YC Focus:** Most jobs are from YC-backed startups
4. **Fresh Data:** Updates frequently
5. **HTML Content:** Descriptions are in HTML format
6. **Manual Posting:** Jobs posted manually by founders

## Debugging

### Get All Job IDs
```bash
curl 'https://hacker-news.firebaseio.com/v0/jobstories.json'
```

### Get Single Job
```bash
curl 'https://hacker-news.firebaseio.com/v0/item/37234567.json'
```

### Common Issues
- **Few results:** HN has limited job postings (100-200 total)
- **Missing company:** Not always in title, need to parse text
- **No salary:** Usually not included in HN postings
- **Timestamps:** Need conversion from Unix to ISO date

## API Behavior

### Job Stories Structure
- Array of IDs (sorted by posting time)
- Newest jobs first
- Includes expired/filled positions
- No pagination (returns all IDs)

### Individual Job
- May have `url` pointing to external site
- May have `text` with inline description
- Sometimes both, sometimes only one
- Always has `title` and `time`

## References
- Plugin: `packages/plugins/source-hackernews/`
- Service: `hackernews.service.ts`
- Constants: `hackernews.constants.ts`
- HN API Docs: https://github.com/HackerNews/API
