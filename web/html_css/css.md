# CSS Comprehensive In-Depth Cheat Sheet

## Selectors

### Basic Selectors
```css
/* Universal selector */
* { margin: 0; }

/* Element selector */
h1 { color: blue; }

/* Class selector */
.highlight { background: yellow; }

/* ID selector */
#header { position: fixed; }

/* Multiple selectors */
h1, h2, h3 { font-family: Arial; }
```

### Combinator Selectors
```css
/* Descendant selector (space) */
.container p { color: gray; }

/* Child selector (>) */
.nav > li { display: inline-block; }

/* Adjacent sibling selector (+) */
h2 + p { margin-top: 0; }

/* General sibling selector (~) */
h2 ~ p { color: #666; }
```

### Attribute Selectors
```css
/* Attribute exists */
[data-type] { border: 1px solid; }

/* Attribute equals value */
[type="text"] { padding: 5px; }

/* Attribute contains value */
[class*="btn"] { cursor: pointer; }

/* Attribute starts with value */
[href^="https"] { color: green; }

/* Attribute ends with value */
[src$=".jpg"] { border: 2px solid; }

/* Attribute contains word */
[title~="important"] { font-weight: bold; }

/* Attribute starts with value or value- */
[lang|="en"] { direction: ltr; }

/* Case insensitive matching */
[type="TEXT" i] { text-transform: uppercase; }
```

### Pseudo-Classes
```css
/* Link states */
a:link { color: blue; }
a:visited { color: purple; }
a:hover { color: red; }
a:active { color: orange; }
a:focus { outline: 2px solid; }

/* Structural pseudo-classes */
:first-child { margin-top: 0; }
:last-child { margin-bottom: 0; }
:nth-child(odd) { background: #f0f0f0; }
:nth-child(3n+1) { color: red; }
:nth-of-type(2) { font-weight: bold; }
:first-of-type { text-decoration: underline; }
:last-of-type { border-bottom: none; }
:only-child { text-align: center; }
:only-of-type { font-style: italic; }

/* Form pseudo-classes */
:checked { background: green; }
:disabled { opacity: 0.5; }
:enabled { cursor: pointer; }
:focus { box-shadow: 0 0 5px blue; }
:focus-within { border: 2px solid blue; }
:focus-visible { outline: 3px solid orange; }
:invalid { border-color: red; }
:valid { border-color: green; }
:required { border-width: 2px; }
:optional { border-style: dashed; }
:in-range { color: green; }
:out-of-range { color: red; }

/* State pseudo-classes */
:empty { display: none; }
:not(.special) { opacity: 0.8; }
:target { background: yellow; }
:root { --main-color: #333; }
```

### Pseudo-Elements
```css
/* Text pseudo-elements */
::first-line { font-weight: bold; }
::first-letter { font-size: 2em; }
::selection { background: yellow; }

/* Generated content */
::before { content: "★ "; color: gold; }
::after { content: " ✓"; color: green; }

/* Form elements */
::placeholder { color: #999; font-style: italic; }
::marker { color: red; }

/* Modern pseudo-elements */
::backdrop { background: rgba(0,0,0,0.5); }
::file-selector-button { 
  background: blue; 
  color: white; 
  border: none; 
}
```

## Box Model

### Box Sizing
```css
/* Default box model */
.default-box {
  width: 300px;
  padding: 20px;
  border: 10px solid;
  margin: 15px;
  /* Total width: 300 + 40 + 20 + 30 = 390px */
}

/* Border box model */
.border-box {
  box-sizing: border-box;
  width: 300px;
  padding: 20px;
  border: 10px solid;
  margin: 15px;
  /* Total width: 300px (includes padding and border) */
}

/* Global border-box */
*, *::before, *::after {
  box-sizing: border-box;
}
```

### Margins and Padding
```css
/* Individual sides */
.spacing {
  margin-top: 10px;
  margin-right: 15px;
  margin-bottom: 20px;
  margin-left: 25px;
  
  padding-top: 5px;
  padding-right: 10px;
  padding-bottom: 15px;
  padding-left: 20px;
}

/* Shorthand syntax */
.shorthand {
  /* All sides */
  margin: 20px;
  padding: 15px;
  
  /* Vertical | Horizontal */
  margin: 10px 20px;
  padding: 15px 25px;
  
  /* Top | Horizontal | Bottom */
  margin: 10px 20px 30px;
  padding: 5px 15px 25px;
  
  /* Top | Right | Bottom | Left */
  margin: 10px 15px 20px 25px;
  padding: 5px 10px 15px 20px;
}

/* Auto margins for centering */
.centered {
  width: 800px;
  margin: 0 auto;
}

/* Negative margins */
.overlap {
  margin-top: -20px;
  margin-left: -10px;
}

/* Logical properties */
.logical {
  margin-block-start: 20px;
  margin-block-end: 30px;
  margin-inline-start: 15px;
  margin-inline-end: 25px;
  
  padding-block: 20px;
  padding-inline: 30px;
}
```

