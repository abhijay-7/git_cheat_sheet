# HTML Comprehensive In-Depth Cheat Sheet

## Document Structure

### Basic HTML5 Document
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Page Title</title>
    <meta name="description" content="Page description for SEO">
    <meta name="keywords" content="keyword1, keyword2, keyword3">
    <meta name="author" content="Your Name">
    <link rel="stylesheet" href="styles.css">
    <link rel="icon" type="image/x-icon" href="/favicon.ico">
</head>
<body>
    <!-- Page content goes here -->
    <script src="script.js"></script>
</body>
</html>
```

### Document Type Declarations
```html
<!-- HTML5 (current standard) -->
<!DOCTYPE html>

<!-- HTML 4.01 Strict (legacy) -->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
"http://www.w3.org/TR/html4/strict.dtd">

<!-- XHTML 1.0 Strict (legacy) -->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
```

### HTML Element
```html
<!-- Basic html element -->
<html lang="en">
<html lang="en-US">
<html lang="es">

<!-- With additional attributes -->
<html lang="en" dir="ltr" class="no-js">
<html lang="ar" dir="rtl">

<!-- With XML namespace (XHTML) -->
<html xmlns="http://www.w3.org/1999/xhtml" lang="en">
```

## Head Section Elements

### Meta Tags
```html
<!-- Essential meta tags -->
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="X-UA-Compatible" content="IE=edge">

<!-- SEO meta tags -->
<meta name="description" content="A concise description of the page content">
<meta name="keywords" content="keyword1, keyword2, keyword3">
<meta name="author" content="Author Name">
<meta name="robots" content="index, follow">
<meta name="googlebot" content="index, follow">
<meta name="revisit-after" content="7 days">

<!-- Open Graph meta tags (Facebook) -->
<meta property="og:title" content="Page Title">
<meta property="og:description" content="Page description">
<meta property="og:image" content="https://example.com/image.jpg">
<meta property="og:url" content="https://example.com/page">
<meta property="og:type" content="website">
<meta property="og:site_name" content="Site Name">
<meta property="og:locale" content="en_US">

<!-- Twitter Card meta tags -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:site" content="@twitterhandle">
<meta name="twitter:creator" content="@twitterhandle">
<meta name="twitter:title" content="Page Title">
<meta name="twitter:description" content="Page description">
<meta name="twitter:image" content="https://example.com/image.jpg">

<!-- Mobile app meta tags -->
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<meta name="apple-mobile-web-app-title" content="App Title">
<meta name="mobile-web-app-capable" content="yes">
<meta name="application-name" content="App Name">

<!-- Theme color -->
<meta name="theme-color" content="#000000">
<meta name="msapplication-TileColor" content="#000000">

<!-- Refresh and redirect -->
<meta http-equiv="refresh" content="30">
<meta http-equiv="refresh" content="5; url=https://example.com">

<!-- Content Security Policy -->
<meta http-equiv="Content-Security-Policy" content="default-src 'self'">

<!-- DNS prefetch -->
<meta http-equiv="x-dns-prefetch-control" content="on">
```

### Link Elements
```html
<!-- Stylesheets -->
<link rel="stylesheet" href="styles.css">
<link rel="stylesheet" href="print.css" media="print">
<link rel="stylesheet" href="mobile.css" media="screen and (max-width: 768px)">

<!-- External stylesheets -->
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Roboto">
<link rel="preconnect" href="https://fonts.gstatic.com">

<!-- Favicons -->
<link rel="icon" type="image/x-icon" href="/favicon.ico">
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
<link rel="manifest" href="/site.webmanifest">

<!-- Resource hints -->
<link rel="preload" href="critical.css" as="style">
<link rel="preload" href="hero-image.jpg" as="image">
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
<link rel="prefetch" href="next-page.html">
<link rel="preconnect" href="https://api.example.com">
<link rel="dns-prefetch" href="https://external-api.com">

<!-- Canonical URL -->
<link rel="canonical" href="https://example.com/canonical-page">

<!-- Alternate versions -->
<link rel="alternate" hreflang="es" href="https://example.com/es/page">
<link rel="alternate" hreflang="fr" href="https://example.com/fr/page">
<link rel="alternate" type="application/rss+xml" href="/feed.xml">

<!-- Navigation hints -->
<link rel="prev" href="https://example.com/page1">
<link rel="next" href="https://example.com/page3">
<link rel="up" href="https://example.com/category">

<!-- Author and license -->
<link rel="author" href="https://example.com/author">
<link rel="license" href="https://example.com/license">
```

### Script Elements
```html
<!-- Basic script inclusion -->
<script src="script.js"></script>

<!-- Script with attributes -->
<script src="script.js" defer></script>
<script src="script.js" async></script>
<script src="script.js" type="module"></script>
<script src="script.js" crossorigin="anonymous"></script>
<script src="script.js" integrity="sha384-..."></script>

<!-- Inline scripts -->
<script>
    console.log('Inline JavaScript');
</script>

<!-- Script with noscript fallback -->
<script>
    // JavaScript enabled content
</script>
<noscript>
    <p>JavaScript is required for this page to function properly.</p>
</noscript>

<!-- JSON-LD structured data -->
<script type="application/ld+json">
{
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": "Article Title",
    "author": "Author Name",
    "datePublished": "2024-01-01"
}
</script>

<!-- Module scripts -->
<script type="module">
    import { myFunction } from './module.js';
    myFunction();
</script>

<!-- Worker scripts -->
<script>
    if ('serviceWorker' in navigator) {
        navigator.serviceWorker.register('/sw.js');
    }
</script>
```

### Other Head Elements
```html
<!-- Title -->
<title>Page Title - Site Name</title>

<!-- Base URL -->
<base href="https://example.com/">
<base target="_blank">

<!-- Style block -->
<style>
    body { font-family: Arial, sans-serif; }
</style>

<!-- Template element -->
<template id="my-template">
    <div class="template-content">
        <h2></h2>
        <p></p>
    </div>
</template>
```

## Semantic HTML5 Elements

### Document Structure
```html
<!-- Main structural elements -->
<header>
    <nav>
        <ul>
            <li><a href="#home">Home</a></li>
            <li><a href="#about">About</a></li>
            <li><a href="#contact">Contact</a></li>
        </ul>
    </nav>
    <h1>Site Title</h1>
</header>

<main>
    <article>
        <header>
            <h1>Article Title</h1>
            <time datetime="2024-01-01">January 1, 2024</time>
        </header>
        
        <section>
            <h2>Section Title</h2>
            <p>Section content...</p>
        </section>
        
        <aside>
            <h3>Related Information</h3>
            <p>Sidebar content...</p>
        </aside>
        
        <footer>
            <p>Article footer information</p>
        </footer>
    </article>
</main>

<aside>
    <section>
        <h2>Site Sidebar</h2>
        <p>Additional content...</p>
    </section>
</aside>

<footer>
    <p>&copy; 2024 Company Name. All rights reserved.</p>
    <address>
        Contact us at <a href="mailto:info@example.com">info@example.com</a>
    </address>
</footer>
```

### Content Sectioning
```html
<!-- Navigation -->
<nav aria-label="Main navigation">
    <ul>
        <li><a href="/" aria-current="page">Home</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/services">Services</a></li>
    </ul>
</nav>

<nav aria-label="Breadcrumb">
    <ol>
        <li><a href="/">Home</a></li>
        <li><a href="/category">Category</a></li>
        <li aria-current="page">Current Page</li>
    </ol>
</nav>

<!-- Article structure -->
<article>
    <header>
        <h1>Article Title</h1>
        <p>By <span rel="author">Author Name</span></p>
        <time datetime="2024-01-01T10:00:00">January 1, 2024 at 10:00 AM</time>
    </header>
    
    <p>Article content...</p>
    
    <section>
        <h2>Comments</h2>
        <article>
            <h3>Comment by User</h3>
            <time datetime="2024-01-01T11:00:00">January 1, 2024 at 11:00 AM</time>
            <p>Comment content...</p>
        </article>
    </section>
</article>

<!-- Aside usage -->
<aside>
    <h2>Related Articles</h2>
    <ul>
        <li><a href="/article1">Related Article 1</a></li>
        <li><a href="/article2">Related Article 2</a></li>
    </ul>
</aside>

<!-- Details and Summary -->
<details>
    <summary>Click to expand</summary>
    <p>Hidden content that is revealed when summary is clicked.</p>
</details>

<details open>
    <summary>Already expanded</summary>
    <p>This content is visible by default.</p>
