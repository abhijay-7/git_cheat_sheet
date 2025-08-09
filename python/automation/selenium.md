# Selenium In-Depth Cheatsheet

## Installation and Setup

```bash
# Install Selenium
pip install selenium

# Install with additional utilities
pip install selenium webdriver-manager requests beautifulsoup4

# Install browser drivers (alternative to webdriver-manager)
# Chrome: Download chromedriver and add to PATH
# Firefox: Download geckodriver and add to PATH
# Edge: Download edgedriver and add to PATH
```

## WebDriver Setup and Configuration

### Basic WebDriver Setup
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager

# Automatic driver management (recommended)
driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))

# Manual driver setup
driver = webdriver.Chrome(executable_path='/path/to/chromedriver')

# Firefox
from webdriver_manager.firefox import GeckoDriverManager
driver = webdriver.Firefox(service=Service(GeckoDriverManager().install()))

# Edge
from webdriver_manager.microsoft import EdgeChromiumDriverManager
driver = webdriver.Edge(service=Service(EdgeChromiumDriverManager().install()))
```

### Chrome Options Configuration
```python
chrome_options = Options()

# Headless mode (no GUI)
chrome_options.add_argument('--headless')

# Window size
chrome_options.add_argument('--window-size=1920,1080')

# Disable images (faster loading)
chrome_options.add_argument('--blink-settings=imagesEnabled=false')

# Disable JavaScript (if not needed)
chrome_options.add_argument('--disable-javascript')

# Disable extensions
chrome_options.add_argument('--disable-extensions')

# Disable GPU acceleration
chrome_options.add_argument('--disable-gpu')

# Disable dev-shm usage (useful in Docker)
chrome_options.add_argument('--disable-dev-shm-usage')

# No sandbox (useful in Docker)
chrome_options.add_argument('--no-sandbox')

# Custom user agent
chrome_options.add_argument('--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36')

# Disable notifications
prefs = {"profile.default_content_setting_values.notifications": 2}
chrome_options.add_experimental_option("prefs", prefs)

# Disable automation detection
chrome_options.add_experimental_option("excludeSwitches", ["enable-automation"])
chrome_options.add_experimental_option('useAutomationExtension', False)

# Set download directory
prefs = {"download.default_directory": "/path/to/downloads"}
chrome_options.add_experimental_option("prefs", prefs)

# Create driver with options
driver = webdriver.Chrome(
    service=Service(ChromeDriverManager().install()),
    options=chrome_options
)

# Execute script to hide automation indicators
driver.execute_script("Object.defineProperty(navigator, 'webdriver', {get: () => undefined})")
```

### Advanced WebDriver Configuration
```python
class WebDriverManager:
    def __init__(self, browser='chrome', headless=True, window_size='1920,1080'):
        self.browser = browser.lower()
        self.headless = headless
        self.window_size = window_size
        self.driver = None
    
    def create_driver(self):
        """Create WebDriver based on browser type"""
        if self.browser == 'chrome':
            return self._create_chrome_driver()
        elif self.browser == 'firefox':
            return self._create_firefox_driver()
        elif self.browser == 'edge':
            return self._create_edge_driver()
        else:
            raise ValueError(f"Unsupported browser: {self.browser}")
    
    def _create_chrome_driver(self):
        """Create Chrome WebDriver with optimized settings"""
        options = Options()
        
        if self.headless:
            options.add_argument('--headless')
        
        options.add_argument(f'--window-size={self.window_size}')
        options.add_argument('--no-sandbox')
        options.add_argument('--disable-dev-shm-usage')
        options.add_argument('--disable-gpu')
        options.add_argument('--disable-extensions')
        options.add_argument('--disable-logging')
        options.add_argument('--disable-notifications')
        options.add_argument('--disable-popup-blocking')
        options.add_argument('--disable-translate')
        options.add_argument('--disable-background-timer-throttling')
        options.add_argument('--disable-backgrounding-occluded-windows')
        options.add_argument('--disable-renderer-backgrounding')
        
        # Performance optimizations
        options.add_argument('--disable-features=TranslateUI')
        options.add_argument('--disable-ipc-flooding-protection')
        options.add_argument('--memory-pressure-off')
        options.add_argument('--max_old_space_size=4096')
        
        # Privacy and security
        options.add_argument('--disable-background-networking')
        options.add_argument('--disable-default-apps')
        options.add_argument('--disable-sync')
        
        # Experimental options
        options.add_experimental_option('excludeSwitches', ['enable-automation'])
        options.add_experimental_option('useAutomationExtension', False)
        
        prefs = {
            'profile.default_content_setting_values.notifications': 2,
            'profile.default_content_settings.popups': 0,
            'profile.managed_default_content_settings.images': 2,  # Block images
        }
        options.add_experimental_option('prefs', prefs)
        
        service = Service(ChromeDriverManager().install())
        driver = webdriver.Chrome(service=service, options=options)
        
        # Execute script to hide webdriver property
        driver.execute_script(
            "Object.defineProperty(navigator, 'webdriver', {get: () => undefined})"
        )
        
        return driver
    
    def _create_firefox_driver(self):
        """Create Firefox WebDriver"""
        from selenium.webdriver.firefox.options import Options as FirefoxOptions
        
        options = FirefoxOptions()
        
        if self.headless:
            options.add_argument('--headless')
        
        options.add_argument(f'--width={self.window_size.split(",")[0]}')
        options.add_argument(f'--height={self.window_size.split(",")[1]}')
        
        # Firefox preferences
        options.set_preference('dom.webnotifications.enabled', False)
        options.set_preference('media.volume_scale', '0.0')
        
        service = Service(GeckoDriverManager().install())
        return webdriver.Firefox(service=service, options=options)
    
    def _create_edge_driver(self):
        """Create Edge WebDriver"""
        from selenium.webdriver.edge.options import Options as EdgeOptions
        
        options = EdgeOptions()
        
        if self.headless:
            options.add_argument('--headless')
        
        options.add_argument(f'--window-size={self.window_size}')
        options.add_argument('--disable-extensions')
        
        service = Service(EdgeChromiumDriverManager().install())
        return webdriver.Edge(service=service, options=options)
    
    def __enter__(self):
        """Context manager entry"""
        self.driver = self.create_driver()
        return self.driver
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """Context manager exit"""
        if self.driver:
            self.driver.quit()