### Borders
```css
/* Border shorthand */
.border-basic {
  border: 2px solid #333;
}

/* Individual border properties */
.border-detailed {
  border-width: 1px 2px 3px 4px;
  border-style: solid dashed dotted double;
  border-color: red green blue yellow;
}

/* Individual sides */
.border-sides {
  border-top: 3px solid red;
  border-right: 2px dashed green;
  border-bottom: 1px dotted blue;
  border-left: 4px double purple;
}

/* Border radius */
.rounded {
  border-radius: 10px;
  border-radius: 10px 20px;
  border-radius: 10px 20px 30px;
  border-radius: 10px 20px 30px 40px;
  
  /* Individual corners */
  border-top-left-radius: 10px;
  border-top-right-radius: 20px;
  border-bottom-right-radius: 30px;
  border-bottom-left-radius: 40px;
  
  /* Elliptical borders */
  border-radius: 50px/25px;
}

/* Border images */
.border-image {
  border: 20px solid;
  border-image: url(border.png) 30 stretch;
  border-image-source: url(border.png);
  border-image-slice: 30;
  border-image-width: 20px;
  border-image-outset: 5px;
  border-image-repeat: stretch;
}

/* Modern border features */
.modern-borders {
  border-inline: 2px solid red;
  border-block: 3px dashed blue;
  border-start-start-radius: 20px;
  border-end-end-radius: 15px;
}
```

## Typography

### Font Properties
```css
/* Font family */
.fonts {
  font-family: "Helvetica Neue", Arial, sans-serif;
  font-family: Georgia, "Times New Roman", serif;
  font-family: "Courier New", monospace;
  font-family: system-ui; /* System font */
}

/* Font size */
.font-sizes {
  font-size: 16px;      /* Absolute */
  font-size: 1.2em;     /* Relative to parent */
  font-size: 1.2rem;    /* Relative to root */
  font-size: 120%;      /* Percentage */
  font-size: larger;    /* Keyword */
  font-size: clamp(16px, 4vw, 24px); /* Responsive */
}

/* Font weight */
.font-weights {
  font-weight: normal;   /* 400 */
  font-weight: bold;     /* 700 */
  font-weight: lighter;  /* Relative to parent */
  font-weight: bolder;   /* Relative to parent */
  font-weight: 100;      /* Thin */
  font-weight: 900;      /* Black */
}

/* Font style and variant */
.font-styles {
  font-style: normal;
  font-style: italic;
  font-style: oblique;
  font-style: oblique 14deg;
  
  font-variant: normal;
  font-variant: small-caps;
  font-variant-caps: small-caps;
  font-variant-numeric: tabular-nums;
}

/* Font shorthand */
.font-shorthand {
  font: italic small-caps bold 16px/1.5 Arial, sans-serif;
  /*    style variant    weight size/height family        */
}
```

### Text Properties
```css
/* Text alignment */
.text-align {
  text-align: left;
  text-align: right;
  text-align: center;
  text-align: justify;
  text-align: start;     /* Logical */
  text-align: end;       /* Logical */
}

/* Text decoration */
.text-decoration {
  text-decoration: underline;
  text-decoration: overline;
  text-decoration: line-through;
  text-decoration: underline overline;
  
  /* Detailed control */
  text-decoration-line: underline;
  text-decoration-color: red;
  text-decoration-style: wavy;
  text-decoration-thickness: 2px;
  text-underline-offset: 5px;
}

/* Text transformation */
.text-transform {
  text-transform: uppercase;
  text-transform: lowercase;
  text-transform: capitalize;
  text-transform: none;
}

/* Text spacing */
.text-spacing {
  letter-spacing: 2px;
  letter-spacing: 0.1em;
  word-spacing: 4px;
  line-height: 1.5;
  line-height: 24px;
  line-height: 150%;
}

/* Text shadow */
.text-shadow {
  text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
  text-shadow: 
    1px 1px 2px red,
    2px 2px 4px blue;
}

/* Advanced text properties */
.advanced-text {
  text-indent: 2em;
  text-overflow: ellipsis;
  white-space: nowrap;
  word-break: break-all;
  word-wrap: break-word;
  hyphens: auto;
  text-justify: inter-word;
  text-orientation: upright;
  writing-mode: vertical-rl;
}
```

### Web Fonts
```css
/* @font-face declaration */
@font-face {
  font-family: 'MyCustomFont';
  src: url('font.woff2') format('woff2'),
       url('font.woff') format('woff'),
       url('font.ttf') format('truetype');
  font-weight: normal;
  font-style: normal;
  font-display: swap;
}

/* Using web fonts */
.custom-font {
  font-family: 'MyCustomFont', Arial, sans-serif;
}

/* Google Fonts */
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap');

.google-font {
  font-family: 'Roboto', sans-serif;
}

/* Variable fonts */
@font-face {
  font-family: 'VariableFont';
  src: url('variable-font.woff2') format('woff2-variations');
  font-weight: 100 900;
  font-stretch: 75% 125%;
}

.variable-font {
  font-family: 'VariableFont';
  font-variation-settings: 'wght' 450, 'wdth' 100;
}
```

## Colors and Backgrounds