</details>
```

## Text Content Elements

### Headings and Paragraphs
```html
<!-- Heading hierarchy -->
<h1>Main Title (only one per page)</h1>
<h2>Major Section</h2>
<h3>Subsection</h3>
<h4>Sub-subsection</h4>
<h5>Minor Heading</h5>
<h6>Smallest Heading</h6>

<!-- Heading groups -->
<hgroup>
    <h1>Main Title</h1>
    <h2>Subtitle</h2>
</hgroup>

<!-- Paragraphs -->
<p>This is a regular paragraph with <strong>strong emphasis</strong> and <em>italic emphasis</em>.</p>

<p>This paragraph has a <br>line break in the middle.</p>

<!-- Preformatted text -->
<pre>
    This text preserves
    whitespace and line breaks
    exactly as written
</pre>

<!-- Code blocks -->
<pre><code>
function example() {
    console.log("Hello, World!");
}
</code></pre>

<!-- Blockquotes -->
<blockquote cite="https://example.com/quote-source">
    <p>This is a quoted passage from another source.</p>
    <footer>
        — <cite>Source Name</cite>
    </footer>
</blockquote>

<!-- Address -->
<address>
    <p>Contact Information:</p>
    <p>123 Main Street<br>
    City, State 12345<br>
    Phone: <a href="tel:+1234567890">(123) 456-7890</a><br>
    Email: <a href="mailto:contact@example.com">contact@example.com</a></p>
</address>
```

### Lists
```html
<!-- Unordered lists -->
<ul>
    <li>First item</li>
    <li>Second item
        <ul>
            <li>Nested item 1</li>
            <li>Nested item 2</li>
        </ul>
    </li>
    <li>Third item</li>
</ul>

<!-- Ordered lists -->
<ol>
    <li>First step</li>
    <li>Second step</li>
    <li>Third step</li>
</ol>

<!-- Ordered list with custom start -->
<ol start="5">
    <li>Fifth item</li>
    <li>Sixth item</li>
</ol>

<!-- Ordered list with different numbering -->
<ol type="A">
    <li>Item A</li>
    <li>Item B</li>
</ol>

<ol type="I">
    <li>Item I</li>
    <li>Item II</li>
</ol>

<!-- Description lists -->
<dl>
    <dt>HTML</dt>
    <dd>HyperText Markup Language</dd>
    
    <dt>CSS</dt>
    <dd>Cascading Style Sheets</dd>
    
    <dt>JavaScript</dt>
    <dd>Programming language for web development</dd>
    <dd>Runs in web browsers</dd>
</dl>
```

### Inline Text Elements
```html
<!-- Emphasis and importance -->
<p><em>Emphasized text</em> (typically italic)</p>
<p><strong>Important text</strong> (typically bold)</p>
<p><b>Bold text</b> (stylistic bold without importance)</p>
<p><i>Italic text</i> (stylistic italic without emphasis)</p>

<!-- Semantic inline elements -->
<p><mark>Highlighted text</mark> for reference</p>
<p><small>Small print or fine text</small></p>
<p><del>Deleted text</del> and <ins>inserted text</ins></p>
<p><s>Struck through text</s> (no longer accurate)</p>
<p><u>Underlined text</u> (avoid for non-links)</p>

<!-- Code and technical elements -->
<p>Use the <code>console.log()</code> function to output to console.</p>
<p>Press <kbd>Ctrl</kbd> + <kbd>C</kbd> to copy.</p>
<p>The program output was: <samp>Hello, World!</samp></p>
<p>Enter your <var>username</var> in the field below.</p>

<!-- Quotes and citations -->
<p>He said, <q cite="https://example.com">This is a short quote.</q></p>
<p>The book <cite>HTML5 Specification</cite> covers this topic.</p>

<!-- Abbreviations and definitions -->
<p><abbr title="HyperText Markup Language">HTML</abbr> is a markup language.</p>
<p><dfn title="Cascading Style Sheets">CSS</dfn> is used for styling.</p>

<!-- Time elements -->
<p>Published on <time datetime="2024-01-01">January 1, 2024</time></p>
<p>The event starts at <time datetime="2024-01-01T19:00">7:00 PM</time></p>
<p>Duration: <time datetime="PT2H30M">2 hours 30 minutes</time></p>

<!-- Ruby annotations (for East Asian typography) -->
<ruby>
    漢 <rt>kan</rt>
    字 <rt>ji</rt>
</ruby>

<!-- Bidirectional text -->
<p>This is <bdi>Arabic text: مرحبا</bdi> in a sentence.</p>
<p>Override direction: <bdo dir="rtl">This text is right-to-left</bdo></p>

<!-- Line break opportunities -->
<p>This is a very long URL: https://example.com/very/long/path/<wbr>with/break/opportunities</p>

<!-- Spans for styling -->
<p>This paragraph has <span class="highlight">highlighted text</span>.</p>
```

## Links and Navigation

### Link Types
```html
<!-- Basic links -->
<a href="https://example.com">External link</a>
<a href="/internal-page">Internal link</a>
<a href="../relative-path">Relative link</a>
<a href="#section1">Link to page section</a>

<!-- Links with additional attributes -->
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
    Open in new tab (secure)
</a>

<a href="https://example.com" rel="nofollow">
    Link with nofollow
</a>

<a href="document.pdf" download>Download PDF</a>
<a href="image.jpg" download="custom-filename.jpg">Download with custom name</a>

<!-- Email and telephone links -->
<a href="mailto:user@example.com">Send email</a>
<a href="mailto:user@example.com?subject=Hello&body=Message%20body">
    Email with subject and body
</a>
<a href="tel:+1234567890">Call phone number</a>
<a href="sms:+1234567890">Send SMS</a>

<!-- Special protocols -->
<a href="ftp://ftp.example.com/file.zip">FTP link</a>
<a href="skype:username?call">Skype call</a>
<a href="whatsapp://send?text=Hello">WhatsApp message</a>

<!-- Link relationships -->
<a href="https://example.com" rel="alternate">Alternate version</a>
<a href="https://example.com" rel="author">Author page</a>
<a href="https://example.com" rel="bookmark">Permalink</a>
<a href="https://example.com" rel="help">Help documentation</a>
<a href="https://example.com" rel="license">License information</a>
<a href="https://example.com" rel="next">Next page</a>
<a href="https://example.com" rel="prev">Previous page</a>
<a href="https://example.com" rel="search">Search page</a>
<a href="https://example.com" rel="tag">Related tag</a>

<!-- Links with media types -->
<a href="document.pdf" type="application/pdf">PDF Document</a>
<a href="archive.zip" type="application/zip">ZIP Archive</a>

<!-- Links with language -->
<a href="https://example.fr" hreflang="fr">French version</a>
<a href="https://example.es" hreflang="es">Spanish version</a>

<!-- Ping attribute for tracking -->
<a href="https://example.com" ping="https://analytics.example.com/track">
    Tracked link
</a>

<!-- Referrer policy -->
<a href="https://example.com" referrerpolicy="no-referrer">
    Link with no referrer
</a>
```

### Navigation Patterns
```html
<!-- Main navigation -->
<nav role="navigation" aria-label="Main">
    <ul>
        <li><a href="/" aria-current="page">Home</a></li>
        <li><a href="/about">About</a></li>
        <li>
            <a href="/products" aria-expanded="false">Products</a>
            <ul>
                <li><a href="/products/web">Web Development</a></li>
                <li><a href="/products/mobile">Mobile Apps</a></li>
            </ul>
        </li>
        <li><a href="/contact">Contact</a></li>
    </ul>
</nav>

<!-- Breadcrumb navigation -->
<nav aria-label="Breadcrumb">
    <ol itemscope itemtype="https://schema.org/BreadcrumbList">
        <li itemprop="itemListElement" itemscope itemtype="https://schema.org/ListItem">
            <a itemprop="item" href="/"><span itemprop="name">Home</span></a>
            <meta itemprop="position" content="1" />
        </li>
        <li itemprop="itemListElement" itemscope itemtype="https://schema.org/ListItem">
            <a itemprop="item" href="/category"><span itemprop="name">Category</span></a>
            <meta itemprop="position" content="2" />
        </li>
        <li itemprop="itemListElement" itemscope itemtype="https://schema.org/ListItem">
            <span itemprop="name" aria-current="page">Current Page</span>
            <meta itemprop="position" content="3" />
        </li>
    </ol>
</nav>

<!-- Pagination -->
<nav aria-label="Pagination">
    <ul>
        <li><a href="/page/1" rel="first">First</a></li>
        <li><a href="/page/2" rel="prev">Previous</a></li>
        <li><a href="/page/2">2</a></li>
        <li><span aria-current="page">3</span></li>
        <li><a href="/page/4">4</a></li>
        <li><a href="/page/4" rel="next">Next</a></li>
        <li><a href="/page/10" rel="last">Last</a></li>
    </ul>
