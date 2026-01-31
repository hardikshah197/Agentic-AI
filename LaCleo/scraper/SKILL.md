---
name: web-scraper
description: Comprehensive web scraping toolkit for extracting data from websites with different structures, APIs, and formats. Use when Claude needs to scrape or extract data from websites, handle different HTML structures, parse API responses (JSON/XML), handle pagination, authentication, rate limiting, or convert scraped data into structured formats (CSV, JSON, Excel, etc.). CRITICAL: All scraped data must be verified for authenticity and validated against constraints before delivery.
---

# Web Scraper Skill

Comprehensive toolkit for web scraping across different website structures, API endpoints, and data formats **with built-in validation to prevent false and synthetic results**.

---

## CRITICAL OPERATING PRINCIPLES

### 1. NO FABRICATED DATA
- **NEVER invent or generate data** - only extract what exists on the source
- **Verify all extractions** - ensure data actually exists at source URL
- **Flag extraction failures** - mark fields as "NOT_FOUND" rather than guessing
- **Preserve source URLs** - always include source URL for each record

### 2. EXTRACTION VALIDATION
- **Cross-check extractions** - verify extracted data makes sense in context
- **Detect empty/blocked pages** - identify captcha, login walls, bot blocks
- **Validate data formats** - ensure emails are emails, phones are phones, etc.
- **Check for placeholder content** - reject template/dummy data

### 3. QUALITY OVER QUANTITY
- **Prefer accuracy over speed** - take time to validate each extraction
- **Stop on detection failure** - halt if bot detection or blocking occurs
- **Log extraction evidence** - keep track of what was extracted and where
- **Provide extraction confidence** - indicate certainty level for each field

---

## Quick Start

For simple HTML scraping:
```python
import requests
from bs4 import BeautifulSoup

response = requests.get(url, headers={'User-Agent': 'Mozilla/5.0'})

# CRITICAL: Check for bot detection BEFORE parsing
if is_blocked_page(response):
    raise Exception("Bot detection / CAPTCHA / Login wall detected")

soup = BeautifulSoup(response.content, 'html.parser')
data = soup.find_all('div', class_='target-class')

# CRITICAL: Validate extracted data
validated_data = validate_extractions(data)
```

For complex scenarios, see references and scripts below.

---

## Core Workflow

1. **Analyze the target** - Determine site structure, authentication, rate limits
2. **Choose the approach** - HTML parsing, API calls, or headless browser
3. **Extract data** - Use appropriate parsing method
4. **VALIDATE EXTRACTIONS** - Verify data is real, not fabricated or blocked content
5. **CHECK FOR BOT DETECTION** - Ensure pages aren't captcha/login walls
6. **Structure output** - Convert to requested format (CSV, JSON, Excel, etc.)
7. **INCLUDE PROVENANCE** - Add source URLs and extraction timestamps

---

## Bot Detection & Blocking Prevention

### Detection Patterns

```python
def is_blocked_page(response):
    """
    Detect if page is blocked, captcha, or login wall
    Returns: (is_blocked: bool, reason: str)
    """
    content = response.text.lower()
    
    # Check status code
    if response.status_code == 403:
        return True, "403 Forbidden - Access denied"
    
    if response.status_code == 429:
        return True, "429 Too Many Requests - Rate limited"
    
    # Check for common blocking indicators
    BLOCKING_INDICATORS = [
        'captcha',
        'recaptcha',
        'cloudflare',
        'access denied',
        'forbidden',
        'bot detected',
        'unusual traffic',
        'verify you are human',
        'please enable javascript',
        'security check',
        'challenge-platform',
        'cf-browser-verification',
        'login to continue',
        'sign in to access',
        'authentication required'
    ]
    
    for indicator in BLOCKING_INDICATORS:
        if indicator in content:
            return True, f"Blocking indicator detected: {indicator}"
    
    # Check for suspiciously short content
    if len(content.strip()) < 100:
        return True, "Page content suspiciously short (possible block)"
    
    # Check for lack of expected content
    soup = BeautifulSoup(response.content, 'html.parser')
    if not soup.find_all(['p', 'div', 'article', 'section']):
        return True, "No substantial content elements found"
    
    return False, ""


def validate_page_content(soup, expected_selectors):
    """
    Validate that expected content is present
    
    Args:
        soup: BeautifulSoup object
        expected_selectors: List of CSS selectors that should exist
        
    Returns:
        (is_valid: bool, missing: list)
    """
    missing = []
    
    for selector in expected_selectors:
        if not soup.select(selector):
            missing.append(selector)
    
    if missing:
        return False, missing
    
    return True, []
```

