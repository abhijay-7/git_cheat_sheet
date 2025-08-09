# Scrapy In-Depth Cheatsheet

## Installation and Setup

```bash
# Install Scrapy
pip install scrapy

# Install with additional dependencies
pip install scrapy scrapy-splash scrapy-user-agents fake-useragent

# Create new Scrapy project
scrapy startproject myproject
cd myproject

# Generate a spider
scrapy genspider example example.com

# Run spider
scrapy crawl example
scrapy crawl example -o output.json
scrapy crawl example -s USER_AGENT='MyBot 1.0'
```

## Project Structure

```
myproject/
    myproject/
        __init__.py
        items.py          # Define data structures
        middlewares.py    # Custom middlewares
        pipelines.py      # Data processing pipelines
        settings.py       # Project settings
        spiders/          # Spider directory
            __init__.py
            example.py
    scrapy.cfg           # Deploy configuration
```

## Basic Spider Creation

### Simple Spider
```python
# spiders/quotes_spider.py
import scrapy

class QuotesSpider(scrapy.Spider):
    name = 'quotes'
    allowed_domains = ['quotes.toscrape.com']
    start_urls = ['http://quotes.toscrape.com/']
    
    def parse(self, response):
        """Parse the main page and extract quotes"""
        # Extract quotes using CSS selectors
        quotes = response.css('div.quote')
        
        for quote in quotes:
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('span small::text').get(),
                'tags': quote.css('div.tags a.tag::text').getall(),
            }
        
        # Follow pagination links
        next_page = response.css('li.next a::attr(href)').get()
        if next_page:
            yield response.follow(next_page, self.parse)

    def parse_author(self, response):
        """Parse author detail page"""
        yield {
            'name': response.css('h3.author-title::text').get(),
            'birth_date': response.css('span.author-born-date::text').get(),
            'birth_location': response.css('span.author-born-location::text').get(),
            'description': response.css('div.author-description::text').get(),
        }
```

### Advanced Spider with Multiple Parsing Methods
```python
import scrapy
from urllib.parse import urljoin
import json
import re

class EcommerceSpider(scrapy.Spider):
    name = 'ecommerce'
    allowed_domains = ['example-shop.com']
    start_urls = ['https://example-shop.com/']
    
    custom_settings = {
        'DOWNLOAD_DELAY': 2,
        'RANDOMIZE_DOWNLOAD_DELAY': True,
        'CONCURRENT_REQUESTS_PER_DOMAIN': 1,
    }
    
    def start_requests(self):
        """Generate initial requests with custom headers"""
        headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
            'Accept-Language': 'en-US,en;q=0.5',
            'Accept-Encoding': 'gzip, deflate',
            'Connection': 'keep-alive',
        }
        
        for url in self.start_urls:
            yield scrapy.Request(
                url=url,
                headers=headers,
                callback=self.parse,
                dont_filter=True
            )
    
    def parse(self, response):
        """Parse category pages and product links"""
        # Extract category links
        category_links = response.css('nav.categories a::attr(href)').getall()
        
        for link in category_links:
            category_url = urljoin(response.url, link)
            yield scrapy.Request(
                url=category_url,
                callback=self.parse_category,
                meta={'category': link.split('/')[-1]}
            )
        
        # Extract product links from current page
        product_links = response.css('.product-item a::attr(href)').getall()
        
        for link in product_links:
            product_url = urljoin(response.url, link)
            yield scrapy.Request(
                url=product_url,
                callback=self.parse_product
            )
    
    def parse_category(self, response):
        """Parse category page for products"""
        category = response.meta.get('category', 'unknown')
        
        # Extract products in this category
        product_links = response.css('.product-grid .product-link::attr(href)').getall()
        
        for link in product_links:
            product_url = urljoin(response.url, link)
            yield scrapy.Request(
                url=product_url,
                callback=self.parse_product,
                meta={'category': category}
            )
        
        # Handle pagination
        next_page = response.css('.pagination .next::attr(href)').get()
        if next_page:
            next_url = urljoin(response.url, next_page)
            yield scrapy.Request(
                url=next_url,
                callback=self.parse_category,
                meta={'category': category}
            )
    
    def parse_product(self, response):
        """Parse individual product page"""
        # Extract basic product information
        name = response.css('h1.product-title::text').get()
        price = response.css('.price .current::text').re_first(r'[\d.]+')
        original_price = response.css('.price .original::text').re_first(r'[\d.]+')
        
        # Extract images
        images = response.css('.product-gallery img::attr(src)').getall()
        images = [urljoin(response.url, img) for img in images]
        
        # Extract specifications from table
        specs = {}
        spec_rows = response.css('.specifications tr')
        for row in spec_rows:
            key = row.css('td:first-child::text').get()
            value = row.css('td:last-child::text').get()
            if key and value:
                specs[key.strip()] = value.strip()
        
        # Extract reviews count and rating
        reviews_count = response.css('.reviews-summary .count::text').re_first(r'\d+')
        rating = response.css('.rating .stars::attr(data-rating)').get()
        
        # Extract availability
        availability = response.css('.availability .status::text').get()
        in_stock = 'in stock' in availability.lower() if availability else False
        
        # Extract description
        description_parts = response.css('.description p::text').getall()
        description = ' '.join(part.strip() for part in description_parts if part.strip())
        
        # Extract JSON-LD structured data if available
        json_ld = response.css('script[type="application/ld+json"]::text').get()
        structured_data = {}
        if json_ld:
            try:
                structured_data = json.loads(json_ld)
            except json.JSONDecodeError:
                pass
        
        yield {
            'url': response.url,
            'name': name,
            'price': float(price) if price else None,
            'original_price': float(original_price) if original_price else None,
            'category': response.meta.get('category'),
            'images': images,
            'specifications': specs,
            'reviews_count': int(reviews_count) if reviews_count else 0,
            'rating': float(rating) if rating else None,
            'in_stock': in_stock,
            'availability': availability,
            'description': description,
            'structured_data': structured_data,
            'scraped_at': response.meta.get('download_slot', ''),
        }
        
        # Extract and follow related product links
        related_links = response.css('.related-products a::attr(href)').getall()
        for link in related_links[:5]:  # Limit to avoid infinite crawling
            related_url = urljoin(response.url, link)
            yield scrapy.Request(
                url=related_url,
                callback=self.parse_product,
                meta={'category': response.meta.get('category')}
            )
```

