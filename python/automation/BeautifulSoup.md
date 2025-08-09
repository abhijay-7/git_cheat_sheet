# BeautifulSoup In-Depth Cheatsheet

## Installation and Setup

```bash
# Install BeautifulSoup
pip install beautifulsoup4

# Install with parsers
pip install beautifulsoup4 lxml html5lib requests

# Install additional utilities
pip install beautifulsoup4 requests selenium fake-useragent
```

## Basic Usage

### Creating BeautifulSoup Objects
```python
from bs4 import BeautifulSoup
import requests
import lxml
from urllib.request import urlopen

# From HTML string
html_content = "<html><body><h1>Hello World</h1></body></html>"
soup = BeautifulSoup(html_content, 'html.parser')

# From file
with open('file.html', 'r', encoding='utf-8') as file:
    soup = BeautifulSoup(file, 'html.parser')

# From URL using requests
response = requests.get('https://example.com')
soup = BeautifulSoup(response.content, 'html.parser')

# From URL using urllib
html = urlopen('https://example.com')
soup = BeautifulSoup(html, 'html.parser')

# Different parsers
soup = BeautifulSoup(html_content, 'html.parser')    # Built-in HTML parser
soup = BeautifulSoup(html_content, 'lxml')           # Fast XML/HTML parser
soup = BeautifulSoup(html_content, 'html5lib')       # Most lenient parser
soup = BeautifulSoup(xml_content, 'xml')             # XML parser
```

### Parser Comparison
```python
# html.parser (built-in)
# - Decent speed, lenient
# - No external dependencies
# - Not as fast as lxml

# lxml
# - Very fast
# - Lenient
# - External C dependency

# html5lib
# - Extremely slow
# - Creates valid HTML5
# - No external dependencies beyond Python
```

## Finding Elements

### Basic Element Selection
```python
# Find by tag name
title = soup.find('title')
titles = soup.find_all('title')

# Find by attributes
link = soup.find('a', href='https://example.com')
links = soup.find_all('a', href=True)  # All links with href attribute

# Find by class (class is reserved keyword, use class_)
content = soup.find('div', class_='content')
contents = soup.find_all('div', class_='content')

# Find by multiple classes
item = soup.find('div', class_=['item', 'featured'])

# Find by ID
header = soup.find('div', id='header')

# Find by multiple attributes
form = soup.find('form', {'method': 'post', 'action': '/submit'})
```

### Advanced Selection Methods
```python
# Using CSS selectors
soup.select('title')                    # Tag selector
soup.select('.content')                 # Class selector
soup.select('#header')                  # ID selector
soup.select('div.content')              # Tag + class
soup.select('div > p')                  # Direct child
soup.select('div p')                    # Descendant
soup.select('a[href]')                  # Attribute exists
soup.select('a[href="https://example.com"]')  # Attribute value
soup.select('p:nth-of-type(2)')         # Nth element
soup.select('tr:nth-child(odd)')        # Odd children

# Complex CSS selectors
soup.select('div.article p:first-child')
soup.select('table tr:not(.header)')
soup.select('a[href^="https://"]')      # Starts with
soup.select('a[href$=".pdf"]')          # Ends with
soup.select('a[href*="download"]')      # Contains

# Lambda functions for complex conditions
soup.find_all(lambda tag: tag.name == 'a' and len(tag.get('href', '')) > 10)
soup.find_all(lambda tag: tag.has_attr('class') and 'highlight' in tag['class'])

# Regular expressions
import re
soup.find_all('a', href=re.compile(r'^https://'))
soup.find_all('img', src=re.compile(r'\.(jpg|png|gif)$', re.I))
soup.find_all(string=re.compile(r'\d{3}-\d{3}-\d{4}'))  # Phone numbers
```

### Navigation Methods
```python
# Parent/child navigation
tag.parent                 # Direct parent
tag.parents               # All parents (generator)
tag.children              # Direct children (generator)
tag.descendants           # All descendants (generator)

# Sibling navigation
tag.next_sibling          # Next sibling
tag.previous_sibling      # Previous sibling
tag.next_siblings         # All next siblings (generator)
tag.previous_siblings     # All previous siblings (generator)

# Element navigation (skipping text nodes)
tag.next_element          # Next element
tag.previous_element      # Previous element
tag.next_elements         # All next elements (generator)
tag.previous_elements     # All previous elements (generator)

# Examples
first_paragraph = soup.find('p')
second_paragraph = first_paragraph.find_next_sibling('p')
parent_div = first_paragraph.find_parent('div')
all_links_after = first_paragraph.find_all_next('a')
```

