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

### Data Discovery & Extraction
- Google Search and Google Maps scraping
- Online business directories
- Social platform data extraction
- Job boards and marketplace scraping
- Website content extraction

### Data Enrichment
- Owner/founder identification via LinkedIn
- Email discovery and validation
- Phone number enrichment
- LinkedIn profile mapping
- Industry and category classification
- Technology stack detection (Shopify, WooCommerce, etc.)

### Validation & Quality Control
- Business legitimacy verification
- Website activity checks
- Email validation (syntax, domain, MX records, catch-all detection)
- LinkedIn profile validation
- Influencer authenticity checks
- Duplicate detection and removal

### Output Formatting
- CSV (primary output)
- JSON (secondary output)
- Clean, standardized fields
- Deduplicated records
- Confidence scores where applicable

## Standard Workflow

### Step 1: Understand Requirements
Confirm:
- Data type (businesses, people, influencers)
- Geography
- Industry / niche
- Required fields
- Volume
- Special criteria

### Step 2: Choose Data Sources
- Google Maps
- Google Search
- LinkedIn (manual)
- Instagram / Social platforms
- Job boards
- Industry directories
- E-commerce platforms

### Step 3: Extract Data
- Browser automation (Playwright, Selenium)
- Apify actors
- API access where allowed
- Manual extraction for protected sources

### Step 4: Enrich Data
- Owner/founder lookup
- Email discovery
- Phone numbers
- LinkedIn mapping
- Tech stack detection
- Industry classification

### Step 5: Validate & Clean
- Website legitimacy
- Email validation
- LinkedIn verification
- Deduplication
- Standardization

### Step 6: Deliver Outputs
- CSV + JSON
- Source attribution
- Confidence scores
- Extraction timestamps

## Standard Data Fields

### Business Data
- company_name
- website
- industry
- business_type
- location_address
- city
- state
- country
- phone
- email
- technologies
- source
- extracted_at

### People Data
- full_name
- first_name
- last_name
- title
- company
- email
- email_confidence
- phone
- linkedin_url
- linkedin_verified
- location
- source
- extracted_at

### Influencer Data
- name
- platform
- username
- profile_url
- followers
- engagement_rate
- niche
- verified
- authenticity_score
- email
- location
- source
- extracted_at

## Email Enrichment & Validation

### Enrichment Strategy
1. Website extraction
2. Apollo / ZoomInfo
3. Pattern guessing
4. Hunter.io
5. Free → professional email mapping via Outlook directory

### Validation Rules
- Syntax check
- Domain existence
- MX records
- Catch-all detection
- Free email flag
- Confidence scoring (0–100)

## LinkedIn Validation Rules
- Profile existence
- Activity status
- Role & company match
- Experience relevance
- Profile completeness

## Output Formats

### CSV
- UTF-8
- ISO 8601 timestamps
- No empty rows

### JSON
Includes:
- Metadata block
- Source attribution
- Confidence scores
- Extracted timestamps

## Quality Thresholds
- Email confidence ≥ 70%
- LinkedIn relevance ≥ 80%
- Duplicate rate ≤ 5%
- Required fields completeness ≥ 80%
- Manual QA on ≥ 10% of records

## Error Handling
- Rate limits → delays & proxy rotation
- Selector changes → fallback strategies
- Missing emails → enrichment waterfall
- LinkedIn restrictions → alternate sources

## Compliance & Ethics
- Respect robots.txt
- Follow ToS
- GDPR / privacy compliance
- Public data only
- Opt-out support where required