### Spider with Form Handling and POST Requests
```python
import scrapy
from urllib.parse import urljoin

class LoginSpider(scrapy.Spider):
    name = 'login_spider'
    start_urls = ['https://example.com/login']
    
    def parse(self, response):
        """Handle login form"""
        # Extract form data
        csrf_token = response.css('input[name="csrf_token"]::attr(value)').get()
        
        # Prepare form data
        form_data = {
            'username': 'your_username',
            'password': 'your_password',
            'csrf_token': csrf_token,
        }
        
        # Submit login form
        return scrapy.FormRequest.from_response(
            response,
            formdata=form_data,
            callback=self.after_login
        )
    
    def after_login(self, response):
        """Handle response after login"""
        # Check if login was successful
        if "Dashboard" in response.text:
            self.logger.info("Login successful")
            
            # Continue to protected pages
            protected_urls = [
                'https://example.com/dashboard',
                'https://example.com/profile',
                'https://example.com/settings',
            ]
            
            for url in protected_urls:
                yield scrapy.Request(
                    url=url,
                    callback=self.parse_protected_page
                )
        else:
            self.logger.error("Login failed")
    
    def parse_protected_page(self, response):
        """Parse protected content"""
        # Extract data from protected pages
        yield {
            'url': response.url,
            'title': response.css('title::text').get(),
            'content': response.css('.main-content').get(),
        }
```

## Data Items and Item Loaders

### Defining Items
```python
# items.py
import scrapy
from itemloaders.processors import TakeFirst, MapCompose, Join
from w3lib.html import remove_tags
import re

def clean_price(value):
    """Clean price string to extract numeric value"""
    if value:
        # Remove currency symbols and extract numbers
        price_match = re.search(r'[\d,]+\.?\d*', value.replace(',', ''))
        if price_match:
            return float(price_match.group())
    return None

def clean_text(value):
    """Clean text by removing extra whitespace"""
    if value:
        return re.sub(r'\s+', ' ', value.strip())
    return value

class ProductItem(scrapy.Item):
    # Basic fields
    name = scrapy.Field()
    price = scrapy.Field()
    original_price = scrapy.Field()
    currency = scrapy.Field()
    description = scrapy.Field()
    
    # Product details
    brand = scrapy.Field()
    model = scrapy.Field()
    sku = scrapy.Field()
    category = scrapy.Field()
    subcategory = scrapy.Field()
    
    # Images and media
    images = scrapy.Field()
    image_urls = scrapy.Field()
    video_urls = scrapy.Field()
    
    # Availability and stock
    in_stock = scrapy.Field()
    stock_quantity = scrapy.Field()
    availability = scrapy.Field()
    
    # Reviews and ratings
    rating = scrapy.Field()
    reviews_count = scrapy.Field()
    reviews = scrapy.Field()
    
    # Technical specifications
    specifications = scrapy.Field()
    features = scrapy.Field()
    dimensions = scrapy.Field()
    weight = scrapy.Field()
    
    # SEO and metadata
    meta_title = scrapy.Field()
    meta_description = scrapy.Field()
    keywords = scrapy.Field()
    
    # Scraping metadata
    url = scrapy.Field()
    scraped_at = scrapy.Field()
    spider_name = scrapy.Field()

class ReviewItem(scrapy.Item):
    product_url = scrapy.Field()
    reviewer_name = scrapy.Field()
    rating = scrapy.Field()
    title = scrapy.Field()
    content = scrapy.Field()
    date = scrapy.Field()
    helpful_votes = scrapy.Field()
    verified_purchase = scrapy.Field()
```

