# CareerJet Integration Guide

## Overview
- **Platform:** CareerJet (80+ countries job aggregator)
- **Method:** REST API (affiliate-based)
- **Authentication:** Affiliate ID required (free sign-up)
- **Difficulty:** Medium (requires registration)

## API Endpoint
```
https://public.api.careerjet.net/search
```

## Authentication

### Get Affiliate ID
1. Visit: https://www.careerjet.com/partners/
2. Sign up as affiliate (FREE)
3. Get your `affid` (affiliate ID)
4. Set in `.env`: `CAREERJET_AFFID=your_id_here`

### Without Affiliate ID
Ever Jobs will log warning and return empty results.

## How It Works

### Job Search Request

**Request:**
```typescript
GET https://public.api.careerjet.net/search?affid=YOUR_AFFID&keywords=software+engineer&location=Mumbai&locale_code=en_IN&page=1&pagesize=50&user_agent=EverJobs/1.0
```

**Required Parameters:**
- `affid` - Your affiliate ID (required)
- `user_agent` - User agent string (required)
- `locale_code` - Country/language code (required)

**Optional Parameters:**
- `keywords` - Search query
- `location` - Location filter
- `page` - Page number (starts at 1)
- `pagesize` - Results per page (default: 50)
- `contracttype` - Employment type (P=permanent, C=contract, T=temporary, I=intern, V=voluntary)
- `contractperiod` - Duration (permanent, contract)

**Response Format:**
```json
{
  "type": "PREMIUM+ORGANIC",
  "solr_time": 12,
  "hits": 1234,
  "pages": 25,
  "page": 1,
  "jobs": [
    {
      "title": "Senior Software Engineer",
      "company": "Tech Corp",
      "location": "Mumbai, India",
      "description": "Job description...",
      "url": "https://www.careerjet.co.in/jobad/...",
      "date": "2026-01-15",
      "salary": "₹15,00,000 - ₹25,00,000",
      "salary_type": "yearly",
      "salary_currency_code": "INR",
      "salary_min": 1500000,
      "salary_max": 2500000,
      "site": "naukri.com"
    }
  ]
}
```

## Key Features

### Country Support (80+ countries)
India locale: `en_IN`

All supported locales:
```typescript
USA: 'en_US'
UK: 'en_GB'
India: 'en_IN'
Germany: 'de_DE'
France: 'fr_FR'
// ... 75+ more
```

### Data Extracted
- Job title
- Company name
- Location
- Description
- Job URL
- Posted date
- Salary (min/max/currency/type)
- Source site

### Rate Limiting
- Page size: 50 per request
- Max pages: 10 (configurable)
- No specific delays needed

### Salary Handling
```typescript
if (job.salary_min || job.salary_max) {
  const intervalMap = {
    'Y': 'YEARLY',
    'M': 'MONTHLY',
    'W': 'WEEKLY',
    'D': 'DAILY',
    'H': 'HOURLY'
  };
  
  compensation = {
    interval: intervalMap[job.salary_type],
    minAmount: job.salary_min,
    maxAmount: job.salary_max,
    currency: job.salary_currency_code
  };
}
```

## Implementation Details

### Job ID
```typescript
id: `careerjet-${hashCode(job.url)}`
```

### Location Parsing
```typescript
location = {
  city: job.location // Full location string
}
```

### Description Format
```typescript
if (descriptionFormat === 'plain') {
  description = plainConverter(job.description);
} else if (descriptionFormat === 'markdown') {
  description = markdownConverter(job.description);
} else {
  description = job.description;
}
```

### Contract Type Mapping
```typescript
// P = permanent → FULLTIME
// C = contract → CONTRACT
// T = temporary → CONTRACT
// I = intern → INTERNSHIP
// V = voluntary → VOLUNTEER
```

## Usage Example

```typescript
const input = {
  searchTerm: 'react developer',
  location: 'Bangalore',
  country: 'India', // Auto-converts to 'en_IN'
  resultsWanted: 100,
  auth: {
    careerjet: {
      affId: process.env.CAREERJET_AFFID
    }
  }
};

const response = await careerJetService.scrape(input);
// Returns JobResponseDto with jobs
```

## Important Notes

1. **Affiliate ID Required:** Must sign up (free)
2. **User Agent Required:** Must provide valid user agent
3. **Locale Required:** Auto-detected from country or defaults to en_US
4. **Job Aggregator:** Shows jobs from multiple sources (Naukri, Indeed, etc.)
5. **Source Attribution:** Response includes original source site

## Environment Setup

```bash
# .env file
CAREERJET_AFFID=your_affiliate_id_here
```

Or provide per-request:
```typescript
input.auth = {
  careerjet: {
    affId: 'your_affiliate_id'
  }
};
```

## Debugging

### Test API Request
```bash
curl 'https://public.api.careerjet.net/search?affid=YOUR_AFFID&keywords=engineer&location=Mumbai&locale_code=en_IN&page=1&pagesize=5&user_agent=EverJobs/1.0'
```

### Common Issues
- **Empty results:** Check if `CAREERJET_AFFID` is set
- **Auth error:** Verify affiliate ID is valid
- **Wrong locale:** India should use `en_IN`
- **Missing user_agent:** Required parameter

## Sign-Up Process

1. Go to https://www.careerjet.com/partners/
2. Click "Join as Partner"
3. Fill registration form
4. Receive affiliate ID via email
5. Add to `.env` file
6. Start using API

## References
- Plugin: `packages/plugins/source-careerjet/`
- Service: `careerjet.service.ts`
- Constants: `careerjet.constants.ts`
- Sign up: https://www.careerjet.com/partners/
