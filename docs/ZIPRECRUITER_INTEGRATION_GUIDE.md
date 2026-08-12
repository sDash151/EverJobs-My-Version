# ZipRecruiter Integration Guide

## Overview
- **Platform:** ZipRecruiter (US/Canada + India presence)
- **Method:** REST API (jobs-app API)
- **Authentication:** Basic Auth (public token)
- **Difficulty:** Easy

## API Endpoint
```
https://api.ziprecruiter.com/jobs-app/jobs
```

## How It Works

### 1. Session Initialization
Before searching, initialize a session:

**Request:**
```typescript
POST https://api.ziprecruiter.com/jobs-app/event
```

**Body:**
```json
{
  "device_make": "Apple",
  "device_model": "Macintosh",
  "device_os": "macOS",
  "event_type": "session",
  "device_form_factor": "desktop",
  "platform": "web"
}
```

**Headers:**
```typescript
{
  'Host': 'api.ziprecruiter.com',
  'accept-encoding': 'gzip',
  'authorization': 'Basic YTBlMTk4NjItMjhiYi00YmU3LTlhZDAtZGNhZGMwZjBmY2M5Og==',
  'x-zr-zva-override': 'utm_source:CARNIVAL;utm_medium:NLX',
  'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36'
}
```

### 2. Job Search
After session init, search for jobs:

**Request:**
```typescript
GET https://api.ziprecruiter.com/jobs-app/jobs?search=software+engineer&location=Remote&radius=25&days_ago=30
```

**Parameters:**
- `search` - Search query (optional)
- `location` - Location filter (optional)
- `radius` - Search radius in miles (optional)
- `days_ago` - Max age of jobs in days (optional)
- `jobs_per_page` - Results per request (default: 20)
- `page` - Page number (starts at 1)
- `continue_token` - Token for pagination (from previous response)

**Response Format:**
```json
{
  "jobs": [
    {
      "id": "job_abc123",
      "name": "Senior Software Engineer",
      "hiring_company": {
        "name": "Tech Corp",
        "url": "https://..."
      },
      "location": "San Francisco, CA",
      "posted_time": "2026-01-15T10:00:00Z",
      "posted_time_friendly": "2 days ago",
      "snippet": "Job description preview...",
      "job_url": "https://www.ziprecruiter.com/c/...",
      "salary_interval": "yearly",
      "salary_min": 120000,
      "salary_max": 160000,
      "salary_source": "employer_est",
      "employment_type": "Full-Time",
      "is_remote": true
    }
  ],
  "continue_token": "next_page_token",
  "total_job_count": 150
}
```

### 3. Pagination
Use `continue_token` from response for next page:

```typescript
GET https://api.ziprecruiter.com/jobs-app/jobs?continue_token=next_page_token
```

## Key Features

### Rate Limiting
- Delay between requests: 5 seconds
- Additional delay: 5 seconds (band delay)
- Page size: 20 jobs per request

### Data Extracted
- Job ID
- Job title
- Company name
- Company URL
- Location
- Salary (min/max/interval)
- Posted time
- Employment type
- Remote flag
- Job URL
- Job snippet/description

### Error Handling
- Session init failure: Log warning, continue
- Search failure: Return empty results
- Pagination: Stops at 10,000 jobs hard limit

## Implementation Details

### Job ID Generation
```typescript
id: `ziprecruiter-${job.id || hashCode(job.job_url)}`
```

### Salary Parsing
```typescript
if (job.salary_min || job.salary_max) {
  compensation = {
    interval: job.salary_interval, // yearly, hourly, etc.
    minAmount: job.salary_min,
    maxAmount: job.salary_max,
    currency: 'USD'
  };
}
```

### Location Handling
```typescript
location = {
  city: job.location // Full location string
}
```

### Remote Detection
```typescript
isRemote: job.is_remote ?? false
```

### Employment Type
```typescript
jobType = [job.employment_type] // "Full-Time", "Part-Time", etc.
```

## Usage Example

```typescript
const input = {
  searchTerm: 'react developer',
  location: 'San Francisco',
  resultsWanted: 50,
  proxies: [], // optional
  requestTimeout: 30 // optional
};

const response = await zipRecruiterService.scrape(input);
// Returns JobResponseDto with jobs array
```

## Important Notes

1. **Auth Token:** Public Basic Auth token included in Ever Jobs
2. **Session Required:** Must call event endpoint before search
3. **Continue Token:** Used for pagination instead of page numbers
4. **Hard Limit:** 10,000 jobs maximum per search
5. **Deduplication:** Uses Set to track seen job IDs

## Debugging

### Test Session Init
```bash
curl -X POST 'https://api.ziprecruiter.com/jobs-app/event' \
  -H 'authorization: Basic YTBlMTk4NjItMjhiYi00YmU3LTlhZDAtZGNhZGMwZjBmY2M5Og==' \
  -H 'Content-Type: application/json' \
  -d '{"event_type":"session","platform":"web"}'
```

### Test Job Search
```bash
curl 'https://api.ziprecruiter.com/jobs-app/jobs?search=engineer&location=Remote&jobs_per_page=5' \
  -H 'authorization: Basic YTBlMTk4NjItMjhiYi00YmU3LTlhZDAtZGNhZGMwZjBmY2M5Og=='
```

### Common Issues
- **Empty results:** Session might not be initialized
- **Auth errors:** Check authorization header
- **Pagination stuck:** Verify continue_token usage

## References
- Plugin: `packages/plugins/source-ziprecruiter/`
- Service: `ziprecruiter.service.ts`
- Constants: `ziprecruiter.constants.ts`