### Item Loaders
```python
# items.py (continued)
from itemloaders import ItemLoader
from datetime import datetime

class ProductLoader(ItemLoader):
    default_item_class = ProductItem
    
    # Default processors
    default_input_processor = MapCompose(remove_tags, clean_text)
    default_output_processor = TakeFirst()
    
    # Specific field processors
    name_in = MapCompose(remove_tags, clean_text)
    name_out = TakeFirst()
    
    price_in = MapCompose(remove_tags, clean_price)
    price_out = TakeFirst()
    
    original_price_in = MapCompose(remove_tags, clean_price)
    original_price_out = TakeFirst()
    
    description_in = MapCompose(remove_tags, clean_text)
    description_out = Join(' ')
    
    images_out = lambda self, values: list(set(values))  # Remove duplicates
    
    features_out = lambda self, values: [v for v in values if v]  # Remove empty values
    
    scraped_at_in = lambda self, values: [datetime.now().isoformat()]
    scraped_at_out = TakeFirst()

# Usage in spider
class ProductSpider(scrapy.Spider):
    name = 'products'
    
    def parse_product(self, response):
        loader = ProductLoader(selector=response)
        
        # Load data using the loader
        loader.add_css('name', 'h1.product-title::text')
        loader.add_css('price', '.price .current::text')
        loader.add_css('original_price', '.price .original::text')
        loader.add_css('description', '.description p::text')
        loader.add_css('images', '.gallery img::attr(src)')
        loader.add_value('url', response.url)
        loader.add_value('spider_name', self.name)
        
        # Load specifications
        specs = {}
        for row in response.css('.specifications tr'):
            key = row.css('td:first-child::text').get()
            value = row.css('td:last-child::text').get()
            if key and value:
                specs[clean_text(key)] = clean_text(value)
        loader.add_value('specifications', specs)
        
        return loader.load_item()
```

## Selectors and Data Extraction

### CSS Selectors
```python
# Basic CSS selectors
response.css('title::text').get()                    # Get title text
response.css('a::attr(href)').getall()              # Get all href attributes
response.css('.product-price::text').get()          # Get text by class
response.css('#product-123 .name::text').get()      # Get text by ID and class
response.css('div.content p:first-child::text').get()  # First paragraph in content

# Advanced CSS selectors
response.css('tr:nth-child(even) td::text').getall()     # Even table rows
response.css('a[href*="product"]::attr(href)').getall()  # Links containing "product"
response.css('img[alt]:not([alt=""])::attr(src)').getall()  # Images with non-empty alt
response.css('div:contains("Price")::text').get()        # Div containing "Price"

# Pseudo-selectors and complex selections
response.css('ul li:last-child a::text').get()          # Last list item link text
response.css('table tr:has(td.price) td:first-child::text').getall()  # Rows with price
```

### XPath Selectors
```python
# Basic XPath
response.xpath('//title/text()').get()                    # Title text
response.xpath('//a/@href').getall()                      # All href attributes
response.xpath('//div[@class="product"]//span[@class="price"]/text()').get()

# Advanced XPath
response.xpath('//tr[position()>1]/td/text()').getall()   # Skip header row
response.xpath('//a[contains(@href, "product")]/@href').getall()  # Links containing "product"
response.xpath('//div[contains(text(), "Price")]/following-sibling::span/text()').get()

# XPath with conditions
response.xpath('//tr[td[@class="price"]]/td[1]/text()').getall()  # First cell of rows with price
response.xpath('//div[@class="review" and @data-rating>4]//p/text()').getall()  # High-rated reviews

# XPath functions
response.xpath('normalize-space(//div[@class="description"]/text())').get()  # Normalize whitespace
response.xpath('substring-after(//span[@class="price"]/text(), "$")').get()  # Extract after $
response.xpath('count(//div[@class="review"])').get()  # Count reviews

# Complex XPath examples
response.xpath('//table//tr[td[1][contains(text(), "Brand")]]/td[2]/text()').get()  # Brand value
response.xpath('//div[@class="breadcrumb"]//a[position()=last()-1]/text()').get()  # Second-to-last breadcrumb
```

### Regular Expressions with Selectors
```python
# Using regex with selectors
price = response.css('.price::text').re_first(r'\$?([\d,]+\.?\d*)')  # Extract price
phone = response.css('.contact::text').re_first(r'(\d{3}-\d{3}-\d{4})')  # Extract phone
all_numbers = response.css('.stats::text').re(r'\d+')  # Extract all numbers

# Complex regex patterns
email = response.css('.contact::text').re_first(r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}')
urls = response.css('::text').re(r'https?://(?:[-\w.])+(?:\:[0-9]+)?(?:/(?:[\w/_.])*(?:\?(?:[\w&=%.])*)?(?:\#(?:[\w.])*)?)?')
```

