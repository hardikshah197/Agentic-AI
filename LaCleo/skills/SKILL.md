# Web Scraper Skill

## Purpose
Automated data research, extraction, enrichment, and validation for business intelligence, sales prospecting, and market research. This skill handles end-to-end web scraping workflows that deliver clean, structured datasets ready for immediate use.

## When to Use This Skill
- Extracting business data (restaurants, law firms, service providers, e-commerce brands)
- Finding decision-makers (founders, CEOs, hiring managers, owners)
- Social media research (Instagram influencers, LinkedIn profiles, Facebook groups)
- Hiring intelligence (companies hiring specific roles, job postings)
- E-commerce and marketplace data (Amazon sellers, Shopify stores)
- Custom demographic research (expats, niche professional groups)

## Core Capabilities

### 1. Data Discovery & Extraction
- Google Search and Google Maps scraping
- Online business directories
- Social platform data extraction
- Job boards and marketplace scraping
- Website content extraction

### 2. Data Enrichment
- Owner/founder identification via LinkedIn
- Email discovery and validation
- Phone number enrichment
- LinkedIn profile mapping
- Industry and category classification
- Technology stack detection (Shopify, WooCommerce, etc.)

### 3. Validation & Quality Control
- Business legitimacy verification
- Website activity checks
- Email validation (syntax, domain, MX records, catch-all detection)
- LinkedIn profile validation
- Influencer authenticity checks
- Duplicate detection and removal

### 4. Output Formatting
- CSV format (primary output)
- JSON format (secondary output)
- Clean, standardized field names
- Deduplicated records
- Confidence scores where applicable

## Standard Workflow

When a scraping task is requested, follow these steps:

### Step 1: Understand Requirements
Ask clarifying questions:
- What type of data? (businesses, people, influencers, etc.)
- Geographic scope? (city, state, country)
- Industry/niche? (restaurants, law firms, e-commerce, etc.)
- Required fields? (name, email, phone, LinkedIn, etc.)
- Volume needed? (approximate number of records)
- Special criteria? (technology used, follower count, hiring status, etc.)

### Step 2: Choose Data Sources
Based on the request, select appropriate sources:
- **Google Maps**: Local businesses, restaurants, service providers
- **Google Search**: General business discovery, websites
- **LinkedIn**: Decision-makers, professionals, company info
- **Instagram/Social**: Influencers, social presence
- **Job Boards**: Hiring companies, open positions
- **Directories**: Industry-specific listings
- **E-commerce platforms**: Amazon, Shopify stores

### Step 3: Extract Data
Use appropriate tools and methods:
- Web scraping scripts (Playwright, Selenium)
- API calls where available
- Automated browser automation
- Manual extraction for protected content

### Step 4: Enrich Data
Add missing information:
- Find owner/founder names via LinkedIn
- Discover email addresses (tools: Apollo, ZoomInfo, Hunter.io)
- Find phone numbers
- Map LinkedIn profiles
- Identify technology stack
- Classify industry/category

### Step 5: Validate & Clean
Quality control checks:
- Verify business legitimacy (active website, real business)
- Validate emails:
  - Syntax check
  - Domain validation
  - MX record verification
  - Catch-all detection
  - Flag free email providers (Gmail, Yahoo)
- Check LinkedIn profile validity
- Remove duplicates
- Standardize formatting

### Step 6: Deliver Outputs
Create two files:
1. **CSV file**: Primary format for spreadsheet use
2. **JSON file**: Secondary format for programmatic use

Both files should contain:
- Clean, consistent field names
- Standardized data formats
- Confidence scores (when applicable)
- Source attribution
- Timestamp of extraction

## Standard Data Fields

### Business Data
- `company_name` - Business name
- `website` - Company website URL
- `industry` - Industry/category
- `business_type` - Type of business
- `location_address` - Full address
- `city` - City
- `state` - State/province
- `country` - Country
- `phone` - Phone number
- `email` - Company email
- `technologies` - Tech stack (comma-separated)
- `source` - Data source URL
- `extracted_at` - Timestamp

