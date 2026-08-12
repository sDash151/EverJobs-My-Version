# Jooble Integration Guide

## Overview
- **Platform:** Jooble (70+ countries job aggregator)
- **Method:** REST API (key-based)
- **Authentication:** API key required (free sign-up)
- **Difficulty:** Medium (requires registration)

## API Endpoint
```
https://jooble.org/api/{API_KEY}
```

## Authentication

### Get API Key
1. Visit: https://jooble.org/api/about
2. Sign up for API access (FREE)
3. Get your API key
4. Set in `.env`: `JOOBLE_API_KEY=your_key_here`

### Without API Key
Ever Jobs will fail to make requests.

## How It Works

### Job Search Request

**Request:**
```typescript
POST https://jooble.org/api/{YOUR_API_KEY}
```

**Headers:**
```typescript
{
  'Content-Type': 'application/json',
  'Accept': 'application/json'
}
```

**Body:**
```json
{
  "keywords": "software engineer",
  "location": "Bangalore",
  "page": "1"
}
```

**Parameters in Body:**
- `keywords` - Search query (required)
- `location` - Location filter (optional)
- `page` - Page number as string (starts at "1")
- `radius` - Search radius in km (optional)
- `salary` - Minimum salary (optional)
- `datecreatedfrom` - Date from (YYYY-MM-DD) (optional)

**Response Format:**
```json
{
  "totalCount": 1234,
  "jobs": [
    {
      "title": "Senior Software Engineer",
      "location": "Bangalore, Karnataka",
      "snippet": "Job description preview...",
      "salary": "$120,000 - $160,000 per year",
      "company": "Tech Corp",
      "type": "Full-Time",
      "link": "https://jooble.org/redirect/...",
      "updated": "2026-01-15T10:30:00.0000000",
      "id": 1234567890123456
    }
  ]
}
```

## Key Features

### Country Support (70+ countries)
API automatically handles country based on location or defaults to US.

### Data Extracted
- Job ID
- Job title
- Company name
- Location
- Snippet/description
- Salary (parsed from string)
- Job type
- Job URL
- Posted/updated date

### Rate Limiting
- Page size: 20 per request
- No specific delays needed
- Pagination via `page` parameter

### Salary Parsing
```typescript
// Parse salary string like "$120,000 - $160,000 per year"
const match = salary.match(/[$€£₹]?\s*(\d{1,3}(?:,\d{3})*|\d+)\s*[-–]\s*[$€£₹]?\s*(\d{1,3}(?:,\d{3})*|\d+)/);

if (match) {
  const min = parseInt(match[1].replace(/,/g, ''));
  const max = parseInt(match[2].replace(/,/g, ''));
  
  // Detect interval from keywords
  const interval = detectInterval(salary);
  // "per year" → YEARLY
  // "per hour" → HOURLY
  // etc.
  
  compensation = { interval, minAmount: min, maxAmount: max };
}
```

### Currency Detection
```typescript
const currencyMap = {
  '$': 'USD',
  '€': 'EUR',
  '£': 'GBP',
  '₹': 'INR'
};
```

## Implementation Details

### Job ID
```typescript
id: `jooble-${job.id}`
```

### Location Parsing
```typescript
location = {
  city: job.location // Full location string
}
```

### Job Type Mapping
```typescript
// Parse from type field
// "Full-Time" → FULLTIME
// "Part-Time" → PARTTIME
// "Contract" → CONTRACT
// "Internship" → INTERNSHIP
```

### Date Handling
```typescript
// Parse ISO date string
datePosted = job.updated.split('T')[0];
// "2026-01-15T10:30:00.0000000" → "2026-01-15"
```

### Email Extraction
```typescript
emails = extractEmails(job.snippet);
```

## Usage Example

```typescript
const input = {
  searchTerm: 'python developer',
  location: 'Mumbai',
  resultsWanted: 50,
  auth: {
    jooble: {
      apiKey: process.env.JOOBLE_API_KEY
    }
  }
};

const response = await joobleService.scrape(input);
// Returns JobResponseDto with jobs
```

## Important Notes

1. **API Key Required:** Must sign up (free tier available)
2. **POST Request:** Unlike most APIs, uses POST with JSON body
3. **Page as String:** Page number must be string ("1", "2", etc.)
4. **Job Aggregator:** Shows jobs from multiple sources
5. **Salary Parsing:** Complex string parsing required
6. **Redirect URLs:** Job links are Jooble redirects

## Environment Setup

```bash
# .env file
JOOBLE_API_KEY=your_api_key_here
```

Or provide per-request:
```typescript
input.auth = {
  jooble: {
    apiKey: 'your_api_key'
  }
};
```

## Debugging

### Test API Request
```bash
curl -X POST 'https://jooble.org/api/YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"keywords":"engineer","location":"Bangalore","page":"1"}'
```

### Common Issues
- **Missing API key:** Check environment variable
- **Page as number:** Must be string, not number
- **Empty location:** Defaults to US if not specified
- **Salary parsing:** May fail if format unusual

## Sign-Up Process

1. Go to https://jooble.org/api/about
2. Fill API access request form
3. Receive API key via email
4. Add to `.env` file
5. Start using API

## Free Tier Limits

Typical free tier includes:
- Limited requests per day
- Basic job data
- Standard support

Check Jooble's website for current limits.

## References
- Plugin: `packages/plugins/source-jooble/`
- Service: `jooble.service.ts`
- Constants: `jooble.constants.ts`
- Sign up: https://jooble.org/api/about