## Settings and Configuration

### Common Settings
```python
# settings.py

# Basic settings
BOT_NAME = 'myproject'
SPIDER_MODULES = ['myproject.spiders']
NEWSPIDER_MODULE = 'myproject.spiders'

# Obey robots.txt
ROBOTSTXT_OBEY = True

# Configure delays
DOWNLOAD_DELAY = 3
RANDOMIZE_DOWNLOAD_DELAY = True  # 0.5 * to 1.5 * DOWNLOAD_DELAY
CONCURRENT_REQUESTS = 16
CONCURRENT_REQUESTS_PER_DOMAIN = 8

# User Agent
USER_AGENT = 'myproject (+http://www.yourdomain.com)'

# HTTP settings
DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en',
    'Accept-Encoding': 'gzip, deflate',
    'Cache-Control': 'no-cache',
}

# AutoThrottle settings
AUTOTHROTTLE_ENABLED = True
AUTOTHROTTLE_START_DELAY = 1
AUTOTHROTTLE_MAX_DELAY = 60
AUTOTHROTTLE_TARGET_CONCURRENCY = 1.0
AUTOTHROTTLE_DEBUG = False

# HTTP caching
HTTPCACHE_ENABLED = True
HTTPCACHE_EXPIRATION_SECS = 3600
HTTPCACHE_DIR = 'httpcache'
HTTPCACHE_IGNORE_HTTP_CODES = [500, 503, 504, 400, 403, 404, 408]

# Retry settings
RETRY_ENABLED = True
RETRY_TIMES = 3
RETRY_HTTP_CODES = [500, 502, 503, 504, 408, 429]

# Download timeout
DOWNLOAD_TIMEOUT = 180

# Memory usage
MEMUSAGE_ENABLED = True
MEMUSAGE_LIMIT_MB = 2048
MEMUSAGE_WARNING_MB = 1024

# Enable pipelines
ITEM_PIPELINES = {
    'myproject.pipelines.ValidationPipeline': 200,
    'myproject.pipelines.DuplicatesPipeline': 300,
    'myproject.pipelines.ImagesPipeline': 400,
    'myproject.pipelines.DatabasePipeline': 500,
}

# Enable middlewares
DOWNLOADER_MIDDLEWARES = {
    'myproject.middlewares.CustomUserAgentMiddleware': 400,
    'myproject.middlewares.ProxyMiddleware': 410,
    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': None,
}

# Logging
LOG_LEVEL = 'INFO'
LOG_FILE = 'scrapy.log'

# Feed settings
FEED_EXPORT_ENCODING = 'utf-8'
FEEDS = {
    'items.json': {
        'format': 'json',
        'encoding': 'utf8',
        'store_empty': False,
        'indent': 2,
    },
    'items.csv': {
        'format': 'csv',
        'encoding': 'utf8',
    },
}
```

### Custom Settings per Spider
```python
class MySpider(scrapy.Spider):
    name = 'myspider'
    
    custom_settings = {
        'DOWNLOAD_DELAY': 5,
        'CONCURRENT_REQUESTS_PER_DOMAIN': 1,
        'ITEM_PIPELINES': {
            'myproject.pipelines.SpecialPipeline': 300,
        },
        'USER_AGENT': 'MySpider Bot 1.0',
        'ROBOTSTXT_OBEY': False,
    }
```

## Pipelines