### Color Values
```css
/* Named colors */
.named-colors {
  color: red;
  color: blue;
  color: transparent;
  color: currentColor;
}

/* Hexadecimal */
.hex-colors {
  color: #ff0000;      /* Red */
  color: #00ff00;      /* Green */
  color: #0000ff;      /* Blue */
  color: #f00;         /* Short hex */
  color: #ff000080;    /* With alpha */
}

/* RGB and RGBA */
.rgb-colors {
  color: rgb(255, 0, 0);
  color: rgba(255, 0, 0, 0.5);
  color: rgb(100% 0% 0%);
  color: rgb(255 0 0 / 50%);  /* New syntax */
}

/* HSL and HSLA */
.hsl-colors {
  color: hsl(0, 100%, 50%);      /* Red */
  color: hsla(120, 100%, 50%, 0.5); /* Semi-transparent green */
  color: hsl(240 100% 50% / 75%); /* New syntax */
}

/* Modern color functions */
.modern-colors {
  color: lab(50% 20 -30);
  color: lch(50% 40 120);
  color: oklch(70% 0.15 180);
  color: color(display-p3 1 0 0);
  color: hwb(0 0% 0%);
}
```

### Gradients
```css
/* Linear gradients */
.linear-gradients {
  background: linear-gradient(to right, red, blue);
  background: linear-gradient(45deg, red, blue);
  background: linear-gradient(to bottom right, red, yellow, blue);
  background: linear-gradient(
    180deg, 
    red 0%, 
    yellow 50%, 
    blue 100%
  );
}

/* Radial gradients */
.radial-gradients {
  background: radial-gradient(circle, red, blue);
  background: radial-gradient(ellipse at top, red, blue);
  background: radial-gradient(
    circle at 50% 50%, 
    red 0%, 
    transparent 70%
  );
}

/* Conic gradients */
.conic-gradients {
  background: conic-gradient(red, yellow, lime, aqua, blue, magenta, red);
  background: conic-gradient(from 45deg, red, blue);
  background: conic-gradient(at 25% 25%, red, blue);
}

/* Repeating gradients */
.repeating-gradients {
  background: repeating-linear-gradient(
    45deg,
    red 0px,
    red 10px,
    blue 10px,
    blue 20px
  );
  
  background: repeating-radial-gradient(
    circle,
    red 0px,
    red 10px,
    blue 10px,
    blue 20px
  );
}
```

### Backgrounds
```css
/* Background image */
.background-image {
  background-image: url('image.jpg');
  background-image: linear-gradient(rgba(0,0,0,0.5), rgba(0,0,0,0.5)), url('image.jpg');
}

/* Background position */
.background-position {
  background-position: center;
  background-position: top right;
  background-position: 50% 75%;
  background-position: 10px 20px;
  background-position: left 10px top 20px;
}

/* Background size */
.background-size {
  background-size: cover;
  background-size: contain;
  background-size: 100% 50%;
  background-size: 300px 200px;
  background-size: auto 100px;
}

/* Background repeat */
.background-repeat {
  background-repeat: no-repeat;
  background-repeat: repeat-x;
  background-repeat: repeat-y;
  background-repeat: space;
  background-repeat: round;
}

/* Background attachment */
.background-attachment {
  background-attachment: scroll;
  background-attachment: fixed;
  background-attachment: local;
}

/* Background origin and clip */
.background-advanced {
  background-origin: border-box;
  background-origin: padding-box;
  background-origin: content-box;
  
  background-clip: border-box;
  background-clip: padding-box;
  background-clip: content-box;
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

/* Multiple backgrounds */
.multiple-backgrounds {
  background: 
    url('overlay.png') center/cover no-repeat,
    linear-gradient(45deg, red, blue),
    url('texture.jpg') repeat;
}

/* Background shorthand */
.background-shorthand {
  background: url('image.jpg') center/cover no-repeat fixed;
  /*          image           position/size repeat attachment */
}
```

## Layout Systems

### Display Property
```css
/* Basic display values */
.display-values {
  display: block;
  display: inline;
  display: inline-block;
  display: none;
  display: contents;
}

/* Table display */
.table-display {
  display: table;
  display: table-row;
  display: table-cell;
  display: table-column;
  display: table-header-group;
  display: table-footer-group;
}

/* Modern display values */
.modern-display {
  display: flex;
  display: inline-flex;
  display: grid;
  display: inline-grid;
  display: subgrid;
}
```

### Flexbox Layout
```css
/* Flex container */
.flex-container {
  display: flex;
  
  /* Direction */
  flex-direction: row;        /* default */
  flex-direction: row-reverse;
  flex-direction: column;
  flex-direction: column-reverse;
  
  /* Wrap */
  flex-wrap: nowrap;         /* default */
  flex-wrap: wrap;
  flex-wrap: wrap-reverse;
  
  /* Shorthand for direction and wrap */
  flex-flow: row wrap;
  
  /* Main axis alignment */
  justify-content: flex-start;  /* default */
  justify-content: flex-end;
  justify-content: center;
  justify-content: space-between;
  justify-content: space-around;
  justify-content: space-evenly;
  
  /* Cross axis alignment */
  align-items: stretch;        /* default */
  align-items: flex-start;
  align-items: flex-end;
  align-items: center;
  align-items: baseline;
  
  /* Multi-line alignment */
  align-content: stretch;      /* default */
  align-content: flex-start;
  align-content: flex-end;
  align-content: center;
  align-content: space-between;
  align-content: space-around;
  
  /* Gap */
  gap: 20px;
  row-gap: 10px;
  column-gap: 15px;
}

/* Flex items */
.flex-item {
  /* Grow factor */
  flex-grow: 1;
  
  /* Shrink factor */
  flex-shrink: 1;
  
  /* Basis */
  flex-basis: auto;
  flex-basis: 200px;
  flex-basis: 50%;
  
  /* Shorthand */
  flex: 1;           /* grow shrink basis */
  flex: 1 0 auto;
  flex: none;        /* 0 0 auto */
  
  /* Individual alignment */
  align-self: auto;     /* inherit from parent */
  align-self: flex-start;
  align-self: flex-end;
  align-self: center;
  align-self: stretch;
  
  /* Order */
  order: 0;          /* default */
  order: 1;
  order: -1;
}
```

