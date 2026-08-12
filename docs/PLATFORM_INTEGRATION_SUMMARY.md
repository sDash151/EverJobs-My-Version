# Platform Integration Summary

## Overview
This document summarizes how Ever Jobs integrates with 6 major job platforms using REST APIs.

## Platforms

| Platform | Auth Required | API Type | Difficulty |
|----------|--------------|----------|------------|
| Dice | ❌ No | REST (public key included) | 🟢 Easy |
| ZipRecruiter | ❌ No | REST (public token included) | 🟢 Easy |
| HackerNews | ❌ No | Firebase REST | 🟢 Easy |
| Freelancer.com | ❌ No | REST (public) | 🟢 Easy |
| CareerJet | ✅ Yes | REST (affiliate ID) | 🟡 Medium |
| Jooble | ✅ Yes | REST (API key) | 🟡 Medium |

## Quick Start

### 1. Platforms Ready Immediately (No Sign-up)

**Dice:**
```typescript
siteType: ['dice']
```
Details: `DICE_INTEGRATION_GUIDE.md`

**ZipRecruiter:**
```typescript
siteType: ['zip_recruiter']
```
Details: `ZIPRECRUITER_INTEGRATION_GUIDE.md`

**HackerNews:**
```typescript
siteType: ['hackernews']
```
Details: `HACKERNEWS_INTEGRATION_GUIDE.md`

**Freelancer.com:**
```typescript
siteType: ['freelancercom']
```
Details: `FREELANCERCOM_INTEGRATION_GUIDE.md`

### 2. Platforms Requiring Sign-up (Free)

**CareerJet:**
1. Sign up: https://www.careerjet.com/partners/
2. Get affiliate ID
3. Add to `.env`: `CAREERJET_AFFID=your_id`
4. Use: `siteType: ['careerjet']`

Details: `CAREERJET_INTEGRATION_GUIDE.md`

**Jooble:**
1. Sign up: https://jooble.org/api/about
2. Get API key
3. Add to `.env`: `JOOBLE_API_KEY=your_key`
4. Use: `siteType: ['jooble']`

Details: `JOOBLE_INTEGRATION_GUIDE.md`

## Technology Stack

### HTTP Client
All platforms use `createHttpClient()` from `@ever-jobs/common`:
- Proxy support
- Timeout handling
- Custom headers
- Retry logic

### Data Processing
- **Cheerio:** HTML parsing (when needed)
- **extractSalary():** Salary string parsing
- **extractEmails():** Email extraction
- **markdownConverter():** HTML to Markdown
- **htmlToPlainText():** HTML to plain text

### Rate Limiting
- **randomSleep():** Delays between requests
- **Configurable:** Min/max delay per platform

## API Patterns

### Pattern 1: Simple GET (Dice, HackerNews, Freelancer.com)
```typescript
const response = await client.get(API_URL, { params });
```

### Pattern 2: Session + GET (ZipRecruiter)
```typescript
// 1. Initialize session
await client.post(SESSION_URL, sessionData);
// 2. Search jobs
const response = await client.get(JOBS_URL, { params });
```

### Pattern 3: POST with JSON (Jooble)
```typescript
const response = await client.post(API_URL, {
  keywords: 'engineer',
  location: 'Mumbai'
});
```

### Pattern 4: Affiliate-based GET (CareerJet)
```typescript
const response = await client.get(API_URL, {
  params: { affid: AFFILIATE_ID, ...otherParams }
});
```

## Common Features

### All Platforms Support
- Search by keywords
- Location filtering
- Result pagination
- Multiple description formats (HTML/Markdown/Plain)
- Proxy configuration
- Timeout settings

### Data Standardization
All platforms return `JobPostDto` with:
- `id` - Unique job ID
- `title` - Job title
- `companyName` - Company name
- `jobUrl` - Job link
- `location` - LocationDto
- `description` - Job description
- `compensation` - CompensationDto (when available)
- `datePosted` - Posted date
- `jobType` - Employment type
- `isRemote` - Remote flag
- `site` - Source platform

## File Structure

```
docs/
├── DICE_INTEGRATION_GUIDE.md           # Dice implementation
├── ZIPRECRUITER_INTEGRATION_GUIDE.md   # ZipRecruiter implementation
├── HACKERNEWS_INTEGRATION_GUIDE.md     # HackerNews implementation
├── FREELANCERCOM_INTEGRATION_GUIDE.md  # Freelancer.com implementation
├── CAREERJET_INTEGRATION_GUIDE.md      # CareerJet implementation
├── JOOBLE_INTEGRATION_GUIDE.md         # Jooble implementation
└── PLATFORM_INTEGRATION_SUMMARY.md     # This file
```

## Next Steps

1. **Immediate:** Enable Dice, ZipRecruiter, HackerNews, Freelancer.com
2. **Quick:** Sign up for CareerJet and Jooble (15 minutes)
3. **Test:** Run searches to verify integration
4. **Monitor:** Check logs for any issues

## Support

For implementation details, refer to individual integration guides.

Each guide contains:
- API endpoints
- Authentication details
- Request/response formats
- Code examples
- Debugging tips
- Common issues