# Usage
with WebDriverManager('chrome', headless=True) as driver:
    driver.get('https://example.com')
    # Your scraping code here
```

## Element Location and Interaction

### Finding Elements
```python
from selenium.webdriver.common.by import By

# Find by ID
element = driver.find_element(By.ID, 'element-id')
elements = driver.find_elements(By.ID, 'element-id')

# Find by class name
element = driver.find_element(By.CLASS_NAME, 'class-name')
elements = driver.find_elements(By.CLASS_NAME, 'class-name')

# Find by tag name
element = driver.find_element(By.TAG_NAME, 'div')
elements = driver.find_elements(By.TAG_NAME, 'div')

# Find by name attribute
element = driver.find_element(By.NAME, 'username')

# Find by CSS selector
element = driver.find_element(By.CSS_SELECTOR, '.class-name')
element = driver.find_element(By.CSS_SELECTOR, '#element-id')
element = driver.find_element(By.CSS_SELECTOR, 'div.class-name')
element = driver.find_element(By.CSS_SELECTOR, 'input[type="submit"]')
element = driver.find_element(By.CSS_SELECTOR, 'a[href*="example"]')

# Find by XPath
element = driver.find_element(By.XPATH, '//div[@class="content"]')
element = driver.find_element(By.XPATH, '//input[@type="submit"]')
element = driver.find_element(By.XPATH, '//a[contains(@href, "example")]')
element = driver.find_element(By.XPATH, '//div[text()="Exact text"]')
element = driver.find_element(By.XPATH, '//div[contains(text(), "Partial text")]')

# Find by link text
element = driver.find_element(By.LINK_TEXT, 'Click here')
element = driver.find_element(By.PARTIAL_LINK_TEXT, 'Click')

# Advanced XPath examples
# Find element by attribute
element = driver.find_element(By.XPATH, '//input[@placeholder="Enter email"]')

# Find element by text content
element = driver.find_element(By.XPATH, '//button[text()="Submit"]')

# Find parent element
parent = driver.find_element(By.XPATH, '//input[@id="username"]/parent::div')

# Find following sibling
sibling = driver.find_element(By.XPATH, '//label[@for="username"]/following-sibling::input')

# Find element with multiple conditions
element = driver.find_element(By.XPATH, '//div[@class="product" and @data-price]')

# Find nth element
element = driver.find_element(By.XPATH, '(//div[@class="item"])[3]')  # 3rd item
```

### Element Interaction
```python
# Click element
element.click()

# Send text to input
element.send_keys('Hello World')
element.send_keys(Keys.ENTER)

# Clear input field
element.clear()

# Get element text
text = element.text

# Get attribute value
href = element.get_attribute('href')
class_name = element.get_attribute('class')
data_value = element.get_attribute('data-value')

# Get CSS property
color = element.value_of_css_property('color')
font_size = element.value_of_css_property('font-size')

# Check if element is displayed/enabled/selected
is_displayed = element.is_displayed()
is_enabled = element.is_enabled()
is_selected = element.is_selected()  # For checkboxes/radio buttons

# Get element size and location
size = element.size
location = element.location
rect = element.rect  # Combined size and location

# Take screenshot of element
element.screenshot('element_screenshot.png')

# Submit form (if element is form or inside form)
element.submit()
```

### Advanced Element Operations
```python
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys

# Action Chains for complex interactions
actions = ActionChains(driver)

# Mouse operations
actions.move_to_element(element).perform()           # Hover
actions.click(element).perform()                     # Click
actions.double_click(element).perform()              # Double click
actions.context_click(element).perform()             # Right click
actions.drag_and_drop(source, target).perform()     # Drag and drop

# Keyboard operations
actions.send_keys('Hello').perform()
actions.send_keys(Keys.CONTROL, 'a').perform()      # Ctrl+A
actions.send_keys(Keys.CONTROL, 'c').perform()      # Ctrl+C
actions.key_down(Keys.SHIFT).send_keys('hello').key_up(Keys.SHIFT).perform()

# Complex action sequences
actions.move_to_element(element1).click().move_to_element(element2).click().perform()

# Scroll operations
driver.execute_script("arguments[0].scrollIntoView();", element)
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")  # Scroll to bottom
driver.execute_script("window.scrollTo(0, 0);")  # Scroll to top

# Custom JavaScript execution
driver.execute_script("arguments[0].style.border='3px solid red'", element)  # Highlight element
driver.execute_script("arguments[0].click();", element)  # JavaScript click
result = driver.execute_script("return arguments[0].textContent;", element)  # Get text content
```

## Waits and Synchronization

### Explicit Waits
```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By

# Basic explicit wait
wait = WebDriverWait(driver, 10)  # Wait up to 10 seconds