### Grid Layout
```css
/* Grid container */
.grid-container {
  display: grid;
  
  /* Grid template columns */
  grid-template-columns: 200px 1fr 100px;
  grid-template-columns: repeat(3, 1fr);
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  grid-template-columns: [sidebar] 250px [content] 1fr [aside] 200px;
  
  /* Grid template rows */
  grid-template-rows: 100px 1fr 50px;
  grid-template-rows: repeat(3, minmax(100px, auto));
  
  /* Grid template areas */
  grid-template-areas: 
    "header header header"
    "sidebar content aside"
    "footer footer footer";
  
  /* Grid template shorthand */
  grid-template: 
    "header header" 100px
    "sidebar content" 1fr
    "footer footer" 50px
    / 250px 1fr;
  
  /* Grid gaps */
  gap: 20px;
  row-gap: 15px;
  column-gap: 25px;
  
  /* Grid alignment */
  justify-items: start;      /* horizontal alignment of items */
  justify-items: end;
  justify-items: center;
  justify-items: stretch;    /* default */
  
  align-items: start;        /* vertical alignment of items */
  align-items: end;
  align-items: center;
  align-items: stretch;      /* default */
  
  justify-content: start;    /* horizontal alignment of grid */
  justify-content: end;
  justify-content: center;
  justify-content: stretch;
  justify-content: space-around;
  justify-content: space-between;
  justify-content: space-evenly;
  
  align-content: start;      /* vertical alignment of grid */
  align-content: end;
  align-content: center;
  align-content: stretch;
  align-content: space-around;
  align-content: space-between;
  align-content: space-evenly;
  
  /* Shorthand alignment */
  place-items: center;       /* align-items justify-items */
  place-content: center;     /* align-content justify-content */
}

/* Grid items */
.grid-item {
  /* Grid column placement */
  grid-column-start: 1;
  grid-column-end: 3;
  grid-column: 1 / 3;
  grid-column: 1 / span 2;
  grid-column: sidebar;      /* named line */
  
  /* Grid row placement */
  grid-row-start: 2;
  grid-row-end: 4;
  grid-row: 2 / 4;
  grid-row: 2 / span 2;
  
  /* Grid area placement */
  grid-area: header;         /* named area */
  grid-area: 1 / 1 / 3 / 3; /* row-start / col-start / row-end / col-end */
  
  /* Individual item alignment */
  justify-self: start;
  justify-self: end;
  justify-self: center;
  justify-self: stretch;
  
  align-self: start;
  align-self: end;
  align-self: center;
  align-self: stretch;
  
  place-self: center;        /* align-self justify-self */
}

/* Subgrid */
.subgrid {
  display: grid;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
}

/* Container queries with grid */
@container (min-width: 700px) {
  .responsive-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

### Positioning
```css
/* Position types */
.positioned {
  position: static;      /* default */
  position: relative;
  position: absolute;
  position: fixed;
  position: sticky;
}

/* Positioning offsets */
.offsets {
  top: 10px;
  right: 20px;
  bottom: 30px;
  left: 40px;
  
  /* Logical properties */
  inset-block-start: 10px;
  inset-block-end: 30px;
  inset-inline-start: 40px;
  inset-inline-end: 20px;
  
  /* Shorthand */
  inset: 10px 20px 30px 40px;
  inset: 10px 20px;
  inset: 10px;
}

/* Z-index stacking */
.stacking {
  z-index: 1;
  z-index: 999;
  z-index: -1;
}

/* Sticky positioning */
.sticky-header {
  position: sticky;
  top: 0;
  z-index: 100;
}

