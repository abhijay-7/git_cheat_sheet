# CSS Selectors & XPath Cheatsheet

## CSS Selectors

### Basic Selectors

| Selector | Description | Example | Matches |
|----------|-------------|---------|---------|
| `*` | Universal selector | `*` | All elements |
| `element` | Type selector | `div` | All `<div>` elements |
| `.class` | Class selector | `.button` | Elements with class="button" |
| `#id` | ID selector | `#header` | Element with id="header" |
| `[attribute]` | Attribute selector | `[disabled]` | Elements with disabled attribute |

### Attribute Selectors (Advanced)

| Selector | Description | Example | Matches |
|----------|-------------|---------|---------|
| `[attr=value]` | Exact match | `[type="text"]` | Elements where type exactly equals "text" |
| `[attr~=value]` | Word match | `[class~="active"]` | Elements with "active" as complete word in class |
| `[attr\|=value]` | Prefix match (hyphen) | `[lang\|="en"]` | Elements with lang="en" or lang="en-US" |
| `[attr^=value]` | Starts with | `[href^="https"]` | Elements where href starts with "https" |
| `[attr$=value]` | Ends with | `[src$=".jpg"]` | Elements where src ends with ".jpg" |
| `[attr*=value]` | Contains | `[title*="hello"]` | Elements where title contains "hello" |
| `[attr operator value i]` | Case insensitive | `[type="TEXT" i]` | Matches regardless of case |

### Combinators

| Combinator | Description | Example | Matches |
|------------|-------------|---------|---------|
| `A B` | Descendant | `div p` | All `<p>` inside `<div>` (any level) |
| `A > B` | Direct child | `ul > li` | `<li>` that are direct children of `<ul>` |
| `A + B` | Adjacent sibling | `h1 + p` | First `<p>` immediately after `<h1>` |
| `A ~ B` | General sibling | `h1 ~ p` | All `<p>` siblings after `<h1>` |

### Pseudo-classes

#### Structural Pseudo-classes

| Pseudo-class | Description | Example |
|--------------|-------------|---------|
| `:first-child` | First child of parent | `li:first-child` |
| `:last-child` | Last child of parent | `li:last-child` |
| `:nth-child(n)` | Nth child | `li:nth-child(3)` |
| `:nth-child(odd)` | Odd children | `tr:nth-child(odd)` |
| `:nth-child(even)` | Even children | `tr:nth-child(even)` |
| `:nth-child(an+b)` | Formula-based | `li:nth-child(3n+2)` |
| `:nth-last-child(n)` | Nth from end | `li:nth-last-child(2)` |
| `:first-of-type` | First of element type | `p:first-of-type` |
| `:last-of-type` | Last of element type | `p:last-of-type` |
| `:nth-of-type(n)` | Nth of element type | `p:nth-of-type(2)` |
| `:only-child` | Only child of parent | `span:only-child` |
| `:only-of-type` | Only one of its type | `p:only-of-type` |
| `:empty` | No children (including text) | `div:empty` |

#### State Pseudo-classes

| Pseudo-class | Description | Example |
|--------------|-------------|---------|
| `:hover` | Mouse over | `a:hover` |
| `:active` | Being activated | `button:active` |
| `:focus` | Has focus | `input:focus` |
| `:checked` | Checked input | `input:checked` |
| `:disabled` | Disabled element | `button:disabled` |
| `:enabled` | Enabled element | `input:enabled` |
| `:required` | Required input | `input:required` |
| `:optional` | Optional input | `input:optional` |
| `:valid` | Valid input | `input:valid` |
| `:invalid` | Invalid input | `input:invalid` |
| `:read-only` | Read-only input | `input:read-only` |
| `:read-write` | Editable input | `input:read-write` |

#### Other Pseudo-classes

| Pseudo-class | Description | Example |
|--------------|-------------|---------|
| `:not(selector)` | Negation | `div:not(.active)` |
| `:is(selector-list)` | Matches any in list | `:is(h1, h2, h3)` |
| `:where(selector-list)` | Same as :is but 0 specificity | `:where(h1, h2)` |
| `:has(selector)` | Parent selector | `div:has(> p)` |
| `:root` | Root element (html) | `:root` |
| `:target` | Targeted by URL fragment | `div:target` |

### Pseudo-elements

| Pseudo-element | Description | Example |
|----------------|-------------|---------|
| `::before` | Insert before content | `p::before` |
| `::after` | Insert after content | `p::after` |
| `::first-letter` | First letter | `p::first-letter` |
| `::first-line` | First line | `p::first-line` |
| `::selection` | Selected text | `::selection` |
| `::placeholder` | Placeholder text | `input::placeholder` |

### Complex Selector Examples