</nav>

<!-- Skip links for accessibility -->
<a href="#main-content" class="skip-link">Skip to main content</a>
<a href="#navigation" class="skip-link">Skip to navigation</a>
```

## Images and Media

### Image Elements
```html
<!-- Basic image -->
<img src="image.jpg" alt="Descriptive text for screen readers">

<!-- Image with additional attributes -->
<img src="image.jpg" 
     alt="Description" 
     title="Tooltip text"
     width="300" 
     height="200"
     loading="lazy"
     decoding="async">

<!-- Responsive images with srcset -->
<img src="image-400.jpg"
     srcset="image-400.jpg 400w,
             image-800.jpg 800w,
             image-1200.jpg 1200w"
     sizes="(max-width: 600px) 400px,
            (max-width: 900px) 800px,
            1200px"
     alt="Responsive image">

<!-- High DPI images -->
<img src="image.jpg"
     srcset="image.jpg 1x,
             image@2x.jpg 2x,
             image@3x.jpg 3x"
     alt="High DPI image">

<!-- Picture element for art direction -->
<picture>
    <source media="(min-width: 800px)" srcset="desktop.jpg">
    <source media="(min-width: 400px)" srcset="tablet.jpg">
    <img src="mobile.jpg" alt="Responsive image with art direction">
</picture>

<!-- Picture with different formats -->
<picture>
    <source type="image/avif" srcset="image.avif">
    <source type="image/webp" srcset="image.webp">
    <source type="image/jpeg" srcset="image.jpg">
    <img src="image.jpg" alt="Image with format fallbacks">
</picture>

<!-- Image maps -->
<img src="image-map.jpg" alt="Interactive image map" usemap="#map1">
<map name="map1">
    <area shape="rect" coords="0,0,50,50" href="/area1" alt="Area 1">
    <area shape="circle" coords="100,100,25" href="/area2" alt="Area 2">
    <area shape="poly" coords="150,150,200,150,175,200" href="/area3" alt="Area 3">
</map>

<!-- Figure and figcaption -->
<figure>
    <img src="chart.jpg" alt="Sales data chart">
    <figcaption>
        Sales performance for Q1 2024 showing 25% growth
    </figcaption>
</figure>

<!-- Multiple images in a figure -->
<figure>
    <img src="before.jpg" alt="Before renovation">
    <img src="after.jpg" alt="After renovation">
    <figcaption>Kitchen renovation: before and after</figcaption>
</figure>
```

### Audio Elements
```html
<!-- Basic audio -->
<audio controls>
    <source src="audio.mp3" type="audio/mpeg">
    <source src="audio.ogg" type="audio/ogg">
    <source src="audio.wav" type="audio/wav">
    Your browser does not support the audio element.
</audio>

<!-- Audio with additional attributes -->
<audio controls 
       autoplay 
       loop 
       muted 
       preload="metadata">
    <source src="audio.mp3" type="audio/mpeg">
    <p>Fallback content for browsers without audio support.</p>
</audio>

<!-- Audio with tracks -->
<audio controls>
    <source src="podcast.mp3" type="audio/mpeg">
    <track kind="captions" 
           src="captions.vtt" 
           srclang="en" 
           label="English Captions">
    <track kind="descriptions" 
           src="descriptions.vtt" 
           srclang="en" 
           label="Audio Descriptions">
</audio>

<!-- Background audio (use with caution) -->
<audio autoplay loop>
    <source src="background.mp3" type="audio/mpeg">
</audio>
```

### Video Elements
```html
<!-- Basic video -->
<video controls width="640" height="360">
    <source src="video.mp4" type="video/mp4">
    <source src="video.webm" type="video/webm">
    <source src="video.ogv" type="video/ogg">
    Your browser does not support the video element.
</video>

<!-- Video with poster and attributes -->
<video controls 
       poster="thumbnail.jpg"
       width="640" 
       height="360"
       preload="metadata"
       playsinline>
    <source src="video.mp4" type="video/mp4">
    <p>Fallback content for browsers without video support.</p>
</video>

<!-- Video with subtitles and captions -->
<video controls>
    <source src="movie.mp4" type="video/mp4">
    <track kind="subtitles" 
           src="subtitles-en.vtt" 
           srclang="en" 
           label="English Subtitles"
           default>
    <track kind="subtitles" 
           src="subtitles-es.vtt" 
           srclang="es" 
           label="Spanish Subtitles">
    <track kind="captions" 
           src="captions.vtt" 
           srclang="en" 
           label="English Captions">
    <track kind="descriptions" 
           src="descriptions.vtt" 
           srclang="en" 
           label="Audio Descriptions">
    <track kind="chapters" 
           src="chapters.vtt" 
           srclang="en" 
           label="Chapters">
</video>

<!-- Responsive video -->
<div class="video-container">
    <video controls>
        <source src="video.mp4" type="video/mp4">
    </video>
</div>

<!-- Picture-in-Picture hint -->
<video controls 
       disablepictureinpicture="false">
    <source src="video.mp4" type="video/mp4">
</video>

<!-- Video with custom controls -->
<video id="custom-video" 
       poster="thumbnail.jpg">
    <source src="video.mp4" type="video/mp4">
</video>
<div class="custom-controls">
    <button onclick="document.getElementById('custom-video').play()">Play</button>
    <button onclick="document.getElementById('custom-video').pause()">Pause</button>
    <input type="range" min="0" max="100" value="0" id="progress">
</div>
```

### Embedded Content
```html
<!-- iframe for external content -->
<iframe src="https://example.com" 
        width="600" 
        height="400"
        title="External content"
        loading="lazy"
        sandbox="allow-scripts allow-same-origin">
    Fallback content for browsers that don't support iframes.
</iframe>

<!-- iframe with security attributes -->
<iframe src="https://trusted-site.com"
        sandbox="allow-scripts allow-same-origin allow-forms"
        referrerpolicy="strict-origin-when-cross-origin"
        csp="default-src 'self'"
        title="Secure embedded content">
</iframe>

<!-- Embed for plugins -->
<embed src="document.pdf" 
       type="application/pdf" 
       width="600" 
       height="400">

<!-- Object for multimedia -->
<object data="movie.mp4" 
        type="video/mp4" 
        width="640" 
        height="480">
    <param name="autoplay" value="false">
    <p>Your browser doesn't support embedded videos.</p>
</object>

<!-- Object for SVG -->
<object data="image.svg" 
        type="image/svg+xml">
    <img src="fallback.png" alt="Fallback image">
</object>
```

## Forms and Input

### Form Structure
```html
<!-- Basic form -->
<form action="/submit" method="post" enctype="application/x-www-form-urlencoded">
    <fieldset>
        <legend>Personal Information</legend>
        
        <label for="name">Name:</label>
        <input type="text" id="name" name="name" required>
        
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
        
        <button type="submit">Submit</button>
    </fieldset>
</form>


<!-- Form with different encoding -->
<form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="upload">
    <button type="submit">Upload File</button>
</form>

<!-- Form with GET method and target -->
<form action="/search" method="get" target="_blank" autocomplete="on">
    <input type="search" name="q" placeholder="Search...">
    <button type="submit">Search</button>
</form>

<!-- Form with validation attributes -->
<form novalidate>
    <input type="text" name="username" required minlength="3" maxlength="20">
    <input type="email" name="email" required>
    <button type="submit">Submit</button>
</form>

<!-- Form with custom validation -->
<form id="customForm">
    <input type="text" name="custom" pattern="[A-Za-z]{3,}" 
           title="Only letters, minimum 3 characters">
    <input type="submit" value="Submit">
</form>
```

### Input Types
```html
<!-- Text inputs -->
<input type="text" name="text" placeholder="Enter text" maxlength="100">
<input type="password" name="password" minlength="8" autocomplete="new-password">
<input type="email" name="email" multiple placeholder="email@example.com">
<input type="url" name="website" placeholder="https://example.com">
<input type="tel" name="phone" pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}">
<input type="search" name="search" results="5" autocomplete="off">

<!-- Number inputs -->
<input type="number" name="quantity" min="1" max="100" step="1" value="1">
<input type="range" name="volume" min="0" max="100" step="5" value="50">

<!-- Date and time inputs -->
<input type="date" name="birthdate" min="1900-01-01" max="2024-12-31">
<input type="time" name="appointment" min="09:00" max="17:00" step="900">
<input type="datetime-local" name="meeting" 
       min="2024-01-01T09:00" max="2024-12-31T17:00">