## Extracting Data

### Text Extraction
```python
# Get text content
tag.string              # Direct text content (None if multiple children)
tag.get_text()          # All text content
tag.text               # Alias for get_text()

# Text extraction options
tag.get_text(strip=True)              # Strip whitespace
tag.get_text(separator=' ')           # Custom separator
tag.get_text(separator='|', strip=True)  # Combined options

# Extract specific text patterns
import re
phone_pattern = re.compile(r'\(\d{3}\) \d{3}-\d{4}')
phones = soup.find_all(string=phone_pattern)

# Clean text extraction
def clean_text(text):
    """Clean extracted text"""
    if text:
        # Remove extra whitespace
        text = re.sub(r'\s+', ' ', text.strip())
        # Remove special characters
        text = re.sub(r'[^\w\s-.]', '', text)
        return text
    return ''

clean_content = clean_text(tag.get_text())
```

### Attribute Extraction
```python
# Get attribute values
link_url = tag.get('href')
link_url = tag['href']              # KeyError if not exists
link_url = tag.attrs['href']        # Access attrs dict directly

# Get all attributes
all_attrs = tag.attrs

# Check if attribute exists
has_href = tag.has_attr('href')
has_href = 'href' in tag.attrs

# Get attribute with default
link_url = tag.get('href', 'default_url')

# Multiple attributes
img_src = tag.get('src')
img_alt = tag.get('alt', 'No description')
img_width = tag.get('width', 0)
```

### Complex Data Extraction
```python
def extract_product_info(soup):
    """Extract comprehensive product information"""
    products = []
    
    for product_div in soup.find_all('div', class_='product'):
        product = {}
        
        # Basic info
        name_tag = product_div.find('h3', class_='product-name')
        product['name'] = name_tag.get_text(strip=True) if name_tag else None
        
        # Price extraction with cleaning
        price_tag = product_div.find('span', class_='price')
        if price_tag:
            price_text = price_tag.get_text(strip=True)
            price_match = re.search(r'[\d,]+\.?\d*', price_text.replace(',', ''))
            product['price'] = float(price_match.group()) if price_match else None
        
        # Image URL
        img_tag = product_div.find('img')
        product['image_url'] = img_tag.get('src') if img_tag else None
        
        # Rating
        rating_div = product_div.find('div', class_='rating')
        if rating_div:
            stars = rating_div.find_all('span', class_='star filled')
            product['rating'] = len(stars)
        
        # Availability
        availability_span = product_div.find('span', class_='availability')
        if availability_span:
            availability_text = availability_span.get_text(strip=True).lower()
            product['in_stock'] = 'in stock' in availability_text
        
        # Features list
        features_ul = product_div.find('ul', class_='features')
        if features_ul:
            features = [li.get_text(strip=True) for li in features_ul.find_all('li')]
            product['features'] = features
        
        # Specifications table
        specs_table = product_div.find('table', class_='specifications')
        if specs_table:
            specs = {}
            for row in specs_table.find_all('tr'):
                cells = row.find_all(['td', 'th'])
                if len(cells) == 2:
                    key = cells[0].get_text(strip=True)
                    value = cells[1].get_text(strip=True)
                    specs[key] = value
            product['specifications'] = specs
        
        products.append(product)
    
    return products

# Usage
products = extract_product_info(soup)
```

## Web Scraping with Requests