```css
/* Multiple classes */
.button.primary.large

/* Class and attribute */
a.external[target="_blank"]

/* Descendant with multiple conditions */
form input[type="text"]:not(:disabled)

/* Direct child with pseudo-class */
ul > li:nth-child(2n+1)

/* Adjacent sibling with class */
h2 + p.intro

/* Complex combination */
section.content > div:not(.sidebar) p:first-of-type
```

---

## XPath

### Basic Syntax

| Expression | Description | Example |
|------------|-------------|---------|
| `/` | Root node | `/html` |
| `//` | Any descendant | `//div` |
| `.` | Current node | `.` |
| `..` | Parent node | `..` |
| `@` | Attribute | `@class` |
| `*` | Any element | `//*` |

### Node Selection

| Expression | Description | Example |
|------------|-------------|---------|
| `nodename` | Select node | `//div` |
| `/nodename` | Child of root | `/html/body` |
| `//nodename` | All descendants | `//p` |
| `.` | Current node | `.//div` |
| `..` | Parent | `//div/..` |
| `//@attr` | All attributes | `//@href` |

### Predicates (Filters)

| Predicate | Description | Example |
|-----------|-------------|---------|
| `[n]` | Nth element | `//div[1]` (first div) |
| `[last()]` | Last element | `//div[last()]` |
| `[position()<3]` | First two | `//div[position()<3]` |
| `[@attr]` | Has attribute | `//div[@id]` |
| `[@attr='value']` | Attribute equals | `//div[@class='active']` |
| `[node]` | Has child node | `//div[p]` |

### Attribute Operations

| Expression | Description | Example |
|------------|-------------|---------|
| `@attr='value'` | Exact match | `//*[@type='text']` |
| `@attr!='value'` | Not equal | `//*[@type!='hidden']` |
| `contains(@attr,'val')` | Contains | `//*[contains(@class,'btn')]` |
| `starts-with(@attr,'val')` | Starts with | `//*[starts-with(@id,'user')]` |
| `ends-with(@attr,'val')` | Ends with (XPath 2.0+) | `//*[ends-with(@src,'.jpg')]` |
| `normalize-space(@attr)` | Trim whitespace | `//*[normalize-space(@class)='active']` |
| `string-length(@attr)` | Attribute length | `//*[string-length(@name)>5]` |

### Text Operations

| Expression | Description | Example |
|------------|-------------|---------|
| `text()` | Text content | `//p[text()='Hello']` |
| `contains(text(),'val')` | Text contains | `//p[contains(text(),'Hello')]` |
| `normalize-space(text())` | Trim text | `//p[normalize-space(text())='Hello']` |
| `string(.)` | String value | `//div[string(.)='Text']` |

### Axes

| Axis | Description | Example |
|------|-------------|---------|
| `ancestor::` | All ancestors | `//div/ancestor::section` |
| `ancestor-or-self::` | Ancestors + current | `//div/ancestor-or-self::*` |
| `parent::` | Parent node | `//div/parent::section` |
| `child::` | Direct children | `//div/child::p` |
| `descendant::` | All descendants | `//div/descendant::span` |
| `descendant-or-self::` | Descendants + current | `//div/descendant-or-self::*` |
| `following::` | After closing tag | `//h1/following::p` |
| `following-sibling::` | Following siblings | `//h1/following-sibling::p` |
| `preceding::` | Before opening tag | `//p/preceding::h1` |
| `preceding-sibling::` | Previous siblings | `//p/preceding-sibling::h1` |
| `self::` | Current node | `//div/self::div` |
| `attribute::` | Attributes | `//div/attribute::class` |

### Logical Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `and` | Logical AND | `//div[@class='active' and @id='main']` |
| `or` | Logical OR | `//div[@class='btn' or @class='button']` |
| `not()` | Negation | `//div[not(@class)]` |
| `\|` | Union | `//div \| //span` |

### Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `=` | Equal | `//div[@id='test']` |
| `!=` | Not equal | `//div[@type!='hidden']` |
| `<` | Less than | `//div[@data-count < 5]` |
| `<=` | Less or equal | `//div[@data-count <= 5]` |
| `>` | Greater than | `//div[@data-count > 5]` |
| `>=` | Greater or equal | `//div[@data-count >= 5]` |

### Functions

#### Node Functions

| Function | Description | Example |
|----------|-------------|---------|
| `count()` | Count nodes | `count(//div)` |
| `position()` | Current position | `//div[position()=2]` |
| `last()` | Last position | `//div[last()]` |
| `name()` | Node name | `//*[name()='div']` |
| `local-name()` | Local name | `//*[local-name()='div']` |

#### String Functions