<input type="month" name="start-month" min="2024-01">
<input type="week" name="vacation-week" min="2024-W01">

<!-- Selection inputs -->
<input type="checkbox" id="terms" name="terms" value="accepted" required>
<label for="terms">I accept the terms and conditions</label>

<input type="radio" id="size-s" name="size" value="small">
<label for="size-s">Small</label>
<input type="radio" id="size-m" name="size" value="medium" checked>
<label for="size-m">Medium</label>
<input type="radio" id="size-l" name="size" value="large">
<label for="size-l">Large</label>

<!-- File inputs -->
<input type="file" name="single-file" accept=".pdf,.doc,.docx">
<input type="file" name="multiple-files" multiple 
       accept="image/jpeg,image/png,image/gif">
<input type="file" name="photos" accept="image/*" capture="camera">

<!-- Color input -->
<input type="color" name="theme-color" value="#ff0000">

<!-- Hidden input -->
<input type="hidden" name="csrf-token" value="abc123">

<!-- Button inputs -->
<input type="submit" value="Submit Form">
<input type="reset" value="Reset Form">
<input type="button" value="Custom Button" onclick="handleClick()">
<input type="image" src="submit-button.png" alt="Submit" width="100" height="30">
```

### Input Attributes
```html
<!-- Validation attributes -->
<input type="text" 
       required 
       minlength="3" 
       maxlength="20" 
       pattern="[A-Za-z0-9]+"
       title="Alphanumeric characters only">

<input type="number" 
       min="0" 
       max="100" 
       step="0.5">

<!-- Autocomplete attributes -->
<input type="text" name="first-name" autocomplete="given-name">
<input type="text" name="last-name" autocomplete="family-name">
<input type="email" name="email" autocomplete="email">
<input type="tel" name="phone" autocomplete="tel">
<input type="text" name="address" autocomplete="street-address">
<input type="text" name="city" autocomplete="address-level2">
<input type="text" name="country" autocomplete="country">
<input type="text" name="postal-code" autocomplete="postal-code">
<input type="text" name="cc-number" autocomplete="cc-number">
<input type="text" name="cc-exp" autocomplete="cc-exp">

<!-- State attributes -->
<input type="text" disabled>
<input type="text" readonly>
<input type="checkbox" checked>
<input type="radio" checked>

<!-- Behavior attributes -->
<input type="text" autofocus>
<input type="text" placeholder="Enter your name">
<input type="text" spellcheck="true">
<input type="password" autocomplete="off">

<!-- Data attributes -->
<input type="text" 
       data-validation="required" 
       data-min-length="3"
       data-custom-attr="value">

<!-- Form association -->
<input type="text" name="external-input" form="external-form">
<form id="external-form" action="/submit" method="post">
    <button type="submit">Submit</button>
</form>

<!-- List attribute for datalist -->
<input type="text" name="browser" list="browsers">
<datalist id="browsers">
    <option value="Chrome">
    <option value="Firefox">
    <option value="Safari">
    <option value="Edge">
</datalist>
```

### Textarea Element
```html
<!-- Basic textarea -->
<textarea name="message" rows="5" cols="40" placeholder="Enter your message"></textarea>

<!-- Textarea with attributes -->
<textarea name="description" 
          rows="10" 
          cols="50" 
          minlength="10" 
          maxlength="500" 
          required
          wrap="soft"
          spellcheck="true"
          autocomplete="off">Default content</textarea>

<!-- Textarea with resize control -->
<textarea name="resizable" style="resize: vertical;"></textarea>
<textarea name="fixed" style="resize: none;"></textarea>
```

### Select Elements
```html
<!-- Basic select -->
<select name="country">
    <option value="">Choose a country</option>
    <option value="us">United States</option>
    <option value="ca">Canada</option>
    <option value="uk" selected>United Kingdom</option>
    <option value="au">Australia</option>
</select>

<!-- Multiple selection -->
<select name="skills" multiple size="5">
    <option value="html">HTML</option>
    <option value="css">CSS</option>
    <option value="js" selected>JavaScript</option>
    <option value="py" selected>Python</option>
    <option value="java">Java</option>
</select>

<!-- Option groups -->
<select name="food-category">
    <optgroup label="Fruits">
        <option value="apple">Apple</option>
        <option value="banana">Banana</option>
        <option value="orange">Orange</option>
    </optgroup>
    <optgroup label="Vegetables">
        <option value="carrot">Carrot</option>
        <option value="broccoli">Broccoli</option>
        <option value="spinach">Spinach</option>
    </optgroup>
    <optgroup label="Proteins" disabled>
        <option value="chicken">Chicken</option>
        <option value="fish">Fish</option>
    </optgroup>
</select>

<!-- Select with data attributes -->
<select name="priority" data-default="medium">
    <option value="low" data-color="green">Low Priority</option>
    <option value="medium" data-color="yellow" selected>Medium Priority</option>
    <option value="high" data-color="red">High Priority</option>
</select>
```

### Labels and Fieldsets
```html
<!-- Explicit labels -->
<label for="username">Username:</label>
<input type="text" id="username" name="username">

<!-- Implicit labels -->
<label>
    Password:
    <input type="password" name="password">
</label>

<!-- Fieldsets for grouping -->
<fieldset>
    <legend>Personal Information</legend>
    <p>
        <label for="first-name">First Name:</label>
        <input type="text" id="first-name" name="first-name">
    </p>
    <p>
        <label for="last-name">Last Name:</label>
        <input type="text" id="last-name" name="last-name">
    </p>
</fieldset>

<fieldset disabled>
    <legend>Disabled Section</legend>
    <input type="text" name="disabled-input">
    <button type="button">Disabled Button</button>
</fieldset>

<!-- Complex form structure -->
<form>
    <fieldset>
        <legend>Account Details</legend>
        
        <div class="form-group">
            <label for="email">Email Address:</label>
            <input type="email" id="email" name="email" required 
                   aria-describedby="email-help">
            <small id="email-help">We'll never share your email.</small>
        </div>
        
        <div class="form-group">
            <label for="password">Password:</label>
            <input type="password" id="password" name="password" 
                   required minlength="8" aria-describedby="password-help">
            <small id="password-help">Minimum 8 characters.</small>
        </div>
    </fieldset>
    
    <fieldset>
        <legend>Preferences</legend>
        
        <div class="checkbox-group">
            <input type="checkbox" id="newsletter" name="preferences" value="newsletter">
            <label for="newsletter">Subscribe to newsletter</label>
        </div>
        
        <div class="radio-group">
            <p>Communication preference:</p>
            <input type="radio" id="email-pref" name="communication" value="email" checked>
            <label for="email-pref">Email</label>
            
            <input type="radio" id="phone-pref" name="communication" value="phone">
            <label for="phone-pref">Phone</label>
            
            <input type="radio" id="none-pref" name="communication" value="none">
            <label for="none-pref">No communication</label>
        </div>
    </fieldset>
</form>
```

### Buttons
```html
<!-- Button types -->
<button type="submit">Submit Form</button>
<button type="reset">Reset Form</button>
<button type="button" onclick="handleClick()">Custom Action</button>

<!-- Button with content -->
<button type="submit">
    <img src="icon.png" alt=""> Submit Application
</button>

<!-- Button attributes -->
<button type="submit" 
        disabled 
        form="external-form"
        formaction="/alternative-submit"
        formmethod="post"
        formenctype="multipart/form-data"
        formtarget="_blank"
        formnovalidate>
    Submit to Alternative
</button>

<!-- Button with accessibility -->
<button type="button" 
        aria-label="Close dialog"
        aria-expanded="false"
        aria-controls="dialog-content">
    ×
</button>

<!-- Button states -->
<button type="button" aria-pressed="false" class="toggle-button">
    Toggle Feature
</button>
```

### Form Validation
```html
<!-- HTML5 validation -->
<form>
    <input type="email" required title="Please enter a valid email">
    <input type="url" title="Please enter a valid URL">
    <input type="number" min="1" max="10" title="Number between 1 and 10">
    <input type="text" pattern="[A-Za-z]{3,}" title="At least 3 letters">
    
    <!-- Custom validation message -->
    <input type="text" 
           oninvalid="this.setCustomValidity('Please enter a valid value')"
           oninput="this.setCustomValidity('')">
    
    <button type="submit">Submit</button>
</form>

