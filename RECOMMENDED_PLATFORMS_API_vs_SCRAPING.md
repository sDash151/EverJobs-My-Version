# Recommended Platforms: API vs Scraping Methods

## 🎯 Quick Summary

| Platform | Method | Needs API Key? | Difficulty | Recommendation |
|----------|--------|----------------|------------|----------------|
| **Dice** | ✅ REST API (Primary) + HTML fallback | ❌ NO | 🟢 Easy | **ADD NOW** |
| **ZipRecruiter** | ✅ REST API | ❌ NO (session-based) | 🟢 Easy | **ADD NOW** |
| **CareerJet** | ✅ REST API | ⚠️ YES (affiliate ID) | 🟡 Medium | Sign up first |
| **Jooble** | ✅ REST API | ⚠️ YES (API key) | 🟡 Medium | Sign up first |
| **HackerNews** | ✅ Firebase API | ❌ NO | 🟢 Easy | **ADD NOW** |
| **Freelancer.com** | ✅ REST API | ❌ NO | 🟢 Easy | **ADD NOW** |
| **BDJobs** | ❌ HTML Scraping (Cheerio) | ❌ NO | 🟡 Medium | Can add |

---

## Detailed Breakdown

### 1. **Dice** 🔥 HIGHLY RECOMMENDED

#### Method:
```typescript
// PRIMARY: REST API (structured JSON)
URL: 'https://job-search-api.svc.dhigroupinc.com/v1/dice/jobs/search'

// FALLBACK: HTML scraping with Playwright if API fails
```

#### API Key Required?
**NO** ❌ - Public API, no authentication needed

#### How It Works:
1. Makes HTTP GET request to Dice API
2. Returns structured JSON with job data
3. Falls back to browser automation only if API blocked

#### Why You Want This:
- **US tech jobs** - Huge demand from Indian IT professionals
- **H1B visa sponsorships** - Indians actively search for these
- **Remote US roles** - Work from India for US companies
- **Reliable API** - No scraping headaches

#### Code Example:
```typescript
const response = await client.get(DICE_API_URL, {
  params: {
    q: 'software engineer',
    countryCode2: 'US',
    radius: '30',
    radiusUnit: 'mi',
    page: '1',
    pageSize: '20'
  }
});
// Returns clean JSON with all job details
```

---

### 2. **ZipRecruiter** ⭐ RECOMMENDED

#### Method:
```typescript
// REST API (jobs-app API)
URL: 'https://api.ziprecruiter.com/jobs-app/jobs'
```

#### API Key Required?
**NO** ❌ - Uses session-based auth (automatic)

#### How It Works:
1. Creates session with event endpoint
2. Makes API calls with session headers
3. Returns paginated JSON results

#### Why You Want This:
- **India presence** - Has ziprecruiter.in domain
- **US + Canada jobs** - Good for NRI opportunities
- **Clean API** - Well-structured responses
- **No scraping** - Pure API calls

#### Code Example:
```typescript
// Step 1: Init session
await client.post('https://api.ziprecruiter.com/jobs-app/event', SESSION_EVENT_DATA);

// Step 2: Search jobs
const response = await client.get('https://api.ziprecruiter.com/jobs-app/jobs', {
  params: {
    search: 'developer',
    location: 'San Francisco',
    radius: 25
  }
});
```

---

### 3. **CareerJet** ⚠️ Requires Sign-up

#### Method:
```typescript
// REST API (affiliate-based)
URL: 'https://public.api.careerjet.net/search'
```

#### API Key Required?
**YES** ⚠️ - Need affiliate ID (free to sign up)

#### How to Get API Key:
1. Visit: https://www.careerjet.com/partners/
2. Sign up as affiliate (FREE)
3. Get your `affId` (affiliate ID)
4. Set in `.env`: `CAREERJET_AFFID=your_id_here`

#### Why You Want This:
- **80+ countries** - Massive global coverage
- **Good India coverage** - Supports IN locale
- **Clean API** - Structured JSON responses
- **Free** - Just need to sign up