### Data Processing Pipelines
```python
# pipelines.py
import json
import sqlite3
import hashlib
from datetime import datetime
from scrapy.exceptions import DropItem
from scrapy.pipelines.images import ImagesPipeline
import logging

class ValidationPipeline:
    """Validate item data"""
    
    required_fields = ['name', 'price']
    
    def process_item(self, item, spider):
        # Check required fields
        for field in self.required_fields:
            if not item.get(field):
                raise DropItem(f"Missing required field: {field}")
        
        # Validate price
        if item.get('price') and item['price'] <= 0:
            raise DropItem("Invalid price value")
        
        # Clean and validate data
        if item.get('name'):
            item['name'] = item['name'].strip()
            if len(item['name']) > 200:
                item['name'] = item['name'][:200]
        
        return item

class DuplicatesPipeline:
    """Filter duplicate items"""
    
    def __init__(self):
        self.ids_seen = set()
    
    def process_item(self, item, spider):
        # Create unique identifier
        item_id = self.get_item_id(item)
        
        if item_id in self.ids_seen:
            raise DropItem(f"Duplicate item found: {item_id}")
        else:
            self.ids_seen.add(item_id)
            return item
    
    def get_item_id(self, item):
        """Generate unique ID for item"""
        unique_string = f"{item.get('name', '')}-{item.get('url', '')}"
        return hashlib.md5(unique_string.encode()).hexdigest()

class PriceConversionPipeline:
    """Convert prices to standard format"""
    
    exchange_rates = {
        'USD': 1.0,
        'EUR': 0.85,
        'GBP': 0.73,
        'JPY': 110.0,
    }
    
    def process_item(self, item, spider):
        if item.get('price') and item.get('currency'):
            # Convert to USD
            currency = item['currency'].upper()
            if currency in self.exchange_rates:
                rate = self.exchange_rates[currency]
                item['price_usd'] = item['price'] / rate
            
        return item

class DatabasePipeline:
    """Save items to database"""
    
    def __init__(self, database_url='sqlite:///items.db'):
        self.database_url = database_url
        self.connection = None
    
    @classmethod
    def from_crawler(cls, crawler):
        db_url = crawler.settings.get("DATABASE_URL", "sqlite:///items.db")
        return cls(database_url=db_url)
    
    def open_spider(self, spider):
        """Initialize database connection"""
        self.connection = sqlite3.connect('items.db')
        self.connection.execute('''
            CREATE TABLE IF NOT EXISTS products (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                price REAL,
                currency TEXT,
                url TEXT UNIQUE,
                description TEXT,
                scraped_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                spider_name TEXT
            )
        ''')
        self.connection.commit()
    
    def close_spider(self, spider):
        """Close database connection"""
        if self.connection:
            self.connection.close()
    
    def process_item(self, item, spider):
        """Insert item into database"""
        try:
            self.connection.execute('''
                INSERT OR REPLACE INTO products (name, price, currency, url, description, spider_name)
                VALUES (?, ?, ?, ?, ?, ?)
            ''', (
                item.get('name'),
                item.get('price'),
                item.get('currency'),
                item.get('url'),
                item.get('description'),
                spider.name
            ))
            self.connection.commit()
            spider.logger.info(f"Item saved to database: {item.get('name')}")
        except sqlite3.Error as e:
            spider.logger.error(f"Database error: {e}")
            raise DropItem(f"Error inserting item: {e}")
        
        return item

class JsonFilesPipeline:
    """Save items to JSON files"""
    
    def __init__(self):
        self.files = {}
    
    def open_spider(self, spider):
        """Open output file for spider"""
        filename = f"{spider.name}_items.json"
        self.files[spider.name] = open(filename, 'w', encoding='utf-8')
        self.files[spider.name].write('[\n')
    
    def close_spider(self, spider):
        """Close output file"""
        if spider.name in self.files:
            self.files[spider.name].write('\n]')
            self.files[spider.name].close()
    
    def process_item(self, item, spider):
        """Write item to JSON file"""
        line = json.dumps(dict(item), ensure_ascii=False, indent=2) + ',\n'
        self.files[spider.name].write(line)
        return item

class CustomImagesPipeline(ImagesPipeline):
    """Custom image downloading pipeline"""
    
    def get_media_requests(self, item, info):
        """Generate requests for image URLs"""
        urls = item.get('image_urls', [])
        for url in urls:
            yield scrapy.Request(
                url,
                meta={'item': item},
                headers={'Referer': item.get('url', '')}
            )
    
    def item_completed(self, results, item, info):
        """Process completed image downloads"""
        image_paths = []
        for ok, result in results:
            if ok:
                image_paths.append(result['path'])
            else:
                info.spider.logger.warning(f"Image download failed: {result}")
        
        item['images'] = image_paths
        return item

class StatsPipeline:
    """Collect statistics about scraped items"""
    
    def __init__(self):
        self.stats = {
            'items_count': 0,
            'items_with_price': 0,
            'items_with_images': 0,
            'avg_price': 0,
            'total_price': 0,
        }
    
    def process_item(self, item, spider):
        """Update statistics"""
        self.stats['items_count'] += 1
        
        if item.get('price'):
            self.stats['items_with_price'] += 1
            self.stats['total_price'] += item['price']
            self.stats['avg_price'] = self.stats['total_price'] / self.stats['items_with_price']
        
        if item.get('images'):
            self.stats['items_with_images'] += 1
        
        return item
    
    def close_spider(self, spider):
        """Log final statistics"""
        spider.logger.info(f"Scraping statistics: {self.stats}")
```

## Middlewares

