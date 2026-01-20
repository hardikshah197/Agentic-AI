---
name: web-scraper
description: Comprehensive web scraping toolkit for extracting data from websites with different structures, APIs, and formats. Use when Claude needs to scrape or extract data from websites, handle different HTML structures, parse API responses (JSON/XML), handle pagination, authentication, rate limiting, or convert scraped data into structured formats (CSV, JSON, Excel, etc.).
---

# Web Scraper Skill

Comprehensive toolkit for web scraping across different website structures, API endpoints, and data formats.

## Quick Start

For simple HTML scraping:
```python
import requests
from bs4 import BeautifulSoup

response = requests.get(url, headers={'User-Agent': 'Mozilla/5.0'})
soup = BeautifulSoup(response.content, 'html.parser')
data = soup.find_all('div', class_='target-class')
```

For complex scenarios, see references and scripts below.

## Core Workflow

1. **Analyze the target** - Determine site structure, authentication, rate limits
2. **Choose the approach** - HTML parsing, API calls, or headless browser
3. **Extract data** - Use appropriate parsing method
4. **Structure output** - Convert to requested format (CSV, JSON, Excel, etc.)
5. **Handle errors** - Implement retries, logging, and error handling

## Scraping Approaches

### Static HTML Scraping
```
#!/usr/bin/env python3
"""
HTML Scraper for static websites
Handles basic HTML parsing with BeautifulSoup
"""

import requests
from bs4 import BeautifulSoup
import json
import csv
import time
from typing import List, Dict, Any, Optional
from urllib.parse import urljoin, urlparse
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


class HTMLScraper:
    """Scraper for static HTML websites"""
    
    def __init__(self, base_url: str, delay: float = 1.0):
        """
        Initialize scraper
        
        Args:
            base_url: Base URL of the website
            delay: Delay between requests in seconds (default: 1.0)
        """
        self.base_url = base_url
        self.delay = delay
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        })
    
    def fetch_page(self, url: str, retries: int = 3) -> Optional[BeautifulSoup]:
        """
        Fetch and parse a webpage
        
        Args:
            url: URL to fetch
            retries: Number of retry attempts
            
        Returns:
            BeautifulSoup object or None if failed
        """
        for attempt in range(retries):
            try:
                response = self.session.get(url, timeout=10)
                response.raise_for_status()
                time.sleep(self.delay)
                return BeautifulSoup(response.content, 'html.parser')
            except requests.RequestException as e:
                logger.warning(f"Attempt {attempt + 1} failed: {e}")
                if attempt < retries - 1:
                    time.sleep(2 ** attempt)  # Exponential backoff
                else:
                    logger.error(f"Failed to fetch {url}")
                    return None
    
    def extract_by_selector(self, soup: BeautifulSoup, selector: str, 
                           attribute: Optional[str] = None) -> List[str]:
        """
        Extract data using CSS selector
        
        Args:
            soup: BeautifulSoup object
            selector: CSS selector
            attribute: HTML attribute to extract (None for text content)
            
        Returns:
            List of extracted values
        """
        elements = soup.select(selector)
        if attribute:
            return [elem.get(attribute, '') for elem in elements]
        return [elem.get_text(strip=True) for elem in elements]
    
    def extract_table(self, soup: BeautifulSoup, table_selector: str = 'table') -> List[Dict]:
        """
        Extract table data
        
        Args:
            soup: BeautifulSoup object
            table_selector: CSS selector for table
            
        Returns:
            List of dictionaries representing rows
        """
        table = soup.select_one(table_selector)
        if not table:
            return []
        
        headers = [th.get_text(strip=True) for th in table.select('thead th')]
        if not headers:
            headers = [th.get_text(strip=True) for th in table.select('tr th')]
        
        rows = []
        for tr in table.select('tbody tr, tr'):
            cells = tr.select('td')
            if cells:
                row_data = {headers[i]: cell.get_text(strip=True) 
                           for i, cell in enumerate(cells) if i < len(headers)}
                rows.append(row_data)
        
        return rows
    
    def save_to_json(self, data: List[Dict], filename: str):
        """Save data to JSON file"""
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(data, f, indent=2, ensure_ascii=False)
        logger.info(f"Saved {len(data)} items to {filename}")
    
    def save_to_csv(self, data: List[Dict], filename: str):
        """Save data to CSV file"""
        if not data:
            logger.warning("No data to save")
            return
        
        with open(filename, 'w', newline='', encoding='utf-8') as f:
            writer = csv.DictWriter(f, fieldnames=data[0].keys())
            writer.writeheader()
            writer.writerows(data)
        logger.info(f"Saved {len(data)} items to {filename}")


def scrape_example():
    """Example usage"""
    scraper = HTMLScraper('https://example.com', delay=1.0)
    
    # Fetch page
    soup = scraper.fetch_page('https://example.com/page')
    if not soup:
        return
    
    # Extract specific elements
    titles = scraper.extract_by_selector(soup, 'h2.title')
    links = scraper.extract_by_selector(soup, 'a.link', attribute='href')
    
    # Extract table
    table_data = scraper.extract_table(soup, 'table.data-table')
    
    # Save results
    scraper.save_to_json(table_data, 'output.json')
    scraper.save_to_csv(table_data, 'output.csv')


if __name__ == '__main__':
    scrape_example()
```