### Respectful Scraping

```python
class RespectfulScraper:
    """Base class with built-in politeness and validation"""
    
    def __init__(self, base_url, delay=2.0, max_retries=3):
        self.base_url = base_url
        self.delay = delay
        self.max_retries = max_retries
        self.session = requests.Session()
        
        # Realistic headers
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36',
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
            'Accept-Language': 'en-US,en;q=0.5',
            'Accept-Encoding': 'gzip, deflate, br',
            'DNT': '1',
            'Connection': 'keep-alive',
            'Upgrade-Insecure-Requests': '1'
        })
        
        # Track request timing
        self.last_request_time = 0
        self.blocked_count = 0
    
    def fetch_with_validation(self, url):
        """
        Fetch URL with bot detection and validation
        """
        # Enforce delay
        import time
        time_since_last = time.time() - self.last_request_time
        if time_since_last < self.delay:
            time.sleep(self.delay - time_since_last)
        
        # Make request
        try:
            response = self.session.get(url, timeout=10)
            self.last_request_time = time.time()
            
            # Check for blocking
            is_blocked, reason = is_blocked_page(response)
            if is_blocked:
                self.blocked_count += 1
                raise BlockedError(f"Page blocked: {reason}")
            
            # Validate response
            if response.status_code != 200:
                raise Exception(f"HTTP {response.status_code}")
            
            return response
        
        except BlockedError:
            raise
        except Exception as e:
            logger.error(f"Request failed: {e}")
            raise


class BlockedError(Exception):
    """Raised when bot detection or blocking occurs"""
    pass
```

---

## Data Extraction with Validation

### Extract with Confidence Scoring

```python
def extract_field_with_confidence(soup, selectors, field_name):
    """
    Extract field using multiple selectors with confidence scoring
    
    Args:
        soup: BeautifulSoup object
        selectors: List of CSS selectors to try (in priority order)
        field_name: Name of the field being extracted
        
    Returns:
        {
            'value': extracted_value or None,
            'confidence': 0-100,
            'source_selector': selector_used,
            'extraction_method': 'css_selector',
            'found': bool
        }
    """
    for selector in selectors:
        elements = soup.select(selector)
        
        if elements:
            # Extract text
            text = elements[0].get_text(strip=True)
            
            # Validate extraction
            if not text or text.lower() in ['n/a', 'none', 'null', '', '-']:
                continue
            
            # Check for placeholder/template text
            if is_placeholder_text(text):
                continue
            
            # Calculate confidence
            confidence = calculate_extraction_confidence(
                text, 
                field_name, 
                selector_index=selectors.index(selector)
            )
            
            return {
                'value': text,
                'confidence': confidence,
                'source_selector': selector,
                'extraction_method': 'css_selector',
                'found': True
            }
    
    # Not found
    return {
        'value': None,
        'confidence': 0,
        'source_selector': None,
        'extraction_method': 'css_selector',
        'found': False
    }


def is_placeholder_text(text):
    """Detect placeholder/template text"""
    PLACEHOLDER_PATTERNS = [
        r'lorem ipsum',
        r'example\.com',
        r'test@test\.com',
        r'placeholder',
        r'your (name|email|phone|company)',
        r'enter your',
        r'click here',
        r'\[.*\]',  # [Bracket text]
        r'{{.*}}',  # {{Template}}
        r'sample (text|data|content)',
        r'dummy (text|data|content)'
    ]
    
    import re
    text_lower = text.lower()
    
    for pattern in PLACEHOLDER_PATTERNS:
        if re.search(pattern, text_lower):
            return True
    
    return False


def calculate_extraction_confidence(value, field_name, selector_index):
    """
    Calculate confidence score for extracted value
    
    Args:
        value: Extracted text value
        field_name: Name of field (email, phone, etc.)
        selector_index: Which selector in priority list was used (0 = highest priority)
        
    Returns:
        Confidence score (0-100)
    """
    confidence = 100
    
    # Penalize lower-priority selectors
    confidence -= (selector_index * 10)
    
    # Field-specific validation
    if field_name == 'email':
        if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', value):
            confidence -= 50
    
    elif field_name == 'phone':
        if not re.search(r'\d{3}[-.\s]?\d{3}[-.\s]?\d{4}', value):
            confidence -= 30
    
    elif field_name == 'url':
        if not value.startswith(('http://', 'https://', 'www.')):
            confidence -= 40
    
    # Penalize very short values (unless expected)
    if field_name not in ['name', 'title'] and len(value) < 3:
        confidence -= 30
    
    # Penalize suspiciously long values
    if len(value) > 500:
        confidence -= 20
    
    return max(confidence, 0)
```