### Custom Middlewares
```python
# middlewares.py
import random
import logging
from scrapy.downloadermiddlewares.retry import RetryMiddleware
from scrapy.downloadermiddlewares.httpproxy import HttpProxyMiddleware
from scrapy.exceptions import NotConfigured, IgnoreRequest
from scrapy.http import HtmlResponse
import time

class CustomUserAgentMiddleware:
    """Rotate user agents"""
    
    def __init__(self):
        self.user_agents = [
            'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36',
            'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36',
            'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36',
            'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:89.0) Gecko/20100101 Firefox/89.0',
            'Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:89.0) Gecko/20100101 Firefox/89.0',
        ]
    
    def process_request(self, request, spider):
        """Set random user agent for each request"""
        user_agent = random.choice(self.user_agents)
        request.headers['User-Agent'] = user_agent
        return None

class ProxyMiddleware:
    """Handle proxy rotation"""
    
    def __init__(self):
        self.proxies = [
            'http://proxy1:8080',
            'http://proxy2:8080',
            'http://proxy3:8080',
        ]
        self.proxy_auth = {
            'http://proxy1:8080': 'username:password',
            'http://proxy2:8080': 'username:password',
        }
    
    def process_request(self, request, spider):
        """Set proxy for request"""
        if self.proxies:
            proxy = random.choice(self.proxies)
            request.meta['proxy'] = proxy
            
            # Set authentication if required
            if proxy in self.proxy_auth:
                auth = self.proxy_auth[proxy]
                request.headers['Proxy-Authorization'] = f'Basic {auth}'
        
        return None

class RetryWithDelayMiddleware(RetryMiddleware):
    """Custom retry middleware with exponential backoff"""
    
    def __init__(self, settings):
        super().__init__(settings)
        self.base_delay = settings.getfloat('RETRY_DELAY', 1.0)
        self.max_delay = settings.getfloat('RETRY_MAX_DELAY', 60.0)
    
    def process_response(self, request, response, spider):
        """Process response and decide whether to retry"""
        if response.status in self.retry_http_codes:
            reason = response_status_message(response.status)
            return self._retry_with_delay(request, reason, spider) or response
        return response
    
    def _retry_with_delay(self, request, reason, spider):
        """Retry request with exponential backoff delay"""
        retries = request.meta.get('retry_times', 0) + 1
        
        if retries <= self.max_retry_times:
            # Calculate delay with exponential backoff
            delay = min(self.base_delay * (2 ** retries), self.max_delay)
            
            spider.logger.info(f"Retrying {request.url} (attempt {retries}) in {delay} seconds: {reason}")
            
            # Add delay
            time.sleep(delay)
            
            retryreq = request.copy()
            retryreq.meta['retry_times'] = retries
            retryreq.dont_filter = True
            
            return retryreq
        else:
            spider.logger.error(f"Gave up retrying {request.url} (failed {retries} times): {reason}")

class JavaScriptMiddleware:
    """Handle JavaScript rendering with Splash"""
    
    def __init__(self):
        self.splash_url = 'http://localhost:8050'
    
    @classmethod
    def from_crawler(cls, crawler):
        if not crawler.settings.getbool('SPLASH_ENABLED'):
            raise NotConfigured('Splash not enabled')
        return cls()
    
    def process_request(self, request, spider):
        """Send request to Splash for JavaScript rendering"""
        if request.meta.get('splash'):
            splash_args = request.meta.get('splash', {})
            splash_url = f"{self.splash_url}/render.html"
            
            # Default Splash arguments
            args = {
                'url': request.url,
                'wait': splash_args.get('wait', 2),
                'html': 1,
                'png': splash_args.get('png', 0),
            }
            
            # Create new request to Splash
            return scrapy.Request(
                url=splash_url,
                method='POST',
                body=json.dumps(args),
                headers={'Content-Type': 'application/json'},
                meta=request.meta,
                callback=request.callback,
                dont_filter=True
            )
        
        return None

class HeadersMiddleware:
    """Add custom headers to requests"""
    
    def process_request(self, request, spider):
        """Add custom headers"""
        # Add common headers
        request.headers.setdefault('Accept-Language', 'en-US,en;q=0.9')
        request.headers.setdefault('Accept-Encoding', 'gzip, deflate, br')
        request.headers.setdefault('DNT', '1')
        request.headers.setdefault('Connection', 'keep-alive')
        request.headers.setdefault('Upgrade-Insecure-Requests', '1')
        
        # Add referer if not present
        if not request.headers.get('Referer') and hasattr(spider, 'allowed_domains'):
            if spider.allowed_domains:
                request.headers['Referer'] = f"https://{spider.allowed_domains[0]}/"
        
        return None

class CacheControlMiddleware:
    """Control caching behavior"""
    
    def process_request(self, request, spider):
        """Set cache control headers"""
        # Disable caching for certain requests
        if request.meta.get('no_cache'):
            request.headers['Cache-Control'] = 'no-cache, no-store, must-revalidate'
            request.headers['Pragma'] = 'no-cache'
            request.headers['Expires'] = '0'
        
        return None

class RequestLimitMiddleware:
    """Limit requests per domain"""
    
    def __init__(self):
        self.domain_requests = {}
        self.max_requests_per_domain = 1000
    
    def process_request(self, request, spider):
        """Check request limits"""
        domain = urlparse(request.url).netloc
        
        if domain not in self.domain_requests:
            self.domain_requests[domain] = 0
        
        if self.domain_requests[domain] >= self.max_requests_per_domain:
            spider.logger.warning(f"Request limit reached for domain: {domain}")
            raise IgnoreRequest(f"Request limit exceeded for {domain}")
        
        self.domain_requests[domain] += 1
        return None
```