### Dynamic Content (JavaScript)
```
#!/usr/bin/env python3
"""
Selenium Scraper for dynamic JavaScript-rendered websites
Handles single-page applications and AJAX-loaded content
"""

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException, NoSuchElementException
from selenium.webdriver.chrome.options import Options
import json
import csv
import time
import logging
from typing import List, Dict, Any, Optional

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


class SeleniumScraper:
    """Scraper for JavaScript-rendered websites"""
    
    def __init__(self, headless: bool = True, delay: float = 1.0):
        """
        Initialize Selenium scraper
        
        Args:
            headless: Run browser in headless mode
            delay: Default delay between actions
        """
        self.delay = delay
        self.driver = self._setup_driver(headless)
    
    def _setup_driver(self, headless: bool) -> webdriver.Chrome:
        """Setup Chrome driver with options"""
        options = Options()
        if headless:
            options.add_argument('--headless')
        options.add_argument('--no-sandbox')
        options.add_argument('--disable-dev-shm-usage')
        options.add_argument('--disable-blink-features=AutomationControlled')
        options.add_argument('user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36')
        
        driver = webdriver.Chrome(options=options)
        driver.implicitly_wait(10)
        return driver
    
    def navigate_to(self, url: str):
        """Navigate to URL"""
        logger.info(f"Navigating to {url}")
        self.driver.get(url)
        time.sleep(self.delay)
    
    def wait_for_element(self, selector: str, by: By = By.CSS_SELECTOR, 
                        timeout: int = 10) -> Optional[Any]:
        """
        Wait for element to be present
        
        Args:
            selector: Element selector
            by: Selector type (CSS_SELECTOR, XPATH, etc.)
            timeout: Wait timeout in seconds
            
        Returns:
            WebElement or None if timeout
        """
        try:
            element = WebDriverWait(self.driver, timeout).until(
                EC.presence_of_element_located((by, selector))
            )
            return element
        except TimeoutException:
            logger.warning(f"Element not found: {selector}")
            return None
    
    def scroll_to_bottom(self, pause_time: float = 1.0, max_scrolls: int = 10):
        """
        Scroll to bottom of page (for infinite scroll)
        
        Args:
            pause_time: Pause between scrolls
            max_scrolls: Maximum number of scrolls
        """
        last_height = self.driver.execute_script("return document.body.scrollHeight")
        scrolls = 0
        
        while scrolls < max_scrolls:
            self.driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
            time.sleep(pause_time)
            
            new_height = self.driver.execute_script("return document.body.scrollHeight")
            if new_height == last_height:
                break
            
            last_height = new_height
            scrolls += 1
            logger.info(f"Scrolled {scrolls} times")
    
    def click_load_more(self, button_selector: str, max_clicks: int = 10):
        """
        Click 'Load More' button repeatedly
        
        Args:
            button_selector: CSS selector for load more button
            max_clicks: Maximum number of clicks
        """
        clicks = 0
        while clicks < max_clicks:
            try:
                button = self.driver.find_element(By.CSS_SELECTOR, button_selector)
                if not button.is_displayed():
                    break
                
                button.click()
                time.sleep(self.delay)
                clicks += 1
                logger.info(f"Clicked load more {clicks} times")
            except NoSuchElementException:
                logger.info("Load more button not found")
                break
    
    def extract_elements(self, selector: str, attribute: Optional[str] = None) -> List[str]:
        """
        Extract data from multiple elements
        
        Args:
            selector: CSS selector
            attribute: HTML attribute to extract (None for text)
            
        Returns:
            List of extracted values
        """
        elements = self.driver.find_elements(By.CSS_SELECTOR, selector)
        if attribute:
            return [elem.get_attribute(attribute) for elem in elements]
        return [elem.text for elem in elements]
    
    def extract_table(self, table_selector: str = 'table') -> List[Dict]:
        """
        Extract table data
        
        Args:
            table_selector: CSS selector for table
            
        Returns:
            List of dictionaries representing rows
        """
        try:
            table = self.driver.find_element(By.CSS_SELECTOR, table_selector)
        except NoSuchElementException:
            logger.warning(f"Table not found: {table_selector}")
            return []
        
        headers = [th.text for th in table.find_elements(By.CSS_SELECTOR, 'thead th')]
        if not headers:
            headers = [th.text for th in table.find_elements(By.CSS_SELECTOR, 'tr th')]
        
        rows = []
        for tr in table.find_elements(By.CSS_SELECTOR, 'tbody tr, tr'):
            cells = tr.find_elements(By.CSS_SELECTOR, 'td')
            if cells:
                row_data = {headers[i]: cell.text 
                           for i, cell in enumerate(cells) if i < len(headers)}
                rows.append(row_data)
        
        return rows
    
    def login(self, username: str, password: str, 
              username_selector: str, password_selector: str, 
              submit_selector: str):
        """
        Perform login
        
        Args:
            username: Username
            password: Password
            username_selector: CSS selector for username field
            password_selector: CSS selector for password field
            submit_selector: CSS selector for submit button
        """
        username_field = self.wait_for_element(username_selector)
        password_field = self.wait_for_element(password_selector)
        submit_button = self.wait_for_element(submit_selector)
        
        if username_field and password_field and submit_button:
            username_field.send_keys(username)
            password_field.send_keys(password)
            submit_button.click()
            time.sleep(2)
            logger.info("Login attempted")
    
    def execute_javascript(self, script: str) -> Any:
        """Execute JavaScript code"""
        return self.driver.execute_script(script)
    
    def save_to_json(self, data: List[Dict], filename: str):
        """Save data to JSON file"""
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(data, f, indent=2, ensure_ascii=False)
        logger.info(f"Saved {len(data)} items to {filename}")
    
    def save_to_csv(self, data: List[Dict], filename: str):
        """Save data to CSV file"""
        if not data:
            logger.warning("No data to save")
            return
        
        with open(filename, 'w', newline='', encoding='utf-8') as f:
            writer = csv.DictWriter(f, fieldnames=data[0].keys())
            writer.writeheader()
            writer.writerows(data)
        logger.info(f"Saved {len(data)} items to {filename}")
    
    def close(self):
        """Close browser"""
        self.driver.quit()
        logger.info("Browser closed")
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()


def scrape_example():
    """Example usage"""
    with SeleniumScraper(headless=True, delay=1.0) as scraper:
        # Navigate to page
        scraper.navigate_to('https://example.com')
        
        # Wait for content to load
        scraper.wait_for_element('div.content')
        
        # Handle infinite scroll
        scraper.scroll_to_bottom(pause_time=1.5)
        
        # Or handle load more button
        # scraper.click_load_more('button.load-more')
        
        # Extract data
        titles = scraper.extract_elements('h2.title')
        links = scraper.extract_elements('a.link', attribute='href')
        
        # Extract table
        table_data = scraper.extract_table('table.data')
        
        # Save results
        scraper.save_to_json(table_data, 'output.json')
        scraper.save_to_csv(table_data, 'output.csv')


if __name__ == '__main__':
    scrape_example()
```