# Wait for element to be present
element = wait.until(EC.presence_of_element_located((By.ID, 'element-id')))

# Wait for element to be visible
element = wait.until(EC.visibility_of_element_located((By.CLASS_NAME, 'content')))

# Wait for element to be clickable
element = wait.until(EC.element_to_be_clickable((By.ID, 'submit-button')))

# Wait for text to be present in element
wait.until(EC.text_to_be_present_in_element((By.ID, 'status'), 'Complete'))

# Wait for title to contain text
wait.until(EC.title_contains('Dashboard'))

# Wait for URL to contain text
wait.until(EC.url_contains('dashboard'))

# Wait for element to disappear
wait.until(EC.invisibility_of_element_located((By.ID, 'loading-spinner')))

# Wait for alert to be present
alert = wait.until(EC.alert_is_present())

# Custom wait conditions
def element_has_class(locator, class_name):
    """Custom condition to wait for element to have specific class"""
    def _predicate(driver):
        element = driver.find_element(*locator)
        return class_name in element.get_attribute('class').split()
    return _predicate

wait.until(element_has_class((By.ID, 'status'), 'success'))

# Wait with custom exception handling
try:
    element = wait.until(EC.presence_of_element_located((By.ID, 'dynamic-content')))
except TimeoutException:
    print("Element not found within timeout period")
    element = None
```

### Custom Wait Conditions
```python
class CustomExpectedConditions:
    """Custom expected conditions for specific scenarios"""
    
    @staticmethod
    def element_attribute_to_be(locator, attribute, value):
        """Wait for element attribute to have specific value"""
        def _predicate(driver):
            element = driver.find_element(*locator)
            return element.get_attribute(attribute) == value
        return _predicate
    
    @staticmethod
    def element_count_to_be(locator, count):
        """Wait for specific number of elements"""
        def _predicate(driver):
            elements = driver.find_elements(*locator)
            return len(elements) == count
        return _predicate
    
    @staticmethod
    def page_loaded():
        """Wait for page to be fully loaded"""
        def _predicate(driver):
            return driver.execute_script("return document.readyState") == "complete"
        return _predicate
    
    @staticmethod
    def jquery_inactive():
        """Wait for jQuery to be inactive"""
        def _predicate(driver):
            try:
                return driver.execute_script("return jQuery.active == 0")
            except:
                return True  # jQuery not present
        return _predicate
    
    @staticmethod
    def element_text_to_match_regex(locator, pattern):
        """Wait for element text to match regex pattern"""
        import re
        def _predicate(driver):
            element = driver.find_element(*locator)
            return re.search(pattern, element.text) is not None
        return _predicate

# Usage
wait = WebDriverWait(driver, 10)
wait.until(CustomExpectedConditions.page_loaded())
wait.until(CustomExpectedConditions.element_count_to_be((By.CLASS_NAME, 'item'), 5))
wait.until(CustomExpectedConditions.element_text_to_match_regex((By.ID, 'price'), r'\$\d+\.\d{2}'))
```

### Implicit Waits
```python
# Set implicit wait (applies to all find operations)
driver.implicitly_wait(10)  # Wait up to 10 seconds for elements to appear

# Note: Mixing implicit and explicit waits can cause unexpected delays
# It's generally recommended to use explicit waits only
```

### Fluent Waits
```python
from selenium.webdriver.support.wait import WebDriverWait

# Fluent wait with custom polling interval
wait = WebDriverWait(driver, timeout=30, poll_frequency=1, ignored_exceptions=[NoSuchElementException])

element = wait.until(EC.presence_of_element_located((By.ID, 'dynamic-element')))
```

## Form Handling and User Input

### Basic Form Operations
```python
# Find form elements
username_field = driver.find_element(By.NAME, 'username')
password_field = driver.find_element(By.NAME, 'password')
submit_button = driver.find_element(By.XPATH, '//input[@type="submit"]')

# Fill form fields
username_field.send_keys('user@example.com')
password_field.send_keys('password123')

# Clear existing text
username_field.clear()
username_field.send_keys('new_username')

# Submit form
submit_button.click()
# Or
password_field.send_keys(Keys.RETURN)
# Or
username_field.submit()  # Submit the form containing this element
```

### Dropdown and Select Elements
```python
from selenium.webdriver.support.ui import Select

# Find select element
select_element = driver.find_element(By.NAME, 'country')
select = Select(select_element)

# Select by visible text
select.select_by_visible_text('United States')

# Select by value attribute
select.select_by_value('us')

# Select by index (0-based)
select.select_by_index(1)

# Get all options
all_options = select.options
for option in all_options:
    print(option.text, option.get_attribute('value'))

# Get selected option
selected_option = select.first_selected_option
print(selected_option.text)

# For multi-select dropdowns
select.deselect_all()
select.select_by_visible_text('Option 1')
select.select_by_visible_text('Option 2')

# Get all selected options
selected_options = select.all_selected_options
```

### Checkboxes and Radio Buttons
```python
# Checkboxes
checkbox = driver.find_element(By.ID, 'newsletter')

# Check if selected
if not checkbox.is_selected():
    checkbox.click()  # Check the box

# Uncheck if selected
if checkbox.is_selected():
    checkbox.click()  # Uncheck the box

# Radio buttons
radio_button = driver.find_element(By.CSS_SELECTOR, 'input[name="gender"][value="male"]')
radio_button.click()

# Find all radio buttons in a group
radio_group = driver.find_elements(By.NAME, 'gender')
for radio in radio_group:
    if radio.get_attribute('value') == 'female':
        radio.click()
        break