/* Absolute centering */
.centered-absolute {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.centered-absolute-modern {
  position: absolute;
  inset: 0;
  margin: auto;
  width: 300px;
  height: 200px;
}
```

### Float and Clear
```css
/* Float property */
.floated {
  float: left;
  float: right;
  float: none;        /* default */
}

/* Clear property */
.cleared {
  clear: left;
  clear: right;
  clear: both;
  clear: none;        /* default */
}

/* Clearfix */
.clearfix::after {
  content: "";
  display: table;
  clear: both;
}

/* Modern clearfix */
.modern-clearfix {
  display: flow-root;
}
```

## Responsive Design

### Media Queries
```css
/* Basic media queries */
@media screen and (max-width: 768px) {
  .mobile-styles { font-size: 14px; }
}

@media screen and (min-width: 769px) and (max-width: 1024px) {
  .tablet-styles { font-size: 16px; }
}

@media screen and (min-width: 1025px) {
  .desktop-styles { font-size: 18px; }
}

/* Media features */
@media (orientation: landscape) {
  .landscape { width: 100%; }
}

@media (orientation: portrait) {
  .portrait { height: 100vh; }
}

@media (aspect-ratio: 16/9) {
  .widescreen { background: blue; }
}

@media (min-aspect-ratio: 16/9) {
  .wide { padding: 2rem; }
}

/* Resolution queries */
@media (min-resolution: 2dppx) {
  .retina { background-image: url('image@2x.jpg'); }
}

@media (-webkit-min-device-pixel-ratio: 2) {
  .webkit-retina { background-image: url('image@2x.jpg'); }
}

/* Color scheme queries */
@media (prefers-color-scheme: dark) {
  .dark-mode { background: #333; color: white; }
}

@media (prefers-color-scheme: light) {
  .light-mode { background: white; color: #333; }
}

/* Motion queries */
@media (prefers-reduced-motion: reduce) {
  .no-animation { animation: none !important; }
}

/* Pointer and hover queries */
@media (hover: hover) {
  .hover-styles:hover { background: blue; }
}

@media (pointer: coarse) {
  .touch-friendly { min-height: 44px; }
}

/* Combined queries */
@media screen and (min-width: 768px) and (hover: hover) {
  .desktop-hover:hover { transform: scale(1.1); }
}

/* Range syntax (modern) */
@media (400px <= width <= 700px) {
  .range-styles { color: blue; }
}

@media (width >= 500px) {
  .min-width { font-size: 18px; }
}
```

### Container Queries
```css
/* Container query setup */
.container {
  container-type: inline-size;
  container-name: sidebar;
}

/* Container queries */
@container (min-width: 400px) {
  .card { 
    display: flex; 
    gap: 1rem;
  }
}

@container sidebar (min-width: 300px) {
  .sidebar-content { 
    columns: 2; 
  }
}

/* Container query units */
.responsive-text {
  font-size: calc(1rem + 2cqi);  /* Container inline size */
  padding: 1cqb;                 /* Container block size */
  margin: 1cqw 1cqh;             /* Container width/height */
  border-radius: 1cqmin;         /* Container minimum */
  line-height: 1cqmax;           /* Container maximum */
}
```

### Responsive Units
```css
/* Viewport units */
.viewport-units {
  width: 100vw;          /* Viewport width */
  height: 100vh;         /* Viewport height */
  font-size: 4vmin;      /* Viewport minimum */
  line-height: 2vmax;    /* Viewport maximum */
  
  /* Dynamic viewport units */
  height: 100dvh;        /* Dynamic viewport height */
  height: 100svh;        /* Small viewport height */
  height: 100lvh;        /* Large viewport height */
  
  width: 100dvi;         /* Dynamic viewport inline */
  width: 100svi;         /* Small viewport inline */
  width: 100lvi;         /* Large viewport inline */
}

/* Relative units */
.relative-units {
  font-size: 1.2em;      /* Relative to parent font-size */
  font-size: 1.2rem;     /* Relative to root font-size */
  
  width: 50%;            /* Percentage of parent */
  line-height: 1.5;      /* Unitless (relative to font-size) */
  
  margin: 1ch;           /* Character width */
  width: 20ex;           /* X-height */
  font-size: 2cap;       /* Cap height */
}

/* Modern responsive functions */
.responsive-functions {
  /* Clamp function */
  font-size: clamp(16px, 4vw, 24px);
  width: clamp(300px, 50%, 800px);
  
  /* Min/Max functions */
  width: min(90%, 600px);
  height: max(300px, 50vh);
  
  /* Calc function */
  width: calc(100% - 40px);
  margin: calc(1rem + 2vw);
  transform: translateX(calc(-50% + 10px));
}
```

## Animations and Transitions

### Transitions
```css
/* Basic transition */
.transition-basic {
  transition: all 0.3s ease;
  transition: width 0.5s ease-in-out;
  transition: transform 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}

/* Multiple transitions */
.transition-multiple {
  transition: 
    opacity 0.3s ease,
    transform 0.5s ease-in-out 0.1s,
    background-color 0.2s linear;
}

/* Transition properties */
.transition-detailed {
  transition-property: opacity, transform;
  transition-duration: 0.3s, 0.5s;
  transition-timing-function: ease, ease-in-out;
  transition-delay: 0s, 0.1s;
}

/* Timing functions */
.timing-functions {
  transition-timing-function: linear;
  transition-timing-function: ease;
  transition-timing-function: ease-in;
  transition-timing-function: ease-out;
  transition-timing-function: ease-in-out;
  transition-timing-function: step-start;
  transition-timing-function: step-end;
  transition-timing-function: steps(4, jump-end);
  transition-timing-function: cubic-bezier(0.25, 0.1, 0.25, 1);
}

/* Common transition patterns */
.button-transition {
  background: #007bff;
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  transition: all 0.2s ease;
}

.button-transition:hover {
  background: #0056b3;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.fade-transition {
  opacity: 1;
  transition: opacity 0.3s ease;
}

.fade-transition.hidden {
  opacity: 0;
}
```

### Animations
```css
/* Keyframe definitions */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideIn {
  0% {
    transform: translateX(-100%);
    opacity: 0;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-20px); }
}

@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes colorChange {
  0% { background-color: red; }
  25% { background-color: yellow; }
  50% { background-color: green; }
  75% { background-color: blue; }
  100% { background-color: red; }
}

/* Animation properties */
.animated {
  animation-name: fadeIn;
  animation-duration: 1s;
  animation-timing-function: ease-in-out;
  animation-delay: 0.5s;
  animation-iteration-count: infinite;
  animation-direction: normal;
  animation-fill-mode: forwards;
  animation-play-state: running;
}

/* Animation shorthand */
.animation-shorthand {
  animation: fadeIn 1s ease-in-out 0.5s infinite normal forwards running;
  /*         name duration timing delay iteration direction fill-mode play-state */
  
  /* Multiple animations */
  animation: 
    fadeIn 1s ease-in-out,
    slideIn 0.5s ease 0.2s,
    bounce 2s infinite;
}

/* Animation direction values */
.animation-directions {
  animation-direction: normal;       /* default */
  animation-direction: reverse;
  animation-direction: alternate;
  animation-direction: alternate-reverse;
}

/* Animation fill mode */
.animation-fill-modes {
  animation-fill-mode: none;         /* default */
  animation-fill-mode: forwards;
  animation-fill-mode: backwards;
  animation-fill-mode: both;
}

/* Animation iteration count */
.animation-iterations {
  animation-iteration-count: 1;      /* default */
  animation-iteration-count: 3;
  animation-iteration-count: infinite;
  animation-iteration-count: 0.5;
}

/* Complex animation example */
.loading-spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  animation: rotate 1s linear infinite;
}

.floating-animation {
  animation: 
    float 3s ease-in-out infinite,
    fade 2s ease-in-out infinite alternate;
}

@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-20px); }
}