### API Scraping
```
#!/usr/bin/env python3
"""
API Scraper for REST APIs
Handles JSON/XML responses with pagination and authentication
"""

import requests
import json
import csv
import time
import logging
from typing import List, Dict, Any, Optional, Callable
from urllib.parse import urljoin

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


class APIScraper:
    """Scraper for REST APIs"""
    
    def __init__(self, base_url: str, delay: float = 0.5):
        """
        Initialize API scraper
        
        Args:
            base_url: Base URL of the API
            delay: Delay between requests in seconds
        """
        self.base_url = base_url
        self.delay = delay
        self.session = requests.Session()
    
    def set_auth_token(self, token: str, token_type: str = 'Bearer'):
        """
        Set authentication token
        
        Args:
            token: Authentication token
            token_type: Token type (Bearer, Token, etc.)
        """
        self.session.headers.update({
            'Authorization': f'{token_type} {token}'
        })
    
    def set_api_key(self, key: str, header_name: str = 'X-API-Key'):
        """
        Set API key
        
        Args:
            key: API key
            header_name: Header name for API key
        """
        self.session.headers.update({header_name: key})
    
    def fetch_json(self, endpoint: str, params: Optional[Dict] = None, 
                   retries: int = 3) -> Optional[Dict]:
        """
        Fetch JSON data from API endpoint
        
        Args:
            endpoint: API endpoint
            params: Query parameters
            retries: Number of retry attempts
            
        Returns:
            JSON response as dictionary or None if failed
        """
        url = urljoin(self.base_url, endpoint)
        
        for attempt in range(retries):
            try:
                response = self.session.get(url, params=params, timeout=10)
                response.raise_for_status()
                time.sleep(self.delay)
                return response.json()
            except requests.RequestException as e:
                logger.warning(f"Attempt {attempt + 1} failed: {e}")
                if attempt < retries - 1:
                    time.sleep(2 ** attempt)
                else:
                    logger.error(f"Failed to fetch {url}")
                    return None
    
    def fetch_all_pages_offset(self, endpoint: str, limit: int = 100,
                               max_items: Optional[int] = None) -> List[Dict]:
        """
        Fetch all pages using offset-based pagination
        
        Args:
            endpoint: API endpoint
            limit: Items per page
            max_items: Maximum total items to fetch
            
        Returns:
            List of all items
        """
        all_items = []
        offset = 0
        
        while True:
            params = {'limit': limit, 'offset': offset}
            data = self.fetch_json(endpoint, params)
            
            if not data or 'items' not in data:
                break
            
            items = data['items']
            if not items:
                break
            
            all_items.extend(items)
            logger.info(f"Fetched {len(items)} items (total: {len(all_items)})")
            
            if max_items and len(all_items) >= max_items:
                all_items = all_items[:max_items]
                break
            
            offset += limit
            
            # Check if we've reached the end
            if len(items) < limit:
                break
        
        return all_items
    
    def fetch_all_pages_cursor(self, endpoint: str, cursor_key: str = 'next_cursor',
                               items_key: str = 'items') -> List[Dict]:
        """
        Fetch all pages using cursor-based pagination
        
        Args:
            endpoint: API endpoint
            cursor_key: Key for next cursor in response
            items_key: Key for items array in response
            
        Returns:
            List of all items
        """
        all_items = []
        cursor = None
        
        while True:
            params = {'cursor': cursor} if cursor else {}
            data = self.fetch_json(endpoint, params)
            
            if not data or items_key not in data:
                break
            
            items = data[items_key]
            if not items:
                break
            
            all_items.extend(items)
            logger.info(f"Fetched {len(items)} items (total: {len(all_items)})")
            
            cursor = data.get(cursor_key)
            if not cursor:
                break
        
        return all_items
    
    def fetch_all_pages_link_header(self, endpoint: str) -> List[Dict]:
        """
        Fetch all pages using Link header pagination (GitHub-style)
        
        Args:
            endpoint: API endpoint
            
        Returns:
            List of all items
        """
        all_items = []
        url = urljoin(self.base_url, endpoint)
        
        while url:
            try:
                response = self.session.get(url, timeout=10)
                response.raise_for_status()
                time.sleep(self.delay)
                
                data = response.json()
                if isinstance(data, list):
                    all_items.extend(data)
                elif isinstance(data, dict) and 'items' in data:
                    all_items.extend(data['items'])
                
                logger.info(f"Total items: {len(all_items)}")
                
                # Get next URL from Link header
                url = None
                if 'Link' in response.headers:
                    links = response.headers['Link'].split(',')
                    for link in links:
                        if 'rel="next"' in link:
                            url = link[link.find('<')+1:link.find('>')]
                            break
            except requests.RequestException as e:
                logger.error(f"Failed to fetch {url}: {e}")
                break
        
        return all_items
    
    def save_to_json(self, data: List[Dict], filename: str):
        """Save data to JSON file"""
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(data, f, indent=2, ensure_ascii=False)
        logger.info(f"Saved {len(data)} items to {filename}")
    
    def save_to_csv(self, data: List[Dict], filename: str):
        """Save data to CSV file"""
        if not data:
            logger.warning("No data to save")
            return
        
        # Flatten nested dictionaries if needed
        flattened = []
        for item in data:
            flat_item = self._flatten_dict(item)
            flattened.append(flat_item)
        
        with open(filename, 'w', newline='', encoding='utf-8') as f:
            writer = csv.DictWriter(f, fieldnames=flattened[0].keys())
            writer.writeheader()
            writer.writerows(flattened)
        logger.info(f"Saved {len(flattened)} items to {filename}")
    
    def _flatten_dict(self, d: Dict, parent_key: str = '', sep: str = '_') -> Dict:
        """Flatten nested dictionary"""
        items = []
        for k, v in d.items():
            new_key = f"{parent_key}{sep}{k}" if parent_key else k
            if isinstance(v, dict):
                items.extend(self._flatten_dict(v, new_key, sep=sep).items())
            elif isinstance(v, list) and v and isinstance(v[0], dict):
                items.append((new_key, json.dumps(v)))
            else:
                items.append((new_key, v))
        return dict(items)


def scrape_example():
    """Example usage"""
    scraper = APIScraper('https://api.example.com/v1', delay=0.5)
    
    # Set authentication
    scraper.set_auth_token('your_token_here')
    # Or: scraper.set_api_key('your_api_key_here')
    
    # Fetch single endpoint
    data = scraper.fetch_json('/users/123')
    
    # Fetch with pagination
    all_items = scraper.fetch_all_pages_offset('/items', limit=50)
    
    # Save results
    scraper.save_to_json(all_items, 'output.json')
    scraper.save_to_csv(all_items, 'output.csv')


if __name__ == '__main__':
    scrape_example()
```