### Basic Web Scraping Setup
```python
import requests
from bs4 import BeautifulSoup
import time
import random
from urllib.parse import urljoin, urlparse
import json

class WebScraper:
    def __init__(self, base_delay=1, max_delay=3):
        self.session = requests.Session()
        self.base_delay = base_delay
        self.max_delay = max_delay
        
        # Set default headers
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36',
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
            'Accept-Language': 'en-US,en;q=0.5',
            'Accept-Encoding': 'gzip, deflate',
            'Connection': 'keep-alive',
            'Upgrade-Insecure-Requests': '1',
        })
    
    def get_page(self, url, **kwargs):
        """Get page with error handling and delays"""
        try:
            # Random delay to avoid being blocked
            delay = random.uniform(self.base_delay, self.max_delay)
            time.sleep(delay)
            
            response = self.session.get(url, **kwargs)
            response.raise_for_status()
            return response
        
        except requests.exceptions.RequestException as e:
            print(f"Error fetching {url}: {e}")
            return None
    
    def parse_page(self, url, parser='html.parser'):
        """Get page and return BeautifulSoup object"""
        response = self.get_page(url)
        if response:
            return BeautifulSoup(response.content, parser)
        return None
    
    def scrape_with_retry(self, url, max_retries=3, backoff_factor=2):
        """Scrape with exponential backoff retry"""
        for attempt in range(max_retries):
            try:
                response = self.session.get(url, timeout=30)
                response.raise_for_status()
                return BeautifulSoup(response.content, 'html.parser')
            
            except requests.exceptions.RequestException as e:
                if attempt == max_retries - 1:
                    print(f"Failed to scrape {url} after {max_retries} attempts")
                    return None
                
                wait_time = backoff_factor ** attempt
                print(f"Attempt {attempt + 1} failed, retrying in {wait_time} seconds...")
                time.sleep(wait_time)

# Usage
scraper = WebScraper()
soup = scraper.parse_page('https://example.com')
```

### Advanced Scraping Techniques
```python
class AdvancedScraper:
    def __init__(self):
        self.session = requests.Session()
        self.setup_session()
    
    def setup_session(self):
        """Configure session with proper settings"""
        # User agents rotation
        user_agents = [
            'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
            'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36',
            'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36',
        ]
        
        self.session.headers.update({
            'User-Agent': random.choice(user_agents),
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
            'Accept-Language': 'en-US,en;q=0.9',
            'Accept-Encoding': 'gzip, deflate, br',
            'DNT': '1',
            'Connection': 'keep-alive',
            'Upgrade-Insecure-Requests': '1',
        })
        
        # Configure connection pooling
        adapter = requests.adapters.HTTPAdapter(
            pool_connections=10,
            pool_maxsize=20,
            max_retries=3
        )
        self.session.mount('http://', adapter)
        self.session.mount('https://', adapter)
    
    def handle_javascript_content(self, url):
        """Handle pages that require JavaScript (basic approach)"""
        # This is a simplified approach - for full JS handling use Selenium
        response = self.session.get(url)
        soup = BeautifulSoup(response.content, 'html.parser')
        
        # Look for data in script tags (common pattern)
        script_tags = soup.find_all('script', type='application/json')
        data = {}
        
        for script in script_tags:
            try:
                json_data = json.loads(script.string)
                data.update(json_data)
            except (json.JSONDecodeError, AttributeError):
                continue
        
        return soup, data
    
    def scrape_with_form_handling(self, login_url, form_data, target_url):
        """Handle form submission and authenticated scraping"""
        # Get login page
        login_response = self.session.get(login_url)
        login_soup = BeautifulSoup(login_response.content, 'html.parser')
        
        # Extract form data (CSRF tokens, etc.)
        form = login_soup.find('form')
        if form:
            # Get all hidden inputs
            hidden_inputs = form.find_all('input', type='hidden')
            for hidden_input in hidden_inputs:
                name = hidden_input.get('name')
                value = hidden_input.get('value')
                if name and value:
                    form_data[name] = value
        
        # Submit form
        form_action = form.get('action', login_url)
        form_url = urljoin(login_url, form_action)
        
        login_response = self.session.post(form_url, data=form_data)
        
        # Now access protected content
        protected_response = self.session.get(target_url)
        return BeautifulSoup(protected_response.content, 'html.parser')
    
    def scrape_paginated_content(self, start_url, next_link_selector):
        """Scrape all pages in paginated content"""
        all_data = []
        current_url = start_url
        
        while current_url:
            print(f"Scraping: {current_url}")
            soup = self.parse_page(current_url)
            
            if not soup:
                break
            
            # Extract data from current page
            page_data = self.extract_page_data(soup)
            all_data.extend(page_data)
            
            # Find next page link
            next_link = soup.select_one(next_link_selector)
            if next_link and next_link.get('href'):
                current_url = urljoin(current_url, next_link['href'])
            else:
                current_url = None
            
            # Polite delay
            time.sleep(random.uniform(1, 3))
        
        return all_data
    
    def parse_page(self, url):
        """Parse page with error handling"""
        try:
            response = self.session.get(url, timeout=30)
            response.raise_for_status()
            return BeautifulSoup(response.content, 'html.parser')
        except Exception as e:
            print(f"Error parsing {url}: {e}")
            return None
    
    def extract_page_data(self, soup):
        """Override this method for specific data extraction"""
        # Example implementation
        items = []
        for item_div in soup.find_all('div', class_='item'):
            item = {
                'title': item_div.find('h3').get_text(strip=True) if item_div.find('h3') else None,
                'description': item_div.find('p').get_text(strip=True) if item_div.find('p') else None,
            }
            items.append(item)
        return items

# Usage
scraper = AdvancedScraper()
data = scraper.scrape_paginated_content(
    'https://example.com/page/1',
    'a.next-page'
)
```