#### Code Example:
```typescript
const response = await client.get(CAREERJET_API_URL, {
  params: {
    affid: process.env.CAREERJET_AFFID,  // Your affiliate ID
    keywords: 'software engineer',
    location: 'Mumbai',
    locale_code: 'en_IN',  // India locale
    page: 1
  }
});
```

#### ⚠️ Important:
If `CAREERJET_AFFID` is not set, Ever Jobs will log a warning and return empty results.

---

### 4. **Jooble** ⚠️ Requires Sign-up

#### Method:
```typescript
// REST API (key-based)
URL: 'https://jooble.org/api/{API_KEY}'
```

#### API Key Required?
**YES** ⚠️ - Need API key (free to sign up)

#### How to Get API Key:
1. Visit: https://jooble.org/api/about
2. Sign up for API access (FREE)
3. Get your API key
4. Set in `.env`: `JOOBLE_API_KEY=your_key_here`

#### Why You Want This:
- **70+ countries** - Excellent global reach
- **India support** - Good coverage
- **Simple API** - Easy to use
- **Free tier** - Generous limits

#### Code Example:
```typescript
const apiKey = process.env.JOOBLE_API_KEY;
const response = await client.post(`https://jooble.org/api/${apiKey}`, {
  keywords: 'developer',
  location: 'Bangalore',
  page: '1'
});
```

---

### 5. **HackerNews Jobs** 🔥 HIGHLY RECOMMENDED

#### Method:
```typescript
// Firebase REST API (public)
URLs:
- Job IDs: 'https://hacker-news.firebaseio.com/v0/jobstories.json'
- Job Details: 'https://hacker-news.firebaseio.com/v0/item/{id}.json'
```

#### API Key Required?
**NO** ❌ - Completely free, public API

#### How It Works:
1. Fetch array of job story IDs
2. Fetch individual job details for each ID
3. Parse HTML from job text
4. Filter by search term

#### Why You Want This:
- **YC startup jobs** - High-quality tech roles
- **Very popular** - Indian devs love HN jobs
- **No limits** - Free, unlimited API access
- **Tech-focused** - Perfect for developers

#### Code Example:
```typescript
// Step 1: Get job IDs
const idsResponse = await client.get('https://hacker-news.firebaseio.com/v0/jobstories.json');
const jobIds = idsResponse.data; // [37234567, 37234568, ...]

// Step 2: Fetch each job
for (const id of jobIds.slice(0, 15)) {
  const jobResponse = await client.get(`https://hacker-news.firebaseio.com/v0/item/${id}.json`);
  const job = jobResponse.data;
  // Process job...
}
```

---

### 6. **Freelancer.com** ⭐ RECOMMENDED

#### Method:
```typescript
// REST API (public projects API)
URL: 'https://www.freelancer.com/api/projects/0.1/projects/active'
```

#### API Key Required?
**NO** ❌ - Public API, no auth needed

#### How It Works:
1. Makes HTTP GET to public projects endpoint
2. Returns JSON with project/job data
3. Supports filtering by skills, location, etc.

#### Why You Want This:
- **Competitor to Upwork** - More options for freelancers
- **Popular in India** - Large Indian freelancer base
- **No authentication** - Easy to implement
- **Good data** - Structured responses

#### Code Example:
```typescript
const response = await client.get('https://www.freelancer.com/api/projects/0.1/projects/active', {
  params: {
    compact: 'true',
    limit: '15',
    full_description: 'true',
    query: 'nodejs developer',
    job_details: 'true'
  }
});
```

---

### 7. **BDJobs** ⚠️ Scraping Required

#### Method:
```typescript
// HTML Scraping with Cheerio
URL: 'https://jobs.bdjobs.com/jobsearch.asp'
```

#### API Key Required?
**NO** ❌ - But uses HTML scraping (not API)

#### How It Works:
1. Sends HTTP GET to search page
2. Parses HTML with Cheerio
3. Extracts job data from page structure
4. Fetches detail pages for full descriptions

#### Difficulty:
🟡 **Medium** - Selectors can break if site changes

#### Why Include It:
- **Bangladesh market** - Some cross-border opportunities
- **South Asian coverage** - Regional jobs
- **Already implemented** - Ready to use

#### Code Example:
```typescript
// Fetch search page
const response = await client.get('https://jobs.bdjobs.com/jobsearch.asp', {
  params: { txtsearch: 'software engineer' }
});