### Pagination Handling
### Common Pagination Types

### 1. Offset/Limit Pagination

**Pattern:** `?offset=0&limit=20`, `?offset=20&limit=20`

**Characteristics:**
- Most common in APIs
- Uses offset/skip and limit/take parameters
- Predictable and easy to implement

**Example URLs:**
```
https://api.example.com/items?offset=0&limit=50
https://api.example.com/items?offset=50&limit=50
https://api.example.com/items?offset=100&limit=50
```

**Implementation:**
```python
def scrape_offset_pagination(base_url, limit=50, max_items=None):
    all_items = []
    offset = 0
    
    while True:
        url = f"{base_url}?offset={offset}&limit={limit}"
        response = requests.get(url)
        data = response.json()
        
        items = data.get('items', [])
        if not items:
            break
            
        all_items.extend(items)
        
        if max_items and len(all_items) >= max_items:
            break
            
        if len(items) < limit:  # Last page
            break
            
        offset += limit
    
    return all_items
```

### 2. Page Number Pagination

**Pattern:** `?page=1`, `?page=2`, `?page=3`

**Characteristics:**
- Common in web interfaces
- Uses page numbers
- Often combined with page size

**Example URLs:**
```
https://example.com/products?page=1
https://example.com/products?page=2&size=25
https://example.com/results?p=3&per_page=100
```