@keyframes fade {
  0% { opacity: 0.5; }
  100% { opacity: 1; }
}
```

### Transform Functions
```css
/* 2D Transforms */
.transform-2d {
  /* Translate */
  transform: translate(50px, 100px);
  transform: translateX(50px);
  transform: translateY(100px);
  
  /* Scale */
  transform: scale(1.5);
  transform: scale(2, 0.5);
  transform: scaleX(2);
  transform: scaleY(0.5);
  
  /* Rotate */
  transform: rotate(45deg);
  transform: rotate(-0.25turn);
  transform: rotate(1.57rad);
  
  /* Skew */
  transform: skew(15deg, 10deg);
  transform: skewX(15deg);
  transform: skewY(10deg);
  
  /* Matrix */
  transform: matrix(1, 0, 0, 1, 50, 100);
}

/* 3D Transforms */
.transform-3d {
  /* 3D Translate */
  transform: translate3d(50px, 100px, 25px);
  transform: translateZ(25px);
  
  /* 3D Scale */
  transform: scale3d(2, 1.5, 0.5);
  transform: scaleZ(0.5);
  
  /* 3D Rotate */
  transform: rotate3d(1, 1, 1, 45deg);
  transform: rotateX(45deg);
  transform: rotateY(45deg);
  transform: rotateZ(45deg);
  
  /* Perspective */
  transform: perspective(500px) rotateX(45deg);
  
  /* Matrix 3D */
  transform: matrix3d(1,0,0,0, 0,1,0,0, 0,0,1,0, 50,100,25,1);
}

/* Transform properties */
.transform-properties {
  transform-origin: center;
  transform-origin: top left;
  transform-origin: 50% 25%;
  transform-origin: 10px 20px;
  transform-origin: 50% 50% 100px; /* 3D */
  
  transform-style: flat;        /* default */
  transform-style: preserve-3d;
  
  perspective: 500px;
  perspective-origin: center;
  perspective-origin: 75% 25%;
  
  backface-visibility: visible; /* default */
  backface-visibility: hidden;
}

/* Multiple transforms */
.multiple-transforms {
  transform: 
    translateX(50px) 
    rotate(45deg) 
    scale(1.2);
}

/* Transform performance tip */
.hardware-accelerated {
  transform: translateZ(0); /* Force hardware acceleration */
  will-change: transform;   /* Hint to browser */
}
```

## Advanced CSS Features

### Custom Properties (CSS Variables)
```css
/* Global custom properties */
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --font-size-base: 16px;
  --font-family-base: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto;
  --spacing-unit: 8px;
  --border-radius: 4px;
  --box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  --transition-duration: 0.2s;
}

/* Using custom properties */
.component {
  color: var(--primary-color);
  font-size: var(--font-size-base);
  font-family: var(--font-family-base);
  padding: calc(var(--spacing-unit) * 2);
  border-radius: var(--border-radius);
  box-shadow: var(--box-shadow);
  transition: all var(--transition-duration) ease;
}

