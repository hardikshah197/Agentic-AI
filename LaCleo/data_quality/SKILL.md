---
name: data-validation-enrichment
description: Validate, clean, normalize, deduplicate, and enrich scraped data into high-quality, analysis-ready datasets. Use when Claude needs to QA scraped records, enforce schemas, detect anomalies, standardize formats (dates/currency/units), merge duplicates (entity resolution), and add enrichment signals (geocoding, categorization, language detection, etc.).
---

# Data Validation & Enrichment Skill (for scraped data)

Turn noisy scraped output into **trusted, structured, and enriched** data suitable for analytics, search, or downstream ML.

## When to use

- Scraped records have **missing/dirty fields**, inconsistent formatting, or duplicates
- You need **schema enforcement** (types, required fields, constraints)
- You need **standardization** (dates, currency, units, addresses, categories)
- You need **entity resolution** (merge multiple pages into one entity)
- You need **enrichment** (geo coords, inferred categories, normalized company/site info)

## Inputs

- **Raw records**: JSON/JSONL/CSV (from scraper/crawler)
- **Schema/spec**
  - Field names, types, required/optional
  - Constraints (regex, ranges, allowed values)
  - Uniqueness keys (e.g., `canonical_url`, `source_id`)
- **Normalization rules**
  - Date formats/timezones, currency/locale rules
  - Unit conversions (kg↔lb, cm↔in), text cleanup
- **Dedupe & merge rules**
  - Entity keys and match thresholds
  - Field precedence (which source wins) and merge strategy
- **Enrichment sources**
  - Lookups (geo, language, classification, domain parsing)
  - Optional external APIs (must be configured by user)

## Outputs

- **Clean dataset** (JSONL/CSV/Parquet/DB rows) matching schema
- **Validation report**
  - missing/invalid counts per field
  - outliers/anomaly flags
  - dedupe merges performed
- **Error/quarantine set**
  - records that failed hard validation
  - reason codes and suggested fixes
- **Provenance**
  - `scraped_at`, `validated_at`, `source_url`, `record_hash`, `pipeline_version`

## Core pipeline

1. **Ingest & parse**
   - Decode encodings, normalize column names, coerce to known structure
2. **Canonicalize identifiers**
   - `canonical_url`, remove tracking params, normalize casing/whitespace
3. **Standardize fields**
   - Trim text, normalize phone/address formats, parse dates, parse numbers
4. **Validate**
   - Types, required fields, constraints, cross-field rules
5. **Detect anomalies**
   - Outlier detection (e.g., negative price), suspicious empty pages, block pages
6. **Deduplicate & entity resolution**
   - Exact dedupe (keys) + fuzzy matching (names/addresses)
   - Merge policy and conflict resolution
7. **Enrich**
   - Derived fields, taxonomy classification, geocoding, language detection, etc.
8. **Export**
   - Write cleaned dataset + reports + quarantined records

## Validation checks (what “good” looks like)

- **Schema validity**
  - required fields present (e.g., `title`, `url`)
  - correct types (dates/numbers/booleans)
- **Constraints**
  - regex: emails, phones, postal codes
  - ranges: price >= 0, rating within [0, 5]
  - enums: allowed categories/statuses
- **Cross-field rules**
  - `currency` required if `price` present
  - `available=false` implies `stock` is 0 or null
  - `start_date <= end_date`
- **Content sanity**
  - minimum text length for descriptions
  - HTML remnants removed (`<div>`, `&nbsp;`)
  - “blocked page” detection (captcha/login wall keywords)

## Standardization & normalization

- **URL canonicalization**
  - remove `utm_*`, `gclid`, `fbclid`
  - normalize trailing slashes, lowercase host, decode/encode safely
- **Link normalization (internal + external)**
  - normalize scheme (`http`→`https` where appropriate), remove fragments, resolve relative → absolute
  - collapse duplicate slashes, decode punycode domains, strip surrounding punctuation
  - detect and drop non-web links when not expected: `javascript:`, `mailto:`, `tel:`, `data:`