### Multi-Source Cross-Validation

```python
def extract_with_cross_validation(url, extractors):
    """
    Extract data using multiple methods and cross-validate
    
    Args:
        url: Target URL
        extractors: List of extraction functions
        
    Returns:
        Validated extraction results
    """
    results = []
    
    for extractor in extractors:
        try:
            result = extractor(url)
            results.append(result)
        except Exception as e:
            logger.warning(f"Extractor failed: {e}")
            continue
    
    if not results:
        return None
    
    # Cross-validate results
    validated = {}
    
    for field in results[0].keys():
        values = [r[field] for r in results if field in r and r[field]]
        
        if not values:
            validated[field] = {
                'value': None,
                'confidence': 0,
                'verified': False,
                'sources': 0
            }
            continue
        
        # Check if values agree
        unique_values = set(values)
        
        if len(unique_values) == 1:
            # All sources agree
            validated[field] = {
                'value': values[0],
                'confidence': 95,
                'verified': True,
                'sources': len(values)
            }
        else:
            # Conflict - use most common value
            from collections import Counter
            most_common = Counter(values).most_common(1)[0]
            
            validated[field] = {
                'value': most_common[0],
                'confidence': (most_common[1] / len(values)) * 100,
                'verified': False,
                'sources': len(values),
                'conflict': True,
                'all_values': list(unique_values)
            }
    
    return validated
```

---

## Scraping Approaches

### Static HTML Scraping (Enhanced with Validation)