/* Custom properties with fallbacks */
.fallback-example {
  color: var(--theme-color, #333);
  margin: var(--component-margin, var(--spacing-unit, 8px));
}

/* Scoped custom properties */
.theme-dark {
  --background-color: #1a1a1a;
  --text-color: #ffffff;
  --border-color: #333333;
}

.theme-light {
  --background-color: #ffffff;
  --text-color: #333333;
  --border-color: #e0e0e0;
}

.themed-component {
  background-color: var(--background-color);
  color: var(--text-color);
  border: 1px solid var(--border-color);
}

/* Dynamic custom properties with JavaScript */
.dynamic-variables {
  --mouse-x: 0;
  --mouse-y: 0;
  --progress: 0%;
  
  background: radial-gradient(
    circle at var(--mouse-x) var(--mouse-y),
    rgba(255,255,255,0.1),
    transparent
  );
  
  width: var(--progress);
}

/* Custom properties in calculations */
.calculated-values {
  --columns: 3;
  --gap: 20px;
  
  width: calc((100% - (var(--columns) - 1) * var(--gap)) / var(--columns));
  margin-right: var(--gap);
}

/* Media query responsive variables */
@media (max-width: 768px) {
  :root {
    --font-size-base: 14px;
    --spacing-unit: 6px;
    --columns: 1;
  }
}
```

### CSS Functions
```css
/* Mathematical functions */
.math-functions {
  /* Basic math */
  width: calc(100% - 40px);
  height: calc(100vh - 60px);
  margin: calc(1rem + 5px);
  
  /* Min/Max */
  width: min(90vw, 1200px);
  height: max(300px, 50vh);
  font-size: max(16px, 1.2vw);
  
  /* Clamp */
  font-size: clamp(14px, 2.5vw, 22px);
  width: clamp(300px, 50%, 800px);
  line-height: clamp(1.3, 1.4 + 0.5vw, 1.8);
}

/* Color functions */
.color-functions {
  /* RGB/RGBA */
  color: rgb(255 128 0);
  color: rgba(255, 128, 0, 0.8);
  
  /* HSL/HSLA */
  background: hsl(200 50% 70%);
  background: hsla(200, 50%, 70%, 0.9);
  
  /* Modern color functions */
  color: lab(70% 20 -30);
  color: lch(70% 40 180);
  color: oklch(0.7 0.15 180);
  color: color(display-p3 1 0.5 0);
  
  /* Color mixing (future) */
  color: color-mix(in srgb, red 30%, blue);
  color: color-mix(in oklch, var(--color1), var(--color2) 25%);
}

/* Filter functions */
.filter-functions {
  filter: blur(5px);
  filter: brightness(150%);
  filter: contrast(120%);
  filter: grayscale(50%);
  filter: hue-rotate(90deg);
  filter: invert(100%);
  filter: saturate(200%);
  filter: sepia(60%);
  filter: opacity(80%);
  filter: drop-shadow(2px 2px 4px rgba(0,0,0,0.5));
  
  /* Multiple filters */
  filter: blur(2px) brightness(120%) contrast(110%);
  
  /* Backdrop filter */
  backdrop-filter: blur(10px) saturate(180%);
}

/* Transform functions */
.transform-functions {
  /* 2D transforms */
  transform: translate(50px, 100px);
  transform: rotate(45deg);
  transform: scale(1.5);
  transform: skew(10deg, 5deg);
  
  /* 3D transforms */
  transform: translate3d(10px, 20px, 30px);
  transform: rotate3d(1, 1, 0, 45deg);
  transform: perspective(500px);
  
  /* Combined transforms */
  transform: perspective(500px) rotateX(45deg) translateZ(100px);
}

/* Gradient functions */
.gradient-functions {
  background: linear-gradient(45deg, red, blue);
  background: radial-gradient(circle at center, red, blue);
  background: conic-gradient(from 0deg, red, yellow, blue, red);
  background: repeating-linear-gradient(
    45deg, 
    red 0px, 
    red 10px, 
    blue 10px, 
    blue 20px
  );
}

/* Shape functions */
.shape-functions {
  clip-path: circle(50%);
  clip-path: ellipse(50% 25%);
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
  clip-path: inset(10px 20px 30px 40px round 5px);
  
  shape-outside: circle(50%);
  shape-outside: polygon(0 0, 100% 0, 100% 100%);
}

/* Counter functions */
.counter-functions {
  counter-reset: section;
  counter-increment: section;
  content: "Section " counter(section) ": ";
  content: counters(section, ".");
}

/* Attribute functions */
.attribute-functions {
  content: attr(data-label);
  content: "Value: " attr(data-value) " units";
  width: attr(data-width px, 100px);
}

/* URL functions */
.url-functions {
  background-image: url('image.jpg');
  background-image: url(data:image/svg+xml;base64,...);
  cursor: url('cursor.cur'), pointer;
}
```

### Pseudo-elements Advanced Usage
```css
/* Generated content techniques */
.tooltip {
  position: relative;
}

.tooltip::before {
  content: attr(data-tooltip);
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  background: #333;
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  font-size: 12px;
  white-space: nowrap;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s, visibility 0.3s;
}

.tooltip::after {
  content: '';
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  border: 5px solid transparent;
  border-top-color: #333;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s, visibility 0.3s;
}

.tooltip:hover::before,
.tooltip:hover::after {
  opacity: 1;
  visibility: visible;
}

/* Decorative elements */
.fancy-quote::before {
  content: '"';
  font-size: 4em;
  color: #ccc;
  line-height: 0.1em;
  margin-right: 0.25em;
  vertical-align: -0.4em;
}

.fancy-quote::after {
  content: '"';
  font-size: 4em;
  color: #ccc;
  line-height: 0.1em;
  margin-left: 0.25em;
  vertical-align: -0.4em;
}

/* Progress bars */
.progress-bar {
  width: 100%;
  height: 20px;
  background: #f0f0f0;
  border-radius: 10px;
  overflow: hidden;
  position: relative;
}

.progress-bar::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: var(--progress, 0%);
  background: linear-gradient(90deg, #007bff, #0056b3);
  border-radius: 10px;
  transition: width 0.3s ease;
}

/* Overlay effects */
.image-overlay {
  position: relative;
  overflow: hidden;
}

.image-overlay::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    45deg,
    rgba(255,0,150,0.3),
    rgba(0,255,255,0.3)
  );
  opacity: 0;
  transition: opacity 0.3s ease;
}