**Implementation:**
```python
def scrape_page_pagination(base_url, per_page=25):
    all_items = []
    page = 1
    
    while True:
        url = f"{base_url}?page={page}&size={per_page}"
        response = requests.get(url)
        data = response.json()
        
        items = data.get('results', [])
        if not items:
            break
            
        all_items.extend(items)
        
        # Check if last page
        total_pages = data.get('total_pages')
        if total_pages and page >= total_pages:
            break
            
        page += 1
    
    return all_items
```

### 3. Cursor-Based Pagination

**Pattern:** Uses cursor/token from previous response

**Characteristics:**
- Common in social media APIs (Twitter, Facebook)
- More reliable for large datasets
- Prevents duplicate/missing items during pagination

**Example Response:**
```json
{
  "items": [...],
  "next_cursor": "eyJpZCI6MTAwfQ==",
  "has_more": true
}
```

**Implementation:**
```python
def scrape_cursor_pagination(base_url):
    all_items = []
    cursor = None
    
    while True:
        params = {'cursor': cursor} if cursor else {}
        response = requests.get(base_url, params=params)
        data = response.json()
        
        items = data.get('items', [])
        all_items.extend(items)
        
        cursor = data.get('next_cursor')
        has_more = data.get('has_more', False)
        
        if not cursor or not has_more:
            break
    
    return all_items
```

### 4. Link Header Pagination (RFC 5988)

**Pattern:** Pagination links in HTTP headers

**Characteristics:**
- Used by GitHub, GitLab APIs
- Links in response headers
- RESTful and self-documenting

**Example Header:**
```
Link: <https://api.example.com/items?page=2>; rel="next",
      <https://api.example.com/items?page=5>; rel="last"
```

**Implementation:**
```python
import requests

def scrape_link_header_pagination(url):
    all_items = []
    
    while url:
        response = requests.get(url)
        data = response.json()
        
        all_items.extend(data)
        
        # Parse Link header
        url = None
        if 'Link' in response.headers:
            links = response.headers['Link'].split(',')
            for link in links:
                if 'rel="next"' in link:
                    url = link[link.find('<')+1:link.find('>')]
                    break
    
    return all_items
```