### People Data
- `full_name` - Person's full name
- `first_name` - First name
- `last_name` - Last name
- `title` - Job title/role
- `company` - Company name
- `email` - Email address
- `email_confidence` - Confidence score (0-100)
- `phone` - Phone number
- `linkedin_url` - LinkedIn profile URL
- `linkedin_verified` - Boolean (true/false)
- `location` - Geographic location
- `source` - Data source URL
- `extracted_at` - Timestamp

### Influencer Data
- `name` - Influencer name
- `platform` - Social platform (Instagram, TikTok, etc.)
- `username` - Platform username
- `profile_url` - Profile URL
- `followers` - Follower count
- `engagement_rate` - Engagement rate (%)
- `niche` - Content niche/category
- `verified` - Verified account (true/false)
- `authenticity_score` - Authenticity score (0-100)
- `email` - Contact email
- `location` - Location
- `source` - Data source
- `extracted_at` - Timestamp

## Common Use Cases (Real-World Projects)

### Use Case 1: Restaurants in Specific Location
**Example**: "Find restaurants in New York City"

**Required Fields**: Restaurant Name, Website, Phone Number, Email, Owner Name, Address, Restaurant Type

**Tools Used**:
- Google Maps (primary data source)
- Google Search (supplementary)
- Apollo (owner/contact enrichment)
- ZoomInfo (owner/contact enrichment)

**Process**:
1. Scrape Google Maps for restaurants in target location
2. Extract: name, address, phone, website, rating, category
3. Visit websites to verify they're active
4. Enrich with owner/manager names using LinkedIn search
5. Find emails through Apollo, ZoomInfo, or website contact pages
6. **Manual Effort**: Verify whether it is an actual restaurant (not closed/fake listing). Manually fill owner details not available in databases.

**Output Fields**: restaurant_name, website, phone, email, owner_name, address, city, state, restaurant_type, source, extracted_at

---

### Use Case 2: Website Detail Scraping
**Example**: "Extract email, phone, social links, and integrations from company websites"

**Required Fields**: Email, Phone Number, Social Links, Integrations

**Tools Used**:
- Apify Actors (website scraping)
- Clay (enrichment and workflow)

**Process**:
1. Use Apify actors to scrape website content
2. Extract contact information from contact pages, footers, headers
3. Parse social media links (LinkedIn, Twitter, Facebook, Instagram)
4. Identify integrations (payment processors, CRM, analytics tools)
5. Output CSV + JSON

**Manual Effort**: Minimal - mostly automated

**Output Fields**: company_name, website, email, phone, linkedin, twitter, facebook, instagram, integrations, source, extracted_at

---

### Use Case 3: Instagram Influencer Profiles
**Example**: "Find fitness influencers with 10k-100k followers"

**Required Fields**: Name, Profile Link, Email, Number of Followers, Views, Phone, Niche, Location, Other Social Links

**Tools Used**:
- Modash (influencer directory)
- Instagram scraping tools

**Process**:
1. Use Modash or similar directories to filter influencers by niche and follower count
2. Extract profile data, engagement metrics, contact info
3. Check authenticity (fake follower detection)
4. Extract bio email or business contact email
5. **Manual Effort**: Verify whether profile has real follower numbers and is a valid/active profile (not bot accounts)

**Output Fields**: name, profile_url, email, followers, avg_views, phone, niche, location, other_social_links, authenticity_score, source, extracted_at

---

### Use Case 4: Law Firm Owners at Specific Location
**Example**: "Find law firm owners in Chicago"

**Required Fields**: Owner Name, Email, Phone Number, Practice Area, Location, Website, Firm Name

**Tools Used**:
- Instant Scraper (law directories)
- Playwright/Selenium (custom scraping)
- Apollo, ZoomInfo (email enrichment)

**Process**:
1. Scrape legal directories (Avvo, Justia, FindLaw) or Google Maps
2. Extract firm names, practice areas, locations
3. Visit firm websites to identify managing partners/owners
4. **Manual Effort**: Find emails through multiple tools (Apollo, ZoomInfo, Hunter.io). If not found, use pattern-based guessing (firstname.lastname@firm.com)