<!-- Validation styling with CSS -->
<style>
input:valid { border-color: green; }
input:invalid { border-color: red; }
input:required { background-color: #fff5f5; }
</style>
```

## Tables

### Basic Table Structure
```html
<!-- Simple table -->
<table>
    <tr>
        <th>Name</th>
        <th>Age</th>
        <th>City</th>
    </tr>
    <tr>
        <td>John Doe</td>
        <td>30</td>
        <td>New York</td>
    </tr>
    <tr>
        <td>Jane Smith</td>
        <td>25</td>
        <td>Los Angeles</td>
    </tr>
</table>

<!-- Table with caption -->
<table>
    <caption>Employee Information for Q1 2024</caption>
    <tr>
        <th>Employee ID</th>
        <th>Name</th>
        <th>Department</th>
        <th>Salary</th>
    </tr>
    <tr>
        <td>001</td>
        <td>John Doe</td>
        <td>Engineering</td>
        <td>$75,000</td>
    </tr>
</table>
```

### Table Sections
```html
<!-- Table with thead, tbody, tfoot -->
<table>
    <caption>Sales Report</caption>
    
    <thead>
        <tr>
            <th scope="col">Product</th>
            <th scope="col">Q1</th>
            <th scope="col">Q2</th>
            <th scope="col">Q3</th>
            <th scope="col">Q4</th>
            <th scope="col">Total</th>
        </tr>
    </thead>
    
    <tbody>
        <tr>
            <th scope="row">Widget A</th>
            <td>$1,000</td>
            <td>$1,200</td>
            <td>$1,100</td>
            <td>$1,300</td>
            <td>$4,600</td>
        </tr>
        <tr>
            <th scope="row">Widget B</th>
            <td>$800</td>
            <td>$900</td>
            <td>$950</td>
            <td>$1,050</td>
            <td>$3,700</td>
        </tr>
    </tbody>
    
    <tfoot>
        <tr>
            <th scope="row">Total</th>
            <td>$1,800</td>
            <td>$2,100</td>
            <td>$2,050</td>
            <td>$2,350</td>
            <td>$8,300</td>
        </tr>
    </tfoot>
</table>
```

### Advanced Table Features
```html
<!-- Table with colspan and rowspan -->
<table>
    <tr>
        <th rowspan="2">Name</th>
        <th colspan="2">Scores</th>
        <th rowspan="2">Average</th>
    </tr>
    <tr>
        <th>Test 1</th>
        <th>Test 2</th>
    </tr>
    <tr>
        <td>John</td>
        <td>85</td>
        <td>90</td>
        <td>87.5</td>
    </tr>
    <tr>
        <td>Jane</td>
        <td>92</td>
        <td>88</td>
        <td>90</td>
    </tr>
</table>

<!-- Table with column groups -->
<table>
    <colgroup>
        <col style="background-color: #f0f0f0;">
        <col span="2" style="background-color: #e0e0e0;">
        <col style="background-color: #d0d0d0;">
    </colgroup>
    
    <thead>
        <tr>
            <th>Product</th>
            <th>Price</th>
            <th>Quantity</th>
            <th>Total</th>
        </tr>
    </thead>
    
    <tbody>
        <tr>
            <td>Laptop</td>
            <td>$999</td>
            <td>2</td>
            <td>$1,998</td>
        </tr>
    </tbody>
</table>

<!-- Accessible table with headers attribute -->
<table>
    <tr>
        <td></td>
        <th id="q1" scope="col">Q1</th>
        <th id="q2" scope="col">Q2</th>
    </tr>
    <tr>
        <th id="revenue" scope="row">Revenue</th>
        <td headers="revenue q1">$100k</td>
        <td headers="revenue q2">$120k</td>
    </tr>
    <tr>
        <th id="expenses" scope="row">Expenses</th>
        <td headers="expenses q1">$80k</td>
        <td headers="expenses q2">$85k</td>
    </tr>
</table>

<!-- Sortable table -->
<table>
    <thead>
        <tr>
            <th>
                <button type="button" aria-label="Sort by name">
                    Name <span aria-hidden="true">↕</span>
                </button>
            </th>
            <th>
                <button type="button" aria-label="Sort by age">
                    Age <span aria-hidden="true">↕</span>
                </button>
            </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>John</td>
            <td>30</td>
        </tr>
        <tr>
            <td>Jane</td>
            <td>25</td>
        </tr>
    </tbody>
</table>

<!-- Responsive table techniques -->
<div class="table-container">
    <table class="responsive-table">
        <thead>
            <tr>
                <th>Product</th>
                <th>Description</th>
                <th>Price</th>
                <th>Stock</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td data-label="Product">Laptop</td>
                <td data-label="Description">High-performance laptop</td>
                <td data-label="Price">$999</td>
                <td data-label="Stock">5</td>
                <td data-label="Actions">
                    <button>Edit</button>
                    <button>Delete</button>
                </td>
            </tr>
        </tbody>
    </table>
</div>
```

## Global Attributes

### Common Global Attributes
```html
<!-- id and class -->
<div id="unique-identifier" class="class1 class2 class3">Content</div>

<!-- Style attribute -->
<p style="color: red; font-weight: bold;">Styled paragraph</p>

<!-- Title attribute for tooltips -->
<abbr title="HyperText Markup Language">HTML</abbr>
<img src="image.jpg" alt="Description" title="Additional information">

<!-- Language attributes -->
<html lang="en">
<p lang="fr">Bonjour le monde</p>
<span lang="es">Hola mundo</span>

<!-- Direction attributes -->
<div dir="ltr">Left-to-right text</div>
<div dir="rtl">Right-to-left text</div>
<div dir="auto">Auto-detected direction</div>

<!-- Access key -->
<button accesskey="s">Save (Alt+S)</button>
<a href="#main" accesskey="m">Main content (Alt+M)</a>

<!-- Tab index -->
<input type="text" tabindex="1">
<button tabindex="2">Button</button>
<div tabindex="0">Focusable div</div>
<a href="#" tabindex="-1">Not in tab order</a>

<!-- Content editable -->
<div contenteditable="true">This text is editable</div>
<div contenteditable="false">This text is not editable</div>
<div contenteditable="plaintext-only">Only plain text allowed</div>

<!-- Draggable -->
<div draggable="true">Draggable element</div>
<img src="image.jpg" draggable="false" alt="Non-draggable image">

<!-- Hidden attribute -->
<div hidden>This content is hidden</div>
<p hidden aria-hidden="true">Screen reader hidden content</p>

<!-- Spellcheck -->
<textarea spellcheck="true">Text with spellcheck enabled</textarea>
<input type="text" spellcheck="false">

<!-- Translate -->
<p translate="no">This text should not be translated</p>
<span translate="yes">This text can be translated</span>
```

### Data Attributes
```html
<!-- Custom data attributes -->
<div data-user-id="12345" 
     data-user-name="john-doe" 
     data-user-role="admin">
    User information
</div>

<!-- Data attributes for JavaScript -->
<button data-action="save" 
        data-confirm="Are you sure?"
        data-target="#modal">
    Save Changes
</button>

<!-- Data attributes for CSS -->
<div data-status="active" data-priority="high">
    Status indicator
</div>

<style>
[data-status="active"] { color: green; }
[data-priority="high"] { font-weight: bold; }
</style>

<!-- Complex data attributes -->
<article data-article-id="123"
         data-author="Jane Smith"
         data-published="2024-01-01"
         data-tags="html,css,javascript"
         data-category="tutorial">
    Article content
</article>
```

## Accessibility (ARIA)

### ARIA Roles
```html
<!-- Landmark roles -->
<header role="banner">Site header</header>
<nav role="navigation">Main navigation</nav>
<main role="main">Main content area</main>
<aside role="complementary">Sidebar content</aside>
<footer role="contentinfo">Site footer</footer>
<section role="region" aria-labelledby="section-title">
    <h2 id="section-title">Section Title</h2>
</section>

<!-- Widget roles -->
<div role="button" tabindex="0">Custom button</div>
<div role="checkbox" aria-checked="false" tabindex="0">Custom checkbox</div>
<div role="slider" aria-valuenow="50" aria-valuemin="0" aria-valuemax="100" tabindex="0">
    Custom slider
</div>
<ul role="tablist">
    <li role="tab" aria-selected="true" tabindex="0">Tab 1</li>
    <li role="tab" aria-selected="false" tabindex="-1">Tab 2</li>
</ul>
<div role="tabpanel" aria-labelledby="tab1">Tab 1 content</div>

<!-- Document structure roles -->
<div role="article">Article content</div>
<div role="group" aria-labelledby="group-title">
    <h3 id="group-title">Group Title</h3>
    <p>Group content</p>
</div>

<!-- Live regions -->
<div role="alert">Important alert message</div>
<div role="status">Status update</div>
<div role="log" aria-live="polite">Log messages</div>
<div role="timer" aria-live="off">Timer display</div>
```

### ARIA Properties and States
```html
<!-- Labels and descriptions -->
<button aria-label="Close dialog">×</button>
<input type="text" aria-labelledby="label1 label2">
<div id="label1">First part</div>
<div id="label2">Second part</div>

<input type="password" aria-describedby="password-help">
<div id="password-help">Password must be at least 8 characters</div>

<!-- States -->
<button aria-pressed="false" onclick="togglePressed(this)">Toggle</button>
<div aria-expanded="false" aria-controls="menu">Menu trigger</div>
<ul id="menu" aria-hidden="true">Menu items</ul>

<input type="checkbox" aria-checked="mixed"> <!-- Indeterminate state -->
<div role="option" aria-selected="true">Selected option</div>

<!-- Properties -->
<div role="progressbar" 
     aria-valuenow="32" 
     aria-valuemin="0" 
     aria-valuemax="100"
     aria-valuetext="32 percent">
    <div style="width: 32%"></div>
</div>

<input type="text" 
       aria-required="true" 
       aria-invalid="false"
       aria-autocomplete="list">

<!-- Relationships -->
<h1 id="settings-title">Settings</h1>
<div role="group" aria-labelledby="settings-title">
    Settings content
</div>

<label id="username-label" for="username">Username</label>
<input type="text" id="username" aria-labelledby="username-label">

<button aria-controls="dropdown-menu" aria-haspopup="menu">Options</button>
<ul id="dropdown-menu" role="menu">
    <li role="menuitem">Option 1</li>
    <li role="menuitem">Option 2</li>
</ul>

<!-- Flow control -->
<div aria-flowto="next-section">Current section</div>
<div id="next-section">Next section</div>

<div aria-owns="child1 child2">
    <div id="child1">Child 1</div>
</div>
<div id="child2">Child 2 (owned by parent)</div>
```

### Accessibility Best Practices
```html
<!-- Skip links -->
<a href="#main-content" class="skip-link">Skip to main content</a>
<a href="#navigation" class="skip-link">Skip to navigation</a>

<main id="main-content" tabindex="-1">
    Main content starts here
</main>

<!-- Focus management -->
<div class="modal" role="dialog" aria-labelledby="modal-title" aria-modal="true">
    <h2 id="modal-title">Modal Title</h2>
    <p>Modal content</p>
    <button aria-label="Close modal">Close</button>
</div>

<!-- Form accessibility -->
<form>
    <fieldset>
        <legend>Required Information</legend>
        
        <label for="name">Name <span aria-label="required">*</span></label>
        <input type="text" id="name" name="name" required aria-required="true">
        
        <label for="email">Email</label>
        <input type="email" id="email" name="email" 
               aria-describedby="email-error" aria-invalid="false">
        <div id="email-error" role="alert" aria-live="polite"></div>
    </fieldset>
</form>

<!-- Table accessibility -->
<table role="table" aria-label="Student grades">
    <thead>
        <tr role="row">
            <th role="columnheader" scope="col">Student</th>
            <th role="columnheader" scope="col">Grade</th>
        </tr>
    </thead>
    <tbody>
        <tr role="row">
            <td role="cell">John Doe</td>
            <td role="cell">A+</td>
        </tr>
    </tbody>
</table>

<!-- Complex widgets -->
<div class="combobox" role="combobox" 
     aria-expanded="false" 
     aria-haspopup="listbox"
     aria-owns="listbox">
    <input type="text" aria-autocomplete="list">
    <ul id="listbox" role="listbox" aria-hidden="true">
        <li role="option" aria-selected="false">Option 1</li>
        <li role="option" aria-selected="false">Option 2</li>
    </ul>
</div>
```

## Microdata and Structured Data

### Schema.org Microdata
```html
<!-- Person schema -->
<div itemscope itemtype="https://schema.org/Person">
    <h1 itemprop="name">John Doe</h1>
    <p>Job Title: <span itemprop="jobTitle">Software Engineer</span></p>
    <p>Company: <span itemprop="worksFor" itemscope itemtype="https://schema.org/Organization">
        <span itemprop="name">Tech Corp</span>
    </span></p>
    <p>Email: <a href="mailto:john@example.com" itemprop="email">john@example.com</a></p>
    <img src="photo.jpg" alt="John Doe" itemprop="image">
</div>

<!-- Article schema -->
<article itemscope itemtype="https://schema.org/Article">
    <header>
        <h1 itemprop="headline">Article Title</h1>
        <p>By <span itemprop="author" itemscope itemtype="https://schema.org/Person">
            <span itemprop="name">Jane Smith</span>
        </span></p>
        <time itemprop="datePublished" datetime="2024-01-01">January 1, 2024</time>
    </header>
    
    <div itemprop="articleBody">
        <p>Article content goes here...</p>
    </div>
    
    <div itemprop="publisher" itemscope itemtype="https://schema.org/Organization">
        <span itemprop="name">News Website</span>
        <img src="logo.png" alt="Logo" itemprop="logo">
    </div>
</article>

<!-- Product schema -->
<div itemscope itemtype="https://schema.org/Product">
    <h1 itemprop="name">Smartphone XYZ</h1>
    <img src="phone.jpg" alt="Phone" itemprop="image">
    
    <div itemprop="offers" itemscope itemtype="https://schema.org/Offer">
        <span itemprop="priceCurrency" content="USD">$</span>
        <span itemprop="price" content="599.99">599.99</span>
        <link itemprop="availability" href="https://schema.org/InStock">
    </div>
    
    <div itemprop="aggregateRating" itemscope itemtype="https://schema.org/AggregateRating">
        Rating: <span itemprop="ratingValue">4.5</span> out of 
        <span itemprop="bestRating">5</span>
        (<span itemprop="reviewCount">123</span> reviews)
    </div>
</div>

<!-- Event schema -->
<div itemscope itemtype="https://schema.org/Event">
    <h1 itemprop="name">Tech Conference 2024</h1>
    <p itemprop="description">Annual technology conference featuring the latest innovations.</p>
    
    <div itemprop="location" itemscope itemtype="https://schema.org/Place">
        <span itemprop="name">Convention Center</span>
        <div itemprop="address" itemscope itemtype="https://schema.org/PostalAddress">
            <span itemprop="streetAddress">123 Conference Blvd</span>
            <span itemprop="addressLocality">Tech City</span>
            <span itemprop="addressRegion">CA</span>
            <span itemprop="postalCode">12345</span>
        </div>
    </div>
    
    <time itemprop="startDate" datetime="2024-06-15T09:00">June 15, 2024 at 9:00 AM</time>
    <time itemprop="endDate" datetime="2024-06-15T17:00">5:00 PM</time>
    
    <div itemprop="offers" itemscope itemtype="https://schema.org/Offer">
        <span itemprop="price">99.00</span>
        <span itemprop="priceCurrency">USD</span>
        <link itemprop="availability" href="https://schema.org/InStock">
    </div>
</div>

<!-- Recipe schema -->
<div itemscope itemtype="https://schema.org/Recipe">
    <h1 itemprop="name">Chocolate Chip Cookies</h1>
    <img src="cookies.jpg" alt="Cookies" itemprop="image">
    
    <p itemprop="description">Delicious homemade chocolate chip cookies</p>
    
    <div itemprop="author" itemscope itemtype="https://schema.org/Person">
        <span itemprop="name">Chef Baker</span>
    </div>
    
    <p>Prep time: <time itemprop="prepTime" datetime="PT15M">15 minutes</time></p>
    <p>Cook time: <time itemprop="cookTime" datetime="PT12M">12 minutes</time></p>
    <p>Total time: <time itemprop="totalTime" datetime="PT27M">27 minutes</time></p>
    <p>Yield: <span itemprop="recipeYield">24 cookies</span></p>
    
    <div itemprop="nutrition" itemscope itemtype="https://schema.org/NutritionInformation">
        <span itemprop="calories">150 calories</span>
    </div>
    
    <h2>Ingredients:</h2>
    <ul>
        <li itemprop="recipeIngredient">2 cups flour</li>
        <li itemprop="recipeIngredient">1 cup butter</li>
        <li itemprop="recipeIngredient">1 cup chocolate chips</li>
    </ul>
    
    <h2>Instructions:</h2>
    <ol>
        <li itemprop="recipeInstructions">Preheat oven to 375°F</li>
        <li itemprop="recipeInstructions">Mix ingredients in large bowl</li>
        <li itemprop="recipeInstructions">Bake for 12 minutes</li>
    </ol>
</div>

<!-- Organization schema -->
<div itemscope itemtype="https://schema.org/Organization">
    <h1 itemprop="name">Tech Company Inc.</h1>
    <p itemprop="description">Leading technology solutions provider</p>
    
    <div itemprop="address" itemscope itemtype="https://schema.org/PostalAddress">
        <span itemprop="streetAddress">456 Business Ave</span>
        <span itemprop="addressLocality">Silicon Valley</span>
        <span itemprop="addressRegion">CA</span>
        <span itemprop="postalCode">94000</span>
        <span itemprop="addressCountry">USA</span>
    </div>
    
    <p>Phone: <span itemprop="telephone">+1-555-123-4567</span></p>
    <p>Email: <a href="mailto:info@techcompany.com" itemprop="email">info@techcompany.com</a></p>
    <p>Website: <a href="https://techcompany.com" itemprop="url">techcompany.com</a></p>
    
    <img src="logo.png" alt="Company Logo" itemprop="logo">
</div>
```

### JSON-LD Structured Data
```html
<!-- Article JSON-LD -->
<script type="application/ld+json">
{
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": "Complete Guide to HTML5",
    "author": {
        "@type": "Person",
        "name": "Web Developer",
        "url": "https://example.com/author"
    },
    "datePublished": "2024-01-01T10:00:00Z",
    "dateModified": "2024-01-15T14:30:00Z",
    "publisher": {
        "@type": "Organization",
        "name": "Tech Blog",
        "logo": {
            "@type": "ImageObject",
            "url": "https://example.com/logo.png",
            "width": 400,
            "height": 60
        }
    },
    "mainEntityOfPage": {
        "@type": "WebPage",
        "@id": "https://example.com/html-guide"
    },
    "image": {
        "@type": "ImageObject",
        "url": "https://example.com/article-image.jpg",
        "width": 1200,
        "height": 630
    }
}
</script>

<!-- Local Business JSON-LD -->
<script type="application/ld+json">
{
    "@context": "https://schema.org",
    "@type": "LocalBusiness",
    "name": "Best Pizza Place",
    "image": "https://example.com/restaurant.jpg",
    "address": {
        "@type": "PostalAddress",
        "streetAddress": "123 Main St",
        "addressLocality": "Anytown",
        "addressRegion": "CA",
        "postalCode": "12345",
        "addressCountry": "US"
    },
    "geo": {
        "@type": "GeoCoordinates",
        "latitude": 40.7128,
        "longitude": -74.0060
    },
    "telephone": "+1-555-123-4567",
    "openingHours": [
        "Mo-Sa 11:00-22:00",
        "Su 12:00-21:00"
    ],
    "priceRange": "$",
    "servesCuisine": "Italian",
    "aggregateRating": {
        "@type": "AggregateRating",
        "ratingValue": "4.5",
        "reviewCount": "127"
    }
}
</script>

<!-- Breadcrumb JSON-LD -->
<script type="application/ld+json">
{
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    "itemListElement": [
        {
            "@type": "ListItem",
            "position": 1,
            "name": "Home",
            "item": "https://example.com/"
        },
        {
            "@type": "ListItem",
            "position": 2,
            "name": "Products",
            "item": "https://example.com/products"
        },
        {
            "@type": "ListItem",
            "position": 3,
            "name": "Electronics",
            "item": "https://example.com/products/electronics"
        }
    ]
}
</script>

<!-- FAQ JSON-LD -->
<script type="application/ld+json">
{
    "@context": "https://schema.org",
    "@type": "FAQPage",
    "mainEntity": [
        {
            "@type": "Question",
            "name": "What is HTML?",
            "acceptedAnswer": {
                "@type": "Answer",
                "text": "HTML (HyperText Markup Language) is the standard markup language for creating web pages."
            }
        },
        {
            "@type": "Question",
            "name": "What is the latest version of HTML?",
            "acceptedAnswer": {
                "@type": "Answer",
                "text": "HTML5 is the latest major version of HTML, which includes many new features and improvements."
            }
        }
    ]
}
</script>
```

## Performance and Optimization

### Resource Loading Optimization
```html
<!-- Preload critical resources -->
<link rel="preload" href="critical.css" as="style">
<link rel="preload" href="hero-image.webp" as="image">
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="critical.js" as="script">

<!-- Prefetch future resources -->
<link rel="prefetch" href="next-page.html">
<link rel="prefetch" href="secondary-image.jpg">

<!-- DNS prefetch for external domains -->
<link rel="dns-prefetch" href="//fonts.googleapis.com">
<link rel="dns-prefetch" href="//api.analytics.com">

<!-- Preconnect to important third-parties -->
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="preconnect" href="https://cdn.example.com">

<!-- Module preload -->
<link rel="modulepreload" href="app.js">
<link rel="modulepreload" href="utils.js">
```

### Image Optimization
```html
<!-- WebP with fallbacks -->
<picture>
    <source type="image/avif" srcset="image.avif">
    <source type="image/webp" srcset="image.webp">
    <img src="image.jpg" alt="Optimized image" loading="lazy">
</picture>

<!-- Responsive images -->
<img src="small.jpg"
     srcset="small.jpg 480w,
             medium.jpg 800w,
             large.jpg 1200w"
     sizes="(max-width: 600px) 480px,
            (max-width: 1000px) 800px,
            1200px"
     alt="Responsive image"
     loading="lazy"
     decoding="async">

<!-- Native lazy loading -->
<img src="image.jpg" alt="Description" loading="lazy">
<iframe src="embed.html" loading="lazy"></iframe>

<!-- Image with intrinsic size -->
<img src="image.jpg" 
     alt="Description" 
     width="400" 
     height="300"
     style="aspect-ratio: 4/3;">
```

### Script Loading Optimization
```html
<!-- Async script loading -->
<script src="analytics.js" async></script>

<!-- Defer script loading -->
<script src="app.js" defer></script>

<!-- Module scripts -->
<script type="module" src="modern-app.js"></script>
<script nomodule src="legacy-app.js"></script>

<!-- Critical inline script -->
<script>
    // Critical path JavaScript
    document.documentElement.className = 'js-enabled';
</script>

<!-- Non-blocking script loading -->
<script>
    function loadScript(src) {
        const script = document.createElement('script');
        script.src = src;
        script.async = true;
        document.head.appendChild(script);
    }
    
    // Load non-critical scripts after page load
    window.addEventListener('load', function() {
        loadScript('non-critical.js');
    });
</script>
```

### CSS Loading Optimization
```html
<!-- Critical CSS inline -->
<style>
    /* Critical above-the-fold styles */
    body { font-family: system-ui, sans-serif; }
    .hero { background: #007bff; color: white; }
</style>

<!-- Non-critical CSS with media query -->
<link rel="stylesheet" href="non-critical.css" media="print" onload="this.media='all'">

<!-- Preload CSS -->
<link rel="preload" href="styles.css" as="style" onload="this.rel='stylesheet'">

<!-- Font loading optimization -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap">

<!-- Font display optimization -->
<style>
    @font-face {
        font-family: 'CustomFont';
        src: url('font.woff2') format('woff2');
        font-display: swap;
    }
</style>
```

## Security

### Content Security Policy
```html
<!-- CSP via meta tag -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'unsafe-inline' https://trusted-cdn.com; 
               style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
               img-src 'self' data: https:;
               font-src 'self' https://fonts.gstatic.com;">

<!-- Stricter CSP -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'none'; 
               script-src 'self' 'nonce-abc123';
               style-src 'self' 'nonce-def456';
               img-src 'self';
               connect-src 'self';
               font-src 'self';">

<!-- Script with nonce -->
<script nonce="abc123">
    console.log('Trusted script');
</script>

<!-- Style with nonce -->
<style nonce="def456">
    body { color: blue; }
</style>
```

### Other Security Headers
```html
<!-- Prevent clickjacking -->
<meta http-equiv="X-Frame-Options" content="DENY">
<meta http-equiv="X-Frame-Options" content="SAMEORIGIN">

<!-- XSS Protection -->
<meta http-equiv="X-XSS-Protection" content="1; mode=block">

<!-- Content type sniffing -->
<meta http-equiv="X-Content-Type-Options" content="nosniff">

<!-- Referrer policy -->
<meta name="referrer" content="strict-origin-when-cross-origin">

<!-- Permissions policy -->
<meta http-equiv="Permissions-Policy" 
      content="geolocation=(), microphone=(), camera=()">
```

### Secure Forms
```html
<!-- CSRF protection -->
<form action="/submit" method="post">
    <input type="hidden" name="csrf_token" value="abc123xyz">
    <input type="text" name="data" required>
    <button type="submit">Submit</button>
</form>

<!-- Honeypot field -->
<form action="/submit" method="post">
    <input type="text" name="username" required>
    <input type="email" name="email" required>
    <!-- Hidden honeypot field -->
    <input type="text" name="website" style="display:none;" tabindex="-1">
    <button type="submit">Submit</button>
</form>

<!-- Secure file upload -->
<form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" 
           name="upload" 
           accept=".pdf,.doc,.docx" 
           required>
    <input type="hidden" name="max_size" value="5242880">
    <button type="submit">Upload</button>
</form>
```

## Advanced HTML5 Features

### Web Components
```html
<!-- Custom element definition -->
<script>
class MyCustomElement extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        this.shadowRoot.innerHTML = `
            <style>
                :host { display: block; padding: 10px; }
                .content { color: blue; }
            </style>
            <div class="content">
                <slot></slot>
            </div>
        `;
    }
}
customElements.define('my-custom-element', MyCustomElement);
</script>