## Data Cleaning and Processing

### Text Processing Functions
```python
import re
from datetime import datetime
from urllib.parse import urljoin

def clean_text(text):
    """Comprehensive text cleaning"""
    if not text:
        return ''
    
    # Remove HTML entities
    import html
    text = html.unescape(text)
    
    # Remove extra whitespace
    text = re.sub(r'\s+', ' ', text.strip())
    
    # Remove special characters (optional)
    # text = re.sub(r'[^\w\s.-]', '', text)
    
    return text

def extract_numbers(text):
    """Extract numbers from text"""
    if not text:
        return []
    
    # Find all numbers (including decimals)
    numbers = re.findall(r'\d+\.?\d*', text.replace(',', ''))
    return [float(num) for num in numbers]

def extract_price(text):
    """Extract price from text"""
    if not text:
        return None
    
    # Remove currency symbols and extract number
    price_match = re.search(r'[\d,]+\.?\d*', text.replace(',', ''))
    if price_match:
        return float(price_match.group())
    return None

def parse_date(date_string):
    """Parse various date formats"""
    if not date_string:
        return None
    
    date_formats = [
        '%Y-%m-%d',
        '%m/%d/%Y',
        '%d/%m/%Y',
        '%B %d, %Y',
        '%b %d, %Y',
        '%Y-%m-%d %H:%M:%S',
    ]
    
    for fmt in date_formats:
        try:
            return datetime.strptime(date_string.strip(), fmt)
        except ValueError:
            continue
    
    return None

def extract_urls(soup, base_url):
    """Extract and normalize URLs"""
    urls = []
    
    for link in soup.find_all('a', href=True):
        href = link['href']
        # Convert relative URLs to absolute
        absolute_url = urljoin(base_url, href)
        urls.append(absolute_url)
    
    return list(set(urls))  # Remove duplicates

def extract_emails(text):
    """Extract email addresses from text"""
    if not text:
        return []
    
    email_pattern = r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'
    return re.findall(email_pattern, text)

def extract_phone_numbers(text):
    """Extract phone numbers from text"""
    if not text:
        return []
    
    phone_patterns = [
        r'\(\d{3}\)\s*\d{3}-\d{4}',  # (123) 456-7890
        r'\d{3}-\d{3}-\d{4}',        # 123-456-7890
        r'\d{3}\.\d{3}\.\d{4}',      # 123.456.7890
        r'\d{10}',                   # 1234567890
    ]
    
    phones = []
    for pattern in phone_patterns:
        phones.extend(re.findall(pattern, text))
    
    return phones
```

