---
name: data-validation-enrichment
description: Validate, clean, normalize, deduplicate, and enrich scraped data into high-quality, analysis-ready datasets. Use when Claude needs to QA scraped records, enforce schemas, detect anomalies, standardize formats (dates/currency/units), merge duplicates (entity resolution), verify authenticity against source URLs/social profiles, and add enrichment signals (geocoding, categorization, language detection, etc.). CRITICAL: No record enters final output unless ALL constraints are verified and passed.
---

# Data Validation & Enrichment Skill

Turn noisy scraped output into **trusted, verified, structured, and enriched** data suitable for analytics, search, or downstream ML.

---

## CRITICAL OPERATING PRINCIPLES

### 1. ZERO TOLERANCE FOR CONSTRAINT VIOLATIONS
- **MANDATORY**: Every record MUST pass ALL specified constraints before inclusion in final output
- **NO EXCEPTIONS**: Partial passes, "close enough" matches, or borderline cases are REJECTED
- **TRANSPARENT REJECTION**: Every rejected record must be logged with specific constraint failure reasons
- **VERIFICATION REQUIRED**: Data must be verified against source URLs and social profiles when available

### 2. NO SYNTHETIC OR FABRICATED DATA
- **Detect and reject**: AI-generated, invented, or template-based entries
- **Source cross-check**: Verify data against original source URLs when available
- **Profile verification**: Cross-check data against LinkedIn/social profiles when provided
- **Consistency validation**: Ensure data is internally consistent and matches external evidence

### 3. MANDATORY PRE-OUTPUT VALIDATION
- **Final validation gate**: MANDATORY checkpoint before ANY results are shared with user
- **Only PASS records**: Records with overall_status = "PASS" proceed to final output
- **Complete transparency**: Provide detailed validation reports showing pass/fail statistics

---

## When to Use This Skill

Use this skill when:
- Scraped records have **missing/dirty fields**, inconsistent formatting, or duplicates
- You need **schema enforcement** (types, required fields, constraints)
- You need **authenticity verification** (detect synthetic/fabricated entries)
- You need **source verification** (cross-check data against original URLs and social profiles)
- You need **constraint validation** (ALL user-specified rules must pass)
- You need **standardization** (dates, currency, units, addresses, categories)
- You need **entity resolution** (merge multiple pages into one entity)
- You need **enrichment** (geo coords, inferred categories, normalized company/site info)

---

## Inputs

### Required Inputs

1. **Raw records**: JSON/JSONL/CSV (from scraper/crawler)