- **Text cleanup**
  - trim, collapse whitespace, normalize unicode (NFKC), remove zero-width chars
- **Numbers & currency**
  - parse locale-aware formats (`1,234.56` vs `1.234,56`)
  - split `price` into `price_amount` + `currency`
- **Dates**
  - parse multiple patterns, normalize to ISO-8601, store timezone
- **Units**
  - extract and convert units to a standard system (e.g., metric)
- **Addresses**
  - separate `street`, `city`, `region`, `postal_code`, `country` when possible

## Link, social profile, and person/org field validation

Scraped datasets often include websites, social profiles, and entity attributes (people/organizations). Validate these fields explicitly to prevent broken links, wrong profile types, and mis-attributed entities.

### Link validation (generic URLs)

- **Rule when sharing links**
  - If you output/share any link (in the final dataset, reports, or messages), it must be validated using the **link metadata you already have** for that link (e.g., `http_status`, `final_url`, `redirect_chain[]`, `checked_at`).
  - If metadata is missing, mark the link as **unverified** (or run a link-check step if allowed) rather than implying it is valid.

- **Syntax**
  - must parse as a URL; allow only expected schemes (typically `http`, `https`)
  - host must exist and be valid; optionally enforce public suffix rules
- **Safety**
  - prevent SSRF-style targets if running in a networked environment (block private IP ranges, localhost, link-local)
  - optionally block unexpected ports
- **Reachability (optional but recommended)**
  - issue `HEAD` (fallback to `GET`) with timeouts and limited redirects
  - treat 2xx as valid; record 3xx final URL; quarantine consistent 4xx/5xx
  - capture: `http_status`, `final_url`, `redirect_chain[]`, `checked_at`
- **Consistency**
  - if `domain` field exists, ensure URL host matches expected domain (or allowed list)
  - validate `canonical_url` vs `url` and keep both with provenance

### Social profile validation (LinkedIn/X/GitHub/etc.)

- **Normalize to canonical forms**
  - `linkedin.com/company/<slug>` vs `linkedin.com/in/<slug>`
  - `x.com/<handle>` or `twitter.com/<handle>` (store as canonical `x.com`)
  - `github.com/<org_or_user>`
  - `facebook.com/<page_or_profile>`, `instagram.com/<handle>`, `youtube.com/@<handle>`/`/channel/<id>`
- **Type correctness**
  - person profile fields must point to person paths (e.g., LinkedIn `/in/`)
  - org profile fields must point to org paths (e.g., LinkedIn `/company/`)
- **Handle rules**
  - validate allowed character sets and lengths per platform where possible
  - strip `@`, trailing slashes, and tracking params
- **Reachability (optional)**
  - check that the profile URL resolves (avoid treating platform login/consent pages as valid content)
- **Deduplication**
  - canonicalize and dedupe social URLs across records
  - reconcile fields like `twitter`, `x`, `linkedin_url` into a single normalized set

### Organization fields (examples) + checks

- **Identity**
  - `org_name` non-empty; normalize casing/whitespace; remove legal suffix noise only when configured (Inc/LLC/Ltd)
  - `org_website` must be a valid URL and domain should match email domain when reasonable
- **Contact**
  - `emails[]`: validate format; optionally validate domain MX (if allowed)
  - `phones[]`: normalize to E.164; flag ambiguous numbers without region
- **Location**
  - address components coherent; country/region codes normalized (ISO-3166 where possible)
- **Consistency**
  - if social profile is org-type, ensure `org_name` exists; if `org_website` host differs wildly from social host, flag for review

### Person fields (examples) + checks

- **Name normalization**
  - split into `first_name`, `last_name` when present; keep `full_name` as raw
  - remove honorifics/suffixes (Dr, Mr, Jr, III) only when configured
- **Role/title**
  - standardize common variants (e.g., “VP”, “Vice President”)
  - prevent title being mistaken as name (common scraping error)