| Function | Description | Example |
|----------|-------------|---------|
| `contains()` | Substring check | `//div[contains(@class,'btn')]` |
| `starts-with()` | Prefix check | `//a[starts-with(@href,'http')]` |
| `substring()` | Extract substring | `//div[substring(@id,1,4)='user']` |
| `substring-before()` | Before delimiter | `substring-before(@email,'@')` |
| `substring-after()` | After delimiter | `substring-after(@email,'@')` |
| `concat()` | Concatenate | `concat(@first,'-',@last)` |
| `translate()` | Replace chars | `translate(@text,'abc','ABC')` |
| `normalize-space()` | Trim whitespace | `normalize-space(text())` |
| `string-length()` | String length | `string-length(@name)>5` |

#### Boolean Functions

| Function | Description | Example |
|----------|-------------|---------|
| `not()` | Negation | `//div[not(@class)]` |
| `boolean()` | Convert to boolean | `boolean(@disabled)` |
| `true()` | True value | `//div[@enabled=true()]` |
| `false()` | False value | `//div[@enabled=false()]` |

### Complex XPath Examples

```xpath
# Multiple conditions with AND
//input[@type='text' and @name='username']

# Multiple conditions with OR
//button[@class='btn' or @class='button']

# Contains class (for multiple classes)
//*[contains(concat(' ', normalize-space(@class), ' '), ' active ')]

# Parent with specific child
//div[child::p[@class='intro']]

# Following sibling
//h1/following-sibling::p[1]

# Nth child of type
//ul/li[position()=3]

# Element with specific text
//button[normalize-space(text())='Submit']

# Contains text ignoring case (XPath 2.0)
//p[contains(lower-case(text()),'hello')]

# Multiple attribute conditions
//input[@type='text' and @required and not(@disabled)]

# Ancestor with condition
//span[ancestor::div[@class='container']]

# Select by index range
//li[position()>=2 and position()<=5]

# Has child matching condition
//div[child::*[@class='highlight']]

# Text contains and has attribute
//a[contains(text(),'Click') and @href]

# Complex navigation
//div[@id='main']//ul/li[last()]/a/@href
```

---

## CSS vs XPath Comparison

| Task | CSS Selector | XPath |
|------|-------------|-------|
| Select by ID | `#myid` | `//*[@id='myid']` |
| Select by class | `.myclass` | `//*[@class='myclass']` or `//*[contains(@class,'myclass')]` |
| Direct child | `div > p` | `//div/p` |
| Any descendant | `div p` | `//div//p` |
| Attribute exists | `[href]` | `//*[@href]` |
| Attribute equals | `[type="text"]` | `//*[@type='text']` |
| Contains text | N/A | `//*[contains(text(),'hello')]` |
| First child | `li:first-child` | `//li[1]` |
| Last child | `li:last-child` | `//li[last()]` |
| Nth child | `li:nth-child(3)` | `//li[3]` |
| Parent selection | `div:has(> p)` | `//div[p]/..` or `//p/parent::div` |
| Following sibling | `h1 ~ p` | `//h1/following-sibling::p` |
| Adjacent sibling | `h1 + p` | `//h1/following-sibling::p[1]` |
| Select by text | N/A | `//*[text()='Submit']` |
| Multiple conditions | `input[type="text"][required]` | `//input[@type='text' and @required]` |

---

## Best Practices

### CSS Selectors
- Use IDs for unique elements (fastest)
- Prefer classes over complex attribute selectors
- Keep selectors short and simple
- Use `:not()` to exclude elements
- Leverage `:has()` for parent selection (modern browsers)
- Use `>` for direct children when possible (more efficient)

### XPath
- Prefer relative paths over absolute (`//` over `/`)
- Use predicates to filter early in the path
- Avoid `//` at the end of expressions (slow)
- Use specific node names instead of `*` when possible
- Leverage axes only when necessary
- Cache compiled XPath expressions in code
- Use `contains()` carefully (can be slow on large documents)
- Prefer `@attribute` over `attribute::attribute`

### When to Use Which

**Use CSS Selectors when:**
- Working with modern browsers/tools
- Need faster performance
- Simple selection patterns
- Styling or JavaScript DOM manipulation

**Use XPath when:**
- Need to select by text content
- Navigate up the DOM (parent/ancestor)
- Complex conditional logic
- Working with XML documents
- Need precise positional queries
- Require string manipulation in queries

---

## Common Pitfalls

### CSS
- Cannot select parent elements (except with `:has()`)
- Cannot select by text content
- Limited text matching capabilities
- Pseudo-classes may not work in all contexts (e.g., Selenium)

### XPath
- Index starts at 1, not 0
- `//div[@class='btn']` matches exact class only (not partial)
- Performance can degrade with complex expressions
- Whitespace in class names requires `normalize-space()`
- Different XPath versions (1.0, 2.0, 3.0) have different functions

### Both
- Dynamic content may require waits/retries
- Shadow DOM requires special handling
- Overly specific selectors break easily
- Generated IDs/classes may change