```python
#!/usr/bin/env python3
"""
HTML Scraper with built-in validation
Prevents extraction of blocked/fake content
"""

import requests
from bs4 import BeautifulSoup
import json
import csv
import time
from typing import List, Dict, Any, Optional
from urllib.parse import urljoin, urlparse
import logging
import re

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


class ValidatedHTMLScraper:
    """Scraper with built-in validation to prevent false data"""
    
    def __init__(self, base_url: str, delay: float = 2.0):
        """
        Initialize scraper
        
        Args:
            base_url: Base URL of the website
            delay: Delay between requests in seconds (default: 2.0 for politeness)
        """
        self.base_url = base_url
        self.delay = delay
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36',
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
            'Accept-Language': 'en-US,en;q=0.5',
        })
        
        self.last_request_time = 0
        self.extraction_stats = {
            'total_attempts': 0,
            'successful_extractions': 0,
            'blocked_pages': 0,
            'validation_failures': 0
        }
    
    def fetch_page(self, url: str, retries: int = 3) -> Optional[BeautifulSoup]:
        """
        Fetch and parse a webpage with validation
        
        Args:
            url: URL to fetch
            retries: Number of retry attempts
            
        Returns:
            BeautifulSoup object or None if failed/blocked
        """
        # Enforce delay
        time_since_last = time.time() - self.last_request_time
        if time_since_last < self.delay:
            time.sleep(self.delay - time_since_last)
        
        for attempt in range(retries):
            try:
                response = self.session.get(url, timeout=10)
                self.last_request_time = time.time()
                
                response.raise_for_status()
                
                # CRITICAL: Check for bot detection
                is_blocked, reason = self._check_if_blocked(response)
                if is_blocked:
                    self.extraction_stats['blocked_pages'] += 1
                    logger.error(f"Page blocked: {reason}")
                    return None
                
                # Parse and validate content
                soup = BeautifulSoup(response.content, 'html.parser')
                
                if not self._validate_page_content(soup):
                    self.extraction_stats['validation_failures'] += 1
                    logger.warning(f"Page validation failed for {url}")
                    return None
                
                return soup
                
            except requests.RequestException as e:
                logger.warning(f"Attempt {attempt + 1} failed: {e}")
                if attempt < retries - 1:
                    time.sleep(2 ** attempt)  # Exponential backoff
                else:
                    logger.error(f"Failed to fetch {url}")
                    return None
    
    def _check_if_blocked(self, response) -> tuple[bool, str]:
        """Check if page is blocked, captcha, or login wall"""
        content = response.text.lower()
        
        # Check status code
        if response.status_code == 403:
            return True, "403 Forbidden"
        if response.status_code == 429:
            return True, "429 Rate Limited"
        
        # Check for blocking indicators
        BLOCKING_INDICATORS = [
            'captcha', 'recaptcha', 'cloudflare', 'access denied',
            'forbidden', 'bot detected', 'unusual traffic',
            'verify you are human', 'security check',
            'login to continue', 'sign in to access'
        ]
        
        for indicator in BLOCKING_INDICATORS:
            if indicator in content:
                return True, f"Blocking indicator: {indicator}"
        
        # Check for suspiciously short content
        if len(content.strip()) < 100:
            return True, "Content too short (possible block)"
        
        return False, ""
    
    def _validate_page_content(self, soup: BeautifulSoup) -> bool:
        """Validate that page has substantial content"""
        # Check for basic content elements
        content_elements = soup.find_all(['p', 'div', 'article', 'section', 'main'])
        
        if len(content_elements) < 3:
            logger.warning("Insufficient content elements")
            return False
        
        # Check total text content
        text_content = soup.get_text(strip=True)
        if len(text_content) < 200:
            logger.warning("Insufficient text content")
            return False
        
        return True
    
    def extract_with_validation(self, soup: BeautifulSoup, field_name: str,
                                selectors: List[str], attribute: Optional[str] = None) -> Dict:
        """
        Extract field with validation and confidence scoring
        
        Args:
            soup: BeautifulSoup object
            field_name: Name of field being extracted
            selectors: List of CSS selectors (in priority order)
            attribute: HTML attribute to extract (None for text)
            
        Returns:
            Extraction result with confidence score
        """
        self.extraction_stats['total_attempts'] += 1
        
        for idx, selector in enumerate(selectors):
            elements = soup.select(selector)
            
            if not elements:
                continue
            
            # Extract value
            if attribute:
                value = elements[0].get(attribute, '').strip()
            else:
                value = elements[0].get_text(strip=True)
            
            # Validate extraction
            if not value or value.lower() in ['n/a', 'none', 'null', '', '-']:
                continue
            
            # Check for placeholder text
            if self._is_placeholder(value):
                logger.warning(f"Placeholder text detected: {value}")
                continue
            
            # Calculate confidence
            confidence = self._calculate_confidence(value, field_name, idx)
            
            if confidence >= 50:  # Minimum confidence threshold
                self.extraction_stats['successful_extractions'] += 1
                
                return {
                    'value': value,
                    'confidence': confidence,
                    'selector_used': selector,
                    'found': True,
                    'validated': True
                }
        
        # Not found or failed validation
        return {
            'value': None,
            'confidence': 0,
            'selector_used': None,
            'found': False,
            'validated': False
        }
    
    def _is_placeholder(self, text: str) -> bool:
        """Detect placeholder/template text"""
        PLACEHOLDER_PATTERNS = [
            r'lorem ipsum',
            r'example\.com',
            r'test@test',
            r'placeholder',
            r'your (name|email|phone|company|address)',
            r'enter your',
            r'\[.*\]',
            r'{{.*}}',
            r'sample (text|data)',
            r'dummy'
        ]
        
        text_lower = text.lower()
        for pattern in PLACEHOLDER_PATTERNS:
            if re.search(pattern, text_lower):
                return True
        
        return False
    
    def _calculate_confidence(self, value: str, field_name: str, selector_index: int) -> int:
        """Calculate extraction confidence score"""
        confidence = 100
        
        # Penalize lower-priority selectors
        confidence -= (selector_index * 15)
        
        # Field-specific validation
        if field_name == 'email':
            if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', value):
                confidence -= 60
            if any(invalid in value.lower() for invalid in ['example', 'test', 'sample']):
                confidence -= 70
        
        elif field_name == 'phone':
            # Clean phone number
            cleaned = re.sub(r'[^\d+]', '', value)
            if len(cleaned) < 10:
                confidence -= 50
        
        elif field_name == 'url':
            if not value.startswith(('http://', 'https://', 'www.')):
                confidence -= 50
        
        # Penalize very short values
        if field_name not in ['name'] and len(value) < 3:
            confidence -= 40
        
        # Penalize suspiciously long values
        if len(value) > 500:
            confidence -= 30
        
        return max(confidence, 0)
    
    def scrape_with_provenance(self, url: str, extraction_config: Dict) -> Dict:
        """
        Scrape with full provenance tracking
        
        Args:
            url: Target URL
            extraction_config: Configuration specifying what to extract
            
        Returns:
            Scraped data with provenance metadata
        """
        soup = self.fetch_page(url)
        
        if not soup:
            return {
                'success': False,
                'error': 'Failed to fetch or validate page',
                'source_url': url
            }
        
        extracted_data = {'source_url': url, 'scraped_at': time.strftime('%Y-%m-%dT%H:%M:%SZ')}
        extraction_metadata = {}
        
        for field_name, selectors in extraction_config.items():
            result = self.extract_with_validation(
                soup, 
                field_name, 
                selectors if isinstance(selectors, list) else [selectors]
            )
            
            extracted_data[field_name] = result['value']
            extraction_metadata[field_name] = {
                'confidence': result['confidence'],
                'found': result['found'],
                'validated': result['validated'],
                'selector_used': result['selector_used']
            }
        
        return {
            'success': True,
            'data': extracted_data,
            'metadata': extraction_metadata,
            'overall_confidence': self._calculate_overall_confidence(extraction_metadata)
        }
    
    def _calculate_overall_confidence(self, metadata: Dict) -> int:
        """Calculate overall record confidence"""
        confidences = [m['confidence'] for m in metadata.values() if m['found']]
        
        if not confidences:
            return 0
        
        # Average confidence
        avg_confidence = sum(confidences) / len(confidences)
        
        # Penalize if many fields not found
        total_fields = len(metadata)
        found_fields = sum(1 for m in metadata.values() if m['found'])
        completeness_ratio = found_fields / total_fields
        
        return int(avg_confidence * completeness_ratio)
    
    def save_with_metadata(self, records: List[Dict], filename: str):
        """Save with extraction metadata"""
        output = {
            'extraction_stats': self.extraction_stats,
            'total_records': len(records),
            'timestamp': time.strftime('%Y-%m-%dT%H:%M:%SZ'),
            'records': records
        }
        
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(output, f, indent=2, ensure_ascii=False)
        
        logger.info(f"Saved {len(records)} records with metadata to {filename}")
```