**Output Fields**: owner_name, email, phone, practice_area, location, website, firm_name, source, extracted_at

---

### Use Case 5: E-commerce Store Data
**Example**: "Find Shopify stores selling jewelry"

**Required Fields**: Company Name, Website, Owner Name, Email, Category, Location, Phone Number

**Tools Used**:
- E-commerce directories (BuiltWith, SimilarTech, Commerce Inspector)
- Apollo, ZoomInfo (owner enrichment)

**Process**:
1. Use tech detection tools to find Shopify/WooCommerce stores
2. Filter by product category (jewelry, fashion, etc.)
3. Extract store names, websites, product categories
4. Find owner/founder names via LinkedIn or About pages
5. Enrich with emails using Apollo/ZoomInfo
6. **Manual Effort**: Verify whether it is an actual e-commerce website (not parked domain or inactive)

**Output Fields**: company_name, website, owner_name, email, category, location, phone, technologies, source, extracted_at

---

### Use Case 6: Startup Founders (Geographic + Industry)
**Example**: "Find German foodtech startup founders"

**Required Fields**: Startup Name, Founder/CEO, Email, Location, Phone Number, Funding

**Tools Used**:
- Startup directories (Crunchbase, AngelList, F6S)
- Apollo, ZoomInfo, Clay (enrichment)

**Process**:
1. Search startup directories for foodtech companies in Germany
2. Extract startup names, websites, funding information
3. Search LinkedIn for founders/CEOs
4. Enrich with email addresses
5. **Manual Effort**: Fill missing founder/CEO details by using LinkedIn manual search

**Output Fields**: startup_name, founder_name, founder_email, location, phone, funding_amount, website, source, extracted_at

---

### Use Case 7: Companies Hiring Specific Roles
**Example**: "Find companies hiring Backend Engineers in USA"

**Required Fields**: Job Post URLs, Company Name, Website, Hiring Person, Location

**Tools Used**:
- Instant Scraper (job boards)
- Scrapers for Indeed, LinkedIn Jobs, Glassdoor

**Process**:
1. Scrape job boards (Indeed, LinkedIn Jobs) for specific role
2. Extract job URLs, company names, locations
3. Find hiring manager/recruiter names via LinkedIn
4. Enrich with company websites and contact info
5. **Manual Effort**: Fill missing fields like email and hiring person name using LinkedIn search

**Output Fields**: job_url, company_name, website, hiring_person, hiring_email, location, job_title, posted_date, source, extracted_at

---

### Use Case 8: Amazon Sellers Data
**Example**: "Find Amazon sellers in electronics category"

**Required Fields**: Brand Name, Email, Category, Location, Owner Name

**Tools Used**:
- Apify (Amazon scraper)

**Process**:
1. Use Apify Amazon scraper to extract seller information
2. Extract brand names, seller names, product categories
3. Find seller websites (if available)
4. Enrich with owner information and emails
5. **Manual Effort**: Find missing emails. Check if website exists and is real.

**Output Fields**: brand_name, email, category, location, owner_name, seller_url, website, source, extracted_at

---

### Use Case 9: Amazon/Myntra Customers (Individual Buyers)
**Example**: "Find customers buying from Amazon/Myntra in specific categories"

**Required Fields**: Customer Name, Email, Phone, Location, Category

**Tools Used**:
- No automated tool found (highly restricted data)

**Process**:
- This data is typically NOT publicly available due to privacy laws (GDPR, CCPA)
- **Manual Effort**: Only possible through partnerships, surveys, or opt-in lists

**Output Fields**: customer_name, email, phone, location, purchase_category (if legally obtainable)

**NOTE**: This use case may violate privacy laws. Only proceed with explicit user consent and legal compliance.

---

### Use Case 10: Home Repair Services (Roofers, Electricians, etc.)
**Example**: "Find roofers in Texas"

**Required Fields**: Company Name, Owner, Email, Phone Number, Category/Service, Location

**Tools Used**:
- Google Maps (primary)
- Clay, Apollo, ZoomInfo (enrichment)