### Data Validation and Cleaning
```python
class DataValidator:
    @staticmethod
    def validate_email(email):
        """Validate email format"""
        if not email:
            return False
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return bool(re.match(pattern, email))
    
    @staticmethod
    def validate_url(url):
        """Validate URL format"""
        if not url:
            return False
        try:
            result = urlparse(url)
            return all([result.scheme, result.netloc])
        except:
            return False
    
    @staticmethod
    def validate_phone(phone):
        """Validate phone number"""
        if not phone:
            return False
        # Remove non-digits
        digits = re.sub(r'\D', '', phone)
        return len(digits) >= 10
    
    @staticmethod
    def clean_currency(price_text):
        """Clean and extract currency amount"""
        if not price_text:
            return None, None
        
        # Extract currency symbol
        currency_match = re.search(r'[$€£¥₹]', price_text)
        currency = currency_match.group() if currency_match else None
        
        # Extract numeric value
        numeric_match = re.search(r'[\d,]+\.?\d*', price_text.replace(',', ''))
        amount = float(numeric_match.group()) if numeric_match else None
        
        return amount, currency

class DataProcessor:
    def __init__(self):
        self.validator = DataValidator()
    
    def process_product_data(self, raw_data):
        """Process and validate product data"""
        processed = {}
        
        # Clean name
        processed['name'] = clean_text(raw_data.get('name', ''))
        
        # Process price
        price_text = raw_data.get('price', '')
        amount, currency = self.validator.clean_currency(price_text)
        processed['price'] = amount
        processed['currency'] = currency
        
        # Validate and clean URL
        url = raw_data.get('url', '')
        if self.validator.validate_url(url):
            processed['url'] = url
        
        # Process description
        processed['description'] = clean_text(raw_data.get('description', ''))
        
        # Extract and validate images
        images = raw_data.get('images', [])
        valid_images = [img for img in images if self.validator.validate_url(img)]
        processed['images'] = valid_images
        
        # Process ratings
        rating_text = raw_data.get('rating', '')
        if rating_text:
            rating_numbers = extract_numbers(rating_text)
            processed['rating'] = rating_numbers[0] if rating_numbers else None
        
        return processed

# Usage
processor = DataProcessor()
clean_data = processor.process_product_data(raw_scraped_data)
```

## Advanced BeautifulSoup Techniques

### Working with XML
```python
# Parse XML content
xml_content = '''
<catalog>
    <book id="1">
        <title>Python Programming</title>
        <author>John Doe</author>
        <price currency="USD">29.99</price>
    </book>
    <book id="2">
        <title>Web Scraping</title>
        <author>Jane Smith</author>
        <price currency="EUR">24.99</price>
    </book>
</catalog>
'''

soup = BeautifulSoup(xml_content, 'xml')

# Extract data from XML
books = []
for book in soup.find_all('book'):
    book_data = {
        'id': book.get('id'),
        'title': book.find('title').text,
        'author': book.find('author').text,
        'price': float(book.find('price').text),
        'currency': book.find('price').get('currency')
    }
    books.append(book_data)
```

### Handling Namespaces
```python
# XML with namespaces
xml_with_ns = '''
<root xmlns:books="http://example.com/books">
    <books:book>
        <books:title>Sample Book</books:title>
    </books:book>
</root>
'''

soup = BeautifulSoup(xml_with_ns, 'xml')

# Find elements with namespaces
title = soup.find('title')  # Works without namespace prefix
book = soup.find('book')    # BeautifulSoup handles namespaces automatically
```

### Custom Parsers and Filters
```python
def custom_filter(tag):
    """Custom filter function"""
    return (tag.name == 'div' and 
            tag.has_attr('class') and 
            'product' in tag.get('class', []) and
            tag.find('span', class_='price'))

# Use custom filter
products = soup.find_all(custom_filter)

# Complex attribute matching
def has_valid_price(tag):
    price_span = tag.find('span', class_='price')
    if price_span:
        price_text = price_span.get_text()
        price_match = re.search(r'\d+\.?\d*', price_text)
        return price_match and float(price_match.group()) > 0
    return False

expensive_products = soup.find_all(has_valid_price)
```

### Memory Optimization
```python
# For large documents, use SoupStrainer to parse only needed parts
from bs4 import SoupStrainer

# Only parse div tags with class 'content'
parse_only = SoupStrainer('div', class_='content')
soup = BeautifulSoup(large_html_content, 'html.parser', parse_only=parse_only)

# Parse only specific tags
parse_only = SoupStrainer(['title', 'h1', 'h2', 'p'])
soup = BeautifulSoup(html_content, 'html.parser', parse_only=parse_only)

# Parse only tags with specific attributes
parse_only = SoupStrainer(attrs={'class': 'important'})
soup = BeautifulSoup(html_content, 'html.parser', parse_only=parse_only)
```