- **Contact**
  - validate email/phone similarly, but be stricter about compliance/PII handling
- **Entity resolution**
  - merge people cautiously: use combinations of `full_name + org + location` and/or unique profiles (LinkedIn/GitHub)
  - keep `person_id` stable and store merge evidence

## Dedupe & entity resolution

### Exact dedupe
- Keyed by `canonical_url` or `source_id`
- Prefer newest `scraped_at` or most complete record

### Fuzzy matching (when URLs differ)
- Compare normalized fields: name/title, address, phone, domain
- Use similarity thresholds (token-based, Jaro-Winkler, etc.)
- Keep merge logs: which records merged, why, and which fields won

### Merge policy (recommended)
- **Prefer non-null** values over null
- **Prefer authoritative source** (user-defined priority list)
- **Prefer structured** fields over raw text (e.g., parsed `price_amount`)
- Record conflicts in `conflicts[]` for audit

## Enrichment options

- **Derived fields**
  - `domain` from URL, `slug`, `record_hash`, `is_active` heuristics
- **Link enrichment**
  - extract and store `domain`, `registered_domain`, `path`, `is_https`, and `url_normalized`
  - keep `final_url` and `redirect_chain[]` from link checks
- **Geocoding**
  - convert address → lat/lng + standardized address (requires configured provider)
- **Taxonomy classification**
  - map free-text categories to a controlled taxonomy
  - rule-based keywords or ML classifier (optional)
- **Language detection**
  - detect content language and store `lang`
- **Currency normalization**
  - convert to base currency with exchange-rate timestamp (requires configured FX source)
- **Contact normalization**
  - normalize phones to E.164, validate emails (avoid scraping sensitive PII unless permitted)

## Example: Schema validation with Pydantic (Python)

```python
from __future__ import annotations

from datetime import datetime
from typing import Optional
from pydantic import BaseModel, Field, HttpUrl, field_validator


class ScrapedItem(BaseModel):
    url: HttpUrl
    title: str = Field(min_length=1)
    scraped_at: datetime
    price_amount: Optional[float] = Field(default=None, ge=0)
    currency: Optional[str] = Field(default=None, min_length=3, max_length=3)

    @field_validator("currency")
    @classmethod
    def currency_upper(cls, v: Optional[str]) -> Optional[str]:
        return v.upper() if v else v

    @field_validator("currency")
    @classmethod
    def currency_if_price_present(cls, v: Optional[str], info):
        price = info.data.get("price_amount")
        if price is not None and not v:
            raise ValueError("currency required when price_amount is present")
        return v
```

## Example: Data quality checks with Pandas

```python
import pandas as pd

df = pd.read_json("items.jsonl", lines=True)

# Missing required fields
missing_title = df["title"].isna() | (df["title"].astype(str).str.strip() == "")

# Outliers
bad_price = df["price_amount"].notna() & (df["price_amount"] < 0)

bad = df[missing_title | bad_price].copy()
good = df[~(missing_title | bad_price)].copy()

good.to_json("items.cleaned.jsonl", orient="records", lines=True, force_ascii=False)
bad.to_json("items.quarantine.jsonl", orient="records", lines=True, force_ascii=False)

report = {
    "total": len(df),
    "clean": len(good),
    "quarantine": len(bad),
    "missing_title": int(missing_title.sum()),
    "bad_price": int(bad_price.sum()),
}
print(report)
```

## Reporting (recommended fields)

- `run_id`, `pipeline_version`
- counts: `input_records`, `clean_records`, `quarantined_records`
- per-field: missing %, invalid %, top error reasons
- dedupe: duplicates found, merges performed
- enrichment: success rates (e.g., geocode hit rate)

## Safety & compliance notes

- Avoid collecting/storing **sensitive personal data** unless necessary and permitted.
- Keep **provenance** (source URL, timestamps) for auditability.
- Log and quarantine suspected **bot-wall/captcha** content instead of treating it as valid data.