**Process**:
1. Scrape Google Maps for home repair services (roofer, electrician, plumber, etc.)
2. Extract company names, phone numbers, addresses
3. Visit websites to check if active
4. Enrich with owner names and emails
5. **Manual Effort**: Check if website is working and real (not parked/404). Fill missing fields using Apollo/ZoomInfo.

**Output Fields**: company_name, owner_name, email, phone, service_category, location, website, source, extracted_at

---

### Use Case 11: Companies Using Specific Technologies (Shopify, WooCommerce, etc.)
**Example**: "Find USA companies using Shopify"

**Required Fields**: Company Name, Website, Tech Stack, Decision Maker, Email, Phone Number, Location

**Tools Used**:
- Apollo, ZoomInfo (tech stack filters)
- BuiltWith, SimilarTech (technology lookup)
- Techlookup tool (verification)

**Process**:
1. Use Apollo/ZoomInfo filters to find companies using specific technologies
2. Extract company names, websites, tech stacks
3. Find decision-makers (CTO, VP Engineering, etc.)
4. Enrich with emails and phone numbers
5. **Manual Effort**: Verify technology using Techlookup tool

**Output Fields**: company_name, website, tech_stack, decision_maker, email, phone, location, source, extracted_at

---

### Use Case 12: LinkedIn Profile History (Detailed Work History)
**Example**: "Extract full LinkedIn profile work history for candidates"

**Required Fields**: Name, Education, Work History, Years of Experience, Email, Phone Number, Address, Skills, Gender

**Tools Used**:
- No automated tool (manual extraction required)

**Process**:
1. Access LinkedIn profiles manually
2. Extract name, current and past roles, education
3. Calculate years of experience
4. Extract skills and certifications
5. **Manual Effort**: Check manually every profile. Enrich with email/phone using Apollo/ZoomInfo.

**Output Fields**: name, education, work_history, years_of_experience, email, phone, address, skills, gender, linkedin_url, extracted_at

**NOTE**: Automated scraping of LinkedIn is against their ToS. This should be done manually or with explicit permission.

---

### Use Case 13: USA Expat Data
**Example**: "Find Indian expats working in USA tech companies"

**Required Fields**: Name, Title, Email, Company Name, Website, Location

**Tools Used**:
- No automated tool (manual LinkedIn search)

**Process**:
1. Use LinkedIn advanced search to filter by ethnicity indicators (name, education, location history)
2. Extract names, titles, companies
3. Enrich with emails using Apollo/ZoomInfo
4. **Manual Effort**: Fully manual LinkedIn searches and verification

**Output Fields**: name, title, email, company_name, website, location, linkedin_url, source, extracted_at

---

### Use Case 14: Facebook Group Members
**Example**: "Extract members from 'E-commerce Entrepreneurs' Facebook group"

**Required Fields**: Name, Profile URL, Email, Phone Number, Location, Company Name, Website

**Tools Used**:
- Apify (Facebook Group Scraper)

**Process**:
1. Use Apify Facebook scraper to extract group member profiles
2. Extract names, profile URLs, locations, bio information
3. Find associated company names and websites from profiles
4. **Manual Effort**: Fill missing fields (Email) using Apollo/ZoomInfo or LinkedIn search

**Output Fields**: name, profile_url, email, phone, location, company_name, website, source, extracted_at

---

### Use Case 15: Custom Apparel Sellers
**Example**: "Find custom t-shirt printing companies"

**Required Fields**: Company Name, Website, Owner, Email, Phone Number, Location

**Tools Used**:
- Apify (Google Search scraper)
- Google Search manual

**Process**:
1. Use Apify or Google Search to find custom apparel sellers
2. Extract company names, websites
3. Visit websites to extract contact information
4. **Manual Effort**: Google search for missing information (owner name, email)

**Output Fields**: company_name, website, owner_name, email, phone, location, source, extracted_at

---

### Use Case 16: Chinese Owners/Founders in USA
**Example**: "Find Chinese founders of USA-based startups"

**Required Fields**: Name, Email, Company Name, Website

**Tools Used**:
- No automated tool (manual LinkedIn search)