### Performance Optimization
```python
import cProfile
import time
from memory_profiler import profile

class OptimizedScraper:
    def __init__(self):
        self.session = requests.Session()
        # Reuse session for connection pooling
        
    @profile  # Memory profiling decorator
    def scrape_efficiently(self, urls):
        """Optimized scraping method"""
        results = []
        
        for url in urls:
            try:
                # Use streaming for large files
                response = self.session.get(url, stream=True)
                
                # Process in chunks for large documents
                content = b''
                for chunk in response.iter_content(chunk_size=8192):
                    content += chunk
                    # Break if we have enough content
                    if len(content) > 1024 * 1024:  # 1MB limit
                        break
                
                # Use SoupStrainer for selective parsing
                parse_only = SoupStrainer(['div', 'span'], class_=['product', 'price'])
                soup = BeautifulSoup(content, 'lxml', parse_only=parse_only)
                
                # Extract data efficiently
                data = self.extract_data_fast(soup)
                results.append(data)
                
                # Clear variables to free memory
                del soup, content, response
                
            except Exception as e:
                print(f"Error processing {url}: {e}")
        
        return results
    
    def extract_data_fast(self, soup):
        """Fast data extraction"""
        # Use generators for memory efficiency
        products = (self.process_product(product) 
                   for product in soup.find_all('div', class_='product'))
        
        return list(products)
    
    def process_product(self, product_div):
        """Process individual product efficiently"""
        # Direct attribute access is faster than multiple find() calls
        name_tag = product_div.find('h3')
        price_tag = product_div.find('span', class_='price')
        
        return {
            'name': name_tag.string if name_tag and name_tag.string else None,
            'price': self.extract_price_fast(price_tag.string if price_tag else None)
        }
    
    def extract_price_fast(self, price_text):
        """Fast price extraction"""
        if not price_text:
            return None
        
        # Use compiled regex for better performance
        import re
        price_pattern = re.compile(r'[\d,]+\.?\d*')
        match = price_pattern.search(price_text.replace(',', ''))
        return float(match.group()) if match else None

# Performance testing
def time_scraping_methods():
    """Compare scraping performance"""
    
    def method1_basic(html):
        soup = BeautifulSoup(html, 'html.parser')
        return [div.find('span').text for div in soup.find_all('div', class_='item')]
    
    def method2_optimized(html):
        parse_only = SoupStrainer('div', class_='item')
        soup = BeautifulSoup(html, 'lxml', parse_only=parse_only)
        return [span.string for span in soup.find_all('span') if span.string]
    
    # Time both methods
    html_content = generate_large_html()  # Your HTML content
    
    start_time = time.time()
    result1 = method1_basic(html_content)
    time1 = time.time() - start_time
    
    start_time = time.time()
    result2 = method2_optimized(html_content)
    time2 = time.time() - start_time
    
    print(f"Basic method: {time1:.4f} seconds")
    print(f"Optimized method: {time2:.4f} seconds")
    print(f"Speedup: {time1/time2:.2f}x")
```

## Error Handling and Debugging