### 5. Infinite Scroll (AJAX)

**Pattern:** Content loads on scroll

**Characteristics:**
- Common in modern SPAs
- Requires JavaScript execution
- Triggered by scroll or button click

**HTML Pattern:**
```html
<div class="container">
  <div class="item">...</div>
  <div class="item">...</div>
  <!-- More items loaded dynamically -->
</div>
<button class="load-more">Load More</button>
```

**Selenium Implementation:**
```python
from selenium.webdriver.common.by import By

def scrape_infinite_scroll(driver, item_selector):
    last_height = driver.execute_script("return document.body.scrollHeight")
    all_items = []
    
    while True:
        # Get current items
        items = driver.find_elements(By.CSS_SELECTOR, item_selector)
        
        # Scroll to bottom
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        time.sleep(2)  # Wait for loading
        
        # Check if new content loaded
        new_height = driver.execute_script("return document.body.scrollHeight")
        if new_height == last_height:
            break
        last_height = new_height
    
    # Extract final set of items
    return [item.text for item in items]
```

**Load More Button:**
```python
def scrape_load_more_button(driver, button_selector, item_selector):
    all_items = []
    
    while True:
        try:
            button = driver.find_element(By.CSS_SELECTOR, button_selector)
            if not button.is_displayed():
                break
            button.click()
            time.sleep(2)
        except:
            break
    
    items = driver.find_elements(By.CSS_SELECTOR, item_selector)
    return [item.text for item in items]
```

### 6. Next/Previous Links

**Pattern:** HTML links to next page

**Characteristics:**
- Traditional web pagination
- Links in page content
- Must parse HTML to find next page

**HTML Pattern:**
```html
<div class="pagination">
  <a href="?page=1">Previous</a>
  <a href="?page=3" class="next">Next</a>
</div>
```

**Implementation:**
```python
from bs4 import BeautifulSoup

def scrape_next_link_pagination(start_url, next_selector='a.next'):
    all_items = []
    url = start_url
    
    while url:
        response = requests.get(url)
        soup = BeautifulSoup(response.content, 'html.parser')
        
        # Extract items from current page
        items = soup.select('div.item')
        all_items.extend(items)
        
        # Find next page link
        next_link = soup.select_one(next_selector)
        if next_link and next_link.get('href'):
            url = urljoin(url, next_link['href'])
        else:
            url = None
    
    return all_items
```

## Pagination Detection

**Identify pagination type:**

1. **Check API documentation** first
2. **Inspect network requests** in DevTools
3. **Look for patterns** in URLs
4. **Check response structure** for pagination metadata
5. **Examine HTML** for pagination controls

## Best Practices

1. **Respect rate limits** - Add delays between requests
2. **Handle errors gracefully** - Implement retries
3. **Save progress** - Checkpoint for large scrapes
4. **Validate data** - Check for duplicates
5. **Log pagination state** - Track current page/cursor
6. **Set maximum limits** - Prevent infinite loops
7. **Cache responses** - Avoid re-fetching

## Error Handling

```python
import time
from requests.exceptions import RequestException

def safe_paginate(fetch_page_func, max_retries=3):
    all_items = []
    page = 1
    
    while True:
        for attempt in range(max_retries):
            try:
                items = fetch_page_func(page)
                if not items:
                    return all_items
                    
                all_items.extend(items)
                break
            except RequestException as e:
                if attempt == max_retries - 1:
                    raise
                wait_time = 2 ** attempt
                time.sleep(wait_time)
        
        page += 1
```

## Performance Optimization

**Parallel pagination** (use with caution):
```python
from concurrent.futures import ThreadPoolExecutor

def scrape_parallel_pages(base_url, total_pages, max_workers=5):
    def fetch_page(page_num):
        url = f"{base_url}?page={page_num}"
        response = requests.get(url)
        return response.json()
    
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        results = list(executor.map(fetch_page, range(1, total_pages + 1)))
    
    # Flatten results
    all_items = []
    for items in results:
        all_items.extend(items)
    
    return all_items
```

**Note:** Only use parallel requests if:
- The API/site allows it
- You respect rate limits
- Pages are independent (not cursor-based)

### Authentication
### 1. API Key Authentication

**Header-based:**
```python
import requests

headers = {
    'X-API-Key': 'your_api_key_here',
    'User-Agent': 'Mozilla/5.0'
}

response = requests.get('https://api.example.com/data', headers=headers)
```

**Query parameter:**
```python
params = {
    'api_key': 'your_api_key_here',
    'limit': 100
}

response = requests.get('https://api.example.com/data', params=params)
```