**Process**:
1. Use LinkedIn advanced search with ethnicity indicators (Chinese name, education in China, etc.)
2. Filter for founders/CEOs in USA
3. Extract company names and websites
4. Enrich with emails
5. **Manual Effort**: Fully manual LinkedIn search and verification

**Output Fields**: name, email, company_name, website, linkedin_url, location, source, extracted_at

---

### Use Case 17: Physiotherapists in USA
**Example**: "Find physiotherapists in California"

**Required Fields**: Therapist Name, Email, Phone Number, Type, Hospital Name

**Tools Used**:
- Google Maps (primary)
- Apollo, ZoomInfo (enrichment)

**Process**:
1. Scrape Google Maps for physiotherapy clinics and hospitals
2. Extract therapist names, phone numbers, clinic names
3. Visit clinic websites to extract emails
4. Identify specialization type (sports therapy, pediatric, etc.)
5. **Manual Effort**: Google search for missing information

**Output Fields**: therapist_name, email, phone, specialization_type, hospital_name, location, website, source, extracted_at

---

### Use Case 18: Signup Email to LinkedIn Profile Mapping
**Example**: "Given a list of emails, find corresponding LinkedIn profiles"

**Required Fields**: LinkedIn Profile, Person Name, Company Name, Title

**Tools Used**:
- Apollo, ZoomInfo, Clay (reverse email lookup)
- **Special**: Use Outlook if free emails (Gmail, Yahoo)

**Process**:
1. Take input list of emails
2. Use Apollo/ZoomInfo reverse lookup to find LinkedIn profiles
3. If email is free provider (Gmail, Yahoo), search Outlook directory for professional email
4. Extract person name, company, title, LinkedIn URL
5. **Manual Effort**: Most cases with free emails require Outlook search to find professional contacts

**Output Fields**: email, linkedin_url, person_name, company_name, title, source, extracted_at

**NOTE**: This is a reverse enrichment workflow. Free emails often require Outlook directory search to map to professional identities.

## Email Enrichment & Validation

### Email Enrichment Strategy

When emails are missing or you need to find professional contacts:

1. **Direct Website Extraction**: Check contact pages, footers, About pages
2. **Apollo/ZoomInfo Lookup**: Use database tools for business emails
3. **Pattern-Based Guessing**: firstname.lastname@company.com, first@company.com
4. **Hunter.io**: Domain search and email pattern verification
5. **Free Email â†’ Professional Email Mapping**:
   - If you have a free email (Gmail, Yahoo, Hotmail), search **Outlook directory** to find the person's professional email
   - This is critical for "Signup Email to LinkedIn" use case
   - Example: john.doe@gmail.com â†’ search Outlook â†’ john.doe@company.com

### Email Validation Rules

Always validate emails using these checks:

1. **Syntax Validation**: Valid email format (regex check)
2. **Domain Validation**: Domain exists and has valid DNS records
3. **MX Record Check**: Domain has mail exchange servers
4. **Catch-All Detection**: Flag catch-all domains (lower confidence)
5. **Free Email Provider**: Flag Gmail, Yahoo, Outlook, etc. (business context)
6. **Deliverability Score**: Combine above into 0-100 confidence score

**Email Confidence Scoring**:
- 90-100: Valid, business domain, MX verified, not catch-all
- 70-89: Valid syntax, MX verified, but catch-all or free provider
- 50-69: Valid syntax, domain exists, but MX issues
- 0-49: Invalid or high risk

### Free Email Handling

When encountering free email addresses (Gmail, Yahoo, Hotmail):
1. **Business context**: Lower confidence score (70-80 range)
2. **LinkedIn mapping**: Search Outlook directory to find professional email
3. **Enrichment**: Use Apollo/ZoomInfo to find company email
4. **Flag in output**: Mark as "free_email" = true in JSON output

## LinkedIn Validation Rules

1. **Profile Exists**: URL is accessible and valid
2. **Profile Active**: Not deleted or suspended
3. **Role Match**: Current role matches expected title/company
4. **Company Match**: Works at expected company
5. **Experience Check**: Has relevant work history
6. **Profile Completeness**: Has sufficient information (not bare profile)