### Example Usage

```python
# Configuration
extraction_config = {
    'name': ['h1.restaurant-name', 'div.name', 'span.title'],
    'phone': ['a[href^="tel:"]', 'span.phone', 'div.contact-phone'],
    'email': ['a[href^="mailto:"]', 'span.email', 'div.contact-email'],
    'address': ['div.address', 'span.location', 'p.addr'],
    'website': ['a.website', 'a.external-link']
}

# Scrape with validation
scraper = ValidatedHTMLScraper('https://example.com', delay=2.0)

results = []
for url in target_urls:
    result = scraper.scrape_with_provenance(url, extraction_config)
    
    if result['success']:
        # Only include if confidence is acceptable
        if result['overall_confidence'] >= 70:
            results.append(result)
        else:
            logger.warning(f"Low confidence ({result['overall_confidence']}%) for {url}")
    else:
        logger.error(f"Failed to scrape {url}: {result.get('error')}")

# Save with metadata
scraper.save_with_metadata(results, 'validated_results.json')

# Print extraction statistics
print(f"Extraction Statistics:")
print(f"  Total attempts: {scraper.extraction_stats['total_attempts']}")
print(f"  Successful: {scraper.extraction_stats['successful_extractions']}")
print(f"  Blocked pages: {scraper.extraction_stats['blocked_pages']}")
print(f"  Validation failures: {scraper.extraction_stats['validation_failures']}")
```

