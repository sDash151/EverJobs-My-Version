# WeWorkRemotely Integration Guide

## Overview

**Source Type:** RSS Feed  
**Technology:** XML/RSS 2.0 Parser  
**Update Frequency:** Every 60 minutes (per feed TTL)  
**Data Freshness:** Excellent (jobs posted within hours)

## How It Works

### 1. Data Source
- **Feed URL:** `https://weworkremotely.com/remote-jobs.rss`
- **Protocol:** HTTPS GET request
- **Format:** RSS 2.0 XML
- **No API key required**

### 2. Technology Stack

```typescript
// HTTP Client
- Uses @ever-jobs/common createHttpClient()
- Custom headers for RSS/XML content
- Configurable proxies, timeouts, CA certs

// Parsing Method
- Regex-based XML parsing (no external XML library)
- Splits by <item> tags
- Extracts CDATA and plain text content

// Data Processing
- Converts HTML descriptions to Plain/Markdown
- Extracts emails from descriptions
- Generates unique job IDs from URLs
```


### 3. RSS Feed Structure

```xml
<item>
  <title>Company: Job Title</title>
  <link>https://weworkremotely.com/remote-jobs/...</link>
  <guid>Unique job URL</guid>
  <description><![CDATA[HTML job description]]></description>
  <pubDate>Tue, 11 Aug 2026 16:03:20 +0000</pubDate>
  <expires_at>Thu, 10 Sep 2026 16:03:20 +0000</expires_at>
  <region>Anywhere in the World</region>
  <country>🇺🇸 United States</country>
  <state>California</state>
  <skills>JavaScript, React, TypeScript</skills>
  <category>Full-Stack Programming</category>
  <type>Full-Time</type>
</item>
```

**Key Fields:**
- `title` - Format: "CompanyName: JobTitle"
- `description` - HTML content (converted to Plain/Markdown)
- `pubDate` - RFC 2822 format
- `expires_at` - 30 days from posting
- `skills` - Comma-separated skills
- `category` - Job category
- `type` - Full-Time, Contract, Part-Time


## Implementation Details

### 4. Request Configuration

```typescript
// Headers
const WWR_HEADERS = {
  Accept: 'application/rss+xml, application/xml, text/xml',
  'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)...'
};

// HTTP Client Setup
const client = createHttpClient({
  proxies: input.proxies,           // Optional proxy rotation
  caCert: input.caCert,              // Optional CA certificate
  timeout: input.requestTimeout,     // Default: 60s
});
client.setHeaders(WWR_HEADERS);

// Fetch Feed
const response = await client.get<string>(WWR_RSS_URL);
const xml = response.data;
```

### 5. Parsing Logic

```typescript
// Split XML into items
const itemBlocks = xml.split(/<item>/i).slice(1);

// Extract each tag using regex
private extractTag(xml: string, tagName: string): string | null {
  // Try CDATA: <tag><![CDATA[content]]></tag>
  const cdataRegex = new RegExp(
    `<${tagName}[^>]*>\\s*<!\\[CDATA\\[([\\s\\S]*?)\\]\\]>\\s*<\\/${tagName}>`, 'i'
  );
  
  // Try plain: <tag>content</tag>
  const plainRegex = new RegExp(
    `<${tagName}[^>]*>([\\s\\S]*?)<\\/${tagName}>`, 'i'
  );
}
```


### 6. Data Mapping

```typescript
// Parse title: "Company: Job Title"
const separatorIndex = title.indexOf(': ');
if (separatorIndex > 0) {
  companyName = title.substring(0, separatorIndex);
  jobTitle = title.substring(separatorIndex + 2);
}

// Description format conversion
if (descriptionFormat === DescriptionFormat.PLAIN) {
  description = htmlToPlainText(description);
} else if (descriptionFormat === DescriptionFormat.MARKDOWN) {
  description = markdownConverter(description);
}

// Generate unique ID
const jobId = extractIdFromUrl(item.guid ?? item.link);
const fullId = `wwr-${jobId}`;

// Date conversion
datePosted = toDateOnly(item.pubDate); // "2026-08-11"

// All jobs marked as remote
isRemote: true
```


