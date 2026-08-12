# Dice Integration Guide

## Overview
- **Platform:** Dice.com (US tech jobs)
- **Method:** REST API (primary) + HTML fallback
- **Authentication:** API key (public, included in Ever Jobs)
- **Difficulty:** Easy

## API Endpoint
```
https://job-search-api.svc.dhigroupinc.com/v1/dice/jobs/search
```

## How It Works

### 1. Primary Method: REST API
Ever Jobs uses Dice's public REST API with a static API key.

**Request:**
```typescript
GET https://job-search-api.svc.dhigroupinc.com/v1/dice/jobs/search?q=software+engineer&location=Remote&page=1&pageSize=20&countryCode2=US&language=en
```

**Headers:**
```typescript
{
  'Accept': 'application/json',
  'Content-Type': 'application/json',
  'x-api-key': '1YAt0R9wBg4WfsF9VB2778F5CHLAPMVW3WAZcKd8',
  'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36'
}
```

**Parameters:**
- `q` - Search query (optional)
- `location` - Location filter (optional)
- `page` - Page number (starts at 1)
- `pageSize` - Results per page (default: 20)
- `countryCode2` - Country code (default: US)
- `language` - Language (default: en)

**Response Format:**
```json
{
  "data": [
    {
      "id": "abc123",
      "jobId": "123456",
      "title": "Senior Software Engineer",
      "companyName": "Tech Corp",
      "detailsPageUrl": "/job-detail/abc123",
      "formattedLocation": "San Francisco, CA",
      "jobLocation": {
        "displayName": "San Francisco, CA"
      },
      "isRemote": false,
      "payRateRange": {
        "min": 120000,
        "max": 160000
      },
      "salary": "$120,000 - $160,000",
      "summary": "Job description...",
      "postedDate": "2026-01-15",
      "modifiedDate": "2026-01-16",
      "employmentType": "FULLTIME"
    }
  ]
}
```

### 2. Fallback Methods
If API fails:
1. **Cheerio HTML scraping** - Parse search results HTML
2. **Playwright** - Browser automation (last resort)

## Key Features

### Rate Limiting
- Delay between requests: 2-4 seconds
- Page size: 20 jobs per request
- Pagination supported

### Data Extracted
- Job title
- Company name
- Job URL
- Location
- Salary (when available)
- Posted date
- Employment type
- Remote flag
- Job description

### Error Handling
- API failure → Cheerio fallback
- Cheerio failure → Playwright fallback
- All failures → Return empty array

## Implementation Details

### Job ID Generation
```typescript
// From API response
id: `dice-${job.id}`

// Or from URL
id: `dice-${extractedFromUrl}`

// Fallback: hash URL
id: `dice-${hashCode(jobUrl)}`
```

### Location Parsing
```typescript
location: job.formattedLocation ?? job.jobLocation?.displayName ?? null
```

### Salary Parsing
1. Use `payRateRange` if available (min/max)
2. Fall back to parsing `salary` string
3. Extract using `extractSalary()` utility

### Remote Detection
```typescript
isRemote: job.isRemote ?? location?.includes('remote')
```

## Usage Example

```typescript
const input = {
  searchTerm: 'react developer',
  location: 'Remote',
  resultsWanted: 50,
  proxies: [], // optional
  requestTimeout: 30 // optional
};

const response = await diceService.scrape(input);
// Returns JobResponseDto with jobs array
```

## Important Notes

1. **API Key:** Public key included in Ever Jobs (no sign-up needed)
2. **Rate Limits:** Respect 2-4 second delays
3. **Pagination:** Automatic, fetches multiple pages if needed
4. **Country:** Defaults to US (countryCode2: 'US')
5. **Reliability:** API is primary, very reliable

## Debugging

### Check API Response
```bash
curl 'https://job-search-api.svc.dhigroupinc.com/v1/dice/jobs/search?q=engineer&page=1&pageSize=5&countryCode2=US' \
  -H 'x-api-key: 1YAt0R9wBg4WfsF9VB2778F5CHLAPMVW3WAZcKd8' \
  -H 'Accept: application/json'
```

### Common Issues
- **Empty results:** Check search term / location
- **API errors:** Falls back to HTML automatically
- **Timeout:** Increase `requestTimeout` parameter

## References
- Plugin: `packages/plugins/source-dice/`
- Service: `dice.service.ts`
- Constants: `dice.constants.ts`
- Types: `dice.types.ts`