### 2. Bearer Token Authentication

**JWT/OAuth tokens:**
```python
headers = {
    'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIs...',
    'Content-Type': 'application/json'
}

response = requests.get('https://api.example.com/data', headers=headers)
```

**Token refresh:**
```python
class TokenAuthSession:
    def __init__(self, token_url, client_id, client_secret):
        self.token_url = token_url
        self.client_id = client_id
        self.client_secret = client_secret
        self.access_token = None
        self.token_expires = 0
    
    def get_token(self):
        import time
        
        if self.access_token and time.time() < self.token_expires:
            return self.access_token
        
        data = {
            'grant_type': 'client_credentials',
            'client_id': self.client_id,
            'client_secret': self.client_secret
        }
        
        response = requests.post(self.token_url, data=data)
        token_data = response.json()
        
        self.access_token = token_data['access_token']
        self.token_expires = time.time() + token_data.get('expires_in', 3600)
        
        return self.access_token
    
    def get(self, url, **kwargs):
        headers = kwargs.get('headers', {})
        headers['Authorization'] = f'Bearer {self.get_token()}'
        kwargs['headers'] = headers
        return requests.get(url, **kwargs)
```

### 3. Basic Authentication

```python
from requests.auth import HTTPBasicAuth

response = requests.get(
    'https://api.example.com/data',
    auth=HTTPBasicAuth('username', 'password')
)

# Or using tuple
response = requests.get(
    'https://api.example.com/data',
    auth=('username', 'password')
)
```

### 4. OAuth 2.0

**Authorization Code Flow:**
```python
from requests_oauthlib import OAuth2Session

client_id = 'your_client_id'
client_secret = 'your_client_secret'
redirect_uri = 'http://localhost:8000/callback'
authorization_base_url = 'https://example.com/oauth/authorize'
token_url = 'https://example.com/oauth/token'

# Authorization
oauth = OAuth2Session(client_id, redirect_uri=redirect_uri)
authorization_url, state = oauth.authorization_url(authorization_base_url)

print(f'Visit: {authorization_url}')
redirect_response = input('Paste redirect URL: ')

# Token exchange
token = oauth.fetch_token(
    token_url,
    authorization_response=redirect_response,
    client_secret=client_secret
)

# Make requests
response = oauth.get('https://api.example.com/data')
```

## Web Login Authentication

### 1. Session-Based Login

**Simple form login:**
```python
import requests
from bs4 import BeautifulSoup

session = requests.Session()

# Get login page (to retrieve CSRF token if needed)
login_page = session.get('https://example.com/login')
soup = BeautifulSoup(login_page.content, 'html.parser')

csrf_token = soup.find('input', {'name': 'csrf_token'})['value']

# Perform login
login_data = {
    'username': 'your_username',
    'password': 'your_password',
    'csrf_token': csrf_token
}

response = session.post('https://example.com/login', data=login_data)

# Check if login successful
if 'dashboard' in response.url or response.status_code == 200:
    # Now make authenticated requests
    protected_page = session.get('https://example.com/protected-data')
```

### 2. Cookie-Based Authentication

**Manual cookie setting:**
```python
cookies = {
    'session_id': 'abc123def456',
    'user_token': 'xyz789'
}

response = requests.get('https://example.com/data', cookies=cookies)
```

**Persist cookies:**
```python
import requests
from http.cookiejar import MozillaCookieJar

# Save cookies
session = requests.Session()
session.cookies = MozillaCookieJar('cookies.txt')

# Login and save
session.post('https://example.com/login', data=login_data)
session.cookies.save(ignore_discard=True, ignore_expires=True)

# Load cookies later
new_session = requests.Session()
new_session.cookies = MozillaCookieJar('cookies.txt')
new_session.cookies.load(ignore_discard=True, ignore_expires=True)
```

### 3. Selenium-Based Login

**For JavaScript-heavy sites:**
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()

# Navigate to login page
driver.get('https://example.com/login')

# Wait for and fill login form
username_field = WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.ID, 'username'))
)
password_field = driver.find_element(By.ID, 'password')
submit_button = driver.find_element(By.CSS_SELECTOR, 'button[type="submit"]')

username_field.send_keys('your_username')
password_field.send_keys('your_password')
submit_button.click()

# Wait for login to complete
WebDriverWait(driver, 10).until(
    EC.url_contains('dashboard')
)

# Transfer cookies to requests session
selenium_cookies = driver.get_cookies()
session = requests.Session()

for cookie in selenium_cookies:
    session.cookies.set(cookie['name'], cookie['value'])