## Advanced Scrapy Features

### Scrapy Shell Usage
```bash
# Start Scrapy shell
scrapy shell "http://example.com"

# Shell commands
response.css('title::text').get()
response.xpath('//title/text()').get()
response.follow('next-page')
fetch('http://example.com/page2')
view(response)  # Open in browser

# Test selectors
response.css('.product').getall()
len(response.css('.product'))
response.css('.product')[0].css('.name::text').get()
```

### Link Extractors
```python
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule

class CrawlingSpider(CrawlSpider):
    name = 'crawler'
    allowed_domains = ['example.com']
    start_urls = ['http://example.com/']
    
    rules = (
        # Extract links matching 'category' and parse them with parse_category
        Rule(LinkExtractor(allow=r'/category/'), callback='parse_category', follow=True),
        
        # Extract product links and parse them
        Rule(LinkExtractor(allow=r'/product/\d+'), callback='parse_product'),
        
        # Follow pagination links
        Rule(LinkExtractor(allow=r'/page/\d+'), follow=True),
        
        # Restrict to certain domains
        Rule(LinkExtractor(allow_domains=['example.com', 'shop.example.com']), follow=True),
        
        # Deny certain patterns
        Rule(LinkExtractor(deny=r'/admin/'), follow=False),
    )
    
    def parse_category(self, response):
        # Extract category data
        yield {
            'category_name': response.css('h1::text').get(),
            'product_count': len(response.css('.product-item')),
        }
    
    def parse_product(self, response):
        # Extract product data
        yield {
            'name': response.css('.product-name::text').get(),
            'price': response.css('.price::text').get(),
        }

# Custom Link Extractor
class CustomLinkExtractor(LinkExtractor):
    def extract_links(self, response):
        links = super().extract_links(response)
        
        # Filter links based on custom logic
        filtered_links = []
        for link in links:
            # Only include links with certain patterns
            if 'product' in link.url and 'id=' in link.url:
                filtered_links.append(link)
        
        return filtered_links
```

### Handling Forms and Sessions
```python
import scrapy
from scrapy import FormRequest

class FormSpider(scrapy.Spider):
    name = 'forms'
    start_urls = ['http://example.com/search']
    
    def parse(self, response):
        """Handle search form"""
        return FormRequest.from_response(
            response,
            formname='search_form',  # Form name attribute
            formdata={'query': 'laptops', 'category': 'electronics'},
            callback=self.parse_results
        )
    
    def parse_results(self, response):
        """Parse search results"""
        for product in response.css('.search-result'):
            yield {
                'name': product.css('.name::text').get(),
                'price': product.css('.price::text').get(),
            }
        
        # Handle pagination
        next_page = response.css('.pagination .next::attr(href)').get()
        if next_page:
            yield response.follow(next_page, self.parse_results)

class SessionSpider(scrapy.Spider):
    name = 'session'
    
    def start_requests(self):
        """Login first to maintain session"""
        return [scrapy.Request('http://example.com/login', callback=self.login)]
    
    def login(self, response):
        """Perform login"""
        return FormRequest.from_response(
            response,
            formdata={'username': 'user', 'password': 'pass'},
            callback=self.after_login
        )
    
    def after_login(self, response):
        """Continue scraping after login"""
        # Session cookies are automatically maintained
        yield scrapy.Request('http://example.com/protected', callback=self.parse_protected)
    
    def parse_protected(self, response):
        """Parse protected content"""
        # Access protected data
        pass
```

### Custom Commands
```python
# myproject/commands/crawl_all.py
from scrapy.commands import ScrapyCommand
from scrapy.utils.project import get_project_settings

class Command(ScrapyCommand):
    requires_project = True
    
    def syntax(self):
        return '<spider1> <spider2> ...'
    
    def short_desc(self):
        return 'Run multiple spiders'
    
    def run(self, args, opts):
        """Run multiple spiders sequentially"""
        spider_names = args
        settings = get_project_settings()
        
        for spider_name in spider_names:
            self.crawler_process.crawl(spider_name, **opts.__dict__)
        
        self.crawler_process.start()

# Usage: scrapy crawl_all spider1 spider2 spider3
```