---

## Dynamic Content (JavaScript) - Enhanced

```python
#!/usr/bin/env python3
"""
Selenium Scraper with validation
Prevents extraction of blocked/placeholder content
"""

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException, NoSuchElementException
from selenium.webdriver.chrome.options import Options
import json
import time
import logging
from typing import List, Dict, Any, Optional
import re

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


class ValidatedSeleniumScraper:
    """Selenium scraper with built-in validation"""
    
    def __init__(self, headless: bool = True, delay: float = 2.0):
        self.delay = delay
        self.driver = self._setup_driver(headless)
        self.extraction_stats = {
            'pages_visited': 0,
            'blocked_pages': 0,
            'successful_extractions': 0,
            'failed_extractions': 0
        }
    
    def _setup_driver(self, headless: bool) -> webdriver.Chrome:
        """Setup Chrome driver with stealth options"""
        options = Options()
        
        if headless:
            options.add_argument('--headless=new')
        
        # Stealth options
        options.add_argument('--no-sandbox')
        options.add_argument('--disable-dev-shm-usage')
        options.add_argument('--disable-blink-features=AutomationControlled')
        options.add_experimental_option('excludeSwitches', ['enable-automation'])
        options.add_experimental_option('useAutomationExtension', False)
        
        # Realistic user agent
        options.add_argument('user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36')
        
        driver = webdriver.Chrome(options=options)
        
        # Remove webdriver flag
        driver.execute_cdp_cmd('Page.addScriptToEvaluateOnNewDocument', {
            'source': '''
                Object.defineProperty(navigator, 'webdriver', {
                    get: () => undefined
                })
            '''
        })
        
        driver.implicitly_wait(10)
        return driver
    
    def navigate_with_validation(self, url: str) -> bool:
        """Navigate to URL and validate page loaded properly"""
        logger.info(f"Navigating to {url}")
        self.driver.get(url)
        time.sleep(self.delay)
        
        self.extraction_stats['pages_visited'] += 1
        
        # Check for bot detection
        if self._check_if_blocked():
            self.extraction_stats['blocked_pages'] += 1
            logger.error("Bot detection or blocking detected")
            return False
        
        # Wait for page to be ready
        try:
            WebDriverWait(self.driver, 10).until(
                lambda d: d.execute_script('return document.readyState') == 'complete'
            )
            return True
        except TimeoutException:
            logger.error("Page load timeout")
            return False
    
    def _check_if_blocked(self) -> bool:
        """Check if current page is blocked or captcha"""
        page_source = self.driver.page_source.lower()
        
        BLOCKING_INDICATORS = [
            'captcha', 'recaptcha', 'cloudflare',
            'access denied', 'bot detected',
            'verify you are human', 'security check'
        ]
        
        for indicator in BLOCKING_INDICATORS:
            if indicator in page_source:
                return True
        
        # Check for very short content
        body = self.driver.find_elements(By.TAG_NAME, 'body')
        if body and len(body[0].text.strip()) < 100:
            return True
        
        return False
    
    def extract_with_validation(self, selector: str, field_name: str,
                                attribute: Optional[str] = None) -> Dict:
        """Extract with validation and confidence scoring"""
        try:
            elements = self.driver.find_elements(By.CSS_SELECTOR, selector)
            
            if not elements:
                return {
                    'value': None,
                    'confidence': 0,
                    'found': False,
                    'validated': False
                }
            
            # Extract value
            if attribute:
                value = elements[0].get_attribute(attribute)
            else:
                value = elements[0].text
            
            value = value.strip() if value else ''
            
            # Validate
            if not value or value.lower() in ['n/a', 'none', 'null']:
                return {
                    'value': None,
                    'confidence': 0,
                    'found': True,
                    'validated': False,
                    'reason': 'Empty or null value'
                }
            
            # Check for placeholder
            if self._is_placeholder(value):
                return {
                    'value': None,
                    'confidence': 0,
                    'found': True,
                    'validated': False,
                    'reason': 'Placeholder text detected'
                }
            
            # Calculate confidence
            confidence = self._calculate_confidence(value, field_name)
            
            if confidence >= 50:
                self.extraction_stats['successful_extractions'] += 1
                return {
                    'value': value,
                    'confidence': confidence,
                    'found': True,
                    'validated': True
                }
            else:
                self.extraction_stats['failed_extractions'] += 1
                return {
                    'value': value,
                    'confidence': confidence,
                    'found': True,
                    'validated': False,
                    'reason': 'Low confidence'
                }
        
        except NoSuchElementException:
            return {
                'value': None,
                'confidence': 0,
                'found': False,
                'validated': False,
                'reason': 'Element not found'
            }
    
    def _is_placeholder(self, text: str) -> bool:
        """Detect placeholder text"""
        PLACEHOLDER_PATTERNS = [
            r'lorem ipsum', r'example\.com', r'test@',
            r'placeholder', r'your (name|email|phone)',
            r'\[.*\]', r'{{.*}}', r'sample', r'dummy'
        ]
        
        text_lower = text.lower()
        return any(re.search(p, text_lower) for p in PLACEHOLDER_PATTERNS)
    
    def _calculate_confidence(self, value: str, field_name: str) -> int:
        """Calculate confidence score"""
        confidence = 100
        
        # Field-specific validation
        if field_name == 'email':
            if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', value):
                confidence -= 60
        
        elif field_name == 'phone':
            cleaned = re.sub(r'[^\d+]', '', value)
            if len(cleaned) < 10:
                confidence -= 50
        
        elif field_name == 'url':
            if not value.startswith(('http://', 'https://', 'www.')):
                confidence -= 50
        
        # Length checks
        if len(value) < 2:
            confidence -= 40
        if len(value) > 500:
            confidence -= 30
        
        return max(confidence, 0)
    
    def scrape_with_provenance(self, url: str, extraction_config: Dict) -> Dict:
        """Scrape with full provenance tracking"""
        if not self.navigate_with_validation(url):
            return {
                'success': False,
                'error': 'Navigation or validation failed',
                'source_url': url
            }
        
        extracted_data = {
            'source_url': url,
            'scraped_at': time.strftime('%Y-%m-%dT%H:%M:%SZ')
        }
        extraction_metadata = {}
        
        for field_name, selector in extraction_config.items():
            result = self.extract_with_validation(selector, field_name)
            
            extracted_data[field_name] = result['value']
            extraction_metadata[field_name] = {
                'confidence': result['confidence'],
                'found': result['found'],
                'validated': result['validated']
            }
        
        # Calculate overall confidence
        confidences = [m['confidence'] for m in extraction_metadata.values() if m['found']]
        overall_confidence = sum(confidences) / len(confidences) if confidences else 0
        
        return {
            'success': True,
            'data': extracted_data,
            'metadata': extraction_metadata,
            'overall_confidence': int(overall_confidence)
        }
    
    def close(self):
        """Close browser"""
        self.driver.quit()
        logger.info("Browser closed")
        logger.info(f"Final stats: {self.extraction_stats}")
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()
```