.image-overlay:hover::before {
  opacity: 1;
}

/* Counter styling */
.custom-counter {
  counter-reset: item;
}

.custom-counter li {
  counter-increment: item;
  list-style: none;
  position: relative;
  padding-left: 40px;
}

.custom-counter li::before {
  content: counter(item);
  position: absolute;
  left: 0;
  top: 0;
  background: #007bff;
  color: white;
  width: 25px;
  height: 25px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  font-weight: bold;
}
```

### Modern CSS Features
```css
/* Container Queries */
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 2fr;
    gap: 1rem;
  }
}

/* Cascade Layers */
@layer reset, base, components, utilities;

@layer reset {
  * { margin: 0; padding: 0; }
}

@layer base {
  body { font-family: system-ui; }
}

@layer components {
  .button { padding: 0.5rem 1rem; }
}

@layer utilities {
  .hidden { display: none !important; }
}

/* CSS Nesting (future) */
.component {
  background: white;
  border: 1px solid #ccc;
  
  &:hover {
    border-color: #999;
  }
  
  & .title {
    font-size: 1.2em;
    font-weight: bold;
    
    & a {
      text-decoration: none;
      
      &:hover {
        text-decoration: underline;
      }
    }
  }
  
  @media (max-width: 768px) {
    & {
      padding: 0.5rem;
    }
  }
}

/* Subgrid */
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

.grid-item {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 3;
}

/* Color Scheme */
:root {
  color-scheme: light dark;
}

.adaptive-colors {
  background: Canvas;
  color: CanvasText;
  border: 1px solid ButtonBorder;
}

/* Scroll behavior */
.smooth-scroll {
  scroll-behavior: smooth;
  scroll-snap-type: y mandatory;
}

.snap-section {
  scroll-snap-align: start;
  scroll-snap-stop: always;
}

/* Scroll timeline (future) */
@scroll-timeline slide-timeline {
  source: selector('.slider');
  orientation: horizontal;
  scroll-offsets: 0%, 100%;
}

.slide-animation {
  animation: slide 1s slide-timeline;
}

/* View Transitions API (future) */
::view-transition-old(root) {
  animation: fade-out 0.3s ease;
}

::view-transition-new(root) {
  animation: fade-in 0.3s ease;
}

/* Anchor Positioning (future) */
.anchor {
  anchor-name: --my-anchor;
}

.positioned {
  position: absolute;
  left: anchor(--my-anchor right);
  top: anchor(--my-anchor bottom);
}
```

## Performance and Optimization

### CSS Performance Best Practices
```css
/* Efficient selectors */
/* Good - specific and fast */
.nav-item { color: blue; }
#sidebar .widget { margin: 10px; }

/* Avoid - slow and inefficient */
/* * { color: red; } */
/* div > div > div > p { color: green; } */

/* Hardware acceleration */
.accelerated {
  transform: translateZ(0);
  will-change: transform;
  backface-visibility: hidden;
}

/* Containment */
.contained {
  contain: layout style paint;
  contain: strict; /* layout + style + paint + size */
  contain: content; /* layout + style + paint */
}

/* Content visibility */
.lazy-content {
  content-visibility: auto;
  contain-intrinsic-size: 500px;
}

/* Efficient animations */
.efficient-animation {
  /* Animate only transform and opacity */
  transition: transform 0.3s ease, opacity 0.3s ease;
}

/* Avoid animating expensive properties */
.avoid-this {
  /* transition: width 0.3s ease; */
  /* transition: height 0.3s ease; */
  /* transition: top 0.3s ease; */
  /* transition: left 0.3s ease; */
}

/* Use transform instead */
.use-this {
  transition: transform 0.3s ease;
}

.use-this.moved {
  transform: translateX(100px);
}

/* Critical CSS patterns */
/* Above-the-fold styles should be inlined */
.hero {
  height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Loading strategies */
.skeleton {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: loading 1.5s infinite;
}

@keyframes loading {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

/* Resource hints */
/* <link rel="preload" href="critical.css" as="style"> */
/* <link rel="prefetch" href="non-critical.css"> */
```

### CSS Optimization Techniques
```css
/* Minimize reflows and repaints */
.optimized-layout {
  /* Use flexbox/grid instead of floats */
  display: flex;
  
  /* Avoid changing box model properties */
  /* width, height, padding, margin, border */
  
  /* Prefer transform for position changes */
  transform: translateX(10px);
  /* instead of left: 10px; */
}

/* Efficient responsive images */
.responsive-image {
  width: 100%;
  height: auto;
  object-fit: cover;
  object-position: center;
}

/* CSS-only lazy loading */
.lazy-load {
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 0.5s ease, transform 0.5s ease;
}

.lazy-load.loaded {
  opacity: 1;
  transform: translateY(0);
}

/* Efficient gradients */
.efficient-gradient {
  /* Use fewer color stops */
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  
  /* Instead of many stops */
  /* background: linear-gradient(45deg, #ff6b6b 0%, #ff8e6b 25%, #ffb16b 50%, #4ecdc4 100%); */
}

/* Bundle and compress CSS */
/* Use build tools to: */
/* - Remove unused CSS */
/* - Minify CSS */
/* - Use CSS modules or scoped styles */
/* - Tree shake unused code */
/* - Generate critical CSS */
```