### Robust Error Handling
```python
import logging
from requests.exceptions import RequestException, Timeout, ConnectionError
from bs4.element import NavigableString

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class RobustScraper:
    def __init__(self, max_retries=3, timeout=30):
        self.max_retries = max_retries
        self.timeout = timeout
        self.session = requests.Session()
        
    def safe_get_page(self, url):
        """Safely get page with comprehensive error handling"""
        for attempt in range(self.max_retries):
            try:
                response = self.session.get(
                    url, 
                    timeout=self.timeout,
                    headers={'User-Agent': 'Mozilla/5.0 (compatible; ScrapyBot/1.0)'}
                )
                response.raise_for_status()
                return response
                
            except ConnectionError as e:
                logger.warning(f"Connection error for {url} (attempt {attempt + 1}): {e}")
                if attempt == self.max_retries - 1:
                    logger.error(f"Failed to connect to {url} after {self.max_retries} attempts")
                    return None
                    
            except Timeout as e:
                logger.warning(f"Timeout for {url} (attempt {attempt + 1}): {e}")
                if attempt == self.max_retries - 1:
                    logger.error(f"Timeout for {url} after {self.max_retries} attempts")
                    return None
                    
            except RequestException as e:
                logger.error(f"Request error for {url}: {e}")
                return None
                
            # Wait before retry
            time.sleep(2 ** attempt)  # Exponential backoff
            
        return None
    
    def safe_parse(self, html_content, parser='html.parser'):
        """Safely parse HTML content"""
        try:
            return BeautifulSoup(html_content, parser)
        except Exception as e:
            logger.error(f"Parsing error: {e}")
            return None
    
    def safe_extract_text(self, element, default=''):
        """Safely extract text from element"""
        try:
            if element is None:
                return default
            if isinstance(element, NavigableString):
                return str(element).strip()
            return element.get_text(strip=True) if element else default
        except Exception as e:
            logger.warning(f"Text extraction error: {e}")
            return default
    
    def safe_extract_attr(self, element, attr, default=None):
        """Safely extract attribute from element"""
        try:
            return element.get(attr, default) if element else default
        except Exception as e:
            logger.warning(f"Attribute extraction error: {e}")
            return default
    
    def safe_find(self, soup, *args, **kwargs):
        """Safely find element with error handling"""
        try:
            return soup.find(*args, **kwargs) if soup else None
        except Exception as e:
            logger.warning(f"Find operation error: {e}")
            return None
    
    def safe_find_all(self, soup, *args, **kwargs):
        """Safely find all elements with error handling"""
        try:
            return soup.find_all(*args, **kwargs) if soup else []
        except Exception as e:
            logger.warning(f"Find all operation error: {e}")
            return []

# Usage example
scraper = RobustScraper()

def scrape_product_safely(url):
    """Example of safe scraping"""
    response = scraper.safe_get_page(url)
    if not response:
        return None
    
    soup = scraper.safe_parse(response.content)
    if not soup:
        return None
    
    # Safe data extraction
    product_data = {
        'name': scraper.safe_extract_text(
            scraper.safe_find(soup, 'h1', class_='product-title')
        ),
        'price': scraper.safe_extract_text(
            scraper.safe_find(soup, 'span', class_='price')
        ),
        'image_url': scraper.safe_extract_attr(
            scraper.safe_find(soup, 'img', class_='product-image'),
            'src'
        ),
        'description': scraper.safe_extract_text(
            scraper.safe_find(soup, 'div', class_='description')
        ),
    }
    
    return product_data
```

### Debugging Techniques
```python
def debug_soup_structure(soup, max_depth=3):
    """Debug BeautifulSoup structure"""
    def print_element(element, depth=0):
        if depth > max_depth:
            return
        
        indent = "  " * depth
        if hasattr(element, 'name'):
            attrs = element.attrs if hasattr(element, 'attrs') else {}
            attr_str = ' '.join(f'{k}="{v}"' for k, v in attrs.items())
            print(f"{indent}<{element.name} {attr_str}>")
            
            for child in element.children:
                if hasattr(child, 'name'):  # Skip text nodes for clarity
                    print_element(child, depth + 1)
        else:
            # Text node
            text = str(element).strip()
            if text:
                print(f"{indent}'{text[:50]}...' " if len(text) > 50 else f"{indent}'{text}'")
    
    print("Soup structure:")
    print_element(soup)

def find_elements_debug(soup, selector):
    """Debug element finding"""
    elements = soup.select(selector)
    print(f"Selector '{selector}' found {len(elements)} elements:")
    
    for i, element in enumerate(elements[:5]):  # Show first 5
        print(f"  {i+1}. {element.name} with attributes: {element.attrs}")
        if element.string:
            print(f"     Text: '{element.string[:100]}...'")

def validate_extracted_data(data):
    """Validate extracted data"""
    issues = []
    
    for key, value in data.items():
        if value is None:
            issues.append(f"Missing value for {key}")
        elif isinstance(value, str) and not value.strip():
            issues.append(f"Empty string for {key}")
        elif key == 'price' and not isinstance(value, (int, float)):
            issues.append(f"Invalid price format: {value}")
        elif key == 'url' and not value.startswith(('http://', 'https://')):
            issues.append(f"Invalid URL format: {value}")
    
    if issues:
        print("Data validation issues:")
        for issue in issues:
            print(f"  - {issue}")
    else:
        print("Data validation passed")
    
    return len(issues) == 0

# Usage
soup = BeautifulSoup(html_content, 'html.parser')
debug_soup_structure(soup)
find_elements_debug(soup, '.product .price')
validate_extracted_data(extracted_data)
```