```

### File Upload
```python
# File upload input
file_input = driver.find_element(By.CSS_SELECTOR, 'input[type="file"]')

# Upload file by sending file path
file_path = '/path/to/your/file.pdf'
file_input.send_keys(file_path)

# Multiple file upload (if supported)
file_paths = '/path/to/file1.pdf\n/path/to/file2.pdf'
file_input.send_keys(file_paths)

# Handle file dialog (alternative approach)
import pyautogui
import time

upload_button = driver.find_element(By.ID, 'upload-btn')
upload_button.click()

time.sleep(2)  # Wait for dialog to open
pyautogui.write('/path/to/file.pdf')
pyautogui.press('enter')
```

### Advanced Form Handling
```python
class FormHandler:
    def __init__(self, driver):
        self.driver = driver
        self.wait = WebDriverWait(driver, 10)
    
    def fill_form(self, form_data):
        """Fill form based on data dictionary"""
        for field_name, value in form_data.items():
            try:
                # Try different methods to find the field
                field = self._find_form_field(field_name)
                
                if field.tag_name.lower() == 'select':
                    self._handle_select(field, value)
                elif field.get_attribute('type') in ['checkbox', 'radio']:
                    self._handle_checkbox_radio(field, value)
                elif field.get_attribute('type') == 'file':
                    field.send_keys(value)
                else:
                    field.clear()
                    field.send_keys(value)
                    
            except Exception as e:
                print(f"Error filling field {field_name}: {e}")
    
    def _find_form_field(self, field_name):
        """Try multiple strategies to find form field"""
        strategies = [
            (By.NAME, field_name),
            (By.ID, field_name),
            (By.CSS_SELECTOR, f'[name="{field_name}"]'),
            (By.CSS_SELECTOR, f'[id="{field_name}"]'),
            (By.XPATH, f'//input[@placeholder="{field_name}"]'),
            (By.XPATH, f'//label[text()="{field_name}"]/following-sibling::input'),
        ]
        
        for by, value in strategies:
            try:
                return self.driver.find_element(by, value)
            except:
                continue
        
        raise Exception(f"Could not find form field: {field_name}")
    
    def _handle_select(self, field, value):
        """Handle select dropdown"""
        select = Select(field)
        try:
            select.select_by_visible_text(value)
        except:
            try:
                select.select_by_value(value)
            except:
                select.select_by_index(int(value))
    
    def _handle_checkbox_radio(self, field, value):
        """Handle checkbox and radio button"""
        if isinstance(value, bool):
            if value and not field.is_selected():
                field.click()
            elif not value and field.is_selected():
                field.click()
        else:
            # For radio buttons, value might be the value attribute
            if field.get_attribute('value') == str(value):
                field.click()

# Usage
form_data = {
    'username': 'john_doe',
    'email': 'john@example.com',
    'password': 'secure_password',
    'country': 'United States',
    'newsletter': True,
    'age_group': '25-34'
}

form_handler = FormHandler(driver)
form_handler.fill_form(form_data)
```

## Browser Control and Navigation

### Basic Navigation
```python
# Navigate to URL
driver.get('https://example.com')

# Get current URL
current_url = driver.current_url

# Navigate back/forward
driver.back()
driver.forward()

# Refresh page
driver.refresh()

# Get page title
title = driver.title

# Get page source
page_source = driver.page_source

# Close current tab
driver.close()

# Quit driver (closes all tabs and browser)
driver.quit()
```

### Window and Tab Management
```python
# Get current window handle
current_window = driver.current_window_handle

# Get all window handles
all_windows = driver.window_handles

# Open new tab
driver.execute_script("window.open('https://example.com','_blank');")

# Switch between windows/tabs
for window in driver.window_handles:
    driver.switch_to.window(window)
    if 'desired_title' in driver.title:
        break

# Switch to specific tab by index
driver.switch_to.window(driver.window_handles[1])  # Switch to second tab

# Close current tab and switch back
driver.close()
driver.switch_to.window(driver.window_handles[0])

# Window size and position
driver.set_window_size(1920, 1080)
driver.set_window_position(0, 0)
driver.maximize_window()
driver.minimize_window()
driver.fullscreen_window()

# Get window size and position
size = driver.get_window_size()
position = driver.get_window_position()
```

### Frames and iframes
```python
# Switch to frame by name or id
driver.switch_to.frame('frame_name')
driver.switch_to.frame('frame_id')

# Switch to frame by index
driver.switch_to.frame(0)  # First frame

# Switch to frame by WebElement
frame_element = driver.find_element(By.TAG_NAME, 'iframe')
driver.switch_to.frame(frame_element)

# Switch back to main content
driver.switch_to.default_content()

# Switch to parent frame
driver.switch_to.parent_frame()
```

### Alert Handling
```python
from selenium.webdriver.support.ui import Alert

# Wait for alert and switch to it
wait = WebDriverWait(driver, 10)
alert = wait.until(EC.alert_is_present())

# Or directly get alert
alert = driver.switch_to.alert

# Get alert text
alert_text = alert.text

# Accept alert (click OK)
alert.accept()

# Dismiss alert (click Cancel/No)
alert.dismiss()

# Send text to prompt alert
alert.send_keys('Hello World')
alert.accept()

# Handle different types of alerts
def handle_alert(action='accept', text=None):
    try:
        alert = WebDriverWait(driver, 5).until(EC.alert_is_present())
        
        if text:
            alert.send_keys(text)
        
        if action == 'accept':
            alert.accept()
        elif action == 'dismiss':
            alert.dismiss()
        
        return True
    except:
        return False