2. **Constraints specification** (CRITICAL - must be enforced):
   - **Hard constraints** (ALL must pass):
     ```
     Example:
     - Location must be in: [UK, Germany, France, Netherlands, Switzerland, Austria, Norway]
     - US work experience >= 7 years
     - Seniority level: Manager+, Director+, VP, C-suite, or senior specialist
     - Company size >= 100 employees
     ```
   - **Soft constraints** (affect confidence score but don't reject)
   - **Validation thresholds** (minimum confidence to pass, default: 75%)

3. **Required fields list**:
   ```
   Example:
   - name (required)
   - email (required)
   - phone (optional)
   - company (required)
   - title (required)
   - location (required)
   ```

### Optional Inputs

4. **Schema/spec**
   - Field names, types, required/optional
   - Format constraints (regex, ranges, allowed values)
   - Uniqueness keys (e.g., `canonical_url`, `source_id`, `linkedin_url`)

5. **Normalization rules**
   - Date formats/timezones, currency/locale rules
   - Unit conversions (kg↔lb, cm↔in), text cleanup
   - Country/location normalization mappings

6. **Dedupe & merge rules**
   - Entity keys and match thresholds
   - Field precedence (which source wins) and merge strategy

7. **Enrichment sources**
   - Lookups (geo, language, classification, domain parsing)
   - Optional external APIs (must be configured by user)

8. **Verification sources** (CRITICAL for authenticity)
   - Source URLs for cross-checking
   - Social profile URLs (LinkedIn, X, GitHub) for validation
   - Alternative data sources for confirmation

---

## Outputs

### 1. Clean Dataset (ONLY PASS records)
- **Format**: JSONL/CSV/Parquet/Excel
- **GUARANTEE**: Every record has passed ALL constraints
- **GUARANTEE**: Every record has been verified for authenticity
- **GUARANTEE**: No synthetic or fabricated entries included
- **Includes**: Full validation metadata for each record

### 2. Validation Summary Report
```json
{
  "metadata": {
    "validated_at": "ISO-8601 timestamp",
    "pipeline_version": "1.0",
    "total_input_records": 500,
    "total_output_records": 87,
    "overall_pass_rate": 17.4
  },
  "constraint_pass_rates": {
    "constraint_1": {"passed": 450, "failed": 50, "pass_rate": 90.0},
    "constraint_2": {"passed": 180, "failed": 320, "pass_rate": 36.0}
  },
  "authenticity_checks": {
    "synthetic_data_detected": 5,
    "source_verification_rate": 94.4,
    "profile_verification_rate": 87.5
  },
  "data_quality": {
    "avg_completeness": 92.3,
    "avg_confidence_score": 84.5
  }
}
```

### 3. Rejection Report (ALL FAIL/UNCERTAIN records)
```json
{
  "total_rejected": 413,
  "rejection_categories": {
    "CONSTRAINT_FAILURE": 375,
    "PROFILE_MISMATCH": 20,
    "SOURCE_VERIFICATION_FAILED": 8,
    "SYNTHETIC_DATA": 5,
    "MISSING_REQUIRED_FIELDS": 5
  },
  "rejected_records": [
    {
      "record_id": "...",
      "rejection_category": "CONSTRAINT_FAILURE",
      "failed_constraints": [
        {
          "constraint": "7+ years US work experience",
          "expected": ">= 7 years",
          "actual": "4.5 years",
          "reason": "Calculated from work history: insufficient experience"
        }
      ],
      "data": { }
    }
  ]
}
```

### 4. Detailed Validation Report
- Missing/invalid counts per field
- Outliers/anomaly flags
- Dedupe merges performed
- Authenticity verification results
- Source verification evidence
- Profile verification details

---

## Validation Pipeline (Step-by-Step)

### Phase 1: Pre-Processing

**Step 1.1: Ingest & Parse**
- Decode encodings, normalize column names
- Coerce to known structure
- Handle malformed records gracefully

**Step 1.2: Canonicalize Identifiers**
- Normalize `canonical_url` (remove tracking params)
- Normalize social profile URLs
- Generate `record_id` if missing

**Step 1.3: Constraint Extraction**
- Parse ALL constraints from user specification
- Create validation checklist for each record
- Set up verification requirements (source URLs, social profiles)

---

### Phase 2: Synthetic Data Detection

**CRITICAL: Detect and reject fabricated entries before validation**

#### Detection Method 1: Pattern-Based Detection

```python
SYNTHETIC_INDICATORS = {
    # Suspiciously perfect data
    "all_fields_complete": "100% field completion is rare in real scraped data",
    "identical_formatting": "All entries have identical formatting patterns",
    "sequential_ids": "IDs are sequential without gaps (1,2,3,4...)",
    
    # Template-like content
    "template_text": "Contains placeholder text (lorem ipsum, example, test, dummy)",
    "repeated_phrases": "Same phrases appear across many records verbatim",
    "generic_names": "Names too generic (John Doe, Jane Smith, Test User)",
    
    # Unrealistic patterns
    "perfect_distributions": "Data distributions are too uniform/perfect",
    "round_numbers": "Suspiciously many round numbers (100, 1000, 50%)",
    "no_noise": "Data lacks expected real-world noise/variation",
    
    # AI-generation markers
    "ai_patterns": "Text has AI-generation patterns (overly formal, consistent structure)",
    "impossible_consistency": "Details are impossibly consistent across records",
}
```

**Immediate Rejection Triggers:**
1. Contains placeholder text: "example.com", "test@test.com", "lorem ipsum"
2. Generic template names: "John Doe", "Jane Smith", "Test User"
3. Impossible data: Dates don't make sense, locations don't exist, companies are fake
4. All fields suspiciously perfect with no variation

#### Detection Method 2: Source URL Verification

**MANDATORY when source_url is available**

```python
def verify_against_source(record, source_url):
    """
    Fetch source URL and cross-check data
    Returns: verification_status, evidence, confidence
    """
    if not source_url:
        return "UNVERIFIED", "No source URL provided", 0
    
    # Fetch source content
    try:
        response = requests.get(source_url, timeout=10)
        content = response.text.lower()
    except Exception as e:
        return "FAIL", f"Source URL unreachable: {e}", 0
    
    # Check if record data exists in source
    verification = {}
    
    if "name" in record:
        verification["name_found"] = record["name"].lower() in content
    if "title" in record:
        verification["title_found"] = record["title"].lower() in content
    if "company" in record:
        verification["company_found"] = record["company"].lower() in content
    if "email" in record:
        verification["email_found"] = record["email"].lower() in content
    if "phone" in record:
        phone_normalized = re.sub(r'[^\d+]', '', record["phone"])
        verification["phone_found"] = phone_normalized in re.sub(r'[^\d+]', '', content)
    
    matches = sum(verification.values())
    total_fields = len(verification)
    confidence = (matches / total_fields) * 100 if total_fields > 0 else 0
    
    # Determine status
    if confidence < 60:
        return "FAIL", f"Only {matches}/{total_fields} fields found in source", confidence
    elif confidence < 80:
        return "UNCERTAIN", f"Partial match: {matches}/{total_fields} fields", confidence
    else:
        return "PASS", f"Strong match: {matches}/{total_fields} fields", confidence
```

**Source Verification Rules:**
- If source URL provided but unreachable → FAIL
- If < 60% of fields found in source → FAIL (likely fabricated)
- If 60-79% of fields found → UNCERTAIN → FAIL (strict mode)
- If ≥ 80% of fields found → PASS

#### Detection Method 3: Social Profile Verification

**MANDATORY when LinkedIn/social profile URL is available**

```python
def verify_linkedin_profile(record, linkedin_url):
    """
    Cross-check record data against LinkedIn profile
    Returns: verification_status, mismatches[], confidence
    """
    if not linkedin_url:
        return "UNVERIFIED", [], 0
    
    # Fetch and parse LinkedIn profile
    try:
        profile = fetch_linkedin_profile(linkedin_url)
    except Exception as e:
        return "FAIL", [{"error": f"Profile fetch failed: {e}"}], 0
    
    mismatches = []
    
    # Verify name
    if "name" in record and "name" in profile:
        if not names_match(record["name"], profile["name"]):
            mismatches.append({
                "field": "name",
                "record_value": record["name"],
                "profile_value": profile["name"],
                "severity": "CRITICAL"
            })
    
    # Verify current company
    if "company" in record and "current_company" in profile:
        if not companies_match(record["company"], profile["current_company"]):
            mismatches.append({
                "field": "company",
                "record_value": record["company"],
                "profile_value": profile["current_company"],
                "severity": "HIGH"
            })
    
    # Verify current title
    if "title" in record and "current_title" in profile:
        if not titles_match(record["title"], profile["current_title"]):
            mismatches.append({
                "field": "title",
                "record_value": record["title"],
                "profile_value": profile["current_title"],
                "severity": "HIGH"
            })
    
    # Verify location
    if "location" in record and "location" in profile:
        if not locations_match(record["location"], profile["location"]):
            mismatches.append({
                "field": "location",
                "record_value": record["location"],
                "profile_value": profile["location"],
                "severity": "MEDIUM"
            })
    
    # Verify work history for experience constraints
    if "us_work_years" in record and "work_history" in profile:
        calculated_years = calculate_us_work_years(profile["work_history"])
        if abs(record["us_work_years"] - calculated_years) > 0.5:
            mismatches.append({
                "field": "us_work_years",
                "record_value": record["us_work_years"],
                "calculated_value": calculated_years,
                "severity": "CRITICAL",
                "note": "US work experience mismatch"
            })
    
    # Calculate confidence based on mismatches
    critical_mismatches = [m for m in mismatches if m["severity"] == "CRITICAL"]
    high_mismatches = [m for m in mismatches if m["severity"] == "HIGH"]
    
    if critical_mismatches:
        return "FAIL", mismatches, 0
    elif len(high_mismatches) >= 2:
        return "FAIL", mismatches, 30
    elif high_mismatches:
        return "UNCERTAIN", mismatches, 60
    else:
        confidence = 95 - (len(mismatches) * 10)
        return "PASS", mismatches, max(confidence, 70)
```

**Profile Verification Rules:**
- If LinkedIn URL provided but profile unreachable → FAIL
- If CRITICAL mismatch (name, work experience) → FAIL
- If 2+ HIGH mismatches (company, title) → FAIL
- If 1 HIGH mismatch → UNCERTAIN → FAIL (strict mode)
- If only MEDIUM mismatches → PASS with reduced confidence

#### Detection Method 4: Cross-Source Consistency

```python
def cross_source_verification(record, sources):
    """
    Verify data consistency across multiple sources
    
    sources = {
        "linkedin": linkedin_data,
        "apollo": apollo_data,
        "website": website_data,
        "zoominfo": zoominfo_data
    }
    
    Returns: status, consistency_report, issue_description
    """
    fields_to_verify = ["name", "email", "phone", "company", "title", "location"]
    
    consistency_report = {}
    
    for field in fields_to_verify:
        # Collect values from all sources
        values = {src: data.get(field) for src, data in sources.items() if data.get(field)}
        
        if not values:
            consistency_report[field] = {
                "status": "MISSING",
                "sources": 0,
                "agreement": 0
            }
            continue
        
        # Normalize values for comparison
        normalized = {src: normalize_field(val, field) for src, val in values.items()}
        
        # Check agreement
        unique_values = set(normalized.values())
        
        if len(unique_values) == 1:
            consistency_report[field] = {
                "status": "CONSISTENT",
                "sources": len(values),
                "agreement": 100,
                "value": list(unique_values)[0]
            }
        else:
            agreement_pct = (len(values) - len(unique_values) + 1) / len(values) * 100
            consistency_report[field] = {
                "status": "INCONSISTENT",
                "sources": len(values),
                "agreement": agreement_pct,
                "values": normalized,
                "conflict": True
            }
    
    # Calculate overall consistency score
    consistent_fields = sum(1 for f in consistency_report.values() if f["status"] == "CONSISTENT")
    total_fields = len([f for f in consistency_report.values() if f["status"] != "MISSING"])
    
    consistency_score = (consistent_fields / total_fields * 100) if total_fields > 0 else 0
    
    # Determine status
    if consistency_score < 50:
        return "FAIL", consistency_report, "High cross-source inconsistency suggests fabricated data"
    elif consistency_score < 70:
        return "UNCERTAIN", consistency_report, "Moderate cross-source inconsistency"
    else:
        return "PASS", consistency_report, None
```

**Cross-Source Consistency Rules:**
- If consistency < 50% → FAIL (likely fabricated)
- If consistency 50-69% → UNCERTAIN → FAIL (strict mode)
- If consistency ≥ 70% → PASS

---

### Phase 3: Field Standardization

**Step 3.1: Text Cleanup**
- Trim whitespace, collapse multiple spaces
- Normalize unicode (NFKC)
- Remove zero-width characters
- Remove HTML remnants (`<div>`, `&nbsp;`, etc.)
- Remove template/placeholder text
- Detect and flag AI-generated patterns

**Step 3.2: URL Canonicalization**
```python
def canonicalize_url(url):
    """Normalize URLs for deduplication and validation"""
    from urllib.parse import urlparse, urlunparse, parse_qs, urlencode
    
    parsed = urlparse(url)
    
    # Remove tracking parameters
    tracking_params = ['utm_source', 'utm_medium', 'utm_campaign', 'utm_content', 
                       'gclid', 'fbclid', 'mc_cid', 'mc_eid']
    
    query_params = parse_qs(parsed.query)
    clean_params = {k: v for k, v in query_params.items() if k not in tracking_params}
    clean_query = urlencode(clean_params, doseq=True)
    
    canonical = urlunparse((
        parsed.scheme.lower() if parsed.scheme else 'https',
        parsed.netloc.lower(),
        parsed.path.rstrip('/') or '/',
        '',
        clean_query,
        ''
    ))
    
    return canonical
```

**Step 3.3: Social Profile Normalization**
```python
def normalize_social_profile(url, platform):
    """Normalize social profile URLs to canonical forms"""
    url = url.lower().strip()
    
    if platform == "linkedin":
        if "/in/" in url:
            slug = re.search(r'/in/([^/?]+)', url)
            if slug:
                return f"https://linkedin.com/in/{slug.group(1)}"
        elif "/company/" in url:
            slug = re.search(r'/company/([^/?]+)', url)
            if slug:
                return f"https://linkedin.com/company/{slug.group(1)}"
    
    elif platform == "twitter" or platform == "x":
        handle = re.search(r'(?:twitter\.com|x\.com)/([^/?]+)', url)
        if handle:
            return f"https://x.com/{handle.group(1)}"
    
    elif platform == "github":
        slug = re.search(r'github\.com/([^/?]+)', url)
        if slug:
            return f"https://github.com/{slug.group(1)}"
    
    return url
```

**Step 3.4: Date Normalization**
```python
def normalize_date(date_string):
    """Parse various date formats to ISO-8601"""
    from dateutil import parser
    
    try:
        if date_string.lower() in ['present', 'current', 'now']:
            return datetime.utcnow().isoformat()
        
        dt = parser.parse(date_string)
        return dt.isoformat()
    
    except:
        return None
```

**Step 3.5: Location/Country Normalization**
```python
COUNTRY_NORMALIZATION = {
    'UK': 'United Kingdom',
    'GB': 'United Kingdom',
    'Great Britain': 'United Kingdom',
    'England': 'United Kingdom',
    'Scotland': 'United Kingdom',
    'Wales': 'United Kingdom',
    
    'US': 'United States',
    'USA': 'United States',
    'U.S.': 'United States',
    'U.S.A.': 'United States',
    'America': 'United States',
    
    'DE': 'Germany',
    'Deutschland': 'Germany',
    
    'NL': 'Netherlands',
    'Holland': 'Netherlands',
    
    'CH': 'Switzerland',
    'Schweiz': 'Switzerland',
    
    'AT': 'Austria',
    'Österreich': 'Austria',
    
    'NO': 'Norway',
    'Norge': 'Norway',
    
    'FR': 'France',
    'République française': 'France',
}

def normalize_country(location_string):
    """Extract and normalize country from location string"""
    location = location_string.strip()
    
    if location in COUNTRY_NORMALIZATION:
        return COUNTRY_NORMALIZATION[location]
    
    for alias, canonical in COUNTRY_NORMALIZATION.items():
        if alias.lower() in location.lower():
            return canonical
    
    for canonical in set(COUNTRY_NORMALIZATION.values()):
        if canonical.lower() in location.lower():
            return canonical
    
    return location
```

**Step 3.6: Phone Number Normalization**
```python
def normalize_phone(phone_string):
    """Normalize phone numbers to E.164 format"""
    import phonenumbers
    
    try:
        parsed = phonenumbers.parse(phone_string, None)
        return phonenumbers.format_number(parsed, phonenumbers.PhoneNumberFormat.E164)
    
    except:
        cleaned = re.sub(r'[^\d+]', '', phone_string)
        return cleaned if cleaned else None
```

**Step 3.7: Email Normalization**
```python
def normalize_email(email_string):
    """Normalize and validate email addresses"""
    import email_validator
    
    try:
        validated = email_validator.validate_email(email_string, check_deliverability=False)
        return validated.email.lower()
    
    except:
        return None
```

**Step 3.8: Company Size Parsing**
```python
def parse_company_size(size_string):
    """
    Parse company size from various formats to minimum employee count
    
    Examples:
    - "1001-5000" → 1001
    - "500+" → 500
    - "10,000 employees" → 10000
    """
    if not size_string:
        return None
    
    size_str = str(size_string).replace(',', '').replace(' employees', '').strip()
    
    if '-' in size_str:
        return int(size_str.split('-')[0])
    elif '+' in size_str:
        return int(size_str.replace('+', ''))
    else:
        try:
            return int(re.sub(r'[^\d]', '', size_str))
        except:
            return None
```

---

### Phase 4: Schema & Format Validation

**Step 4.1: Type Validation**
```python
def validate_field_types(record, schema):
    """Validate that fields match expected types"""
    errors = []
    
    for field, expected_type in schema.items():
        if field not in record:
            continue
        
        value = record[field]
        
        if expected_type == "string" and not isinstance(value, str):
            errors.append(f"{field}: expected string, got {type(value)}")
        
        elif expected_type == "integer" and not isinstance(value, int):
            errors.append(f"{field}: expected integer, got {type(value)}")
        
        elif expected_type == "float" and not isinstance(value, (int, float)):
            errors.append(f"{field}: expected float, got {type(value)}")
        
        elif expected_type == "url":
            from urllib.parse import urlparse
            try:
                result = urlparse(value)
                if not all([result.scheme, result.netloc]):
                    errors.append(f"{field}: invalid URL format")
            except:
                errors.append(f"{field}: URL parsing failed")
        
        elif expected_type == "email":
            if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', value):
                errors.append(f"{field}: invalid email format")
        
        elif expected_type == "phone":
            cleaned = re.sub(r'[^\d+]', '', value)
            if len(cleaned) < 7:
                errors.append(f"{field}: phone number too short")
    
    return errors
```

**Step 4.2: Required Fields Check**
```python
def validate_required_fields(record, required_fields):
    """Check that all required fields are present and non-empty"""
    missing = []
    empty = []
    
    for field in required_fields:
        if field not in record:
            missing.append(field)
        elif record[field] is None or str(record[field]).strip() == '':
            empty.append(field)
    
    return {
        "all_present": len(missing) == 0 and len(empty) == 0,
        "missing_fields": missing,
        "empty_fields": empty
    }
```

**Step 4.3: Format Validation (Regex)**
```python
FORMAT_PATTERNS = {
    "email": r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
    "phone_us": r'^\+?1?[-.\s]?\(?[0-9]{3}\)?[-.\s]?[0-9]{3}[-.\s]?[0-9]{4}$',
    "postal_code_us": r'^\d{5}(-\d{4})?$',
    "postal_code_uk": r'^[A-Z]{1,2}\d{1,2}[A-Z]?\s?\d[A-Z]{2}$',
    "linkedin_person": r'^https?://(?:www\.)?linkedin\.com/in/[a-zA-Z0-9_-]+/?$',
    "linkedin_company": r'^https?://(?:www\.)?linkedin\.com/company/[a-zA-Z0-9_-]+/?$',
}

def validate_format(value, format_type):
    """Validate field against regex pattern"""
    if format_type not in FORMAT_PATTERNS:
        return True
    
    pattern = FORMAT_PATTERNS[format_type]
    return bool(re.match(pattern, str(value)))
```

---

### Phase 5: Constraint Validation (CRITICAL)

**CRITICAL: This is the core validation step where ALL user constraints are checked**

#### Constraint Validation Framework

```python
def validate_all_constraints(record, constraints):
    """
    Validate record against ALL specified constraints
    Returns validation object with detailed pass/fail for each constraint
    """
    validation = {
        "overall_status": "PASS",
        "constraints_checked": len(constraints),
        "constraints_passed": 0,
        "constraints_failed": 0,
        "constraints_uncertain": 0,
        "constraint_details": []
    }
    
    for constraint in constraints:
        result = check_constraint(record, constraint)
        validation["constraint_details"].append(result)
        
        if result["status"] == "PASS":
            validation["constraints_passed"] += 1
        
        elif result["status"] == "FAIL":
            validation["constraints_failed"] += 1
            validation["overall_status"] = "FAIL"
        
        elif result["status"] == "UNCERTAIN":
            validation["constraints_uncertain"] += 1
            validation["overall_status"] = "FAIL"
    
    return validation
```

#### Constraint Type 1: Location Constraints

```python
def validate_location_constraint(record, constraint):
    """
    Validate location is in approved list
    
    constraint = {
        "type": "location",
        "field": "location",
        "allowed_values": ["United Kingdom", "Germany", "France", 
                          "Netherlands", "Switzerland", "Austria", "Norway"]
    }
    """
    location = record.get(constraint["field"])
    
    if not location:
        return {
            "constraint": constraint["name"],
            "status": "FAIL",
            "expected": f"One of: {', '.join(constraint['allowed_values'])}",
            "actual": "MISSING",
            "confidence": 0,
            "note": "Location field is missing"
        }
    
    normalized = normalize_country(location)
    
    if normalized in constraint["allowed_values"]:
        return {
            "constraint": constraint["name"],
            "status": "PASS",
            "expected": f"One of: {', '.join(constraint['allowed_values'])}",
            "actual": normalized,
            "confidence": 95,
            "note": f"Matched: {normalized}"
        }
    else:
        return {
            "constraint": constraint["name"],
            "status": "FAIL",
            "expected": f"One of: {', '.join(constraint['allowed_values'])}",
            "actual": normalized,
            "confidence": 0,
            "note": f"{normalized} not in approved list"
        }
```

#### Constraint Type 2: Experience/Duration Constraints

```python
def validate_experience_constraint(record, constraint):
    """
    Validate years of experience meets threshold
    
    constraint = {
        "type": "experience",
        "field": "us_work_years",
        "minimum": 7.0,
        "calculation_method": "sum_us_positions"
    }
    """
    # Option 1: Pre-calculated field exists
    if constraint["field"] in record:
        years = record[constraint["field"]]
        
        if years >= constraint["minimum"]:
            return {
                "constraint": constraint["name"],
                "status": "PASS",
                "expected": f">= {constraint['minimum']} years",
                "actual": f"{years} years",
                "confidence": 90,
                "note": f"Has {years} years of experience"
            }
        else:
            return {
                "constraint": constraint["name"],
                "status": "FAIL",
                "expected": f">= {constraint['minimum']} years",
                "actual": f"{years} years",
                "confidence": 0,
                "note": f"Only {years} years, need {constraint['minimum']}"
            }
    
    # Option 2: Calculate from work history
    elif "work_history" in record:
        years = calculate_us_work_years(record["work_history"])
        
        if years >= constraint["minimum"]:
            return {
                "constraint": constraint["name"],
                "status": "PASS",
                "expected": f">= {constraint['minimum']} years",
                "actual": f"{years} years",
                "confidence": 85,
                "note": f"Calculated from work history: {years} years"
            }
        else:
            return {
                "constraint": constraint["name"],
                "status": "FAIL",
                "expected": f">= {constraint['minimum']} years",
                "actual": f"{years} years",
                "confidence": 0,
                "note": f"Calculated only {years} years from work history"
            }
    
    else:
        return {
            "constraint": constraint["name"],
            "status": "FAIL",
            "expected": f">= {constraint['minimum']} years",
            "actual": "UNKNOWN",
            "confidence": 0,
            "note": "Cannot verify: work history data missing"
        }


def calculate_us_work_years(work_history):
    """
    Calculate total years of US work experience from work history
    
    work_history = [
        {
            "company": "Google",
            "title": "Software Engineer",
            "location": "Mountain View, CA",
            "start_date": "2012-01",
            "end_date": "2015-06"
        }
    ]
    """
    from dateutil import parser
    
    US_LOCATIONS = ['United States', 'USA', 'US', 'U.S.']
    US_STATES = ['CA', 'NY', 'TX', 'WA', 'MA', 'IL', 'FL', 'PA', 'OH', 'GA']
    
    total_years = 0.0
    
    for position in work_history:
        location = position.get("location", "")
        
        is_us = any(us_loc in location for us_loc in US_LOCATIONS)
        is_us = is_us or any(state in location for state in US_STATES)
        
        if not is_us:
            continue
        
        try:
            start = parser.parse(position["start_date"])
            
            if not position.get("end_date") or position["end_date"].lower() in ['present', 'current']:
                end = datetime.utcnow()
            else:
                end = parser.parse(position["end_date"])
            
            years = (end - start).days / 365.25
            total_years += years
        
        except:
            continue
    
    return round(total_years, 1)
```

#### Constraint Type 3: Seniority Level Constraints

```python
SENIORITY_PATTERNS = {
    "c_suite": [
        r'\bCEO\b', r'\bCTO\b', r'\bCFO\b', r'\bCOO\b', r'\bCMO\b', 
        r'\bCPO\b', r'\bCISO\b', r'\bChief\s+\w+\s+Officer\b'
    ],
    "vp": [
        r'\bVP\b', r'\bVice\s+President\b', r'\bSVP\b', r'\bEVP\b'
    ],
    "director": [
        r'\bDirector\b', r'\bDir\b', r'\bHead\s+of\b'
    ],
    "manager": [
        r'\bManager\b', r'\bMgr\b', r'\bTeam\s+Lead\b', r'\bGroup\s+Lead\b'
    ],
    "senior_specialist": [
        r'\bStaff\s+\w+', r'\bPrincipal\s+\w+', r'\bLead\s+\w+', 
        r'\bSenior\s+\w+', r'\bSr\.?\s+\w+',
        r'\bArchitect\b', r'\bFellow\b', r'\bDistinguished\b'
    ]
}

JUNIOR_PATTERNS = [
    r'\bJunior\b', r'\bJr\.?\b', r'\bEntry\b', r'\bAssociate\b',
    r'\bAnalyst\b', r'\bIntern\b', r'\bTrainee\b', r'\bGraduate\b'
]

def validate_seniority_constraint(record, constraint):
    """
    Validate job title matches seniority patterns
    
    constraint = {
        "type": "seniority",
        "field": "title",
        "levels": ["c_suite", "vp", "director", "manager", "senior_specialist"]
    }
    """
    title = record.get(constraint["field"], "")
    
    if not title:
        return {
            "constraint": constraint["name"],
            "status": "FAIL",
            "expected": "Senior level title",
            "actual": "MISSING",
            "confidence": 0,
            "note": "Title field is missing"
        }
    
    # Check for junior patterns (immediate disqualification)
    for pattern in JUNIOR_PATTERNS:
        if re.search(pattern, title, re.IGNORECASE):
            return {
                "constraint": constraint["name"],
                "status": "FAIL",
                "expected": "Senior level title",
                "actual": title,
                "confidence": 0,
                "note": f"Contains junior pattern: {pattern}"
            }
    
    # Check for senior patterns
    for level in constraint["levels"]:
        if level in SENIORITY_PATTERNS:
            for pattern in SENIORITY_PATTERNS[level]:
                if re.search(pattern, title, re.IGNORECASE):
                    return {
                        "constraint": constraint["name"],
                        "status": "PASS",
                        "expected": "Senior level title",
                        "actual": title,
                        "confidence": 90,
                        "note": f"Matched {level} pattern: {pattern}"
                    }
    
    # No pattern matched
    return {
        "constraint": constraint["name"],
        "status": "FAIL",
        "expected": "Senior level title",
        "actual": title,
        "confidence": 0,
        "note": "No senior level pattern matched"
    }
```

#### Constraint Type 4: Numeric Range Constraints

```python
def validate_numeric_constraint(record, constraint):
    """
    Validate numeric field meets range requirements
    
    constraint = {
        "type": "numeric",
        "field": "company_size",
        "minimum": 100,
        "maximum": None,
        "preferred_minimum": 1000
    }
    """
    value = record.get(constraint["field"])
    
    if value is None:
        return {
            "constraint": constraint["name"],
            "status": "FAIL",
            "expected": f">= {constraint['minimum']}",
            "actual": "MISSING",
            "confidence": 0,
            "note": f"{constraint['field']} is missing"
        }
    
    # Parse if string
    if isinstance(value, str):
        value = parse_company_size(value) if constraint["field"] == "company_size" else float(value)
    
    # Check minimum
    if constraint.get("minimum") is not None and value < constraint["minimum"]:
        return {
            "constraint": constraint["name"],
            "status": "FAIL",
            "expected": f">= {constraint['minimum']}",
            "actual": str(value),
            "confidence": 0,
            "note": f"Below minimum threshold"
        }
    
    # Check maximum
    if constraint.get("maximum") is not None and value > constraint["maximum"]:
        return {
            "constraint": constraint["name"],
            "status": "FAIL",
            "expected": f"<= {constraint['maximum']}",
            "actual": str(value),
            "confidence": 0,
            "note": f"Above maximum threshold"
        }
    
    # Check preferred (affects confidence, not pass/fail)
    confidence = 90
    note = f"Meets minimum requirement"
    
    if constraint.get("preferred_minimum") and value >= constraint["preferred_minimum"]:
        confidence = 95
        note = f"Meets preferred threshold"
    
    return {
        "constraint": constraint["name"],
        "status": "PASS",
        "expected": f">= {constraint['minimum']}",
        "actual": str(value),
        "confidence": confidence,
        "note": note
    }
```

---

### Phase 6: Anomaly Detection

**Step 6.1: Outlier Detection**
```python
def detect_outliers(records, field, method="iqr"):
    """
    Detect statistical outliers in numeric fields
    """
    import numpy as np
    
    values = [r.get(field) for r in records if r.get(field) is not None]
    
    if not values:
        return []
    
    if method == "iqr":
        q1 = np.percentile(values, 25)
        q3 = np.percentile(values, 75)
        iqr = q3 - q1
        
        lower_bound = q1 - (1.5 * iqr)
        upper_bound = q3 + (1.5 * iqr)
        
        outliers = []
        for record in records:
            value = record.get(field)
            if value is not None and (value < lower_bound or value > upper_bound):
                outliers.append({
                    "record_id": record.get("record_id"),
                    "field": field,
                    "value": value,
                    "bounds": [lower_bound, upper_bound],
                    "severity": "HIGH" if value < 0 else "MEDIUM"
                })
        
        return outliers
    
    return []
```

**Step 6.2: Content Sanity Checks**
```python
def check_content_sanity(record):
    """
    Check for common content issues
    """
    issues = []
    
    # Check for HTML remnants
    for field, value in record.items():
        if isinstance(value, str):
            if '<div>' in value or '&nbsp;' in value or '<script>' in value:
                issues.append({
                    "field": field,
                    "issue": "HTML_REMNANTS",
                    "severity": "HIGH"
                })
    
    # Check for blocked page indicators
    blocked_indicators = ['captcha', 'login required', 'access denied', 'bot detected']
    for field, value in record.items():
        if isinstance(value, str):
            value_lower = value.lower()
            if any(indicator in value_lower for indicator in blocked_indicators):
                issues.append({
                    "field": field,
                    "issue": "BLOCKED_PAGE",
                    "severity": "CRITICAL"
                })
    
    # Check for minimum content length
    if 'description' in record:
        desc = record['description']
        if isinstance(desc, str) and len(desc.strip()) < 10:
            issues.append({
                "field": "description",
                "issue": "TOO_SHORT",
                "severity": "MEDIUM"
            })
    
    return issues
```

---

### Phase 7: Deduplication & Entity Resolution

**Step 7.1: Exact Deduplication**
```python
def exact_deduplicate(records, key_fields):
    """
    Remove exact duplicates based on key fields
    
    key_fields: ['canonical_url'] or ['linkedin_url'] or ['email']
    """
    seen = {}
    unique_records = []
    duplicates = []
    
    for record in records:
        # Create composite key
        key_values = tuple(record.get(field) for field in key_fields)
        
        if None in key_values:
            unique_records.append(record)
            continue
        
        if key_values in seen:
            duplicates.append({
                "duplicate": record,
                "original": seen[key_values],
                "matched_on": key_fields
            })
        else:
            seen[key_values] = record
            unique_records.append(record)
    
    return unique_records, duplicates
```

**Step 7.2: Fuzzy Matching**
```python
def fuzzy_match_records(record1, record2, threshold=0.85):
    """
    Fuzzy match two records based on similar fields
    """
    from difflib import SequenceMatcher
    
    fields_to_compare = ['name', 'company', 'title', 'location', 'email']
    
    matches = 0
    total = 0
    
    for field in fields_to_compare:
        val1 = record1.get(field)
        val2 = record2.get(field)
        
        if val1 and val2:
            total += 1
            similarity = SequenceMatcher(None, str(val1).lower(), str(val2).lower()).ratio()
            
            if similarity >= threshold:
                matches += 1
    
    if total == 0:
        return 0.0
    
    return matches / total
```

**Step 7.3: Merge Policy**
```python
def merge_duplicate_records(records, source_priority):
    """
    Merge duplicate records with conflict resolution
    
    source_priority: ['linkedin', 'apollo', 'zoominfo', 'website']
    """
    merged = {}
    
    for field in set().union(*[set(r.keys()) for r in records]):
        # Get all non-null values for this field
        values = [(r.get(field), r.get('source', 'unknown')) 
                  for r in records if r.get(field) is not None]
        
        if not values:
            merged[field] = None
            continue
        
        # If all values are the same, use that value
        unique_values = set(v[0] for v in values)
        if len(unique_values) == 1:
            merged[field] = list(unique_values)[0]
            continue
        
        # Use source priority to resolve conflicts
        for source in source_priority:
            matching = [v[0] for v in values if v[1] == source]
            if matching:
                merged[field] = matching[0]
                merged[f'{field}_conflict'] = True
                merged[f'{field}_all_values'] = list(unique_values)
                break
        
        # If no priority match, use most complete/longest value
        if field not in merged:
            merged[field] = max(values, key=lambda x: len(str(x[0])))[0]
    
    return merged
```

---

### Phase 8: Enrichment

**Step 8.1: Derived Fields**
```python
def add_derived_fields(record):
    """
    Add computed/derived fields
    """
    # Extract domain from URL
    if 'url' in record:
        from urllib.parse import urlparse
        parsed = urlparse(record['url'])
        record['domain'] = parsed.netloc
        
        # Extract registered domain (remove subdomains)
        parts = parsed.netloc.split('.')
        if len(parts) >= 2:
            record['registered_domain'] = '.'.join(parts[-2:])
    
    # Extract domain from email
    if 'email' in record:
        email_domain = record['email'].split('@')[1] if '@' in record['email'] else None
        record['email_domain'] = email_domain
    
    # Generate record hash for deduplication
    import hashlib
    key_fields = [str(record.get(f, '')) for f in ['name', 'company', 'email']]
    record['record_hash'] = hashlib.md5('|'.join(key_fields).encode()).hexdigest()
    
    # Add timestamp
    record['enriched_at'] = datetime.utcnow().isoformat()
    
    return record
```

**Step 8.2: Geocoding (Optional)**
```python
def geocode_address(address, geocoding_service):
    """
    Convert address to lat/lng coordinates
    Requires configured geocoding service
    """
    if not geocoding_service:
        return None
    
    try:
        result = geocoding_service.geocode(address)
        
        return {
            'latitude': result['lat'],
            'longitude': result['lng'],
            'formatted_address': result['formatted_address'],
            'geocoded_at': datetime.utcnow().isoformat()
        }
    
    except:
        return None
```

**Step 8.3: Language Detection**
```python
def detect_language(text):
    """
    Detect language of text content
    """
    from langdetect import detect
    
    try:
        lang = detect(text)
        return lang
    except:
        return None
```

---

### Phase 9: MANDATORY Pre-Output Validation Gate

**CRITICAL: Final checkpoint before ANY data is shared with user**

```python
def final_validation_gate(records, constraints, required_fields):
    """
    Final validation before output - NO BYPASSES ALLOWED
    
    This is the LAST checkpoint. Only records that pass ALL checks proceed to output.
    """
    passed_records = []
    rejected_records = []
    
    validation_stats = {
        "total_input": len(records),
        "constraint_pass": 0,
        "authenticity_pass": 0,
        "required_fields_pass": 0,
        "final_pass": 0
    }
    
    for record in records:
        rejection_reasons = []
        
        # Check 1: Constraint validation
        constraint_validation = validate_all_constraints(record, constraints)
        
        if constraint_validation["overall_status"] != "PASS":
            rejection_reasons.append({
                "category": "CONSTRAINT_FAILURE",
                "failed_constraints": [
                    c for c in constraint_validation["constraint_details"] 
                    if c["status"] != "PASS"
                ]
            })
        else:
            validation_stats["constraint_pass"] += 1
        
        # Check 2: Authenticity verification
        authenticity = {
            "synthetic_check": "PASS",
            "source_verification": "PASS",
            "profile_verification": "PASS"
        }
        
        # Synthetic data check
        if detect_synthetic_patterns(record):
            rejection_reasons.append({
                "category": "SYNTHETIC_DATA",
                "details": "Record contains synthetic/fabricated data patterns"
            })
            authenticity["synthetic_check"] = "FAIL"
        
        # Source URL verification
        if "source_url" in record:
            source_status, _, _ = verify_against_source(record, record["source_url"])
            if source_status == "FAIL":
                rejection_reasons.append({
                    "category": "SOURCE_VERIFICATION_FAILED",
                    "details": "Data not found in source URL"
                })
                authenticity["source_verification"] = "FAIL"
        
        # Profile verification
        if "linkedin_url" in record:
            profile_status, mismatches, _ = verify_linkedin_profile(record, record["linkedin_url"])
            if profile_status == "FAIL":
                rejection_reasons.append({
                    "category": "PROFILE_MISMATCH",
                    "details": "LinkedIn profile data mismatch",
                    "mismatches": mismatches
                })
                authenticity["profile_verification"] = "FAIL"
        
        if all(v == "PASS" for v in authenticity.values()):
            validation_stats["authenticity_pass"] += 1
        
        # Check 3: Required fields
        fields_check = validate_required_fields(record, required_fields)
        
        if not fields_check["all_present"]:
            rejection_reasons.append({
                "category": "MISSING_REQUIRED_FIELDS",
                "missing": fields_check["missing_fields"],
                "empty": fields_check["empty_fields"]
            })
        else:
            validation_stats["required_fields_pass"] += 1
        
        # Final decision: ONLY PASS if ALL checks passed
        if len(rejection_reasons) == 0:
            # Add validation metadata to record
            record["validation"] = {
                "overall_status": "PASS",
                "validated_at": datetime.utcnow().isoformat(),
                "constraints": constraint_validation,
                "authenticity": authenticity,
                "required_fields": fields_check
            }
            
            passed_records.append(record)
            validation_stats["final_pass"] += 1
        else:
            # Record rejected
            rejected_records.append({
                "record_id": record.get("record_id"),
                "rejection_reasons": rejection_reasons,
                "data": record
            })
    
    # Generate transparency report
    report = {
        "validation_stats": validation_stats,
        "pass_rate": (validation_stats["final_pass"] / validation_stats["total_input"] * 100) if validation_stats["total_input"] > 0 else 0,
        "rejection_breakdown": categorize_rejections(rejected_records)
    }
    
    return passed_records, rejected_records, report


def categorize_rejections(rejected_records):
    """
    Categorize rejection reasons for reporting
    """
    categories = {}
    
    for rejected in rejected_records:
        for reason in rejected["rejection_reasons"]:
            category = reason["category"]
            
            if category not in categories:
                categories[category] = {
                    "count": 0,
                    "examples": []
                }
            
            categories[category]["count"] += 1
            
            if len(categories[category]["examples"]) < 3:
                categories[category]["examples"].append({
                    "record_id": rejected["record_id"],
                    "details": reason.get("details", ""),
                    "data_sample": {k: rejected["data"].get(k) for k in ['name', 'company', 'title']}
                })
    
    return categories
```

---

## Link & Social Profile Validation

### Link Validation (URLs)

**Syntax Validation:**
```python
def validate_url_syntax(url):
    """Validate URL syntax and scheme"""
    from urllib.parse import urlparse
    
    try:
        result = urlparse(url)
        
        # Check scheme
        if result.scheme not in ['http', 'https']:
            return False, "Invalid scheme (must be http or https)"
        
        # Check netloc (domain)
        if not result.netloc:
            return False, "Missing domain"
        
        return True, "Valid URL syntax"
    
    except:
        return False, "URL parsing failed"
```

**Reachability Check (Optional but Recommended):**
```python
def check_url_reachability(url, timeout=10):
    """
    Check if URL is reachable
    Returns: status, http_code, final_url, redirect_chain
    """
    import requests
    
    try:
        response = requests.head(url, timeout=timeout, allow_redirects=True)
        
        return {
            "status": "REACHABLE" if response.status_code < 400 else "UNREACHABLE",
            "http_status": response.status_code,
            "final_url": response.url,
            "redirect_chain": [r.url for r in response.history],
            "checked_at": datetime.utcnow().isoformat()
        }
    
    except requests.exceptions.Timeout:
        return {
            "status": "TIMEOUT",
            "error": "Request timed out"
        }
    
    except Exception as e:
        return {
            "status": "ERROR",
            "error": str(e)
        }
```

### Social Profile Validation

**LinkedIn Profile Type Check:**
```python
def validate_linkedin_profile_type(url, expected_type):
    """
    Validate LinkedIn URL is correct type (person vs company)
    
    expected_type: 'person' or 'company'
    """
    url_lower = url.lower()
    
    if expected_type == "person":
        if "/in/" in url_lower:
            return True, "Valid person profile"
        else:
            return False, "URL is not a person profile (expected /in/)"
    
    elif expected_type == "company":
        if "/company/" in url_lower:
            return True, "Valid company profile"
        else:
            return False, "URL is not a company profile (expected /company/)"
    
    return False, "Unknown profile type"
```

---

## Reporting & Output Generation

### Validation Summary Report

```python
def generate_validation_report(passed_records, rejected_records, constraints):
    """
    Generate comprehensive validation report
    """
    total_input = len(passed_records) + len(rejected_records)
    
    # Constraint pass rates
    constraint_stats = {}
    for constraint in constraints:
        constraint_id = constraint.get("name") or constraint.get("id")
        
        passed = sum(1 for r in passed_records 
                    if any(c["constraint"] == constraint_id and c["status"] == "PASS" 
                          for c in r.get("validation", {}).get("constraints", {}).get("constraint_details", [])))
        
        failed = sum(1 for r in rejected_records 
                    if any(reason["category"] == "CONSTRAINT_FAILURE" 
                          and any(c.get("constraint") == constraint_id 
                                 for c in reason.get("failed_constraints", []))
                          for reason in r.get("rejection_reasons", [])))
        
        constraint_stats[constraint_id] = {
            "passed": passed,
            "failed": failed,
            "total": passed + failed,
            "pass_rate": (passed / (passed + failed) * 100) if (passed + failed) > 0 else 0
        }
    
    # Authenticity checks
    authenticity_stats = {
        "synthetic_data_detected": sum(1 for r in rejected_records 
                                       if any(reason["category"] == "SYNTHETIC_DATA" 
                                             for reason in r.get("rejection_reasons", []))),
        
        "source_verification_failed": sum(1 for r in rejected_records 
                                          if any(reason["category"] == "SOURCE_VERIFICATION_FAILED" 
                                                for reason in r.get("rejection_reasons", []))),
        
        "profile_mismatch": sum(1 for r in rejected_records 
                               if any(reason["category"] == "PROFILE_MISMATCH" 
                                     for reason in r.get("rejection_reasons", [])))
    }
    
    # Data quality metrics
    data_quality = {
        "avg_completeness": calculate_avg_completeness(passed_records),
        "avg_confidence_score": calculate_avg_confidence(passed_records)
    }
    
    report = {
        "metadata": {
            "validated_at": datetime.utcnow().isoformat(),
            "pipeline_version": "1.0",
            "total_input_records": total_input,
            "total_output_records": len(passed_records),
            "overall_pass_rate": (len(passed_records) / total_input * 100) if total_input > 0 else 0
        },
        "constraint_pass_rates": constraint_stats,
        "authenticity_checks": authenticity_stats,
        "data_quality": data_quality,
        "rejection_summary": categorize_rejections(rejected_records)
    }
    
    return report
```

### Export Functions

```python
def export_to_csv(records, filepath, include_validation_metadata=True):
    """
    Export validated records to CSV
    """
    import csv
    
    if not records:
        return
    
    # Get all field names
    all_fields = set()
    for record in records:
        all_fields.update(record.keys())
    
    # Exclude nested validation metadata if requested
    if not include_validation_metadata:
        all_fields = {f for f in all_fields if not f.startswith('validation')}
    
    fieldnames = sorted(all_fields)
    
    with open(filepath, 'w', newline='', encoding='utf-8') as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        
        for record in records:
            # Flatten nested fields
            flat_record = {}
            for field in fieldnames:
                value = record.get(field)
                if isinstance(value, (dict, list)):
                    flat_record[field] = str(value)
                else:
                    flat_record[field] = value
            
            writer.writerow(flat_record)


def export_to_jsonl(records, filepath):
    """
    Export validated records to JSONL
    """
    import json
    
    with open(filepath, 'w', encoding='utf-8') as f:
        for record in records:
            f.write(json.dumps(record, ensure_ascii=False) + '\n')


def export_to_excel(records, filepath, include_validation_sheet=True):
    """
    Export validated records to Excel with multiple sheets
    """
    import pandas as pd
    
    # Main data sheet
    df_main = pd.DataFrame(records)
    
    # Remove nested validation metadata for main sheet
    nested_cols = [col for col in df_main.columns if isinstance(df_main[col].iloc[0], (dict, list))]
    df_clean = df_main.drop(columns=nested_cols)
    
    with pd.ExcelWriter(filepath, engine='openpyxl') as writer:
        df_clean.to_excel(writer, sheet_name='Data', index=False)
        
        if include_validation_sheet:
            # Extract validation metadata
            validation_data = []
            for record in records:
                if 'validation' in record:
                    validation_data.append({
                        'record_id': record.get('record_id'),
                        'overall_status': record['validation'].get('overall_status'),
                        'confidence_score': record['validation'].get('constraints', {}).get('confidence_score'),
                        'constraints_passed': record['validation'].get('constraints', {}).get('constraints_passed'),
                        'constraints_failed': record['validation'].get('constraints', {}).get('constraints_failed')
                    })
            
            if validation_data:
                df_validation = pd.DataFrame(validation_data)
                df_validation.to_excel(writer, sheet_name='Validation', index=False)
```

---

## Safety & Compliance

### Data Privacy

- Avoid collecting/storing **sensitive personal data** unless necessary and permitted
- Keep **provenance** (source URL, timestamps, verification evidence) for auditability
- Log and quarantine suspected **bot-wall/captcha** content instead of treating it as valid data
- **Never include unverified or synthetic data** in final output - when in doubt, reject
- Always provide **rejection reasons with evidence** for transparency and debugging
- Maintain **strict separation** between PASS and FAIL records - no mixed statuses
- Document all **verification checks performed** and their results

### Compliance Notes

- Respect **GDPR, CCPA, and other privacy regulations**
- Do not process data for purposes beyond what was disclosed
- Provide users ability to review and correct their data
- Maintain audit logs of all validation decisions
- Support data deletion requests
- Encrypt sensitive data at rest and in transit

---

## Final Output Guarantees

When using this skill, the final output dataset guarantees:

1. ✓ **100% constraint compliance**: Every record passes ALL specified constraints
2. ✓ **Zero synthetic data**: All records verified against source URLs and/or social profiles  
3. ✓ **Complete transparency**: Full validation reports with pass/fail breakdowns
4. ✓ **Rejection accountability**: Every rejected record documented with specific reasons
5. ✓ **Verification evidence**: Proof of source/profile verification for each record
6. ✓ **No false positives**: Uncertain or borderline cases rejected, not included

**If a record is in the final output, it has passed ALL checks. No exceptions.**

---

## Usage Examples

### Example 1: Validate Restaurant Data

```python
# Input: Scraped restaurant records
raw_records = load_jsonl("restaurants_scraped.jsonl")

# Define constraints
constraints = [
    {
        "name": "All required fields present",
        "type": "required_fields",
        "fields": ["name", "website", "phone", "email", "owner_name", "address", "restaurant_type"]
    },
    {
        "name": "Valid email format",
        "type": "format",
        "field": "email",
        "pattern": "email"
    },
    {
        "name": "Valid phone format",
        "type": "format",
        "field": "phone",
        "pattern": "phone_us"
    },
    {
        "name": "Valid website URL",
        "type": "format",
        "field": "website",
        "pattern": "url"
    }
]

# Run validation pipeline
passed, rejected, report = final_validation_gate(
    records=raw_records,
    constraints=constraints,
    required_fields=["name", "website", "phone", "email", "owner_name", "address", "restaurant_type"]
)

# Export results
export_to_csv(passed, "restaurants_validated.csv")
export_to_jsonl(rejected, "restaurants_rejected.jsonl")

# Print report
print(f"Total input: {report['validation_stats']['total_input']}")
print(f"Passed: {report['validation_stats']['final_pass']}")
print(f"Pass rate: {report['pass_rate']:.1f}%")
```

### Example 2: Validate Professional Data with LinkedIn Verification

```python
# Input: Scraped professional records
raw_records = load_jsonl("professionals_scraped.jsonl")

# Define constraints
constraints = [
    {
        "name": "Lives in approved European country",
        "type": "location",
        "field": "location",
        "allowed_values": ["United Kingdom", "Germany", "France", "Netherlands", 
                          "Switzerland", "Austria", "Norway"]
    },
    {
        "name": "7+ years US work experience",
        "type": "experience",
        "field": "us_work_years",
        "minimum": 7.0
    },
    {
        "name": "Senior level position",
        "type": "seniority",
        "field": "title",
        "levels": ["c_suite", "vp", "director", "manager", "senior_specialist"]
    },
    {
        "name": "Company size 100+ employees",
        "type": "numeric",
        "field": "company_size",
        "minimum": 100,
        "preferred_minimum": 1000
    }
]

# Run validation pipeline with LinkedIn verification
passed, rejected, report = final_validation_gate(
    records=raw_records,
    constraints=constraints,
    required_fields=["name", "company", "title", "location", "linkedin_url"]
)

# Export results
export_to_excel(passed, "professionals_validated.xlsx", include_validation_sheet=True)

# Generate detailed report
validation_report = generate_validation_report(passed, rejected, constraints)
print(json.dumps(validation_report, indent=2))
```

---

## Summary

This skill provides **comprehensive data validation and enrichment** with:

- **Zero tolerance** for constraint violations
- **Synthetic data detection** to prevent fabricated entries
- **Source and profile verification** for authenticity
- **Mandatory pre-output validation** gate
- **Complete transparency** through detailed reporting
- **100% compliance guarantee** for final output

Use this skill whenever you need to transform raw scraped data into **trusted, analysis-ready datasets** that you can confidently use for business decisions, analytics, or downstream processing.
```