<!-- Using custom element -->
<my-custom-element>
    <p>This content will be slotted</p>
</my-custom-element>

<!-- Template element -->
<template id="my-template">
    <div class="card">
        <h3 class="title"></h3>
        <p class="content"></p>
        <button class="action">Click me</button>
    </div>
</template>

<script>
    const template = document.getElementById('my-template');
    const clone = template.content.cloneNode(true);
    clone.querySelector('.title').textContent = 'Card Title';
    clone.querySelector('.content').textContent = 'Card content';
    document.body.appendChild(clone);
</script>
```

### Service Worker Registration
```html
<script>
if ('serviceWorker' in navigator) {
    window.addEventListener('load', function() {
        navigator.serviceWorker.register('/sw.js')
            .then(function(registration) {
                console.log('SW registered: ', registration);
            })
            .catch(function(registrationError) {
                console.log('SW registration failed: ', registrationError);
            });
    });
}
</script>

<!-- Web App Manifest -->
<link rel="manifest" href="/manifest.json">

<!-- PWA meta tags -->
<meta name="theme-color" content="#000000">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black">
<link rel="apple-touch-icon" href="/icon-192x192.png">
```

### Intersection Observer
```html
<div class="lazy-load-container">
    <div class="placeholder" data-src="image1.jpg">Loading...</div>
    <div class="placeholder" data-src="image2.jpg">Loading...</div>
    <div class="placeholder" data-src="image3.jpg">Loading...</div>