```

## Advanced Selenium Techniques

### JavaScript Execution
```python
# Execute JavaScript
result = driver.execute_script("return document.title;")

# Execute JavaScript with arguments
element = driver.find_element(By.ID, 'element-id')
driver.execute_script("arguments[0].style.border = '3px solid red';", element)

# Scroll operations
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")  # Scroll to bottom
driver.execute_script("arguments[0].scrollIntoView();", element)  # Scroll to element

# Modify element properties
driver.execute_script("arguments[0].value = 'new value';", input_element)
driver.execute_script("arguments[0].checked = true;", checkbox_element)

# Get element properties
text_content = driver.execute_script("return arguments[0].textContent;", element)
inner_html = driver.execute_script("return arguments[0].innerHTML;", element)

# Wait for page load
driver.execute_script("return document.readyState") == "complete"

# Execute async JavaScript
def execute_async_script_example():
    script = """
    var callback = arguments[arguments.length - 1];
    setTimeout(function() {
        callback('Async operation completed');
    }, 2000);
    """
    result = driver.execute_async_script(script)
    return result

# Custom JavaScript utilities
js_utils = """
window.seleniumUtils = {
    waitForElement: function(selector, timeout = 5000) {
        return new Promise((resolve, reject) => {
            const startTime = Date.now();
            const check = () => {
                const element = document.querySelector(selector);
                if (element) {
                    resolve(element);
                } else if (Date.now() - startTime >= timeout) {
                    reject(new Error('Element not found within timeout'));
                } else {
                    setTimeout(check, 100);
                }
            };
            check();
        });
    },
    
    simulateHumanTyping: function(element, text, delay = 100) {
        return new Promise((resolve) => {
            let i = 0;
            const type = () => {
                if (i < text.length) {
                    element.value += text.charAt(i);
                    element.dispatchEvent(new Event('input', { bubbles: true }));
                    i++;
                    setTimeout(type, delay + Math.random() * 50);
                } else {
                    resolve();
                }
            };
            type();
        });
    }
};
"""

driver.execute_script(js_utils)

# Use custom utilities
driver.execute_async_script("""
    var callback = arguments[arguments.length - 1];
    window.seleniumUtils.waitForElement('.dynamic-content')
        .then(() => callback('Element found'))
        .catch(() => callback('Element not found'));
""")
```

### Screenshot and Visual Testing
```python
import os
from datetime import datetime

# Take full page screenshot
driver.save_screenshot('full_page.png')

# Take element screenshot
element = driver.find_element(By.ID, 'content')
element.screenshot('element.png')

# Get screenshot as base64 (useful for embedding)
screenshot_base64 = driver.get_screenshot_as_base64()

# Get screenshot as binary data
screenshot_binary = driver.get_screenshot_as_png()

# Advanced screenshot utility
class ScreenshotManager:
    def __init__(self, driver, base_path='screenshots'):
        self.driver = driver
        self.base_path = base_path
        os.makedirs(base_path, exist_ok=True)
    
    def take_screenshot(self, name=None, element=None):
        """Take screenshot with timestamp"""
        if not name:
            name = f"screenshot_{datetime.now().strftime('%Y%m%d_%H%M%S')}"
        
        filepath = os.path.join(self.base_path, f"{name}.png")
        
        if element:
            element.screenshot(filepath)
        else:
            self.driver.save_screenshot(filepath)
        
        return filepath
    
    def take_full_page_screenshot(self, name=None):
        """Take full page screenshot by scrolling"""
        # Get page dimensions
        total_width = self.driver.execute_script("return document.body.scrollWidth")
        total_height = self.driver.execute_script("return document.body.scrollHeight")
        
        # Set window size to capture full page
        self.driver.set_window_size(total_width, total_height)
        
        # Take screenshot
        return self.take_screenshot(name)
    
    def compare_screenshots(self, screenshot1, screenshot2):
        """Basic screenshot comparison"""
        from PIL import Image, ImageChops
        
        img1 = Image.open(screenshot1)
        img2 = Image.open(screenshot2)
        
        diff = ImageChops.difference(img1, img2)
        
        if diff.getbbox():
            return False  # Images are different
        else:
            return True   # Images are the same

# Usage
screenshot_manager = ScreenshotManager(driver)
screenshot_manager.take_screenshot('login_page')
screenshot_manager.take_full_page_screenshot('full_page')
```

### Performance Monitoring
```python
import json
import time

class PerformanceMonitor:
    def __init__(self, driver):
        self.driver = driver
    
    def get_performance_metrics(self):
        """Get browser performance metrics"""
        # Get navigation timing
        nav_timing = self.driver.execute_script("""
            return performance.getEntriesByType('navigation')[0];
        """)
        
        # Get resource timing
        resource_timing = self.driver.execute_script("""
            return performance.getEntriesByType('resource');
        """)
        
        # Get memory info (Chrome only)
        try:
            memory_info = self.driver.execute_script("""
                return performance.memory;
            """)
        except:
            memory_info = None
        
        return {
            'navigation': nav_timing,
            'resources': resource_timing,
            'memory': memory_info
        }
    
    def get_page_load_time(self):
        """Get page load time"""
        return self.driver.execute_script("""
            return performance.timing.loadEventEnd - performance.timing.navigationStart;
        """)
    
    def get_dom_content_loaded_time(self):
        """Get DOM content loaded time"""
        return self.driver.execute_script("""
            return performance.timing.domContentLoadedEventEnd - performance.timing.navigationStart;
        """)
    
    def measure_element_render_time(self, locator):
        """Measure time for element to appear"""
        start_time = time.time()
        
        try:
            wait = WebDriverWait(self.driver, 30)
            wait.until(EC.presence_of_element_located(locator))
            end_time = time.time()
            return (end_time - start_time) * 1000  # Convert to milliseconds
        except:
            return None
    
    def get_network_logs(self):
        """Get network logs (Chrome only with logging enabled)"""
        try:
            logs = self.driver.get_log('performance')
            network_logs = []
            
            for log in logs:
                message = json.loads(log['message'])
                if message['message']['method'].startswith('Network.'):
                    network_logs.append(message)
            
            return network_logs
        except:
            return []

