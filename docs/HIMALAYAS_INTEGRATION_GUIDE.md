# Himalayas Integration Guide

## Overview

**Himalayas** is a remote job board providing **100K+ fresh jobs** via a simple REST API. Jobs are posted daily and sorted by most recent first.

**Status**: ✅ Working perfectly (verified August 2026)

---

## API Endpoint

```
GET https://himalayas.app/jobs/api
```

**Authentication**: None required  
**Rate Limiting**: Unknown (works reliably)  
**Response Format**: JSON

---

## Request Parameters

| Parameter | Type   | Required | Description                    |
|-----------|--------|----------|--------------------------------|
| `limit`   | number | No       | Jobs per page (default: 20)    |
| `offset`  | number | No       | Starting position (default: 0) |

**Example Request:**
```
https://himalayas.app/jobs/api?limit=20&offset=0
```

---

## Required Headers

```http
Accept: application/json
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36
```

---

## Response Structure

```json
{
  "jobs": [...],
  "totalCount": 99747,
  "offset": 0,
  "limit": 20
}
```


## Job Object Schema

```typescript
{
  guid: string;              // Unique job ID
  title: string;             // Job title
  companyName: string;       // Company name
  companyLogo: string;       // Logo URL
  applicationLink: string;   // Apply URL
  description: string;       // HTML format
  excerpt: string;           // Short description
  
  // Compensation
  minSalary: number;         // Minimum salary
  maxSalary: number;         // Maximum salary
  currency: string;          // USD, EUR, etc.
  
  // Location & Remote
  locationRestrictions: string[];   // ["United States", "Canada"]
  timezoneRestrictions: string[];   // ["-8", "-7", "-6"]
  
  // Classification
  employmentType: string;    // "Full Time", "Part Time", "Contract"
  seniority: string[];       // ["Senior", "Mid-level"]
  categories: string[];      // Job categories
  
  // Dates (Unix timestamp in seconds)
  pubDate: number;           // Publication date
  expiryDate: number;        // Expiry date
}
```

---

## Implementation Technology Stack

**Language**: TypeScript  
**Framework**: NestJS  
**HTTP Client**: Custom wrapper (`@ever-jobs/common`)  
**HTML Parser**: Cheerio (for description processing)  
**Markdown Converter**: Turndown  
**Validation**: Zod schemas

---

## EverJobs Implementation Flow

### 1. Plugin Registration
- Location: `packages/plugins/source-himalayas/`
- Service: `HimalayasService` implements `IScraper` interface
- Decorator: `@SourcePlugin({ site: Site.HIMALAYAS, category: 'remote' })`

### 2. Scraping Process
```typescript
1. Initialize HTTP client with headers
2. Loop: while (jobs.length < resultsWanted)
   - Request: GET with limit & offset
   - Parse: JSON response
   - Map: Each job to JobPostDto
   - Deduplicate: Track seen IDs
   - Increment: offset += rawJobs.length
   - Break: if no more jobs or limit reached
3. Return: JobResponseDto
```


### 3. Data Mapping

**Key Transformations:**

| Source Field          | Target Field         | Transformation                          |
|-----------------------|----------------------|-----------------------------------------|
| `guid`                | `id`                 | Prefix: `himalayas-{guid}`              |
| `description`         | `description`        | HTML → Plain/Markdown                   |
| `pubDate`             | `datePosted`         | Unix seconds → ISO date string          |
| `locationRestrictions`| `location.country`   | First element                           |
| `minSalary/maxSalary` | `compensation`       | Build CompensationDto (yearly interval)|
| All jobs              | `isRemote`           | Always `true`                           |

**Email Extraction**: Automatically extracts emails from description text

---

## Critical Success Factors

### ✅ What Makes It Work

1. **No Authentication** - Simple GET requests
2. **Stable Endpoint** - API hasn't changed
3. **Fresh Data** - Jobs posted daily, sorted newest first
4. **Reliable Pagination** - Standard limit/offset pattern
5. **Rich Metadata** - Salary, location, seniority, categories
6. **Clean JSON** - No HTML scraping needed
7. **No Rate Limits** - Works without throttling

### ⚠️ Important Notes

- **HTML Descriptions**: Need conversion to plain text/markdown
- **Unix Timestamps**: In seconds (multiply by 1000 for JS Date)
- **Deduplication**: Required when fetching multiple pages
- **Timezone Format**: String numbers like "-8", "-7"
- **All Remote**: Every job is remote-friendly

---

## Configuration Constants

```typescript
HIMALAYAS_API_URL = 'https://himalayas.app/jobs/api'
HIMALAYAS_PAGE_SIZE = 20  // Jobs per request
```


---

## Quick Implementation Guide

### Minimal Working Example (Node.js)

```javascript
const https = require('https');

async function fetchHimalayasJobs(limit = 20, offset = 0) {
  const url = `https://himalayas.app/jobs/api?limit=${limit}&offset=${offset}`;
  
  return new Promise((resolve, reject) => {
    https.get(url, {
      headers: {
        'Accept': 'application/json',
        'User-Agent': 'Mozilla/5.0'
      }
    }, (res) => {
      let data = '';
      res.on('data', chunk => data += chunk);
      res.on('end', () => resolve(JSON.parse(data)));
    }).on('error', reject);
  });
}

// Usage
const response = await fetchHimalayasJobs(10, 0);
console.log(`Fetched ${response.jobs.length} of ${response.totalCount} jobs`);
```

### Required Processing Steps

1. **Build Job ID**: `himalayas-${job.guid}`
2. **Convert Date**: `new Date(job.pubDate * 1000).toISOString().split('T')[0]`
3. **Extract Country**: `job.locationRestrictions[0]`
4. **Build Compensation**: Check if `minSalary` or `maxSalary` exists
5. **Convert Description**: HTML → Plain text or Markdown
6. **Set Remote Flag**: Always `true`

---

## Testing

**Test Script**: `test-himalayas.js`

```bash
node test-himalayas.js
```

**Expected Results:**
- ✅ Status 200
- ✅ 20 jobs per request
- ✅ All jobs posted today or recently
- ✅ Total count ~100K jobs
- ✅ Pagination works (offset increments)

---

## File Locations

```
packages/plugins/source-himalayas/
├── src/
│   ├── himalayas.service.ts      # Main scraper logic
│   ├── himalayas.module.ts       # NestJS module
│   ├── himalayas.types.ts        # TypeScript interfaces
│   ├── himalayas.constants.ts    # API URL & headers
│   └── index.ts                  # Barrel exports
└── package.json
```

---

## Last Verified

**Date**: August 12, 2026  
**Status**: ✅ Fully operational  
**Jobs Available**: 99,747  
**Freshness**: All tested jobs posted same day