// Parse HTML
const $ = cheerio.load(response.data);
$('.job-card').each((_, el) => {
  const title = $(el).find('.title').text();
  const company = $(el).find('.company').text();
  // Extract job data...
});
```

---

## 🎯 Final Recommendations

### **Add Immediately (No API Key Needed):**

```typescript
const easyToAdd = [
  'dice',           // ✅ REST API - US tech jobs
  'zip_recruiter',  // ✅ REST API - US/India jobs  
  'hackernews',     // ✅ Firebase API - YC startups
  'freelancercom'   // ✅ REST API - Freelance
];
```

**Why:** All use **public APIs**, no authentication, easy integration!

---

### **Add After Sign-up (Free API Keys):**

```typescript
const needSignup = [
  'careerjet',  // ⚠️ Need affiliate ID (free) - 80+ countries
  'jooble'      // ⚠️ Need API key (free) - 70+ countries
];
```

**Steps:**
1. Sign up at their developer portals (free)
2. Get API credentials
3. Add to `.env` file
4. Enable in Ever Jobs

---

### **Optional (Scraping):**

```typescript
const scraping = [
  'bdjobs'  // ⚠️ HTML scraping - Bangladesh/South Asia
];
```

**Why Optional:** 
- Scraping is less reliable
- Bangladesh focus (less relevant for pure India)
- Already have better options

---

## 📊 Implementation Priority

### **Priority 1: Add These 4 Now** (10 minutes setup)
1. **Dice** - REST API, no auth
2. **ZipRecruiter** - REST API, no auth
3. **HackerNews** - Firebase API, no auth
4. **Freelancer.com** - REST API, no auth

**Total: 18 (current) + 4 = 22 platforms**

---

### **Priority 2: Sign Up & Add** (1 hour setup)
5. **CareerJet** - Sign up → Get affiliate ID → Add
6. **Jooble** - Sign up → Get API key → Add

**Total: 22 + 2 = 24 platforms**

---

### **Priority 3: Optional**
7. **BDJobs** - If you want Bangladesh market

**Total: 24 + 1 = 25 platforms**

---

## 🚀 Quick Start Commands

### Step 1: Add to Your Job Aggregator Config

```typescript
// In your Ever Jobs search request
const response = await fetch('/api/jobs/search', {
  method: 'POST',
  body: JSON.stringify({
    searchTerm: 'software engineer',
    siteType: [
      // Your existing 18 platforms...
      'naukri', 'linkedin', 'indeed', 'internshala', 'glassdoor',
      'google', 'upwork', 'remoteok', 'weworkremotely', 'wellfound',
      'bayt', 'monster', 'himalayas', 'jobicy', 'remotive',
      'adzuna', /* 'jsearch' */,
      
      // NEW: Add these 4 immediately (no API keys needed)
      'dice',
      'zip_recruiter', 
      'hackernews',
      'freelancercom'
    ],
    location: 'India',
    resultsWanted: 15
  })
});
```

### Step 2: For CareerJet & Jooble (After Sign-up)

```bash
# Add to your .env file
CAREERJET_AFFID=your_affiliate_id_here
JOOBLE_API_KEY=your_api_key_here
```

Then add to siteType:
```typescript
siteType: [
  // ... other platforms
  'careerjet',  // After getting affiliate ID
  'jooble'      // After getting API key
]
```

---

## ✅ Summary: API vs Scraping

| Method | Count | Platforms | Difficulty |
|--------|-------|-----------|------------|
| **REST API (No Auth)** | 4 | Dice, ZipRecruiter, HackerNews, Freelancer.com | 🟢 Easy |
| **REST API (Need Key)** | 2 | CareerJet, Jooble | 🟡 Medium |
| **HTML Scraping** | 1 | BDJobs | 🟡 Medium |

**Recommendation:** 
1. Add the 4 **REST API (No Auth)** platforms immediately
2. Sign up for CareerJet & Jooble later
3. Skip BDJobs unless you specifically need Bangladesh coverage

**You'll have 22-24 platforms with mostly API-based integrations = reliable, fast, no scraping headaches!** 🎯