## Output Format Requirements

### CSV Format
- Headers in first row
- UTF-8 encoding
- Comma-separated
- Quoted fields containing commas
- No empty rows
- Consistent date format (ISO 8601: YYYY-MM-DD HH:MM:SS)

### JSON Format
```json
{
  "metadata": {
    "total_records": 100,
    "extraction_date": "2026-01-18T14:30:00Z",
    "source": "Google Maps",
    "query": "Restaurants in New York City",
    "fields": ["company_name", "website", "phone", "email"]
  },
  "data": [
    {
      "company_name": "Example Restaurant",
      "website": "https://example.com",
      "phone": "+1-555-0100",
      "email": "info@example.com",
      "email_confidence": 85,
      "location": "New York, NY",
      "extracted_at": "2026-01-18T14:30:00Z"
    }
  ]
}
```

## Quality Thresholds

Maintain these quality standards:

- **Email Accuracy**: Minimum 70% confidence score
- **LinkedIn Match**: Minimum 80% relevance to search criteria
- **Business Legitimacy**: Active website (no 404s, parked domains)
- **Duplicate Rate**: Maximum 5% duplicates in final dataset
- **Completeness**: Minimum 80% of required fields populated
- **Verification**: Manual spot-check 10% of records for high-value projects

## Error Handling

### Common Issues and Solutions

**Issue**: Rate limiting / IP blocks
- **Solution**: Implement delays, rotate IPs, use residential proxies

**Issue**: Website structure changed
- **Solution**: Update scraping selectors, use multiple fallback methods

**Issue**: Email not found
- **Solution**: Try multiple enrichment tools, use pattern-based guessing (first.last@domain), flag as low confidence

**Issue**: LinkedIn restricted access
- **Solution**: Use alternate research methods, search for public profiles, use company website team pages

**Issue**: Catch-all email domains
- **Solution**: Flag in output, attempt verification via alternate sources, lower confidence score

**Issue**: Duplicate records
- **Solution**: Deduplicate by email, website, or phone number (configurable unique key)

## Tool Recommendations

Based on LacLeo's proven workflow and tools:

### Scraping Tools

**Primary Scraping**:
- **Google Maps**: Local businesses, restaurants, service providers, healthcare
- **Google Search**: General business discovery, websites, news
- **Apify**: Pre-built actors for:
  - Google Maps scraper
  - Instagram scraper
  - Facebook Group scraper
  - Amazon seller scraper
  - Website content scraper
- **Instant Scraper**: Browser extension for quick data extraction (job boards, directories)
- **Playwright**: Custom scraping for dynamic websites
- **Selenium**: Custom browser automation for complex workflows

**Specialized Scrapers**:
- **LinkedIn**: Manual profile research (automated scraping violates ToS)
- **Indeed/LinkedIn Jobs/Glassdoor**: Job posting scrapers
- **Legal Directories**: Avvo, Justia, FindLaw (for law firms)
- **Startup Directories**: Crunchbase, AngelList, F6S

### Enrichment Tools

**Primary Enrichment**:
- **Apollo**: B2B contact data, email finding, tech stack filters, reverse email lookup
- **ZoomInfo**: Company and contact intelligence, decision-maker data
- **Clay**: Workflow automation, multi-source enrichment, waterfall enrichment
- **Hunter.io**: Email finding, domain search, email pattern detection, verification
- **Clearbit**: Company enrichment, firmographic data

**Special Purpose**:
- **Outlook Directory**: Critical for free email â†’ professional email mapping (Gmail/Yahoo â†’ corporate email)
- **LinkedIn**: Manual founder/CEO research, work history extraction
- **Company Websites**: Owner names, team pages, contact information

### Validation Tools

**Email Validation**:
- **NeverBounce**: Email validation and verification
- **ZeroBounce**: Bulk email verification with catch-all detection
- **EmailListVerify**: Bulk email verification
- **Hunter.io**: Email verification API
- **Custom scripts**: MX record checks, catch-all detection, syntax validation