# Enable performance logging for Chrome
chrome_options = Options()
chrome_options.add_argument('--enable-logging')
chrome_options.add_argument('--log-level=0')
chrome_options.set_capability('goog:loggingPrefs', {'performance': 'ALL'})

driver = webdriver.Chrome(options=chrome_options)
performance_monitor = PerformanceMonitor(driver)

# Use performance monitoring
driver.get('https://example.com')
metrics = performance_monitor.get_performance_metrics()
load_time = performance_monitor.get_page_load_time()
```

### Cookie and Session Management
```python
# Get all cookies
cookies = driver.get_cookies()

# Get specific cookie
cookie = driver.get_cookie('session_id')

# Add cookie
driver.add_cookie({
    'name': 'test_cookie',
    'value': 'test_value',
    'domain': 'example.com',
    'path': '/',
    'secure': True,
    'httpOnly': False
})

# Delete specific cookie
driver.delete_cookie('cookie_name')

# Delete all cookies
driver.delete_all_cookies()

# Session management utility
class SessionManager:
    def __init__(self, driver):
        self.driver = driver
        self.session_file = 'session_cookies.json'
    
    def save_session(self):
        """Save current session cookies to file"""
        cookies = self.driver.get_cookies()
        with open(self.session_file, 'w') as f:
            json.dump(cookies, f)
    
    def load_session(self):
        """Load session cookies from file"""
        try:
            with open(self.session_file, 'r') as f:
                cookies = json.load(f)
            
            for cookie in cookies:
                self.driver.add_cookie(cookie)
            
            return True
        except FileNotFoundError:
            return False
    
    def clear_session(self):
        """Clear all cookies and session file"""
        self.driver.delete_all_cookies()
        if os.path.exists(self.session_file):
            os.remove(self.session_file)

# Usage
session_manager = SessionManager(driver)

# Login and save session
driver.get('https://example.com/login')
# ... perform login ...
session_manager.save_session()

# Later, load session
driver.get('https://example.com')
session_manager.load_session()
driver.refresh()  # Refresh to apply cookies
```

## Error Handling and Debugging

### Exception Handling
```python
from selenium.common.exceptions import (
    NoSuchElementException,
    TimeoutException,
    ElementNotInteractableException,
    StaleElementReferenceException,
    WebDriverException
)

def safe_find_element(driver, by, value, timeout=10):
    """Safely find element with error handling"""
    try:
        wait = WebDriverWait(driver, timeout)
        element = wait.until(EC.presence_of_element_located((by, value)))
        return element
    except TimeoutException:
        print(f"Element not found: {by}={value}")
        return None

def safe_click(element, driver=None):
    """Safely click element with multiple strategies"""
    try:
        # Try regular click
        element.click()
        return True
    except ElementNotInteractableException:
        # Try JavaScript click
        if driver:
            driver.execute_script("arguments[0].click();", element)
            return True
    except StaleElementReferenceException:
        print("Element is stale, need to re-find it")
        return False
    except Exception as e:
        print(f"Click failed: {e}")
        return False

def safe_send_keys(element, text, clear_first=True):
    """Safely send keys to element"""
    try:
        if clear_first:
            element.clear()
        element.send_keys(text)
        return True
    except StaleElementReferenceException:
        print("Element is stale, need to re-find it")
        return False
    except Exception as e:
        print(f"Send keys failed: {e}")
        return False

# Robust element interaction
class RobustElementHandler:
    def __init__(self, driver, max_retries=3):
        self.driver = driver
        self.max_retries = max_retries
        self.wait = WebDriverWait(driver, 10)
    
    def find_and_click(self, locator, timeout=10):
        """Find element and click with retries"""
        for attempt in range(self.max_retries):
            try:
                element = self.wait.until(EC.element_to_be_clickable(locator))
                element.click()
                return True
            except (StaleElementReferenceException, ElementNotInteractableException):
                if attempt < self.max_retries - 1:
                    time.sleep(1)
                    continue
                else:
                    # Try JavaScript click as last resort
                    try:
                        element = self.driver.find_element(*locator)
                        self.driver.execute_script("arguments[0].click();", element)
                        return True
                    except:
                        return False
            except Exception as e:
                print(f"Click attempt {attempt + 1} failed: {e}")
                if attempt == self.max_retries - 1:
                    return False
                time.sleep(1)
        return False
    
    def find_and_send_keys(self, locator, text, clear_first=True):
        """Find element and send keys with retries"""
        for attempt in range(self.max_retries):
            try:
                element = self.wait.until(EC.presence_of_element_located(locator))
                if clear_first:
                    element.clear()
                element.send_keys(text)
                return True
            except StaleElementReferenceException:
                if attempt < self.max_retries - 1:
                    time.sleep(1)
                    continue
            except Exception as e:
                print(f"Send keys attempt {attempt + 1} failed: {e}")
                if attempt == self.max_retries - 1:
                    return False
                time.sleep(1)
        return False
    
    def get_element_text(self, locator):
        """Get element text with error handling"""
        try:
            element = self.wait.until(EC.presence_of_element_located(locator))
            return element.text
        except Exception as e:
            print(f"Failed to get element text: {e}")
            return None