---

## API Scraping (Enhanced)

*(Keep the existing API scraping section but add validation for API responses)*

```python
class ValidatedAPIScraper(APIScraper):
    """API scraper with response validation"""
    
    def fetch_json_validated(self, endpoint: str, params: Optional[Dict] = None) -> Optional[Dict]:
        """Fetch JSON with validation"""
        data = self.fetch_json(endpoint, params)
        
        if not data:
            return None
        
        # Validate response structure
        if not isinstance(data, dict):
            logger.error("Response is not a dictionary")
            return None
        
        # Check for error indicators
        if 'error' in data or 'errors' in data:
            logger.error(f"API returned error: {data}")
            return None
        
        # Check for empty/placeholder data
        if self._contains_placeholder_data(data):
            logger.warning("API response contains placeholder data")
            return None
        
        return data
    
    def _contains_placeholder_data(self, data: Dict) -> bool:
        """Check if API response contains placeholder/test data"""
        data_str = str(data).lower()
        
        PLACEHOLDER_INDICATORS = [
            'example.com', 'test@test', 'lorem ipsum',
            'sample data', 'placeholder', 'dummy'
        ]
        
        return any(indicator in data_str for indicator in PLACEHOLDER_INDICATORS)
```

---

## Best Practices (Updated)

1. **Always validate extractions** - Never trust scraped data blindly
2. **Check for bot detection** - Halt scraping if blocked
3. **Include provenance** - Always track source URLs and timestamps
4. **Set realistic delays** - Minimum 1-2 seconds between requests
5. **Use confidence scores** - Filter out low-confidence extractions
6. **Log everything** - Track successes, failures, and blocking
7. **Respect robots.txt** - Check and honor robots.txt rules
8. **Handle errors gracefully** - Implement retries with exponential backoff
9. **Validate data formats** - Ensure emails are emails, phones are phones
10. **Never fabricate data** - Mark as "NOT_FOUND" rather than guessing

---

## Legal and Ethical Guidelines

- Review website's Terms of Service
- Respect robots.txt directives
- Implement reasonable rate limiting (minimum 1-2 seconds between requests)
- Don't overload servers
- Attribute data sources when publishing
- Obtain permission for commercial use
- **Never fabricate or invent data**
- **Stop immediately if bot detection occurs**
- **Validate all extracted data before use**
```

## Key Enhancements Made:

1. **Bot Detection & Blocking Prevention**: Added comprehensive checks for captcha, login walls, and access blocks
2. **Extraction Validation**: Confidence scoring for each extracted field
3. **Placeholder Detection**: Identifies and rejects template/dummy text
4. **Provenance Tracking**: Every record includes source URL and extraction timestamp
5. **Cross-Validation**: Multi-source validation when possible
6. **Format Validation**: Field-specific validation (emails, phones, URLs)
7. **Extraction Statistics**: Tracks success/failure rates
8. **Stealth Mode**: Enhanced Selenium configuration to avoid detection
9. **Error Handling**: Comprehensive validation at every step
10. **Quality Metrics**: Overall confidence scoring for records

This ensures no synthetic or false data makes it into the final output from scraping.