</div>

<script>
const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            const img = document.createElement('img');
            img.src = entry.target.dataset.src;
            img.alt = 'Lazy loaded image';
            entry.target.replaceWith(img);
            observer.unobserve(entry.target);
        }
    });
});

document.querySelectorAll('.placeholder').forEach(el => {
    observer.observe(el);
});
</script>
```

## Browser Compatibility

### Feature Detection
```html
<script>
// Check for feature support
if ('serviceWorker' in navigator) {
    // Service Worker supported
}

if ('IntersectionObserver' in window) {
    // Intersection Observer supported
}

if (CSS.supports('display', 'grid')) {
    // CSS Grid supported
}

if (typeof Storage !== 'undefined') {
    // localStorage supported
}
</script>

<!-- Polyfill loading -->
<script>
if (!window.Promise) {
    document.write('<script src="promise-polyfill.js"><\/script>');
}

if (!window.fetch) {
    document.write('<script src="fetch-polyfill.js"><\/script>');
}
</script>

<!-- Progressive enhancement -->
<div class="enhanced-feature" data-fallback="basic-feature">
    <!-- Enhanced content -->
</div>

<noscript>
    <div class="basic-feature">
        <!-- Fallback content for no JavaScript -->
    </div>
</noscript>
```

### Graceful Degradation
```html
<!-- Audio with multiple fallbacks -->
<audio controls>
    <source src="audio.mp3" type="audio/mpeg">
    <source src="audio.ogg" type="audio/ogg">
    <source src="audio.wav" type="audio/wav">
    <embed src="audio.mp3" width="300" height="50">
    <p>Your browser doesn't support audio playback. 
       <a href="audio.mp3">Download the audio file</a>.</p>