# Usage
handler = RobustElementHandler(driver)
success = handler.find_and_click((By.ID, 'submit-button'))
text = handler.get_element_text((By.CLASS_NAME, 'result'))
```

### Debugging Utilities
```python
import logging
from selenium.webdriver.remote.remote_connection import LOGGER

# Configure Selenium logging
LOGGER.setLevel(logging.WARNING)

# Custom logger
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class SeleniumDebugger:
    def __init__(self, driver):
        self.driver = driver
        self.screenshot_counter = 0
    
    def debug_screenshot(self, description=""):
        """Take debug screenshot"""
        self.screenshot_counter += 1
        filename = f"debug_{self.screenshot_counter:03d}_{description}.png"
        self.driver.save_screenshot(filename)
        logger.info(f"Debug screenshot saved: {filename}")
        return filename
    
    def log_element_info(self, element):
        """Log detailed element information"""
        try:
            info = {
                'tag_name': element.tag_name,
                'text': element.text,
                'location': element.location,
                'size': element.size,
                'displayed': element.is_displayed(),
                'enabled': element.is_enabled(),
                'attributes': {}
            }
            
            # Get common attributes
            common_attrs = ['id', 'class', 'name', 'type', 'value', 'href', 'src']
            for attr in common_attrs:
                value = element.get_attribute(attr)
                if value:
                    info['attributes'][attr] = value
            
            logger.info(f"Element info: {info}")
            return info
        except Exception as e:
            logger.error(f"Failed to get element info: {e}")
            return None
    
    def log_page_info(self):
        """Log current page information"""
        info = {
            'url': self.driver.current_url,
            'title': self.driver.title,
            'window_handles': len(self.driver.window_handles),
            'page_source_length': len(self.driver.page_source)
        }
        logger.info(f"Page info: {info}")
        return info
    
    def highlight_element(self, element, color='red', duration=2):
        """Highlight element for debugging"""
        original_style = element.get_attribute('style')
        
        self.driver.execute_script(
            f"arguments[0].style.border='3px solid {color}';",
            element
        )
        
        time.sleep(duration)
        
        # Restore original style
        self.driver.execute_script(
            f"arguments[0].style.border='{original_style}';",
            element
        )
    
    def wait_for_user_input(self, message="Press Enter to continue..."):
        """Pause execution for debugging"""
        input(f"DEBUG: {message}")

# Usage
debugger = SeleniumDebugger(driver)

# Take debug screenshot
debugger.debug_screenshot("before_login")

# Log element info
element = driver.find_element(By.ID, 'username')
debugger.log_element_info(element)

# Highlight element
debugger.highlight_element(element)