### Contracts and Testing
```python
# contracts.py
from scrapy.contracts import Contract
from scrapy.exceptions import ContractFail

class ReturnsItemsContract(Contract):
    """Contract to ensure spider returns items
    
    @returns items 10 20
    """
    
    name = 'returns_items'
    
    def pre_process(self, request, response, spider):
        self.expected_min = int(self.args[0])
        self.expected_max = int(self.args[1])
        self.items_count = 0
    
    def post_process(self, request, response, spider):
        if not (self.expected_min <= self.items_count <= self.expected_max):
            raise ContractFail(
                f"Expected {self.expected_min}-{self.expected_max} items, got {self.items_count}"
            )
    
    def adjust_request_args(self, args):
        return args

# Usage in spider
class TestSpider(scrapy.Spider):
    name = 'test'
    
    def parse(self, response):
        """Parse page
        
        @url http://example.com
        @returns items 5 15
        @scrapes name price
        """
        # Spider implementation
        pass

# Run contracts: scrapy check
```

## Performance Optimization

### Memory Management
```python
# settings.py

# Memory usage monitoring
MEMUSAGE_ENABLED = True
MEMUSAGE_LIMIT_MB = 2048
MEMUSAGE_WARNING_MB = 1024

# Disable certain extensions to save memory
EXTENSIONS = {
    'scrapy.extensions.telnet.TelnetConsole': None,
    'scrapy.extensions.memusage.MemoryUsage': 500,
}

# Optimize item processing
CONCURRENT_ITEMS = 100

# Disable cookies if not needed
COOKIES_ENABLED = False

# Reduce memory usage for large responses
DOWNLOAD_MAXSIZE = 1073741824  # 1GB
DOWNLOAD_WARNSIZE = 33554432   # 32MB
```

### Speed Optimization
```python
# settings.py

# Increase concurrency
CONCURRENT_REQUESTS = 32
CONCURRENT_REQUESTS_PER_DOMAIN = 16

# Reduce delays
DOWNLOAD_DELAY = 0.25
RANDOMIZE_DOWNLOAD_DELAY = False

# Disable unnecessary middlewares
DOWNLOADER_MIDDLEWARES = {
    'scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware': None,
    'scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware': None,
}

# Optimize DNS
DNSCACHE_ENABLED = True
DNSCACHE_SIZE = 10000
DNS_TIMEOUT = 60

# Connection pooling
REACTOR_THREADPOOL_MAXSIZE = 20
```

### Distributed Scraping with Scrapy-Redis
```python
# Install: pip install scrapy-redis

# settings.py
SCHEDULER = "scrapy_redis.scheduler.Scheduler"
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"

ITEM_PIPELINES = {
    'scrapy_redis.pipelines.RedisPipeline': 300,
}

REDIS_URL = 'redis://localhost:6379'

# Spider using Redis
from scrapy_redis.spiders import RedisSpider

class DistributedSpider(RedisSpider):
    name = 'distributed'
    redis_key = 'distributed_spider:start_urls'
    
    def parse(self, response):
        # Parse and yield items
        pass

# Add URLs to Redis: redis-cli lpush distributed_spider:start_urls "http://example.com"
```

## Deployment and Production

### Scrapyd Deployment
```bash
# Install Scrapyd
pip install scrapyd scrapyd-client

# Start Scrapyd server
scrapyd

# Deploy project
scrapyd-deploy

# Schedule spider
curl http://localhost:6800/schedule.json -d project=myproject -d spider=myspider

# Check jobs
curl http://localhost:6800/listjobs.json?project=myproject
```

### Docker Deployment
```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

RUN apt-get update && apt-get install -y \
    gcc \
    libc-dev \
    libffi-dev \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["scrapy", "crawl", "myspider"]
```

### Production Settings
```python
# production_settings.py
from .settings import *

# Production-specific settings
LOG_LEVEL = 'WARNING'
DOWNLOAD_DELAY = 2
CONCURRENT_REQUESTS = 8
CONCURRENT_REQUESTS_PER_DOMAIN = 2

# Enable monitoring
STATSMAILER_RCPTS = ['admin@example.com']

# Database connection pooling
DATABASE_URL = 'postgresql://user:pass@host/db'

# Use Redis for caching
HTTPCACHE_ENABLED = True
HTTPCACHE_STORAGE = 'scrapy_redis.cache.RedisCacheStorage'
```

### Monitoring and Logging
```python
# Custom stats collection
class StatsCollectorPipeline:
    def __init__(self, stats):
        self.stats = stats
    
    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler.stats)
    
    def process_item(self, item, spider):
        self.stats.inc_value('items_scraped')
        if item.get('price'):
            self.stats.inc_value('items_with_price')
        return item

# Logging configuration
import logging
from scrapy.utils.log import configure_logging

configure_logging({'LOG_FORMAT': '%(levelname)s: %(message)s'})
logging.getLogger('scrapy').setLevel(logging.WARNING)
logging.getLogger('myproject').setLevel(logging.INFO)
```
