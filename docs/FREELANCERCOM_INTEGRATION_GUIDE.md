# Freelancer.com Integration Guide

## Overview
- **Platform:** Freelancer.com (global freelance marketplace)
- **Method:** REST API (public projects API)
- **Authentication:** None required
- **Difficulty:** Easy

## API Endpoint
```
https://www.freelancer.com/api/projects/0.1/projects/active
```

## How It Works

### Job Search Request

**Request:**
```typescript
GET https://www.freelancer.com/api/projects/0.1/projects/active?compact=true&limit=25&full_description=true&query=nodejs&job_details=true
```

**Headers:**
```typescript
{
  'Accept': 'application/json',
  'User-Agent': 'EverJobs/1.0'
}
```

**Parameters:**
- `compact` - Return compact response (true/false)
- `limit` - Number of results (default: 25, max: 50)
- `full_description` - Include full description (true/false)
- `query` - Search keywords (optional)
- `job_details` - Include additional job details (true/false)
- `min_avg_price` - Minimum budget (optional)
- `max_avg_price` - Maximum budget (optional)
- `offset` - Pagination offset (optional)

**Response Format:**
```json
{
  "status": "success",
  "result": {
    "projects": [
      {
        "id": 12345678,
        "title": "Build a React Native App",
        "description": "Full project description...",
        "seo_url": "/projects/react-native/build-app.html",
        "type": "fixed",
        "currency": {
          "code": "USD",
          "sign": "$"
        },
        "budget": {
          "minimum": 250,
          "maximum": 750
        },
        "time_submitted": 1705920000,
        "owner": {
          "id": 123456,
          "username": "client123"
        },
        "jobs": [
          {
            "id": 45,
            "name": "React Native"
          }
        ],
        "time_free_bids_expire": 1705930000,
        "bid_stats": {
          "bid_count": 15,
          "bid_avg": 500
        }
      }
    ]
  }
}
```

## Key Features

### Data Extracted
- Project ID
- Title
- Description (full text)
- Budget (min/max)
- Currency
- Posted time
- Owner username
- Skills/jobs required
- Bid count
- Project type (fixed/hourly)
- Project URL

### Rate Limiting
- Max results per request: 50
- No delays needed (public API)
- Pagination via `offset` parameter

### Budget Handling
```typescript
if (project.budget) {
  compensation = {
    interval: project.type === 'hourly' ? 'HOURLY' : 'ONE_TIME',
    minAmount: project.budget.minimum,
    maxAmount: project.budget.maximum,
    currency: project.currency.code
  };
}
```

### Skills Extraction
```typescript
skills = project.jobs?.map(job => job.name) || null;
```

### Date Conversion
```typescript
// Unix timestamp to ISO date
datePosted = new Date(project.time_submitted * 1000).toISOString().split('T')[0];
```

## Implementation Details

### Job ID
```typescript
id: `freelancercom-${project.id}`
```

### Job URL
```typescript
jobUrl = `https://www.freelancer.com${project.seo_url}`;
```

### Project Type
```typescript
// Convert to JobType
jobType = project.type === 'hourly' ? ['CONTRACT'] : null;
```

### Description Format
```typescript
if (descriptionFormat === 'plain') {
  description = htmlToPlainText(project.description);
} else if (descriptionFormat === 'markdown') {
  description = markdownConverter(project.description);
} else {
  description = project.description; // HTML
}
```

## Usage Example

```typescript
const input = {
  searchTerm: 'python developer',
  resultsWanted: 50,
  descriptionFormat: 'markdown'
};

const response = await freelancerComService.scrape(input);
// Returns JobResponseDto with projects
```

## Important Notes

1. **Public API:** No authentication needed
2. **Projects, not Jobs:** These are freelance projects/gigs
3. **Budget Instead of Salary:** Shows project budget range
4. **Skills List:** Each project has skill tags
5. **Bidding System:** Shows number of bids received
6. **Max 50 Results:** Hard limit per request

## Debugging

### Test API Request
```bash
curl 'https://www.freelancer.com/api/projects/0.1/projects/active?compact=true&limit=5&full_description=true' \
  -H 'Accept: application/json'
```

### Common Issues
- **Max 50 limit:** Cannot fetch more than 50 per request
- **Pagination:** Use `offset` for additional results
- **Budget only:** No hourly rate for fixed projects

## References
- Plugin: `packages/plugins/source-freelancercom/`
- Service: `freelancercom.service.ts`
- Constants: `freelancercom.constants.ts`