# Pause for manual inspection
debugger.wait_for_user_input("Check the page state")
```

## Web Scraping with Selenium

### Complete Scraping Example
```python
class SeleniumScraper:
    def __init__(self, headless=True, implicit_wait=10):
        self.driver = None
        self.wait = None
        self.setup_driver(headless)
        
    def setup_driver(self, headless):
        """Setup Chrome driver with optimal settings"""
        options = Options()
        
        if headless:
            options.add_argument('--headless')
        
        # Performance optimizations
        options.add_argument('--no-sandbox')
        options.add_argument('--disable-dev-shm-usage')
        options.add_argument('--disable-gpu')
        options.add_argument('--disable-features=TranslateUI')
        options.add_argument('--disable-extensions')
        options.add_argument('--disable-plugins')
        options.add_argument('--disable-images')
        options.add_argument('--disable-javascript')  # Only if JS not needed
        
        # User agent
        options.add_argument('--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36')
        
        self.driver = webdriver.Chrome(
            service=Service(ChromeDriverManager().install()),
            options=options
        )
        
        self.driver.implicitly_wait(10)
        self.wait = WebDriverWait(self.driver, 10)
    
    def scrape_ecommerce_site(self, base_url, max_pages=5):
        """Scrape e-commerce product listings"""
        all_products = []
        current_page = 1
        
        try:
            self.driver.get(base_url)
            
            while current_page <= max_pages:
                print(f"Scraping page {current_page}")
                
                # Wait for products to load
                self.wait.until(EC.presence_of_element_located((By.CLASS_NAME, 'product-item')))
                
                # Scroll to load all products (infinite scroll handling)
                self.scroll_to_bottom()
                
                # Extract products from current page
                products = self.extract_products()
                all_products.extend(products)
                
                # Try to go to next page
                if not self.go_to_next_page():
                    break
                
                current_page += 1
                time.sleep(2)  # Polite delay
            
        except Exception as e:
            print(f"Scraping error: {e}")
        
        return all_products
    
    def extract_products(self):
        """Extract product information from current page"""
        products = []
        product_elements = self.driver.find_elements(By.CLASS_NAME, 'product-item')
        
        for element in product_elements:
            try:
                product = self.extract_single_product(element)
                if product:
                    products.append(product)
            except Exception as e:
                print(f"Error extracting product: {e}")
        
        return products
    
    def extract_single_product(self, element):
        """Extract single product data"""
        try:
            # Extract basic information
            name_elem = element.find_element(By.CLASS_NAME, 'product-name')
            price_elem = element.find_element(By.CLASS_NAME, 'product-price')
            
            # Extract optional information
            try:
                image_elem = element.find_element(By.TAG_NAME, 'img')
                image_url = image_elem.get_attribute('src')
            except:
                image_url = None
            
            try:
                rating_elem = element.find_element(By.CLASS_NAME, 'rating')
                rating = rating_elem.get_attribute('data-rating')
            except:
                rating = None
            
            try:
                link_elem = element.find_element(By.TAG_NAME, 'a')
                product_url = link_elem.get_attribute('href')
            except:
                product_url = None
            
            return {
                'name': name_elem.text.strip(),
                'price': self.clean_price(price_elem.text),
                'image_url': image_url,
                'rating': rating,
                'product_url': product_url,
                'scraped_at': datetime.now().isoformat()
            }
            
        except Exception as e:
            print(f"Error extracting product data: {e}")
            return None
    
    def clean_price(self, price_text):
        """Clean price text and extract numeric value"""
        if not price_text:
            return None
        
        import re
        price_match = re.search(r'[\d,]+\.?\d*', price_text.replace(',', ''))
        return float(price_match.group()) if price_match else None
    
    def scroll_to_bottom(self):
        """Scroll to bottom of page to load all content"""
        last_height = self.driver.execute_script("return document.body.scrollHeight")
        
        while True:
            # Scroll down to bottom
            self.driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
            
            # Wait for new content to load
            time.sleep(2)
            
            # Calculate new scroll height
            new_height = self.driver.execute_script("return document.body.scrollHeight")
            
            if new_height == last_height:
                break
            
            last_height = new_height
    
    def go_to_next_page(self):
        """Navigate to next page"""
        try:
            next_button = self.driver.find_element(By.CSS_SELECTOR, 'a.next, .pagination .next')
            
            if next_button.is_enabled():
                next_button.click()
                return True
            else:
                return False
                
        except NoSuchElementException:
            return False
    
    def scrape_product_details(self, product_urls):
        """Scrape detailed product information"""
        detailed_products = []
        
        for url in product_urls:
            try:
                self.driver.get(url)
                
                # Wait for page to load
                self.wait.until(EC.presence_of_element_located((By.CLASS_NAME, 'product-details')))
                
                # Extract detailed information
                details = {
                    'url': url,
                    'name': self.safe_get_text((By.CLASS_NAME, 'product-title')),
                    'price': self.safe_get_text((By.CLASS_NAME, 'price')),
                    'description': self.safe_get_text((By.CLASS_NAME, 'description')),
                    'specifications': self.extract_specifications(),
                    'images': self.extract_image_urls(),
                    'reviews': self.extract_reviews()
                }
                
                detailed_products.append(details)
                time.sleep(1)  # Polite delay
                
            except Exception as e:
                print(f"Error scraping product details for {url}: {e}")
        
        return detailed_products
    
    def safe_get_text(self, locator):
        """Safely get text from element"""
        try:
            element = self.driver.find_element(*locator)
            return element.text.strip()
        except:
            return None
    
    def extract_specifications(self):
        """Extract product specifications"""
        specs = {}
        try:
            spec_rows = self.driver.find_elements(By.CSS_SELECTOR, '.specifications tr')
            
            for row in spec_rows:
                cells = row.find_elements(By.TAG_NAME, 'td')
                if len(cells) >= 2:
                    key = cells[0].text.strip()
                    value = cells[1].text.strip()
                    specs[key] = value
        except:
            pass
        
        return specs
    
    def extract_image_urls(self):
        """Extract product image URLs"""
        image_urls = []
        try:
            img_elements = self.driver.find_elements(By.CSS_SELECTOR, '.product-gallery img')
            image_urls = [img.get_attribute('src') for img in img_elements if img.get_attribute('src')]
        except:
            pass
        
        return image_urls
    
    def extract_reviews(self):
        """Extract product reviews"""
        reviews = []
        try:
            review_elements = self.driver.find_elements(By.CLASS_NAME, 'review-item')
            
            for review_elem in review_elements[:5]:  # Limit to first 5 reviews
                review = {
                    'author': self.safe_get_text_from_element(review_elem, By.CLASS_NAME, 'review-author'),
                    'rating': self.safe_get_attribute_from_element(
                        review_elem, By.CLASS_NAME, 'review-rating', 'data-rating'
                    ),
                    'content': self.safe_get_text_from_element(review_elem, By.CLASS_NAME, 'review-content')
                }
                reviews.append(review)
        except:
            pass
        
        return reviews
    
    def safe_get_text_from_element(self, parent_element, by, value):
        """Safely get text from child element"""
        try:
            element = parent_element.find_element(by, value)
            return element.text.strip()
        except:
            return None
    
    def safe_get_attribute_from_element(self, parent_element, by, value, attribute):
        """Safely get attribute from child element"""
        try:
            element = parent_element.find_element(by, value)
            return element.get_attribute(attribute)
        except:
            return None
    
    def close(self):
        """Close the driver"""
        if self.driver:
            self.driver.quit()
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()

# Usage
with SeleniumScraper(headless=True) as scraper:
    # Scrape product listings
    products = scraper.scrape_ecommerce_site('https://example-shop.com', max_pages=3)
    
    # Extract product URLs for detailed scraping
    product_urls = [p['product_url'] for p in products if p.get('product_url')]
    
    # Scrape detailed product information
    detailed_products = scraper.scrape_product_details(product_urls[:10])  # First 10 products
    
    # Save results
    import json
    with open('scraped_products.json', 'w') as f:
        json.dump(detailed_products, f, indent=2)
```