**Business Legitimacy**:
- **Manual website checks**: Verify active websites, not parked domains or 404s
- **Techlookup**: Verify technology stack claims
- **Domain age/reputation tools**: Check business legitimacy

### Social Intelligence

- **Modash**: Instagram influencer discovery, audience analytics, authenticity scores
- **Instagram**: Profile scraping for engagement metrics, follower counts
- **Facebook**: Group member extraction (via Apify)
- **LinkedIn**: Profile research, work history (manual extraction only)

### Technology Detection

- **BuiltWith**: Technology stack detection (Shopify, WooCommerce, etc.)
- **SimilarTech**: E-commerce platform detection
- **Commerce Inspector**: Shopify store analysis
- **Techlookup**: Verify technology claims
- **Apollo/ZoomInfo tech filters**: Filter companies by technology used

## Manual Verification Checklist

LacLeo's workflow includes critical manual verification steps. Always document what requires manual checking:

### Business Legitimacy Checks
- âœ… **Restaurant verification**: Confirm it's an actual restaurant, not closed or fake listing
- âœ… **E-commerce verification**: Check if website is real, not parked domain
- âœ… **Website activity**: Verify website is working (not 404, not under construction)
- âœ… **Technology verification**: Use Techlookup to confirm tech stack claims

### Contact Information Enrichment
- âœ… **Missing owner/founder**: Use LinkedIn manual search to fill missing data
- âœ… **Missing emails**: Waterfall through multiple tools (Apollo â†’ ZoomInfo â†’ Hunter.io â†’ pattern guessing)
- âœ… **Free emails (Gmail/Yahoo)**: Search Outlook directory for professional email
- âœ… **Hiring person names**: LinkedIn search for recruiters/hiring managers

### Social Media Verification
- âœ… **Instagram influencers**: Verify real follower numbers vs bot/fake followers
- âœ… **Profile validity**: Check if social profiles are active, not deleted/suspended

### Specialized Manual Work
- âœ… **LinkedIn profile history**: Manually extract work history, education (no automated tool)
- âœ… **USA expat data**: Manual LinkedIn searches with ethnicity filters
- âœ… **Chinese founders in USA**: Manual LinkedIn searches with demographic filters
- âœ… **Amazon/Myntra customers**: No automated tool available (privacy-restricted)

## Agent Instructions

When executing this skill:

1. **Always confirm scope** before starting extraction
2. **Provide progress updates** for large datasets (every 25-50 records)
3. **Flag issues immediately** (rate limits, access problems, data quality concerns)
4. **Show sample data** before final delivery (first 5-10 records for approval)
5. **Document limitations** in metadata (missing fields, confidence scores, exclusions)
6. **Deliver both formats** (CSV + JSON) unless otherwise specified
7. **Include summary statistics**: total records, field completion rates, confidence score distribution
8. **Flag manual verification needs**: Clearly indicate which records or fields require manual checking
9. **Document tool waterfall**: Show which enrichment tools were tried for missing data

## Example Invocation

**User Request**: "Find 50 law firms in Chicago with their managing partners' emails"

**Agent Response**:
```
I'll help you find law firms in Chicago with managing partner contact information.

Scope confirmation:
- Target: Law firms in Chicago, IL
- Volume: 50 records
- Required fields: Firm name, website, managing partner name, email, phone, address
- Data sources: Google Maps, law firm directories, LinkedIn
- Deliverables: CSV + JSON format

Proceeding with:
1. Extract law firms from Google Maps (Chicago)
2. Enrich with managing partner names (LinkedIn + website research)
3. Find email addresses (Apollo, Hunter.io, pattern-based)
4. Validate emails and business legitimacy
5. Clean and deduplicate
6. Output files

Starting extraction...
```

## Notes

- **Manual verification** may still be required for high-value/high-accuracy projects
- **Always respect** robots.txt and website terms of service
- **Rate limit responsibly** to avoid IP blocks
- **Rotate sources** when one source has limited data
- **Document data provenance** for compliance and auditing
- **GDPR/Privacy considerations**: Only scrape publicly available data, provide opt-out mechanisms where required