## Filtering & Features

### 7. Available Filters

**Currently Implemented:**
- `searchTerm` - Searches in title and skills field (case-insensitive)
- `resultsWanted` - Limit results (default: 25)
- `descriptionFormat` - HTML, PLAIN, or MARKDOWN

**Filter Logic:**
```typescript
private matchesSearch(item: WwrRssItem, searchTerm: string): boolean {
  const term = searchTerm.toLowerCase();
  const title = (item.title ?? '').toLowerCase();
  const skills = (item.skills ?? '').toLowerCase();
  
  return title.includes(term) || skills.includes(term);
}
```

**Not Implemented (but data available):**
- Location filtering (region, country, state fields exist)
- Job type filtering (type field exists)
- Category filtering (category field exists)
- Freshness filtering (pubDate available)


## Why It Works Flawlessly

### 8. Key Success Factors

**1. Reliable Data Source**
- Official RSS feed (not scraping)
- Updates every 60 minutes
- Jobs posted within hours (very fresh)
- 30-day expiration built-in

**2. Simple Technology**
- No external XML libraries needed
- Regex parsing = fast & lightweight
- No authentication required
- No rate limits

**3. Rich Metadata**
- Company name, skills, location included
- Job type, category provided
- Expiration dates available
- HTML descriptions convertible

**4. Error Handling**
```typescript
try {
  const job = this.mapJob(item, input.descriptionFormat);
  if (job) jobs.push(job);
} catch (err) {
  this.logger.warn(`Error mapping WWR job: ${err.message}`);
  // Continue processing other jobs
}
```

**5. Performance**
- Single HTTP request fetches all jobs
- Sequential processing (no concurrency needed)
- Bounded by resultsWanted parameter
- Minimal memory footprint


## Output Format

### 9. JobPostDto Structure

```typescript
{
  id: "wwr-{jobId}",
  title: "Software Engineer",
  companyName: "Coinbase",
  jobUrl: "https://weworkremotely.com/remote-jobs/...",
  location: {
    city: "Anywhere in the World",
    country: "United States of America",
    state: "California"
  },
  description: "Converted HTML/Plain/Markdown",
  compensation: null,  // Not provided in RSS
  datePosted: "2026-08-11",
  jobType: ["FULL_TIME"],
  isRemote: true,
  emails: ["jobs@company.com"],  // Extracted from description
  site: Site.WEWORKREMOTELY
}
```


## Quick Implementation Guide

### 10. For AI Assistants

**To replicate WeWorkRemotely integration:**

1. **Fetch RSS Feed**
   - GET `https://weworkremotely.com/remote-jobs.rss`
   - Set Accept header: `application/rss+xml`
   - Set User-Agent (any modern browser UA)

2. **Parse XML**
   - Split by `<item>` tags
   - Use regex to extract: title, link, guid, description, pubDate, region, country, state, skills, category, type
   - Handle both CDATA and plain text content

3. **Transform Data**
   - Split title by `: ` to get company and job title
   - Convert HTML description to desired format
   - Parse pubDate to ISO date string
   - Extract last URL segment as job ID
   - Extract emails from description using regex

4. **Filter Results**
   - Apply searchTerm to title + skills
   - Limit by resultsWanted
   - Return as JobPostDto array

5. **Set Defaults**
   - isRemote: always true
   - site: WEWORKREMOTELY
   - compensation: null (not in feed)

**That's it!** No complex logic, no API keys, no rate limiting.

---

**Last Updated:** 2026-08-12  
**Feed TTL:** 60 minutes  
**Average Jobs in Feed:** ~50-100  
**Data Freshness:** Excellent