</audio>

<!-- Video with poster and fallback -->
<video controls poster="video-poster.jpg">
    <source src="video.mp4" type="video/mp4">
    <source src="video.webm" type="video/webm">
    <object data="video.mp4" type="video/mp4">
        <embed src="video.mp4" type="video/mp4">
        <p>Your browser doesn't support video playback. 
           <a href="video.mp4">Download the video</a>.</p>
    </object>
</video>

<!-- Picture with multiple formats -->
<picture>
    <source type="image/avif" srcset="image.avif">
    <source type="image/webp" srcset="image.webp">
    <source type="image/jpeg" srcset="image.jpg">
    <img src="image.jpg" alt="Description">
</picture>
```

## Testing and Validation

### HTML Validation
```html
<!-- Well-formed HTML5 document -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Valid HTML5 Document</title>
</head>
<body>
    <main>
        <h1>Main Heading</h1>
        <p>Content paragraph with <a href="#link">valid link</a>.</p>
        
        <section>
            <h2>Section Heading</h2>
            <ul>
                <li>List item 1</li>
                <li>List item 2</li>
            </ul>
        </section>
    </main>
</body>
</html>

<!-- Common validation errors to avoid -->
<!-- ❌ Missing alt attribute -->
<!-- <img src="image.jpg"> -->

<!-- ✅ Correct -->
<img src="image.jpg" alt="Description">

<!-- ❌ Improper nesting -->
<!-- <p><div>Invalid nesting</div></p> -->

<!-- ✅ Correct -->
<div><p>Valid nesting</p></div>

<!-- ❌ Missing closing tags -->
<!-- <p>Paragraph without closing tag -->

<!-- ✅ Correct -->
<p>Paragraph with closing tag</p>

<!-- ❌ Duplicate IDs -->
<!-- <div id="duplicate">First</div>
<div id="duplicate">Second</div> -->

<!-- ✅ Correct -->
<div id="unique1">First</div>
<div id="unique2">Second</div>
```

### Accessibility Testing
```html
<!-- Landmark structure for screen readers -->
<body>
    <header role="banner">
        <nav role="navigation" aria-label="Main navigation">
            <!-- Navigation content -->
        </nav>
    </header>
    
    <main role="main">
        <h1>Page Title</h1>
        <!-- Main content -->
    </main>
    
    <aside role="complementary">
        <!-- Sidebar content -->
    </aside>
    
    <footer role="contentinfo">
        <!-- Footer content -->
    </footer>
</body>

<!-- Proper heading hierarchy -->
<h1>Main Title</h1>
    <h2>Major Section</h2>
        <h3>Subsection</h3>
            <h4>Sub-subsection</h4>
    <h2>Another Major Section</h2>
        <h3>Another Subsection</h3>

<!-- Focus management -->
<div class="modal" role="dialog" aria-labelledby="modal-title">
    <h2 id="modal-title">Modal Title</h2>
    <button type="button" aria-label="Close modal" onclick="closeModal()">×</button>
    <div class="modal-content">
        <!-- Modal content -->
    </div>
</div>

<!-- Color contrast consideration -->
<style>
/* Ensure sufficient contrast ratios */
.high-contrast { 
    background: #000; 
    color: #fff; 
}

.sufficient-contrast { 
    background: #007bff; 
    color: #fff; 
}
</style>
```

## Best Practices Summary

### Performance Best Practices
```html
<!-- 1. Optimize critical rendering path -->
<head>
    <!-- Critical CSS inline -->
    <style>/* Critical styles */</style>
    
    <!-- Preload important resources -->
    <link rel="preload" href="hero.jpg" as="image">
    <link rel="preload" href="app.js" as="script">
</head>

<!-- 2. Use semantic HTML -->
<main>
    <article>
        <header>
            <h1>Article Title</h1>
            <time datetime="2024-01-01">January 1, 2024</time>
        </header>
        <p>Article content...</p>
    </article>
</main>

<!-- 3. Optimize images -->
<img src="image.jpg" 
     alt="Description" 
     loading="lazy" 
     width="300" 
     height="200">

<!-- 4. Minimize DOM manipulation -->
<script>
// ❌ Poor performance
for (let i = 0; i < 1000; i++) {
    document.body.appendChild(document.createElement('div'));
}

// ✅ Better performance
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
    fragment.appendChild(document.createElement('div'));
}
document.body.appendChild(fragment);
</script>

<!-- 5. Use appropriate input types -->
<input type="email" name="email" required>
<input type="tel" name="phone">
<input type="date" name="birthday">
<input type="number" name="quantity" min="1" max="10">
</head>
</body>
</html>
```