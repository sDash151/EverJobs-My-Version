# Jobicy Integration Guide

## Overview

Jobicy provides a **public REST API** for remote job listings. No scraping required - clean JSON responses with structured data.

**API Endpoint**: `https://jobicy.com/api/v2/remote-jobs`

---

## Technology Stack

### HTTP Client
- Uses `@ever-jobs/common` `createHttpClient()`
- Supports proxies, custom CA certs, configurable timeouts
- Built-in retry logic and error handling

### Request Configuration
```typescript
Headers: {
  'Accept': 'application/json',
  'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) ...'
}
```

### Query Parameters
- `count`: Number of jobs (max 50)
- `tag`: Search keyword/term (optional)
- `geo`: Location filter (optional)

---

## API Response Structure

### Response Format
```json
{
  "jobs": [
    {
      "id": 150095,
      "url": "https://jobicy.com/jobs/...",
      "jobSlug": "head-of-customer-education",
      "jobTitle": "Head of Customer Education",
      "companyName": "Ashby",
      "companyLogo": "https://jobicy.com/data/.../logo.png",
      "jobIndustry": ["Education & E-learning"],
      "jobType": ["Full-Time"],
      "jobGeo": "USA",
      "jobLevel": "Director",
      "jobExcerpt": "Brief summary...",
      "jobDescription": "<html>Full description...</html>",
      "pubDate": "2026-08-12T03:35:03+00:00",
      "annualSalaryMin": 120000,
      "annualSalaryMax": 180000,
      "salaryCurrency": "USD"
    }
  ]
}
```

### Field Details
- **HTML Content**: `jobDescription` is in HTML format
- **Arrays**: `jobIndustry` and `jobType` are arrays
- **Salary**: Optional fields (can be null/undefined)
- **Remote**: All jobs are remote positions

---

## Implementation Flow

### 1. Make API Request
```typescript
const params = {
  count: Math.min(resultsWanted ?? 50, 50),
  ...(location && { geo: location }),
  ...(searchTerm && { tag: searchTerm })
};

const response = await client.get<JobicyApiResponse>(JOBICY_API_URL, { params });
```

### 2. Process Each Job
```typescript
for (const raw of response.data.jobs) {
  const job = mapJob(raw, descriptionFormat);
  jobs.push(job);
}
```

### 3. Transform Data
- **Description**: Convert HTML → Plain Text or Markdown
  - Plain: `htmlToPlainText(description)`
  - Markdown: `markdownConverter(description)`
- **Compensation**: Build from salary fields
- **Location**: Extract city from `jobGeo`
- **Emails**: Extract from description using `extractEmails()`
- **Date**: Parse and convert to date-only format

### 4. Return Standardized Format
Map to `JobPostDto` with standardized fields for EverJobs

---

## Why Jobicy Works Flawlessly

### 1. **Official API** (No Scraping)
- Stable JSON structure
- No HTML parsing complexity
- No DOM structure changes to handle

### 2. **Clean Data**
- Well-structured response
- Consistent field naming
- Proper data types

### 3. **Rich Metadata**
- Industry tags
- Job level (Senior, Director, etc.)
- Job type (Full-Time, Contract)
- Company logos
- Salary ranges with currency

### 4. **Reliable**
- Cloudflare CDN (fast, cached responses)
- CORS enabled (`Access-Control-Allow-Origin: *`)
- High availability
- Fresh data (real-time updates)

### 5. **Flexible Filtering**
- Keyword search via `tag`
- Location filter via `geo`
- Adjustable result count
- Combinable parameters

---

## Code Example

### Basic Fetch
```typescript
const client = createHttpClient({
  proxies: input.proxies,
  caCert: input.caCert,
  timeout: input.requestTimeout,
});

client.setHeaders({
  Accept: 'application/json',
  'User-Agent': 'Mozilla/5.0 ...'
});

const response = await client.get('https://jobicy.com/api/v2/remote-jobs', {
  params: { count: 10, tag: 'developer', geo: 'USA' }
});

const jobs = response.data.jobs;
```

### Data Mapping
```typescript
const job = new JobPostDto({
  id: `jobicy-${raw.id}`,
  title: raw.jobTitle,
  companyName: raw.companyName,
  companyLogo: raw.companyLogo,
  jobUrl: raw.url,
  location: new LocationDto({ city: raw.jobGeo }),
  description: processDescription(raw.jobDescription, format),
  compensation: buildCompensation(raw),
  datePosted: toDateOnly(raw.pubDate),
  isRemote: true,
  jobLevel: raw.jobLevel,
  companyIndustry: raw.jobIndustry?.join(', '),
  emails: extractEmails(description),
  site: Site.JOBICY
});
```

---

## Key Utilities Used

### From `@ever-jobs/common`:
1. **`createHttpClient()`** - HTTP client with proxy/timeout support
2. **`htmlToPlainText()`** - Strips HTML tags from description
3. **`markdownConverter()`** - Converts HTML to Markdown
4. **`extractEmails()`** - Extracts email addresses from text
5. **`toDateOnly()`** - Formats date to YYYY-MM-DD

### Error Handling
```typescript
try {
  const job = mapJob(raw, descriptionFormat);
  if (job) jobs.push(job);
} catch (err) {
  logger.warn(`Error mapping job ${raw.id}: ${err.message}`);
  // Continue processing other jobs
}
```

**Strategy**: Log individual job errors, don't fail entire batch

---

## Testing

Verify API works:
```bash
node test-jobicy.js
```

Expected: 200 OK, JSON with jobs array, all fields populated

---

## Summary

**What makes it work:**
- ✅ Public REST API (no auth needed)
- ✅ JSON responses (no HTML parsing)
- ✅ Consistent structure
- ✅ Rich metadata included
- ✅ Flexible search filters
- ✅ High reliability (Cloudflare CDN)
- ✅ Proper error handling in code
- ✅ Standard HTTP utilities

**Result**: Clean, maintainable integration with minimal failure points