# Now use session for requests
driver.quit()
```

## Advanced Authentication

### 1. Two-Factor Authentication (2FA)

**Manual OTP entry:**
```python
def login_with_2fa(username, password, otp_callback):
    session = requests.Session()
    
    # Initial login
    session.post('https://example.com/login', data={
        'username': username,
        'password': password
    })
    
    # Get OTP from user
    otp_code = otp_callback()  # Function that gets OTP from user
    
    # Submit OTP
    session.post('https://example.com/verify-otp', data={
        'otp': otp_code
    })
    
    return session

# Usage
session = login_with_2fa('user', 'pass', lambda: input('Enter OTP: '))
```

**TOTP (Time-based One-Time Password):**
```python
import pyotp

secret = 'YOUR_SECRET_KEY'
totp = pyotp.TOTP(secret)
otp = totp.now()  # Generate current OTP

# Use in login
session.post('https://example.com/verify-otp', data={'otp': otp})
```

### 2. Headless Browser with Authentication

**Persistent login:**
```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import pickle

def save_cookies(driver, filepath):
    cookies = driver.get_cookies()
    with open(filepath, 'wb') as f:
        pickle.dump(cookies, f)

def load_cookies(driver, filepath):
    with open(filepath, 'rb') as f:
        cookies = pickle.load(f)
    for cookie in cookies:
        driver.add_cookie(cookie)

# First time login
options = Options()
options.add_argument('--headless')
driver = webdriver.Chrome(options=options)

driver.get('https://example.com/login')
# ... perform login ...

save_cookies(driver, 'session_cookies.pkl')

# Subsequent visits
driver = webdriver.Chrome(options=options)
driver.get('https://example.com')
load_cookies(driver, 'session_cookies.pkl')
driver.refresh()
```

## Authentication Best Practices

1. **Store credentials securely**
   ```python
   import os
   from dotenv import load_dotenv
   
   load_dotenv()
   API_KEY = os.getenv('API_KEY')
   USERNAME = os.getenv('USERNAME')
   PASSWORD = os.getenv('PASSWORD')
   ```

2. **Handle token expiration**
   - Implement automatic token refresh
   - Catch 401 errors and re-authenticate
   - Store token expiry times

3. **Rate limiting**
   - Respect API rate limits
   - Implement exponential backoff
   - Track request counts

4. **Session management**
   - Reuse sessions when possible
   - Close sessions properly
   - Handle session timeouts

5. **Security considerations**
   - Use HTTPS only
   - Don't log credentials
   - Rotate API keys regularly
   - Use environment variables

## Error Handling

```python
class AuthenticationError(Exception):
    pass

def authenticated_request(session, url, max_retries=3):
    for attempt in range(max_retries):
        response = session.get(url)
        
        if response.status_code == 401:
            # Re-authenticate
            if not re_authenticate(session):
                raise AuthenticationError("Failed to re-authenticate")
            continue
        
        if response.status_code == 429:
            # Rate limited
            retry_after = int(response.headers.get('Retry-After', 60))
            time.sleep(retry_after)
            continue
        
        response.raise_for_status()
        return response
    
    raise AuthenticationError("Max retries exceeded")

def re_authenticate(session):
    # Re-login logic
    response = session.post('https://example.com/login', data={
        'username': USERNAME,
        'password': PASSWORD
    })
    return response.status_code == 200
```

## Common Authentication Patterns by Platform

**GitHub:**
```python
headers = {'Authorization': f'token {GITHUB_TOKEN}'}
```

**Twitter/X:**
```python
headers = {'Authorization': f'Bearer {BEARER_TOKEN}'}
```

**Reddit:**
```python
auth = HTTPBasicAuth(CLIENT_ID, CLIENT_SECRET)
# Then get OAuth token
```

**Google APIs:**
```python
from google.oauth2 import service_account
credentials = service_account.Credentials.from_service_account_file('key.json')
```

**AWS:**
```python
import boto3
session = boto3.Session(
    aws_access_key_id=ACCESS_KEY,
    aws_secret_access_key=SECRET_KEY
)
```

## Best Practices

- Always set proper User-Agent headers
- Respect robots.txt and rate limits
- Implement exponential backoff for retries
- Cache responses when appropriate
- Handle errors gracefully
- Log scraping activities
- Validate extracted data before saving

## Legal and Ethical Guidelines

- Review website's Terms of Service
- Respect robots.txt directives
- Implement reasonable rate limiting
- Don't overload servers
- Attribute data sources when publishing
- Obtain permission for commercial use