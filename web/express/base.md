# Complete Express.js Cheatsheet

## Table of Contents
1. [Setup & Installation](#setup--installation)
2. [Basic Server Setup](#basic-server-setup)
3. [Routing](#routing)
4. [Middleware](#middleware)
5. [Request & Response](#request--response)
6. [Error Handling](#error-handling)
7. [Static Files](#static-files)
8. [Template Engines](#template-engines)
9. [Database Integration](#database-integration)
10. [Authentication & Authorization](#authentication--authorization)
11. [Validation](#validation)
12. [File Uploads](#file-uploads)
13. [API Design](#api-design)
14. [Testing](#testing)
15. [Security](#security)
16. [Performance & Optimization](#performance--optimization)
17. [Deployment](#deployment)
18. [Best Practices](#best-practices)

---

## Setup & Installation

### Initialize New Project
```bash
# Create new directory
mkdir my-express-app
cd my-express-app

# Initialize package.json
npm init -y

# Install Express
npm install express

# Install development dependencies
npm install --save-dev nodemon

# Install commonly used packages
npm install cors helmet morgan dotenv
npm install bcryptjs jsonwebtoken
npm install express-validator
npm install mongoose  # For MongoDB
npm install pg        # For PostgreSQL
npm install mysql2    # For MySQL
```

### Package.json Scripts
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```

### Environment Configuration
```bash
# .env file
NODE_ENV=development
PORT=3000
DB_CONNECTION_STRING=mongodb://localhost:27017/myapp
JWT_SECRET=your_jwt_secret_key
API_KEY=your_api_key
```

---

## Basic Server Setup

### Minimal Express Server
```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Complete Server Setup
```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(helmet()); // Security headers
app.use(cors()); // Enable CORS
app.use(morgan('combined')); // Logging
app.use(express.json({ limit: '10mb' })); // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies

// Routes
app.get('/', (req, res) => {
  res.json({ 
    message: 'API is running!',
    version: '1.0.0',
    timestamp: new Date().toISOString()
  });
});

// Health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'OK',
    uptime: process.uptime(),
    timestamp: new Date().toISOString()
  });
});

// Global error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ 
    message: 'Something went wrong!',
    error: process.env.NODE_ENV === 'development' ? err.message : {}
  });
});

// Handle 404
app.use('*', (req, res) => {
  res.status(404).json({ message: 'Route not found' });
});

app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT}`);
});
```

### Server with Graceful Shutdown
```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

// Your middleware and routes here...

const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM received. Shutting down gracefully...');
  server.close(() => {
    console.log('Process terminated');
  });
});

process.on('SIGINT', () => {
  console.log('SIGINT received. Shutting down gracefully...');
  server.close(() => {
    console.log('Process terminated');
  });
});
```

---

## Routing

### Basic Routes
```javascript
const express = require('express');
const app = express();

// GET route
app.get('/users', (req, res) => {
  res.json({ message: 'Get all users' });
});

// POST route
app.post('/users', (req, res) => {
  const { name, email } = req.body;
  res.status(201).json({ 
    message: 'User created',
    user: { name, email }
  });
});

// PUT route
app.put('/users/:id', (req, res) => {
  const { id } = req.params;
  const { name, email } = req.body;
  res.json({ 
    message: 'User updated',
    user: { id, name, email }
  });
});

// DELETE route
app.delete('/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ message: `User ${id} deleted` });
});

// PATCH route
app.patch('/users/:id', (req, res) => {
  const { id } = req.params;
  const updates = req.body;
  res.json({ 
    message: 'User partially updated',
    id,
    updates
  });
});
```

### Route Parameters
```javascript
// Single parameter
app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ userId: id });
});

// Multiple parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});

// Optional parameters
app.get('/posts/:year/:month?', (req, res) => {
  const { year, month } = req.params;
  res.json({ year, month: month || 'all' });
});

// Wildcard parameters
app.get('/files/*', (req, res) => {
  const filePath = req.params[0];
  res.json({ filePath });
});

// Parameter validation
app.param('id', (req, res, next, id) => {
  if (!/^\d+$/.test(id)) {
    return res.status(400).json({ error: 'Invalid ID format' });
  }
  req.userId = parseInt(id);
  next();
});
```

### Query Parameters
```javascript
// GET /search?q=express&limit=10&page=1
app.get('/search', (req, res) => {
  const { 
    q: query,
    limit = 10,
    page = 1,
    sort = 'asc'
  } = req.query;
  
  res.json({
    query,
    limit: parseInt(limit),
    page: parseInt(page),
    sort
  });
});

// Multiple values in query
// GET /filter?tags=node&tags=express&tags=api
app.get('/filter', (req, res) => {
  let { tags } = req.query;
  
  // Ensure tags is always an array
  if (typeof tags === 'string') {
    tags = [tags];
  }
  
  res.json({ tags: tags || [] });
});
```

### Express Router
```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

// Middleware specific to this router
router.use((req, res, next) => {
  console.log('User route accessed:', new Date().toISOString());
  next();
});

// Routes
router.get('/', (req, res) => {
  res.json({ message: 'Get all users' });
});

router.get('/:id', (req, res) => {
  res.json({ message: `Get user ${req.params.id}` });
});

router.post('/', (req, res) => {
  res.status(201).json({ message: 'User created' });
});

router.put('/:id', (req, res) => {
  res.json({ message: `User ${req.params.id} updated` });
});

router.delete('/:id', (req, res) => {
  res.json({ message: `User ${req.params.id} deleted` });
});

module.exports = router;

// server.js
const userRoutes = require('./routes/users');
app.use('/users', userRoutes);
```

### Route Organization
```javascript
// routes/index.js
const express = require('express');
const router = express.Router();

const userRoutes = require('./users');
const postRoutes = require('./posts');
const authRoutes = require('./auth');

router.use('/users', userRoutes);
router.use('/posts', postRoutes);
router.use('/auth', authRoutes);

// API versioning
router.use('/v1', require('./v1'));
router.use('/v2', require('./v2'));

module.exports = router;

// server.js
const routes = require('./routes');
app.use('/api', routes);
```

---

## Middleware

### Built-in Middleware
```javascript
const express = require('express');
const app = express();

// Parse JSON bodies
app.use(express.json({
  limit: '10mb',
  type: 'application/json'
}));

// Parse URL-encoded bodies
app.use(express.urlencoded({
  extended: true,
  limit: '10mb'
}));

// Serve static files
app.use(express.static('public'));
app.use('/uploads', express.static('uploads'));

// Custom static options
app.use('/assets', express.static('public', {
  maxAge: '1d',
  etag: false
}));
```

### Third-party Middleware
```javascript
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
const compression = require('compression');
const rateLimit = require('express-rate-limit');

// CORS
app.use(cors({
  origin: ['http://localhost:3000', 'https://myapp.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));

// Security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"]
    }
  }
}));

// Logging
app.use(morgan('combined'));
// Custom log format
app.use(morgan(':method :url :status :response-time ms - :res[content-length]'));

// Compression
app.use(compression());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});
app.use('/api/', limiter);

// Strict rate limiting for auth routes
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true
});
app.use('/api/auth', authLimiter);
```

### Custom Middleware
```javascript
// Logging middleware
const logger = (req, res, next) => {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.url}`);
  next();
};

// Authentication middleware
const authenticate = (req, res, next) => {
  const token = req.header('Authorization')?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'Access denied. No token provided.' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token.' });
  }
};

// Authorization middleware
const authorize = (roles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Access denied. No user found.' });
    }
    
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Access denied. Insufficient permissions.' });
    }
    
    next();
  };
};

// Request ID middleware
const requestId = (req, res, next) => {
  req.id = Math.random().toString(36).substr(2, 9);
  res.setHeader('X-Request-ID', req.id);
  next();
};

// Response time middleware
const responseTime = (req, res, next) => {
  const startTime = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - startTime;
    console.log(`${req.method} ${req.url} - ${duration}ms`);
  });
  
  next();
};

// Validation middleware
const validateUser = (req, res, next) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ 
      error: 'Name and email are required' 
    });
  }
  
  if (!/\S+@\S+\.\S+/.test(email)) {
    return res.status(400).json({ 
      error: 'Invalid email format' 
    });
  }
  
  next();
};

// Usage
app.use(logger);
app.use(requestId);
app.use(responseTime);

app.post('/users', validateUser, (req, res) => {
  // Route handler
});

app.get('/protected', authenticate, authorize(['admin', 'user']), (req, res) => {
  res.json({ message: 'Protected route accessed', user: req.user });
});
```

### Error Handling Middleware
```javascript
// 404 handler
const notFound = (req, res, next) => {
  const error = new Error(`Route not found - ${req.originalUrl}`);
  error.status = 404;
  next(error);
};

// Global error handler
const errorHandler = (err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;

  // Log error
  console.error(err);

  // Mongoose bad ObjectId
  if (err.name === 'CastError') {
    const message = 'Resource not found';
    error = { message, status: 404 };
  }

  // Mongoose duplicate key
  if (err.code === 11000) {
    const message = 'Duplicate field value entered';
    error = { message, status: 400 };
  }

  // Mongoose validation error
  if (err.name === 'ValidationError') {
    const message = Object.values(err.errors).map(val => val.message);
    error = { message, status: 400 };
  }

  // JWT errors
  if (err.name === 'JsonWebTokenError') {
    const message = 'Invalid token';
    error = { message, status: 401 };
  }

  if (err.name === 'TokenExpiredError') {
    const message = 'Token expired';
    error = { message, status: 401 };
  }

  res.status(error.status || 500).json({
    success: false,
    message: error.message || 'Server Error',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
};

// Usage
app.use(notFound);
app.use(errorHandler);
```

---

## Request & Response

### Request Object
```javascript
app.get('/request-info', (req, res) => {
  console.log('Request details:');
  
  // Basic properties
  console.log('Method:', req.method);
  console.log('URL:', req.url);
  console.log('Path:', req.path);
  console.log('Original URL:', req.originalUrl);
  console.log('Base URL:', req.baseUrl);
  console.log('Protocol:', req.protocol);
  console.log('Secure:', req.secure);
  console.log('IP:', req.ip);
  console.log('IPs:', req.ips);
  console.log('Hostname:', req.hostname);
  
  // Headers
  console.log('Headers:', req.headers);
  console.log('User-Agent:', req.get('User-Agent'));
  console.log('Authorization:', req.get('Authorization'));
  
  // Parameters and query
  console.log('Params:', req.params);
  console.log('Query:', req.query);
  console.log('Body:', req.body);
  
  // Cookies (requires cookie-parser middleware)
  console.log('Cookies:', req.cookies);
  
  res.json({ message: 'Check console for request details' });
});

// Request with file upload info
app.post('/upload', (req, res) => {
  console.log('Files:', req.files); // requires multer
  console.log('Body:', req.body);
  res.json({ message: 'Upload received' });
});
```

### Response Methods
```javascript
app.get('/response-examples', (req, res) => {
  // Basic response methods
  
  // Send JSON
  // res.json({ message: 'Hello World' });
  
  // Send text
  // res.send('Hello World');
  
  // Send with status
  // res.status(201).json({ message: 'Created' });
  
  // Send file
  // res.sendFile(path.join(__dirname, 'files', 'document.pdf'));
  
  // Download file
  // res.download('path/to/file.pdf', 'filename.pdf');
  
  // Redirect
  // res.redirect('/new-path');
  // res.redirect(301, '/new-path'); // permanent redirect
  
  // Set headers
  res.set('X-Custom-Header', 'MyValue');
  res.setHeader('Content-Type', 'application/json');
  
  // Set cookies
  res.cookie('sessionId', '123456', {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  });
  
  // Clear cookie
  // res.clearCookie('sessionId');
  
  res.json({ message: 'Response sent with headers and cookies' });
});

// Streaming response
app.get('/stream', (req, res) => {
  res.setHeader('Content-Type', 'text/plain');
  res.setHeader('Transfer-Encoding', 'chunked');
  
  let counter = 0;
  const interval = setInterval(() => {
    res.write(`Chunk ${counter}\n`);
    counter++;
    
    if (counter > 10) {
      clearInterval(interval);
      res.end('Stream finished\n');
    }
  }, 1000);
});

// Server-Sent Events
app.get('/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  res.setHeader('Access-Control-Allow-Origin', '*');
  
  // Send initial event
  res.write('data: Connected to event stream\n\n');
  
  // Send periodic updates
  const interval = setInterval(() => {
    const data = {
      timestamp: new Date().toISOString(),
      message: 'Hello from server'
    };
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  }, 5000);
  
  // Clean up on client disconnect
  req.on('close', () => {
    clearInterval(interval);
    res.end();
  });
});
```

### Content Negotiation
```javascript
app.get('/content', (req, res) => {
  const data = { message: 'Hello World', timestamp: new Date() };
  
  res.format({
    'text/plain': () => {
      res.send(`${data.message} at ${data.timestamp}`);
    },
    
    'text/html': () => {
      res.send(`<h1>${data.message}</h1><p>Time: ${data.timestamp}</p>`);
    },
    
    'application/json': () => {
      res.json(data);
    },
    
    'application/xml': () => {
      const xml = `
        <response>
          <message>${data.message}</message>
          <timestamp>${data.timestamp}</timestamp>
        </response>
      `;
      res.type('xml').send(xml);
    },
    
    default: () => {
      res.status(406).json({ error: 'Not Acceptable' });
    }
  });
});
```

---

## Error Handling

### Synchronous Error Handling
```javascript
app.get('/sync-error', (req, res, next) => {
  try {
    // Potentially throwing operation
    const result = riskyOperation();
    res.json({ result });
  } catch (error) {
    next(error); // Pass error to error handler
  }
});

// Or using wrapper function
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/wrapped-route', asyncHandler(async (req, res) => {
  const result = await someAsyncOperation();
  res.json({ result });
}));
```

### Asynchronous Error Handling
```javascript
// Manual error handling
app.get('/async-error', async (req, res, next) => {
  try {
    const data = await fetchDataFromDatabase();
    res.json(data);
  } catch (error) {
    next(error);
  }
});

// Custom async wrapper
const catchAsync = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
};

app.get('/users/:id', catchAsync(async (req, res, next) => {
  const user = await User.findById(req.params.id);
  if (!user) {
    const error = new Error('User not found');
    error.statusCode = 404;
    return next(error);
  }
  res.json(user);
}));
```

### Custom Error Classes
```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
    this.isOperational = true;

    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message) {
    super(message, 400);
  }
}

class NotFoundError extends AppError {
  constructor(resource = 'Resource') {
    super(`${resource} not found`, 404);
  }
}

class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized access') {
    super(message, 401);
  }
}

// Usage
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      throw new NotFoundError('User');
    }
    res.json(user);
  } catch (error) {
    next(error);
  }
});
```

### Comprehensive Error Handler
```javascript
const sendErrorDev = (err, res) => {
  res.status(err.statusCode || 500).json({
    status: err.status || 'error',
    error: err,
    message: err.message,
    stack: err.stack
  });
};

const sendErrorProd = (err, res) => {
  // Operational errors: send message to client
  if (err.isOperational) {
    res.status(err.statusCode).json({
      status: err.status,
      message: err.message
    });
  } else {
    // Programming errors: don't leak error details
    console.error('ERROR:', err);
    res.status(500).json({
      status: 'error',
      message: 'Something went wrong!'
    });
  }
};

const globalErrorHandler = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  if (process.env.NODE_ENV === 'development') {
    sendErrorDev(err, res);
  } else {
    let error = { ...err };
    error.message = err.message;

    // Handle specific error types
    if (error.name === 'CastError') {
      error = new AppError('Invalid ID format', 400);
    }
    
    if (error.code === 11000) {
      const value = err.errmsg.match(/(["'])(\\?.)*?\1/)[0];
      error = new AppError(`Duplicate field value: ${value}`, 400);
    }
    
    if (error.name === 'ValidationError') {
      const errors = Object.values(err.errors).map(el => el.message);
      error = new AppError(`Invalid input data: ${errors.join('. ')}`, 400);
    }

    sendErrorProd(error, res);
  }
};

module.exports = globalErrorHandler;
```

---

## Static Files

### Basic Static File Serving
```javascript
const express = require('express');
const path = require('path');
const app = express();

// Serve static files from 'public' directory
app.use(express.static('public'));

// Serve with virtual path prefix
app.use('/static', express.static('public'));

// Multiple static directories
app.use(express.static('public'));
app.use(express.static('files'));

// Static files with options
app.use('/assets', express.static('public', {
  maxAge: '1d', // Cache for 1 day
  etag: false,
  index: false, // Disable directory indexing
  dotfiles: 'ignore', // Ignore dotfiles
  setHeaders: (res, path, stat) => {
    res.set('X-Timestamp', Date.now());
  }
}));
```

### File Downloads
```javascript
app.get('/download/:filename', (req, res) => {
  const { filename } = req.params;
  const filePath = path.join(__dirname, 'downloads', filename);
  
  // Check if file exists
  if (!fs.existsSync(filePath)) {
    return res.status(404).json({ error: 'File not found' });
  }
  
  // Download with original filename
  res.download(filePath);
  
  // Download with custom filename
  // res.download(filePath, 'custom-name.pdf');
  
  // Download with callback
  // res.download(filePath, (err) => {
  //   if (err) {
  //     console.error('Download error:', err);
  //   } else {
  //     console.log('File downloaded successfully');
  //   }
  // });
});

// Send file (doesn't prompt download)
app.get('/view/:filename', (req, res) => {
  const { filename } = req.params;
  const filePath = path.join(__dirname, 'files', filename);
  
  res.sendFile(filePath, (err) => {
    if (err) {
      res.status(404).json({ error: 'File not found' });
    }
  });
});
```

### Image Serving with Processing
```javascript
const sharp = require('sharp'); // npm install sharp

app.get('/images/:filename', (req, res) => {
  const { filename } = req.params;
  const { width, height, quality } = req.query;
  
  const imagePath = path.join(__dirname, 'images', filename);
  
  if (!fs.existsSync(imagePath)) {
    return res.status(404).json({ error: 'Image not found' });
  }
  
  let transform = sharp(imagePath);
  
  if (width || height) {
    transform = transform.resize(
      parseInt(width) || null,
      parseInt(height) || null,
      { fit: 'inside', withoutEnlargement: true }
    );
  }
  
  if (quality) {
    transform = transform.jpeg({ quality: parseInt(quality) });
  }
  
  res.setHeader('Content-Type', 'image/jpeg');
  transform.pipe(res);
});
```

---

## Template Engines

### EJS Templates
```javascript
// npm install ejs
const express = require('express');
const app = express();

// Set view engine
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Route with template
app.get('/', (req, res) => {
  const data = {
    title: 'My App',
    user: { name: 'John Doe', email: 'john@example.com' },
    posts: [
      { title: 'Post 1', content: 'Content 1' },
      { title: 'Post 2', content: 'Content 2' }
    ]
  };
  
  res.render('index', data);
});

// Dynamic template selection
app.get('/admin', (req, res) => {
  const template = req.user?.isAdmin ? 'admin' : 'user';
  res.render(template, { user: req.user });
});
```

```html
<!-- views/index.ejs -->
<!DOCTYPE html>
<html>
<head>
    <title><%= title %></title>
</head>
<body>
    <h1>Welcome, <%= user.name %>!</h1>
    <p>Email: <%= user.email %></p>
    
    <h2>Posts</h2>
    <% posts.forEach(post => { %>
        <article>
            <h3><%= post.title %></h3>
            <p><%= post.content %></p>
        </article>
    <% }) %>
    
    <!-- Include partial -->
    <%- include('partials/footer') %>
</body>
</html>
```

### Handlebars Templates
```javascript
// npm install express-handlebars
const exphbs = require('express-handlebars');

// Configure Handlebars
app.engine('handlebars', exphbs.engine({
  defaultLayout: 'main',
  layoutsDir: path.join(__dirname, 'views/layouts/'),
  partialsDir: path.join(__dirname, 'views/partials/'),
  helpers: {
    formatDate: (date) => new Date(date).toLocaleDateString(),
    uppercase: (str) => str.toUpperCase(),
    eq: (a, b) => a === b
  }
}));

app.set('view engine', 'handlebars');

app.get('/', (req, res) => {
  res.render('home', {
    title: 'Home Page',
    users: [
      { name: 'John', active: true },
      { name: 'Jane', active: false }
    ]
  });
});
```

```html
<!-- views/layouts/main.handlebars -->
<!DOCTYPE html>
<html>
<head>
    <title>{{title}}</title>
</head>
<body>
    {{{body}}}
</body>
</html>

<!-- views/home.handlebars -->
<h1>{{title}}</h1>
{{#each users}}
    <div>
        <h3>{{uppercase name}}</h3>
        {{#if active}}
            <span>Active User</span>
        {{else}}
            <span>Inactive User</span>
        {{/if}}
    </div>
{{/each}}
```

### Pug Templates
```javascript
// npm install pug
app.set('view engine', 'pug');
app.set('views', path.join(__dirname, 'views'));

app.get('/pug-example', (req, res) => {
  res.render('example', {
    title: 'Pug Example',
    message: 'Hello from Pug!',
    items: ['Item 1', 'Item 2', 'Item 3']
  });
});
```

```pug
//- views/example.pug
doctype html
html
  head
    title= title
  body
    h1= message
    ul
      each item in items
        li= item
```

---

## Database Integration

### MongoDB with Mongoose
```javascript
// npm install mongoose
const mongoose = require('mongoose');

// Connection
mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// Schema definition
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    maxlength: 50
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    validate: {
      validator: (email) => /\S+@\S+\.\S+/.test(email),
      message: 'Invalid email format'
    }
  },
  age: {
    type: Number,
    min: 0,
    max: 120
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
}, {
  timestamps: true
});

// Instance methods
userSchema.methods.getFullName = function() {
  return `${this.firstName} ${this.lastName}`;
};

// Static methods
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email });
};

// Middleware
userSchema.pre('save', async function(next) {
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 12);
  }
  next();
});

const User = mongoose.model('User', userSchema);

// CRUD Operations
app.get('/users', async (req, res, next) => {
  try {
    const { page = 1, limit = 10, sortBy = 'createdAt' } = req.query;
    
    const users = await User
      .find()
      .sort({ [sortBy]: -1 })
      .limit(limit * 1)
      .skip((page - 1) * limit)
      .select('-password');
    
    const total = await User.countDocuments();
    
    res.json({
      users,
      totalPages: Math.ceil(total / limit),
      currentPage: page,
      total
    });
  } catch (error) {
    next(error);
  }
});

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id).select('-password');
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    next(error);
  }
});

app.post('/users', async (req, res, next) => {
  try {
    const user = new User(req.body);
    await user.save();
    res.status(201).json(user);
  } catch (error) {
    next(error);
  }
});

app.put('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    );
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    next(error);
  }
});

app.delete('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json({ message: 'User deleted successfully' });
  } catch (error) {
    next(error);
  }
});
```

### PostgreSQL with pg
```javascript
// npm install pg
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: process.env.NODE_ENV === 'production' ? { rejectUnauthorized: false } : false
});

// Database helper functions
const query = (text, params) => pool.query(text, params);

// CRUD Operations
app.get('/users', async (req, res, next) => {
  try {
    const { page = 1, limit = 10 } = req.query;
    const offset = (page - 1) * limit;
    
    const result = await query(
      'SELECT id, name, email, created_at FROM users ORDER BY created_at DESC LIMIT $1 OFFSET $2',
      [limit, offset]
    );
    
    const countResult = await query('SELECT COUNT(*) FROM users');
    const total = parseInt(countResult.rows[0].count);
    
    res.json({
      users: result.rows,
      totalPages: Math.ceil(total / limit),
      currentPage: parseInt(page),
      total
    });
  } catch (error) {
    next(error);
  }
});

app.get('/users/:id', async (req, res, next) => {
  try {
    const { id } = req.params;
    const result = await query(
      'SELECT id, name, email, created_at FROM users WHERE id = $1',
      [id]
    );
    
    if (result.rows.length === 0) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json(result.rows[0]);
  } catch (error) {
    next(error);
  }
});

app.post('/users', async (req, res, next) => {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN');
    
    const { name, email, password } = req.body;
    const hashedPassword = await bcrypt.hash(password, 12);
    
    const result = await client.query(
      'INSERT INTO users (name, email, password) VALUES ($1, $2, $3) RETURNING id, name, email, created_at',
      [name, email, hashedPassword]
    );
    
    await client.query('COMMIT');
    
    res.status(201).json(result.rows[0]);
  } catch (error) {
    await client.query('ROLLBACK');
    next(error);
  } finally {
    client.release();
  }
});

// Connection handling
pool.on('connect', () => {
  console.log('Connected to PostgreSQL database');
});

pool.on('error', (err) => {
  console.error('Unexpected error on idle client', err);
  process.exit(-1);
});
```

### MySQL with mysql2
```javascript
// npm install mysql2
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

// Database operations
app.get('/users', async (req, res, next) => {
  try {
    const [rows] = await pool.execute(
      'SELECT id, name, email, created_at FROM users ORDER BY created_at DESC'
    );
    res.json(rows);
  } catch (error) {
    next(error);
  }
});

app.post('/users', async (req, res, next) => {
  const connection = await pool.getConnection();
  
  try {
    await connection.beginTransaction();
    
    const { name, email, password } = req.body;
    const hashedPassword = await bcrypt.hash(password, 12);
    
    const [result] = await connection.execute(
      'INSERT INTO users (name, email, password) VALUES (?, ?, ?)',
      [name, email, hashedPassword]
    );
    
    const [user] = await connection.execute(
      'SELECT id, name, email, created_at FROM users WHERE id = ?',
      [result.insertId]
    );
    
    await connection.commit();
    
    res.status(201).json(user[0]);
  } catch (error) {
    await connection.rollback();
    next(error);
  } finally {
    connection.release();
  }
});
```

---

## Authentication & Authorization

### JWT Authentication
```javascript
// npm install jsonwebtoken bcryptjs
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

// Register user
app.post('/auth/register', async (req, res, next) => {
  try {
    const { name, email, password } = req.body;
    
    // Check if user already exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(400).json({ error: 'User already exists' });
    }
    
    // Hash password
    const hashedPassword = await bcrypt.hash(password, 12);
    
    // Create user
    const user = new User({
      name,
      email,
      password: hashedPassword
    });
    
    await user.save();
    
    // Generate token
    const token = jwt.sign(
      { userId: user._id, email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    );
    
    res.status(201).json({
      message: 'User created successfully',
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email
      }
    });
  } catch (error) {
    next(error);
  }
});

// Login user
app.post('/auth/login', async (req, res, next) => {
  try {
    const { email, password } = req.body;
    
    // Find user
    const user = await User.findOne({ email }).select('+password');
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Check password
    const isValidPassword = await bcrypt.compare(password, user.password);
    if (!isValidPassword) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Generate token
    const token = jwt.sign(
      { userId: user._id, email: user.email, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    );
    
    // Set cookie
    res.cookie('token', token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
    });
    
    res.json({
      message: 'Login successful',
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role
      }
    });
  } catch (error) {
    next(error);
  }
});

// Logout
app.post('/auth/logout', (req, res) => {
  res.clearCookie('token');
  res.json({ message: 'Logout successful' });
});

// Refresh token
app.post('/auth/refresh', async (req, res, next) => {
  try {
    const { refreshToken } = req.body;
    
    if (!refreshToken) {
      return res.status(401).json({ error: 'Refresh token required' });
    }
    
    const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
    const user = await User.findById(decoded.userId);
    
    if (!user) {
      return res.status(401).json({ error: 'User not found' });
    }
    
    const newToken = jwt.sign(
      { userId: user._id, email: user.email, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );
    
    res.json({ token: newToken });
  } catch (error) {
    next(error);
  }
});
```

### Authentication Middleware
```javascript
// JWT middleware
const authenticateToken = async (req, res, next) => {
  try {
    // Get token from header or cookie
    let token = req.header('Authorization')?.replace('Bearer ', '');
    if (!token && req.cookies?.token) {
      token = req.cookies.token;
    }
    
    if (!token) {
      return res.status(401).json({ error: 'Access denied. No token provided.' });
    }
    
    // Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // Get user from database
    const user = await User.findById(decoded.userId).select('-password');
    if (!user) {
      return res.status(401).json({ error: 'Invalid token. User not found.' });
    }
    
    req.user = user;
    next();
  } catch (error) {
    if (error.name === 'JsonWebTokenError') {
      return res.status(401).json({ error: 'Invalid token.' });
    }
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired.' });
    }
    next(error);
  }
};

// Authorization middleware
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Access denied. Please login.' });
    }
    
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ 
        error: 'Access denied. Insufficient permissions.' 
      });
    }
    
    next();
  };
};

// Resource ownership middleware
const checkOwnership = (Model, resourceParam = 'id') => {
  return async (req, res, next) => {
    try {
      const resource = await Model.findById(req.params[resourceParam]);
      
      if (!resource) {
        return res.status(404).json({ error: 'Resource not found' });
      }
      
      // Check if user owns the resource or is admin
      if (resource.userId.toString() !== req.user._id.toString() && 
          req.user.role !== 'admin') {
        return res.status(403).json({ 
          error: 'Access denied. You can only access your own resources.' 
        });
      }
      
      req.resource = resource;
      next();
    } catch (error) {
      next(error);
    }
  };
};

// Usage examples
app.get('/profile', authenticateToken, (req, res) => {
  res.json({ user: req.user });
});

app.get('/admin', authenticateToken, authorize('admin'), (req, res) => {
  res.json({ message: 'Admin access granted' });
});

app.get('/posts/:id', 
  authenticateToken, 
  checkOwnership(Post, 'id'), 
  (req, res) => {
    res.json({ post: req.resource });
  }
);
```

### Session-based Authentication
```javascript
// npm install express-session connect-mongo
const session = require('express-session');
const MongoStore = require('connect-mongo');

// Session configuration
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  store: MongoStore.create({
    mongoUrl: process.env.MONGODB_URI,
    touchAfter: 24 * 3600 // lazy session update
  }),
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}));

// Session-based login
app.post('/login', async (req, res, next) => {
  try {
    const { email, password } = req.body;
    
    const user = await User.findOne({ email }).select('+password');
    if (!user || !await bcrypt.compare(password, user.password)) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    req.session.userId = user._id;
    req.session.user = {
      id: user._id,
      name: user.name,
      email: user.email,
      role: user.role
    };
    
    res.json({ message: 'Login successful', user: req.session.user });
  } catch (error) {
    next(error);
  }
});

// Session middleware
const requireAuth = (req, res, next) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  next();
};

// Logout
app.post('/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({ error: 'Could not log out' });
    }
    res.clearCookie('connect.sid');
    res.json({ message: 'Logout successful' });
  });
});
```

---

## Validation

### Express Validator
```javascript
// npm install express-validator
const { body, param, query, validationResult } = require('express-validator');

// Validation middleware
const validate = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      error: 'Validation failed',
      details: errors.array()
    });
  }
  next();
};

// User validation rules
const userValidationRules = () => {
  return [
    body('name')
      .trim()
      .isLength({ min: 2, max: 50 })
      .withMessage('Name must be between 2 and 50 characters')
      .matches(/^[a-zA-Z\s]+$/)
      .withMessage('Name can only contain letters and spaces'),
    
    body('email')
      .isEmail()
      .normalizeEmail()
      .withMessage('Please provide a valid email')
      .custom(async (email) => {
        const user = await User.findOne({ email });
        if (user) {
          throw new Error('Email already in use');
        }
      }),
    
    body('password')
      .isLength({ min: 8 })
      .withMessage('Password must be at least 8 characters long')
      .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
      .withMessage('Password must contain at least one lowercase letter, one uppercase letter, one number, and one special character'),
    
    body('confirmPassword')
      .custom((value, { req }) => {
        if (value !== req.body.password) {
          throw new Error('Passwords do not match');
        }
        return true;
      }),
    
    body('age')
      .optional()
      .isInt({ min: 13, max: 120 })
      .withMessage('Age must be between 13 and 120'),
    
    body('role')
      .optional()
      .isIn(['user', 'admin'])
      .withMessage('Role must be either user or admin')
  ];
};

// Parameter validation
const idValidation = () => {
  return [
    param('id')
      .isMongoId()
      .withMessage('Invalid ID format')
  ];
};

// Query validation
const paginationValidation = () => {
  return [
    query('page')
      .optional()
      .isInt({ min: 1 })
      .withMessage('Page must be a positive integer'),
    
    query('limit')
      .optional()
      .isInt({ min: 1, max: 100 })
      .withMessage('Limit must be between 1 and 100'),
    
    query('sortBy')
      .optional()
      .isIn(['name', 'email', 'createdAt'])
      .withMessage('Invalid sort field')
  ];
};

// Usage
app.post('/users', 
  userValidationRules(), 
  validate, 
  async (req, res, next) => {
    try {
      const user = new User(req.body);
      await user.save();
      res.status(201).json(user);
    } catch (error) {
      next(error);
    }
  }
);

app.get('/users/:id', 
  idValidation(), 
  validate, 
  async (req, res, next) => {
    // Route handler
  }
);

app.get('/users', 
  paginationValidation(), 
  validate, 
  async (req, res, next) => {
    // Route handler
  }
);
```

### Custom Validation Middleware
```javascript
// Custom validation functions
const validateUser = (req, res, next) => {
  const { name, email, password } = req.body;
  const errors = [];
  
  // Name validation
  if (!name || name.trim().length === 0) {
    errors.push({ field: 'name', message: 'Name is required' });
  } else if (name.trim().length < 2 || name.trim().length > 50) {
    errors.push({ field: 'name', message: 'Name must be between 2 and 50 characters' });
  }
  
  // Email validation
  if (!email || email.trim().length === 0) {
    errors.push({ field: 'email', message: 'Email is required' });
  } else if (!/\S+@\S+\.\S+/.test(email)) {
    errors.push({ field: 'email', message: 'Invalid email format' });
  }
  
  // Password validation
  if (!password) {
    errors.push({ field: 'password', message: 'Password is required' });
  } else if (password.length < 8) {
    errors.push({ field: 'password', message: 'Password must be at least 8 characters' });
  } else if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(password)) {
    errors.push({ 
      field: 'password', 
      message: 'Password must contain at least one lowercase letter, one uppercase letter, and one number' 
    });
  }
  
  if (errors.length > 0) {
    return res.status(400).json({ 
      error: 'Validation failed',
      details: errors 
    });
  }
  
  next();
};

// Sanitization middleware
const sanitizeUser = (req, res, next) => {
  if (req.body.name) {
    req.body.name = req.body.name.trim();
  }
  if (req.body.email) {
    req.body.email = req.body.email.trim().toLowerCase();
  }
  next();
};

// Usage
app.post('/users', sanitizeUser, validateUser, async (req, res, next) => {
  // Route handler
});
```

---

## File Uploads

### Multer for File Uploads
```javascript
// npm install multer
const multer = require('multer');
const path = require('path');
const fs = require('fs').promises;

// Storage configuration
const storage = multer.diskStorage({
  destination: async (req, file, cb) => {
    const uploadPath = path.join(__dirname, 'uploads');
    try {
      await fs.access(uploadPath);
    } catch {
      await fs.mkdir(uploadPath, { recursive: true });
    }
    cb(null, uploadPath);
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
  }
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowedTypes = /jpeg|jpg|png|gif|pdf|doc|docx/;
  const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
  const mimetype = allowedTypes.test(file.mimetype);
  
  if (mimetype && extname) {
    return cb(null, true);
  } else {
    cb(new Error('Invalid file type. Only JPEG, PNG, GIF, PDF, DOC, and DOCX files are allowed.'));
  }
};

// Upload configuration
const upload = multer({
  storage: storage,
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB limit
    files: 5 // Maximum 5 files
  },
  fileFilter: fileFilter
});

// Single file upload
app.post('/upload/single', upload.single('file'), (req, res, next) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }
    
    res.json({
      message: 'File uploaded successfully',
      file: {
        filename: req.file.filename,
        originalname: req.file.originalname,
        size: req.file.size,
        mimetype: req.file.mimetype,
        path: req.file.path
      }
    });
  } catch (error) {
    next(error);
  }
});

// Multiple files upload
app.post('/upload/multiple', upload.array('files', 5), (req, res, next) => {
  try {
    if (!req.files || req.files.length === 0) {
      return res.status(400).json({ error: 'No files uploaded' });
    }
    
    const files = req.files.map(file => ({
      filename: file.filename,
      originalname: file.originalname,
      size: file.size,
      mimetype: file.mimetype,
      path: file.path
    }));
    
    res.json({
      message: 'Files uploaded successfully',
      files
    });
  } catch (error) {
    next(error);
  }
});

// Fields upload (different field names)
app.post('/upload/fields', 
  upload.fields([
    { name: 'avatar', maxCount: 1 },
    { name: 'documents', maxCount: 3 }
  ]), 
  (req, res, next) => {
    try {
      res.json({
        message: 'Files uploaded successfully',
        avatar: req.files.avatar?.[0],
        documents: req.files.documents || []
      });
    } catch (error) {
      next(error);
    }
  }
);

// Image upload with processing
const sharp = require('sharp'); // npm install sharp

app.post('/upload/image', upload.single('image'), async (req, res, next) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No image uploaded' });
    }
    
    const { buffer, mimetype } = req.file;
    
    // Process image
    const processedImage = await sharp(buffer)
      .resize(800, 600, { fit: 'inside', withoutEnlargement: true })
      .jpeg({ quality: 85 })
      .toBuffer();
    
    // Save processed image
    const filename = `processed-${Date.now()}.jpg`;
    const filepath = path.join(__dirname, 'uploads', filename);
    await fs.writeFile(filepath, processedImage);
    
    res.json({
      message: 'Image uploaded and processed successfully',
      filename,
      originalSize: buffer.length,
      processedSize: processedImage.length
    });
  } catch (error) {
    next(error);
  }
});
```



### Cloud Storage (AWS S3) - Complete
```javascript
// npm install aws-sdk multer-s3
const AWS = require('aws-sdk');
const multerS3 = require('multer-s3');

// Configure AWS
AWS.config.update({
  accessKeyId: process.env.AWS_ACCESS_KEY_ID,
  secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  region: process.env.AWS_REGION
});

const s3 = new AWS.S3();

// S3 upload configuration
const s3Upload = multer({
  storage: multerS3({
    s3: s3,
    bucket: process.env.AWS_S3_BUCKET,
    key: (req, file, cb) => {
      const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
      cb(null, `uploads/${uniqueSuffix}-${file.originalname}`);
    },
    contentType: multerS3.AUTO_CONTENT_TYPE,
    metadata: (req, file, cb) => {
      cb(null, { fieldName: file.fieldname });
    }
  }),
  limits: {
    fileSize: 10 * 1024 * 1024 // 10MB
  }
});

// Upload to S3
app.post('/upload/s3', s3Upload.single('file'), (req, res) => {
  res.json({
    message: 'File uploaded to S3 successfully',
    file: {
      filename: req.file.key,
      location: req.file.location,
      size: req.file.size
    }
  });
});

// Delete file from S3
app.delete('/files/:key', async (req, res, next) => {
  try {
    await s3.deleteObject({
      Bucket: process.env.AWS_S3_BUCKET,
      Key: req.params.key
    }).promise();
    
    res.json({ message: 'File deleted successfully' });
  } catch (error) {
    next(error);
  }
});
```

### File Upload Error Handling
```javascript
// Global upload error handler
app.use((error, req, res, next) => {
  if (error instanceof multer.MulterError) {
    switch (error.code) {
      case 'LIMIT_FILE_SIZE':
        return res.status(400).json({
          error: 'File too large',
          maxSize: '5MB'
        });
      case 'LIMIT_FILE_COUNT':
        return res.status(400).json({
          error: 'Too many files',
          maxFiles: 5
        });
      case 'LIMIT_UNEXPECTED_FILE':
        return res.status(400).json({
          error: 'Unexpected field name'
        });
      default:
        return res.status(400).json({
          error: 'File upload error',
          details: error.message
        });
    }
  }
  next(error);
});
```

---

## API Design

### RESTful API Structure
```javascript
// Users API - Complete CRUD
const express = require('express');
const router = express.Router();

// GET /api/users - Get all users with pagination and filtering
router.get('/', async (req, res, next) => {
  try {
    const {
      page = 1,
      limit = 10,
      sortBy = 'createdAt',
      sortOrder = 'desc',
      search,
      role,
      status
    } = req.query;

    // Build query
    const query = {};
    if (search) {
      query.$or = [
        { name: { $regex: search, $options: 'i' } },
        { email: { $regex: search, $options: 'i' } }
      ];
    }
    if (role) query.role = role;
    if (status) query.status = status;

    // Execute query with pagination
    const users = await User
      .find(query)
      .select('-password')
      .sort({ [sortBy]: sortOrder === 'desc' ? -1 : 1 })
      .limit(limit * 1)
      .skip((page - 1) * limit);

    const total = await User.countDocuments(query);

    res.json({
      success: true,
      data: users,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total,
        pages: Math.ceil(total / limit)
      },
      meta: {
        totalUsers: total,
        activeUsers: await User.countDocuments({ ...query, status: 'active' })
      }
    });
  } catch (error) {
    next(error);
  }
});

// GET /api/users/:id - Get single user
router.get('/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id)
      .select('-password')
      .populate('posts', 'title createdAt');
    
    if (!user) {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }

    res.json({
      success: true,
      data: user
    });
  } catch (error) {
    next(error);
  }
});

// POST /api/users - Create new user
router.post('/', async (req, res, next) => {
  try {
    const user = new User(req.body);
    await user.save();

    // Remove password from response
    const userResponse = user.toObject();
    delete userResponse.password;

    res.status(201).json({
      success: true,
      message: 'User created successfully',
      data: userResponse
    });
  } catch (error) {
    next(error);
  }
});

// PUT /api/users/:id - Update user (full update)
router.put('/:id', async (req, res, next) => {
  try {
    const user = await User.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    ).select('-password');

    if (!user) {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }

    res.json({
      success: true,
      message: 'User updated successfully',
      data: user
    });
  } catch (error) {
    next(error);
  }
});

// PATCH /api/users/:id - Partial update
router.patch('/:id', async (req, res, next) => {
  try {
    const updates = Object.keys(req.body);
    const allowedUpdates = ['name', 'email', 'role', 'status'];
    const isValidUpdate = updates.every(update => allowedUpdates.includes(update));

    if (!isValidUpdate) {
      return res.status(400).json({
        success: false,
        message: 'Invalid updates',
        allowedFields: allowedUpdates
      });
    }

    const user = await User.findByIdAndUpdate(
      req.params.id,
      { $set: req.body },
      { new: true, runValidators: true }
    ).select('-password');

    if (!user) {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }

    res.json({
      success: true,
      message: 'User updated successfully',
      data: user
    });
  } catch (error) {
    next(error);
  }
});

// DELETE /api/users/:id - Delete user
router.delete('/:id', async (req, res, next) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);

    if (!user) {
      return res.status(404).json({
        success: false,
        message: 'User not found'
      });
    }

    res.json({
      success: true,
      message: 'User deleted successfully'
    });
  } catch (error) {
    next(error);
  }
});

module.exports = router;
```

### API Versioning
```javascript
// Version 1 API
// routes/v1/users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({ version: '1.0', message: 'Users API v1' });
});

module.exports = router;

// Version 2 API
// routes/v2/users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({ 
    version: '2.0', 
    message: 'Users API v2 - Enhanced',
    features: ['pagination', 'filtering', 'sorting']
  });
});

module.exports = router;

// Main app setup
app.use('/api/v1/users', require('./routes/v1/users'));
app.use('/api/v2/users', require('./routes/v2/users'));

// Default to latest version
app.use('/api/users', require('./routes/v2/users'));

// Version detection middleware
const apiVersion = (req, res, next) => {
  // From header
  let version = req.get('API-Version') || req.get('Accept-Version');
  
  // From query parameter
  if (!version && req.query.version) {
    version = req.query.version;
  }
  
  // Default version
  if (!version) {
    version = '2.0';
  }
  
  req.apiVersion = version;
  next();
};
```

### Response Formatting
```javascript
// Standard API response format
const sendResponse = (res, statusCode, success, message, data = null, meta = null) => {
  const response = {
    success,
    message,
    timestamp: new Date().toISOString(),
    ...(data && { data }),
    ...(meta && { meta })
  };

  res.status(statusCode).json(response);
};

// Success responses
const sendSuccess = (res, message, data, statusCode = 200) => {
  sendResponse(res, statusCode, true, message, data);
};

const sendCreated = (res, message, data) => {
  sendResponse(res, 201, true, message, data);
};

// Error responses
const sendError = (res, message, statusCode = 500, data = null) => {
  sendResponse(res, statusCode, false, message, data);
};

const sendValidationError = (res, errors) => {
  sendResponse(res, 400, false, 'Validation failed', { errors });
};

// Middleware to add response helpers
app.use((req, res, next) => {
  res.sendSuccess = (message, data, statusCode) => sendSuccess(res, message, data, statusCode);
  res.sendCreated = (message, data) => sendCreated(res, message, data);
  res.sendError = (message, statusCode, data) => sendError(res, message, statusCode, data);
  res.sendValidationError = (errors) => sendValidationError(res, errors);
  next();
});

// Usage in routes
app.get('/users', async (req, res, next) => {
  try {
    const users = await User.find();
    res.sendSuccess('Users retrieved successfully', users);
  } catch (error) {
    next(error);
  }
});
```

### API Documentation with Swagger
```javascript
// npm install swagger-jsdoc swagger-ui-express
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'My API',
      version: '1.0.0',
      description: 'A comprehensive API for managing users and posts',
    },
    servers: [
      {
        url: 'http://localhost:3000/api',
        description: 'Development server',
      },
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT',
        },
      },
      schemas: {
        User: {
          type: 'object',
          required: ['name', 'email'],
          properties: {
            id: {
              type: 'string',
              description: 'User ID',
            },
            name: {
              type: 'string',
              description: 'User name',
            },
            email: {
              type: 'string',
              format: 'email',
              description: 'User email',
            },
            role: {
              type: 'string',
              enum: ['user', 'admin'],
              description: 'User role',
            },
          },
        },
      },
    },
  },
  apis: ['./routes/*.js'], // Path to the API docs
};

const specs = swaggerJsdoc(options);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));

/**
 * @swagger
 * /users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *         description: Page number
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *         description: Number of users per page
 *     responses:
 *       200:
 *         description: List of users
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 */
app.get('/users', getUsersHandler);

/**
 * @swagger
 * /users:
 *   post:
 *     summary: Create a new user
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/User'
 *     responses:
 *       201:
 *         description: User created successfully
 *       400:
 *         description: Validation error
 *       401:
 *         description: Unauthorized
 */
app.post('/users', createUserHandler);
```

---

## Testing

### Jest Testing Setup
```javascript
// npm install --save-dev jest supertest
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "jest": {
    "testEnvironment": "node",
    "collectCoverageFrom": [
      "src/**/*.js",
      "!src/server.js",
      "!src/config/**"
    ]
  }
}

// tests/setup.js
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');

let mongoServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  const mongoUri = mongoServer.getUri();
  await mongoose.connect(mongoUri);
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});

beforeEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    const collection = collections[key];
    await collection.deleteMany({});
  }
});
```

### API Testing
```javascript
// tests/auth.test.js
const request = require('supertest');
const app = require('../src/app');
const User = require('../src/models/User');

describe('Authentication', () => {
  describe('POST /auth/register', () => {
    it('should register a new user', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'Password123!'
      };

      const response = await request(app)
        .post('/auth/register')
        .send(userData)
        .expect(201);

      expect(response.body.success).toBe(true);
      expect(response.body.data.user.email).toBe(userData.email);
      expect(response.body.data.token).toBeDefined();

      // Verify user was saved to database
      const user = await User.findOne({ email: userData.email });
      expect(user).toBeTruthy();
    });

    it('should not register user with invalid email', async () => {
      const userData = {
        name: 'John Doe',
        email: 'invalid-email',
        password: 'Password123!'
      };

      const response = await request(app)
        .post('/auth/register')
        .send(userData)
        .expect(400);

      expect(response.body.success).toBe(false);
      expect(response.body.message).toContain('validation');
    });

    it('should not register user with duplicate email', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'Password123!'
      };

      // Create first user
      await request(app)
        .post('/auth/register')
        .send(userData)
        .expect(201);

      // Try to create duplicate
      const response = await request(app)
        .post('/auth/register')
        .send(userData)
        .expect(400);

      expect(response.body.success).toBe(false);
      expect(response.body.message).toContain('already exists');
    });
  });

  describe('POST /auth/login', () => {
    it('should login with valid credentials', async () => {
      // Create user first
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'Password123!'
      };

      await request(app)
        .post('/auth/register')
        .send(userData);

      // Login
      const response = await request(app)
        .post('/auth/login')
        .send({
          email: userData.email,
          password: userData.password
        })
        .expect(200);

      expect(response.body.success).toBe(true);
      expect(response.body.data.token).toBeDefined();
      expect(response.body.data.user.email).toBe(userData.email);
    });

    it('should not login with invalid password', async () => {
      // Create user first
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'Password123!'
      };

      await request(app)
        .post('/auth/register')
        .send(userData);

      // Try to login with wrong password
      const response = await request(app)
        .post('/auth/login')
        .send({
          email: userData.email,
          password: 'wrongpassword'
        })
        .expect(401);

      expect(response.body.success).toBe(false);
      expect(response.body.message).toContain('Invalid credentials');
    });
  });
});

// tests/users.test.js
describe('Users API', () => {
  let authToken;
  let userId;

  beforeEach(async () => {
    // Create and login user for authentication
    const userData = {
      name: 'Test User',
      email: 'test@example.com',
      password: 'Password123!'
    };

    const registerResponse = await request(app)
      .post('/auth/register')
      .send(userData);

    authToken = registerResponse.body.data.token;
    userId = registerResponse.body.data.user.id;
  });

  describe('GET /users', () => {
    it('should get all users', async () => {
      const response = await request(app)
        .get('/users')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.success).toBe(true);
      expect(Array.isArray(response.body.data)).toBe(true);
    });

    it('should get users with pagination', async () => {
      const response = await request(app)
        .get('/users?page=1&limit=5')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.pagination).toBeDefined();
      expect(response.body.pagination.page).toBe(1);
      expect(response.body.pagination.limit).toBe(5);
    });

    it('should require authentication', async () => {
      const response = await request(app)
        .get('/users')
        .expect(401);

      expect(response.body.success).toBe(false);
    });
  });

  describe('GET /users/:id', () => {
    it('should get user by id', async () => {
      const response = await request(app)
        .get(`/users/${userId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.success).toBe(true);
      expect(response.body.data.id).toBe(userId);
    });

    it('should return 404 for non-existent user', async () => {
      const fakeId = new mongoose.Types.ObjectId();
      const response = await request(app)
        .get(`/users/${fakeId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(404);

      expect(response.body.success).toBe(false);
    });
  });
});
```

### Unit Testing
```javascript
// tests/unit/userService.test.js
const UserService = require('../../src/services/UserService');
const User = require('../../src/models/User');

// Mock the User model
jest.mock('../../src/models/User');

describe('UserService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('createUser', () => {
    it('should create a user successfully', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'hashedpassword'
      };

      const mockUser = { id: '123', ...userData, save: jest.fn() };
      User.mockImplementation(() => mockUser);
      User.findOne.mockResolvedValue(null); // No existing user

      const result = await UserService.createUser(userData);

      expect(User.findOne).toHaveBeenCalledWith({ email: userData.email });
      expect(mockUser.save).toHaveBeenCalled();
      expect(result).toEqual(mockUser);
    });

    it('should throw error if user already exists', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'hashedpassword'
      };

      User.findOne.mockResolvedValue({ id: '456' }); // Existing user

      await expect(UserService.createUser(userData))
        .rejects
        .toThrow('User already exists');

      expect(User.findOne).toHaveBeenCalledWith({ email: userData.email });
    });
  });

  describe('getUserById', () => {
    it('should return user when found', async () => {
      const userId = '123';
      const mockUser = { id: userId, name: 'John Doe' };
      
      User.findById.mockResolvedValue(mockUser);

      const result = await UserService.getUserById(userId);

      expect(User.findById).toHaveBeenCalledWith(userId);
      expect(result).toEqual(mockUser);
    });

    it('should return null when user not found', async () => {
      const userId = '123';
      
      User.findById.mockResolvedValue(null);

      const result = await UserService.getUserById(userId);

      expect(User.findById).toHaveBeenCalledWith(userId);
      expect(result).toBeNull();
    });
  });
});
```

### Testing Middleware
```javascript
// tests/middleware/auth.test.js
const jwt = require('jsonwebtoken');
const { authenticate } = require('../../src/middleware/auth');
const User = require('../../src/models/User');

jest.mock('jsonwebtoken');
jest.mock('../../src/models/User');

describe('Authentication Middleware', () => {
  let req, res, next;

  beforeEach(() => {
    req = {
      header: jest.fn(),
      cookies: {}
    };
    res = {
      status: jest.fn().mockReturnThis(),
      json: jest.fn()
    };
    next = jest.fn();
    jest.clearAllMocks();
  });

  it('should authenticate valid token', async () => {
    const mockUser = { id: '123', name: 'John Doe' };
    const mockDecoded = { userId: '123' };

    req.header.mockReturnValue('Bearer validtoken');
    jwt.verify.mockReturnValue(mockDecoded);
    User.findById.mockResolvedValue(mockUser);

    await authenticate(req, res, next);

    expect(jwt.verify).toHaveBeenCalledWith('validtoken', process.env.JWT_SECRET);
    expect(User.findById).toHaveBeenCalledWith('123');
    expect(req.user).toEqual(mockUser);
    expect(next).toHaveBeenCalledWith();
  });

  it('should reject request without token', async () => {
    req.header.mockReturnValue(null);

    await authenticate(req, res, next);

    expect(res.status).toHaveBeenCalledWith(401);
    expect(res.json).toHaveBeenCalledWith({
      error: 'Access denied. No token provided.'
    });
    expect(next).not.toHaveBeenCalled();
  });

  it('should reject invalid token', async () => {
    req.header.mockReturnValue('Bearer invalidtoken');
    jwt.verify.mockImplementation(() => {
      throw new Error('Invalid token');
    });

    await authenticate(req, res, next);

    expect(res.status).toHaveBeenCalledWith(401);
    expect(res.json).toHaveBeenCalledWith({
      error: 'Invalid token.'
    });
    expect(next).not.toHaveBeenCalled();
  });
});
```

---

## Security

### Security Headers and Configuration
```javascript
// npm install helmet express-rate-limit express-brute hpp xss
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const ExpressBrute = require('express-brute');
const hpp = require('hpp');
const xss = require('xss');

// Security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
      fontSrc: ["'self'", "https://fonts.gstatic.com"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'"],
      frameSrc: ["'none'"],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: []
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: {
    error: 'Too many requests from this IP, please try again later.'
  },
  standardHeaders: true,
  legacyHeaders: false,
  skip: (req) => {
    // Skip rate limiting for certain IPs or conditions
    const whitelist = ['127.0.0.1', '::1'];
    return whitelist.includes(req.ip);
  }
});

// Strict rate limiting for authentication endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
  message: {
    error: 'Too many login attempts, please try again later.'
  }
});

// Brute force protection
const store = new ExpressBrute.MemoryStore();
const bruteforce = new ExpressBrute(store, {
  freeRetries: 3,
  minWait: 5 * 60 * 1000, // 5 minutes
  maxWait: 60 * 60 * 1000, // 1 hour
  failCallback: (req, res, next, nextValidRequestDate) => {
    res.status(429).json({
      error: 'Too many failed attempts, try again later.',
      nextValidRequestDate
    });
  }
});

// Parameter pollution protection
app.use(hpp({
  whitelist: ['tags', 'categories'] // Allow arrays for these parameters
}));

// Apply security middleware
app.use('/api', limiter);
app.use('/api/auth', authLimiter);
app.use('/api/auth/login', bruteforce.prevent);

// XSS protection middleware
const sanitizeInput = (req, res, next) => {
  for (let key in req.body) {
    if (typeof req.body[key] === 'string') {
      req.body[key] = xss(req.body[key]);
    }
  }
  next();
};

app.use(sanitizeInput);
```

### Input Validation and Sanitization
```javascript
// npm install validator sanitize-html
const validator = require('validator');
const sanitizeHtml = require('sanitize-html');

// Comprehensive input sanitization
const sanitizeAndValidate = {
  email: (email) => {
    if (!email) return null;
    const sanitized = validator.normalizeEmail(email);
    return validator.isEmail(sanitized) ? sanitized : null;
  },

  string: (str, options = {}) => {
    if (!str) return null;
    
    const {
      minLength = 1,
      maxLength = 1000,
      allowHTML = false,
      trim = true
    } = options;

    let sanitized = str;
    
    if (trim) {
      sanitized = sanitized.trim();
    }
    
    if (!allowHTML) {
      sanitized = sanitizeHtml(sanitized, {
        allowedTags: [],
        allowedAttributes: {}
      });
    }
    
    if (sanitized.length < minLength || sanitized.length > maxLength) {
      throw new Error(`String length must be between ${minLength} and ${maxLength}`);
    }
    
    return sanitized;
  },

  integer: (num, options = {}) => {
    const { min, max } = options;
    
    if (num === null || num === undefined) return null;
    
    const parsed = parseInt(num, 10);
    if (isNaN(parsed)) {
      throw new Error('Invalid integer');
    }
    
    if (min !== undefined && parsed < min) {
      throw new Error(`Value must be at least ${min}`);
    }
    
    if (max !== undefined && parsed > max) {
      throw new Error(`Value must be at most ${max}`);
    }
    
    return parsed;
  },

  boolean: (bool) => {
    if (bool === null || bool === undefined) return null;
    if (typeof bool === 'boolean') return bool;
    if (typeof bool === 'string') {
      return bool.toLowerCase() === 'true';
    }
    return Boolean(bool);
  },

  url: (url) => {
    if (!url) return null;
    return validator.isURL(url) ? url : null;
  },

  mongoId: (id) => {
    if (!id) return null;
    return validator.isMongoId(id) ? id : null;
  }
};

// Validation middleware factory
const createValidator = (schema) => {
  return (req, res, next) => {
    try {
      for (const [field, config] of Object.entries(schema)) {
        const { type, required = false, options = {} } = config;
        const value = req.body[field];
        
        if (required && (value === null || value === undefined || value === '')) {
          return res.status(400).json({
            error: `Field '${field}' is required`
          });
        }
        
        if (value !== null && value !== undefined && value !== '') {
          req.body[field] = sanitizeAndValidate[type](value, options);
        }
      }
      next();
    } catch (error) {
      res.status(400).json({
        error: 'Validation failed',
        details: error.message
      });
    }
  };
};

// Usage example
const userValidationSchema = {
  name: { 
    type: 'string', 
    required: true, 
    options: { minLength: 2, maxLength: 50 } 
  },
  email: { 
    type: 'email', 
    required: true 
  },
  age: { 
    type: 'integer', 
    options: { min: 13, max: 120 } 
  },
  website: { 
    type: 'url' 
  }
};

app.post('/users', createValidator(userValidationSchema), (req, res) => {
  // Validated and sanitized data in req.body
  res.json({ message: 'User created', data: req.body });
});
```

### SQL Injection Prevention
```javascript
// Using parameterized queries with PostgreSQL
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// GOOD: Parameterized query
app.get('/users/search', async (req, res, next) => {
  try {
    const { name, email } = req.query;
    
    let query = 'SELECT id, name, email FROM users WHERE 1=1';
    const params = [];
    let paramIndex = 1;
    
    if (name) {
      query += ` AND name ILIKE ${paramIndex}`;
      params.push(`%${name}%`);
      paramIndex++;
    }
    
    if (email) {
      query += ` AND email = ${paramIndex}`;
      params.push(email);
      paramIndex++;
    }
    
    const result = await pool.query(query, params);
    res.json({ users: result.rows });
  } catch (error) {
    next(error);
  }
});

// BAD: String concatenation (vulnerable to SQL injection)
// app.get('/users/search', async (req, res) => {
//   const { name } = req.query;
//   const query = `SELECT * FROM users WHERE name = '${name}'`; // NEVER DO THIS
//   const result = await pool.query(query);
//   res.json(result.rows);
// });

// MongoDB injection prevention with Mongoose
app.get('/posts', async (req, res, next) => {
  try {
    const { author, status, limit = 10 } = req.query;
    
    // Build query object safely
    const query = {};
    
    if (author && typeof author === 'string') {
      query.author = author;
    }
    
    if (status && ['published', 'draft', 'archived'].includes(status)) {
      query.status = status;
    }
    
    // Sanitize limit
    const sanitizedLimit = Math.min(parseInt(limit) || 10, 100);
    
    const posts = await Post.find(query).limit(sanitizedLimit);
    res.json({ posts });
  } catch (error) {
    next(error);
  }
});
```

### Password Security
```javascript
// npm install bcryptjs argon2 zxcvbn
const bcrypt = require('bcryptjs');
const argon2 = require('argon2');
const zxcvbn = require('zxcvbn');

// Password strength validation
const validatePasswordStrength = (password) => {
  const result = zxcvbn(password);
  
  if (result.score < 3) {
    throw new Error(`Weak password. ${result.feedback.warning || 'Please use a stronger password.'}`);
  }
  
  return true;
};

// Bcrypt hashing (good)
const hashPasswordBcrypt = async (password) => {
  const saltRounds = 12;
  return await bcrypt.hash(password, saltRounds);
};

const verifyPasswordBcrypt = async (password, hash) => {
  return await bcrypt.compare(password, hash);
};

// Argon2 hashing (better - recommended)
const hashPasswordArgon2 = async (password) => {
  return await argon2.hash(password, {
    type: argon2.argon2id,
    memoryCost: 2 ** 16, // 64 MB
    timeCost: 3,
    parallelism: 1,
  });
};

const verifyPasswordArgon2 = async (password, hash) => {
  return await argon2.verify(hash, password);
};

// Password change endpoint
app.patch('/auth/change-password', authenticateToken, async (req, res, next) => {
  try {
    const { currentPassword, newPassword } = req.body;
    
    // Get user with password
    const user = await User.findById(req.user.id).select('+password');
    
    // Verify current password
    const isCurrentPasswordValid = await verifyPasswordArgon2(currentPassword, user.password);
    if (!isCurrentPasswordValid) {
      return res.status(400).json({ error: 'Current password is incorrect' });
    }
    
    // Validate new password strength
    validatePasswordStrength(newPassword);
    
    // Hash new password
    const hashedPassword = await hashPasswordArgon2(newPassword);
    
    // Update password
    await User.findByIdAndUpdate(user.id, { password: hashedPassword });
    
    res.json({ message: 'Password updated successfully' });
  } catch (error) {
    next(error);
  }
});
```

### CORS Configuration
```javascript
// npm install cors
const cors = require('cors');

// Basic CORS
app.use(cors());

// Advanced CORS configuration
const corsOptions = {
  origin: (origin, callback) => {
    const allowedOrigins = [
      'http://localhost:3000',
      'http://localhost:3001',
      'https://myapp.com',
      'https://app.myapp.com'
    ];
    
    // Allow requests with no origin (mobile apps, Postman, etc.)
    if (!origin) return callback(null, true);
    
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-Requested-With',
    'Accept',
    'Origin'
  ],
  exposedHeaders: ['X-Total-Count', 'X-Page-Count'],
  credentials: true, // Allow cookies
  maxAge: 86400 // 24 hours
};

app.use(cors(corsOptions));

// Environment-specific CORS
const getDynamicCorsOptions = () => {
  if (process.env.NODE_ENV === 'production') {
    return {
      origin: ['https://myapp.com', 'https://app.myapp.com'],
      credentials: true
    };
  } else {
    return {
      origin: true, // Allow all origins in development
      credentials: true
    };
  }
};

app.use(cors(getDynamicCorsOptions()));
```

---

## Performance & Optimization

### Caching Strategies
```javascript
// npm install memory-cache redis compression
const cache = require('memory-cache');
const redis = require('redis');
const compression = require('compression');

// Memory caching middleware
const memoryCache = (duration) => {
  return (req, res, next) => {
    const key = '__express__' + req.originalUrl || req.url;
    const cached = cache.get(key);
    
    if (cached) {
      return res.json(cached);
    }
    
    res.sendResponse = res.json;
    res.json = (body) => {
      cache.put(key, body, duration * 1000);
      res.sendResponse(body);
    };
    
    next();
  };
};

// Redis caching
const redisClient = redis.createClient({
  host: process.env.REDIS_HOST || 'localhost',
  port: process.env.REDIS_PORT || 6379,
  password: process.env.REDIS_PASSWORD
});

const redisCache = (duration = 300) => {
  return async (req, res, next) => {
    const key = `cache:${req.originalUrl}`;
    
    try {
      const cached = await redisClient.get(key);
      if (cached) {
        return res.json(JSON.parse(cached));
      }
      
      res.sendResponse = res.json;
      res.json = async (body) => {
        await redisClient.setex(key, duration, JSON.stringify(body));
        res.sendResponse(body);
      };
      
      next();
    } catch (error) {
      next();
    }
  };
};

// Cache invalidation
const invalidateCache = (pattern) => {
  return async (req, res, next) => {
    try {
      const keys = await redisClient.keys(pattern);
      if (keys.length > 0) {
        await redisClient.del(keys);
      }
    } catch (error) {
      console.error('Cache invalidation error:', error);
    }
    next();
  };
};

// Usage
app.get('/users', redisCache(300), async (req, res) => {
  const users = await User.find();
  res.json({ users });
});

app.post('/users', invalidateCache('cache:/users*'), async (req, res) => {
  const user = new User(req.body);
  await user.save();
  res.json({ user });
});

// Compression middleware
app.use(compression({
  level: 6,
  threshold: 1024, // Only compress if bigger than 1KB
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));
```

### Database Optimization
```javascript
// MongoDB optimization with Mongoose
const userSchema = new mongoose.Schema({
  name: { type: String, required: true, index: true },
  email: { type: String, required: true, unique: true, index: true },
  status: { type: String, enum: ['active', 'inactive'], index: true },
  createdAt: { type: Date, default: Date.now, index: true }
});

// Compound indexes for common queries
userSchema.index({ status: 1, createdAt: -1 });
userSchema.index({ email: 1, status: 1 });

// Text search index
userSchema.index({ 
  name: 'text', 
  email: 'text',
  bio: 'text' 
}, {
  weights: {
    name: 10,
    email: 5,
    bio: 1
  }
});

// Optimized queries with proper field selection and population
app.get('/users', async (req, res, next) => {
  try {
    const { page = 1, limit = 10, search, status } = req.query;
    const skip = (page - 1) * limit;
    
    // Build query
    let query = User.find();
    
    if (search) {
      query = query.find({ $text: { $search: search } });
    }
    
    if (status) {
      query = query.find({ status });
    }
    
    // Select only needed fields
    query = query.select('name email status createdAt');
    
    // Apply pagination and sorting
    query = query.sort({ createdAt: -1 }).skip(skip).limit(parseInt(limit));
    
    // Execute query with lean() for better performance
    const users = await query.lean();
    
    // Get total count efficiently
    const total = await User.countDocuments(query.getQuery());
    
    res.json({
      users,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total,
        pages: Math.ceil(total / limit)
      }
    });
  } catch (error) {
    next(error);
  }
});

// Aggregation pipeline for complex queries
app.get('/user-stats', async (req, res, next) => {
  try {
    const stats = await User.aggregate([
      // Match active users
      { $match: { status: 'active' } },
      
      // Group by month
      {
        $group: {
          _id: {
            year: { $year: '$createdAt' },
            month: { $month: '$createdAt' }
          },
          count: { $sum: 1 },
          averageAge: { $avg: '$age' }
        }
      },
      
      // Sort by date
      { $sort: { '_id.year': -1, '_id.month': -1 } },
      
      // Limit results
      { $limit: 12 },
      
      // Project final format
      {
        $project: {
          _id: 0,
          year: '$_id.year',
          month: '$_id.month',
          userCount: '$count',
          averageAge: { $round: ['$averageAge', 2] }
        }
      }
    ]);
    
    res.json({ stats });
  } catch (error) {
    next(error);
  }
});
```

### Request Optimization
```javascript
// npm install express-slow-down express-status-monitor
const slowDown = require('express-slow-down');
const monitor = require('express-status-monitor');

// Request slowing middleware
const speedLimiter = slowDown({
  windowMs: 15 * 60 * 1000, // 15 minutes
  delayAfter: 100, // allow 100 requests per 15 minutes at full speed
  delayMs: 500 // slow down subsequent requests by 500ms per request
});

// Request monitoring
app.use(monitor({
  title: 'API Status',
  path: '/status',
  spans: [
    { interval: 1, retention: 60 },
    { interval: 5, retention: 60 },
    { interval: 15, retention: 60 }
  ],
  chartVisibility: {
    cpu: true,
    mem: true,
    load: true,
    responseTime: true,
    rps: true,
    statusCodes: true
  }
}));

// Request timeout middleware
const timeout = require('connect-timeout');

app.use(timeout('30s'));
app.use((req, res, next) => {
  if (!req.timedout) next();
});

// Request size limiting
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ limit: '10mb', extended: true }));

// Graceful shutdown handling
let server;

const gracefulShutdown = () => {
  console.log('Received kill signal, shutting down gracefully');
  
  server.close(() => {
    console.log('HTTP server closed');
    
    // Close database connections
    mongoose.connection.close(false, () => {
      console.log('MongoDB connection closed');
      process.exit(0);
    });
  });
  
  // Force close after 30 seconds
  setTimeout(() => {
    console.error('Could not close connections in time, forcefully shutting down');
    process.exit(1);
  }, 30000);
};

server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

process.on('SIGTERM', gracefulShutdown);
process.on('SIGINT', gracefulShutdown);
```

### Static Asset Optimization
```javascript
const path = require('path');

// Static file serving with caching
app.use('/static', express.static('public', {
  maxAge: '1y', // Cache static assets for 1 year
  etag: true,
  lastModified: true,
  setHeaders: (res, filePath) => {
    // Set different cache headers for different file types
    if (path.extname(filePath) === '.html') {
      res.setHeader('Cache-Control', 'public, max-age=0');
    } else if (path.extname(filePath).match(/\.(css|js)$/)) {
      res.setHeader('Cache-Control', 'public, max-age=31536000'); // 1 year
    } else if (path.extname(filePath).match(/\.(jpg|jpeg|png|gif|ico|svg)$/)) {
      res.setHeader('Cache-Control', 'public, max-age=2592000'); // 30 days
    }
  }
}));

// Conditional requests support
app.use('/api', (req, res, next) => {
  // Add ETag support
  const originalSend = res.send;
  res.send = function(data) {
    if (data && typeof data === 'object') {
      const etag = require('crypto')
        .createHash('md5')
        .update(JSON.stringify(data))
        .digest('hex');
      
      res.set('ETag', `"${etag}"`);
      
      if (req.headers['if-none-match'] === `"${etag}"`) {
        return res.status(304).end();
      }
    }
    
    originalSend.call(this, data);
  };
  
  next();
});
```

---

## Deployment

### PM2 Configuration
```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'my-api',
    script: './server.js',
    instances: 'max', // Use all CPU cores
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'development',
      PORT: 3000
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000,
      DATABASE_URL: 'your_production_db_url',
      JWT_SECRET: 'your_production_jwt_secret'
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_file: './logs/combined.log',
    time: true,
    max_memory_restart: '1G',
    node_args: '--max_old_space_size=4096',
    watch: false,
    ignore_watch: ['node_modules', 'logs'],
    restart_delay: 4000,
    max_restarts: 10,
    min_uptime: '10s'
  }],

  deploy: {
    production: {
      user: 'ubuntu',
      host: 'your-server.com',
      ref: 'origin/main',
      repo: 'git@github.com:yourname/your-repo.git',
      path: '/home/ubuntu/your-app',
      'post-deploy': 'npm install && pm2 reload ecosystem.config.js --env production'
    }
  }
};

// package.json scripts for PM2
{
  "scripts": {
    "start": "pm2 start ecosystem.config.js --env production",
    "dev": "pm2 start ecosystem.config.js --watch",
    "stop": "pm2 stop ecosystem.config.js",
    "restart": "pm2 restart ecosystem.config.js",
    "logs": "pm2 logs",
    "monitor": "pm2 monit"
  }
}
```

### Docker Configuration
```dockerfile
# Dockerfile
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && npm cache clean --force

# Create non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001

# Copy application code
COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# Start application
CMD ["npm", "start"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=mongodb://mongo:27017/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongo
      - redis
    restart: unless-stopped
    volumes:
      - ./logs:/app/logs

  mongo:
    image: mongo:5
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo_data:/data/db
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/ssl
    depends_on:
      - app
    restart: unless-stopped

volumes:
  mongo_data:
  redis_data:
```

### Environment Configuration
```javascript
// config/config.js
const config = {
  development: {
    port: process.env.PORT || 3000,
    database: {
      uri: process.env.DATABASE_URL || 'mongodb://localhost:27017/myapp_dev',
      options: {
        useNewUrlParser: true,
        useUnifiedTopology: true
      }
    },
    redis: {
      host: process.env.REDIS_HOST || 'localhost',
      port: process.env.REDIS_PORT || 6379
    },
    jwt: {
      secret: process.env.JWT_SECRET || 'dev_secret',
      expiresIn: '7d'
    },
    logging: {
      level: 'debug'
    }
  },

  test: {
    port: process.env.PORT || 3001,
    database: {
      uri: process.env.TEST_DATABASE_URL || 'mongodb://localhost:27017/myapp_test'
    },
    jwt: {
      secret: 'test_secret',
      expiresIn: '1h'
    },
    logging: {
      level: 'error'
    }
  },

  production: {
    port: process.env.PORT || 3000,
    database: {
      uri: process.env.DATABASE_URL,
      options: {
        useNewUrlParser: true,
        useUnifiedTopology: true,
        maxPoolSize: 10,
        serverSelectionTimeoutMS: 5000,
        socketTimeoutMS: 45000
      }
    },
    redis: {
      host: process.env.REDIS_HOST,
      port: process.env.REDIS_PORT,
      password: process.env.REDIS_PASSWORD
    },
    jwt: {
      secret: process.env.JWT_SECRET,
      expiresIn: '7d'
    },
    logging: {
      level: 'info'
    },
    cors: {
      origin: process.env.CORS_ORIGIN?.split(',') || []
    }
  }
};

module.exports = config[process.env.NODE_ENV || 'development'];
```

### Health Checks and Monitoring
```javascript
// healthcheck.js
const http = require('http');
const mongoose = require('mongoose');

const options = {
  hostname: 'localhost',
  port: process.env.PORT || 3000,
  path: '/health',
  method: 'GET',
  timeout: 2000
};

const req = http.request(options, (res) => {
  if (res.statusCode === 200) {
    process.exit(0);
  } else {
    process.exit(1);
  }
});

req.on('error', () => {
  process.exit(1);
});

req.on('timeout', () => {
  req.destroy();
  process.exit(1);
});

req.end();

// health.js - Comprehensive health check endpoint
app.get('/health', async (req, res) => {
  const health = {
    status: 'OK',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    checks: {}
  };

  // Database check
  try {
    await mongoose.connection.db.admin().ping();
    health.checks.database = 'OK';
  } catch (error) {
    health.checks.database = 'FAIL';
    health.status = 'FAIL';
  }

  // Redis check
  try {
    await redisClient.ping();
    health.checks.redis = 'OK';
  } catch (error) {
    health.checks.redis = 'FAIL';
    health.status = 'FAIL';
  }

  // Memory check
  const memUsage = process.memoryUsage();
  health.checks.memory = {
    used: Math.round(memUsage.heapUsed / 1024 / 1024) + 'MB',
    total: Math.round(memUsage.heapTotal / 1024 / 1024) + 'MB'
  };

  const statusCode = health.status === 'OK' ? 200 : 503;
  res.status(statusCode).json(health);
});

// Readiness check
app.get('/ready', async (req, res) => {
  // Check if app is ready to receive traffic
  if (mongoose.connection.readyState === 1) {
    res.status(200).json({ status: 'ready' });
  } else {
    res.status(503).json({ status: 'not ready' });
  }
});

// Liveness check
app.get('/live', (req, res) => {
  res.status(200).json({ status: 'alive' });
});
```

---

## Best Practices

### Project Structure
```
my-express-app/
├── src/
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── userController.js
│   │   └── postController.js
│   ├── models/
│   │   ├── User.js
│   │   └── Post.js
│   ├── routes/
│   │   ├── auth.js
│   │   ├── users.js
│   │   └── posts.js
│   ├── middleware/
│   │   ├── auth.js
│   │   ├── validation.js
│   │   └── errorHandler.js
│   ├── services/
│   │   ├── authService.js
│   │   ├── userService.js
│   │   └── emailService.js
│   ├── utils/
│   │   ├── database.js
│   │   ├── logger.js
│   │   └── helpers.js
│   ├── config/
│   │   ├── database.js
│   │   ├── redis.js
│   │   └── config.js
│   └── app.js
├── tests/
│   ├── integration/
│   ├── unit/
│   └── setup.js
├── docs/
├── logs/
├── uploads/
├── .env
├── .env.example
├── .gitignore
├── .dockerignore
├── Dockerfile
├── docker-compose.yml
├── ecosystem.config.js
├── package.json
└── server.js
```

### Error Handling Best Practices
```javascript
// Custom error classes
class APIError extends Error {
  constructor(message, statusCode = 500, isOperational = true) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends APIError {
  constructor(message, errors = []) {
    super(message, 400);
    this.errors = errors;
  }
}

class NotFoundError extends APIError {
  constructor(resource = 'Resource') {
    super(`${resource} not found`, 404);
  }
}

class UnauthorizedError extends APIError {
  constructor(message = 'Unauthorized') {
    super(message, 401);
  }
}

// Async error wrapper
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Global error handler with logging
const globalErrorHandler = (err, req, res, next) => {
  // Default error values
  let error = { ...err };
  error.message = err.message;

  // Log error
  if (err.isOperational) {
    logger.warn('Operational Error:', {
      message: err.message,
      statusCode: err.statusCode,
      path: req.path,
      method: req.method,
      ip: req.ip
    });
  } else {
    logger.error('Programming Error:', {
      message: err.message,
      stack: err.stack,
      path: req.path,
      method: req.method,
      ip: req.ip
    });
  }

  // Send appropriate error response
  if (process.env.NODE_ENV === 'production') {
    if (err.isOperational) {
      res.status(err.statusCode).json({
        success: false,
        message: err.message,
        ...(err.errors && { errors: err.errors })
      });
    } else {
      // Don't leak error details in production
      res.status(500).json({
        success: false,
        message: 'Something went wrong!'
      });
    }
  } else {
    // Development - send full error details
    res.status(err.statusCode || 500).json({
      success: false,
      message: err.message,
      stack: err.stack,
      error: err
    });
  }
};

// Usage examples
const userController = {
  getUser: asyncHandler(async (req, res, next) => {
    const { id } = req.params;
    
    const user = await User.findById(id);
    if (!user) {
      throw new NotFoundError('User');
    }
    
    res.json({
      success: true,
      data: { user }
    });
  }),

  createUser: asyncHandler(async (req, res, next) => {
    const { name, email, password } = req.body;
    
    // Check if user exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      throw new ValidationError('User already exists', [
        { field: 'email', message: 'Email is already in use' }
      ]);
    }
    
    const user = new User({ name, email, password });
    await user.save();
    
    res.status(201).json({
      success: true,
      message: 'User created successfully',
      data: { user: { id: user._id, name, email } }
    });
  })
};
```

### Logging Best Practices
```javascript
// npm install winston morgan
const winston = require('winston');
const morgan = require('morgan');

// Winston logger configuration
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: 'my-api',
    version: process.env.npm_package_version
  },
  transports: [
    // Write all logs with level 'error' and below to error.log
    new winston.transports.File({ 
      filename: 'logs/error.log', 
      level: 'error',
      maxsize: 5242880, // 5MB
      maxFiles: 5
    }),
    
    // Write all logs with level 'info' and below to combined.log
    new winston.transports.File({ 
      filename: 'logs/combined.log',
      maxsize: 5242880, // 5MB
      maxFiles: 5
    }),
  ],
  
  // Handle uncaught exceptions and rejections
  exceptionHandlers: [
    new winston.transports.File({ filename: 'logs/exceptions.log' })
  ],
  rejectionHandlers: [
    new winston.transports.File({ filename: 'logs/rejections.log' })
  ]
});

// Add console logging in development
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple()
    )
  }));
}

// Morgan HTTP request logging
const morganFormat = process.env.NODE_ENV === 'production' 
  ? 'combined' 
  : 'dev';

app.use(morgan(morganFormat, {
  stream: {
    write: (message) => logger.info(message.trim())
  },
  skip: (req, res) => {
    // Skip health check logs in production
    return process.env.NODE_ENV === 'production' && 
           req.url.includes('/health');
  }
}));

// Custom logging middleware
const requestLogger = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    const logData = {
      method: req.method,
      url: req.originalUrl,
      statusCode: res.statusCode,
      duration: `${duration}ms`,
      userAgent: req.get('User-Agent'),
      ip: req.ip,
      userId: req.user?.id
    };
    
    if (res.statusCode >= 400) {
      logger.error('HTTP Request Error', logData);
    } else {
      logger.info('HTTP Request', logData);
    }
  });
  
  next();
};

// Structured logging examples
const logUserAction = (userId, action, details = {}) => {
  logger.info('User Action', {
    userId,
    action,
    timestamp: new Date().toISOString(),
    ...details
  });
};

const logSecurityEvent = (event, details = {}) => {
  logger.warn('Security Event', {
    event,
    timestamp: new Date().toISOString(),
    severity: 'high',
    ...details
  });
};

// Usage in routes
app.post('/auth/login', async (req, res, next) => {
  try {
    const { email, password } = req.body;
    
    const user = await User.findOne({ email }).select('+password');
    if (!user) {
      logSecurityEvent('login_attempt_invalid_email', { email, ip: req.ip });
      throw new UnauthorizedError('Invalid credentials');
    }
    
    const isValidPassword = await bcrypt.compare(password, user.password);
    if (!isValidPassword) {
      logSecurityEvent('login_attempt_invalid_password', { 
        email, 
        ip: req.ip,
        userId: user._id 
      });
      throw new UnauthorizedError('Invalid credentials');
    }
    
    const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET);
    
    logUserAction(user._id, 'login', { ip: req.ip });
    
    res.json({
      success: true,
      message: 'Login successful',
      data: { token, user: { id: user._id, email, name: user.name } }
    });
  } catch (error) {
    next(error);
  }
});

module.exports = { logger, requestLogger, logUserAction, logSecurityEvent };
```

### API Response Standards
```javascript
// Standard response format
const ResponseFormatter = {
  success: (data, message = 'Success', meta = null) => ({
    success: true,
    message,
    data,
    ...(meta && { meta }),
    timestamp: new Date().toISOString()
  }),

  error: (message, errors = null, statusCode = 500) => ({
    success: false,
    message,
    ...(errors && { errors }),
    statusCode,
    timestamp: new Date().toISOString()
  }),

  paginated: (data, pagination, message = 'Success') => ({
    success: true,
    message,
    data,
    pagination,
    timestamp: new Date().toISOString()
  })
};

// Response middleware
app.use((req, res, next) => {
  res.success = (data, message, meta) => {
    res.json(ResponseFormatter.success(data, message, meta));
  };
  
  res.error = (message, errors, statusCode = 500) => {
    res.status(statusCode).json(ResponseFormatter.error(message, errors, statusCode));
  };
  
  res.paginated = (data, pagination, message) => {
    res.json(ResponseFormatter.paginated(data, pagination, message));
  };
  
  next();
});

// Consistent error codes
const ErrorCodes = {
  VALIDATION_ERROR: 'VALIDATION_ERROR',
  AUTHENTICATION_FAILED: 'AUTHENTICATION_FAILED',
  AUTHORIZATION_FAILED: 'AUTHORIZATION_FAILED',
  RESOURCE_NOT_FOUND: 'RESOURCE_NOT_FOUND',
  DUPLICATE_RESOURCE: 'DUPLICATE_RESOURCE',
  RATE_LIMIT_EXCEEDED: 'RATE_LIMIT_EXCEEDED',
  INTERNAL_SERVER_ERROR: 'INTERNAL_SERVER_ERROR'
};

// Enhanced error responses
const sendErrorResponse = (res, statusCode, code, message, errors = null) => {
  res.status(statusCode).json({
    success: false,
    error: {
      code,
      message,
      ...(errors && { details: errors })
    },
    timestamp: new Date().toISOString()
  });
};
```

### Security Checklist
```javascript
// Security middleware stack
const securityMiddleware = () => {
  const middlewares = [];
  
  // Security headers
  middlewares.push(helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", "data:", "https:"],
        connectSrc: ["'self'"],
        fontSrc: ["'self'", "https://fonts.gstatic.com"],
        objectSrc: ["'none'"],
        mediaSrc: ["'self'"],
        frameSrc: ["'none'"],
      },
    },
    crossOriginEmbedderPolicy: false
  }));
  
  // Rate limiting
  middlewares.push(rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
    message: { error: 'Too many requests' },
    standardHeaders: true,
    legacyHeaders: false
  }));
  
  // Parameter pollution
  middlewares.push(hpp({
    whitelist: ['tags', 'categories', 'fields']
  }));
  
  // Request size limiting
  middlewares.push(express.json({ limit: '10mb' }));
  middlewares.push(express.urlencoded({ limit: '10mb', extended: true }));
  
  return middlewares;
};

// Security validation middleware
const securityValidation = (req, res, next) => {
  // Check for suspicious patterns
  const suspiciousPatterns = [
    /<script/i,
    /javascript:/i,
    /on\w+\s*=/i,
    /union.*select/i,
    /drop.*table/i
  ];
  
  const checkValue = (value) => {
    if (typeof value === 'string') {
      return suspiciousPatterns.some(pattern => pattern.test(value));
    }
    if (typeof value === 'object' && value !== null) {
      return Object.values(value).some(checkValue);
    }
    return false;
  };
  
  const hasSecurityThreat = 
    checkValue(req.body) || 
    checkValue(req.query) || 
    checkValue(req.params);
  
  if (hasSecurityThreat) {
    logger.warn('Security threat detected', {
      ip: req.ip,
      url: req.originalUrl,
      body: req.body,
      query: req.query
    });
    
    return res.status(400).json({
      success: false,
      message: 'Invalid request format'
    });
  }
  
  next();
};

app.use(...securityMiddleware());
app.use(securityValidation);
```

### Performance Monitoring
```javascript
// npm install @sentry/node newrelic
const Sentry = require('@sentry/node');

// Sentry error tracking
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
  integrations: [
    new Sentry.Integrations.Http({ tracing: true }),
    new Sentry.Integrations.Express({ app })
  ]
});

// Performance monitoring middleware
const performanceMonitor = (req, res, next) => {
  const start = process.hrtime();
  
  res.on('finish', () => {
    const [seconds, nanoseconds] = process.hrtime(start);
    const duration = seconds * 1000 + nanoseconds / 1000000;
    
    // Log slow requests
    if (duration > 1000) { // More than 1 second
      logger.warn('Slow request detected', {
        method: req.method,
        url: req.originalUrl,
        duration: `${duration.toFixed(2)}ms`,
        statusCode: res.statusCode
      });
    }
    
    // Send to monitoring service
    if (process.env.NODE_ENV === 'production') {
      // Example: Send to custom monitoring service
      // monitoringService.recordMetric('request_duration', duration, {
      //   method: req.method,
      //   route: req.route?.path || req.originalUrl,
      //   status: res.statusCode
      // });
    }
  });
  
  next();
};

app.use(performanceMonitor);

// Memory usage monitoring
const monitorMemory = () => {
  const usage = process.memoryUsage();
  const formatMB = (bytes) => Math.round(bytes / 1024 / 1024 * 100) / 100;
  
  logger.info('Memory Usage', {
    rss: `${formatMB(usage.rss)}MB`,
    heapTotal: `${formatMB(usage.heapTotal)}MB`,
    heapUsed: `${formatMB(usage.heapUsed)}MB`,
    external: `${formatMB(usage.external)}MB`
  });
  
  // Alert if memory usage is high
  if (usage.heapUsed > 500 * 1024 * 1024) { // 500MB
    logger.warn('High memory usage detected', {
      heapUsed: `${formatMB(usage.heapUsed)}MB`
    });
  }
};

// Monitor memory every 5 minutes
setInterval(monitorMemory, 5 * 60 * 1000);

// CPU usage monitoring
const monitorCPU = () => {
  const usage = process.cpuUsage();
  logger.info('CPU Usage', {
    user: `${usage.user / 1000}ms`,
    system: `${usage.system / 1000}ms`
  });
};

setInterval(monitorCPU, 5 * 60 * 1000);
```

### Code Organization Patterns
```javascript
// Service Layer Pattern
class UserService {
  static async createUser(userData) {
    // Validate input
    const validatedData = await this.validateUserData(userData);
    
    // Check business rules
    await this.checkBusinessRules(validatedData);
    
    // Create user
    const user = new User(validatedData);
    await user.save();
    
    // Trigger side effects
    await this.triggerPostCreationEffects(user);
    
    return user;
  }
  
  static async validateUserData(data) {
    // Validation logic
    return data;
  }
  
  static async checkBusinessRules(data) {
    // Business rule validation
    const existingUser = await User.findOne({ email: data.email });
    if (existingUser) {
      throw new ValidationError('Email already exists');
    }
  }
  
  static async triggerPostCreationEffects(user) {
    // Send welcome email, create profile, etc.
    await EmailService.sendWelcomeEmail(user);
    await ProfileService.createDefaultProfile(user);
  }
}

// Repository Pattern
class UserRepository {
  static async findById(id) {
    return await User.findById(id);
  }
  
  static async findByEmail(email) {
    return await User.findOne({ email });
  }
  
  static async create(userData) {
    const user = new User(userData);
    return await user.save();
  }
  
  static async update(id, updates) {
    return await User.findByIdAndUpdate(id, updates, { 
      new: true, 
      runValidators: true 
    });
  }
  
  static async delete(id) {
    return await User.findByIdAndDelete(id);
  }
  
  static async findPaginated(query, options) {
    const { page = 1, limit = 10, sortBy = 'createdAt', sortOrder = 'desc' } = options;
    const skip = (page - 1) * limit;
    
    const users = await User
      .find(query)
      .sort({ [sortBy]: sortOrder === 'desc' ? -1 : 1 })
      .skip(skip)
      .limit(limit);
    
    const total = await User.countDocuments(query);
    
    return {
      users,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit)
      }
    };
  }
}

// Controller Pattern
class UserController {
  static async getUsers(req, res, next) {
    try {
      const options = {
        page: parseInt(req.query.page) || 1,
        limit: parseInt(req.query.limit) || 10,
        sortBy: req.query.sortBy || 'createdAt',
        sortOrder: req.query.sortOrder || 'desc'
      };
      
      const query = {};
      if (req.query.search) {
        query.$text = { $search: req.query.search };
      }
      
      const result = await UserRepository.findPaginated(query, options);
      
      res.success(result.users, 'Users retrieved successfully', {
        pagination: result.pagination
      });
    } catch (error) {
      next(error);
    }
  }
  
  static async createUser(req, res, next) {
    try {
      const user = await UserService.createUser(req.body);
      res.status(201).success(user, 'User created successfully');
    } catch (error) {
      next(error);
    }
  }
  
  static async getUserById(req, res, next) {
    try {
      const user = await UserRepository.findById(req.params.id);
      if (!user) {
        throw new NotFoundError('User');
      }
      res.success(user, 'User retrieved successfully');
    } catch (error) {
      next(error);
    }
  }
  
  static async updateUser(req, res, next) {
    try {
      const user = await UserRepository.update(req.params.id, req.body);
      if (!user) {
        throw new NotFoundError('User');
      }
      res.success(user, 'User updated successfully');
    } catch (error) {
      next(error);
    }
  }
  
  static async deleteUser(req, res, next) {
    try {
      const user = await UserRepository.delete(req.params.id);
      if (!user) {
        throw new NotFoundError('User');
      }
      res.success(null, 'User deleted successfully');
    } catch (error) {
      next(error);
    }
  }
}

// Route definitions
const userRoutes = express.Router();

userRoutes.get('/', UserController.getUsers);
userRoutes.post('/', validateUser, UserController.createUser);
userRoutes.get('/:id', UserController.getUserById);
userRoutes.put('/:id', validateUser, UserController.updateUser);
userRoutes.delete('/:id', UserController.deleteUser);

module.exports = userRoutes;
```

### Environment-Specific Configuration
```javascript
// config/index.js - Environment configuration manager
const path = require('path');

const requiredEnvVars = {
  production: [
    'DATABASE_URL',
    'JWT_SECRET',
    'REDIS_URL',
    'SENTRY_DSN'
  ],
  development: [
    'JWT_SECRET'
  ],
  test: []
};

const validateEnvironment = () => {
  const env = process.env.NODE_ENV || 'development';
  const required = requiredEnvVars[env] || [];
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
};

const config = {
  env: process.env.NODE_ENV || 'development',
  port: parseInt(process.env.PORT) || 3000,
  
  database: {
    url: process.env.DATABASE_URL || 'mongodb://localhost:27017/myapp',
    options: {
      useNewUrlParser: true,
      useUnifiedTopology: true,
      maxPoolSize: parseInt(process.env.DB_POOL_SIZE) || 10
    }
  },
  
  redis: {
    url: process.env.REDIS_URL || 'redis://localhost:6379',
    options: {
      retryDelayOnFailover: 100,
      maxRetriesPerRequest: 3
    }
  },
  
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '7d',
    refreshSecret: process.env.JWT_REFRESH_SECRET,
    refreshExpiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '30d'
  },
  
  cors: {
    origin: process.env.CORS_ORIGIN ? 
      process.env.CORS_ORIGIN.split(',') : 
      ['http://localhost:3000'],
    credentials: true
  },
  
  upload: {
    maxSize: parseInt(process.env.UPLOAD_MAX_SIZE) || 5 * 1024 * 1024, // 5MB
    allowedTypes: (process.env.UPLOAD_ALLOWED_TYPES || 'image/jpeg,image/png,image/gif').split(',')
  },
  
  email: {
    from: process.env.EMAIL_FROM || 'noreply@myapp.com',
    smtp: {
      host: process.env.SMTP_HOST,
      port: parseInt(process.env.SMTP_PORT) || 587,
      secure: process.env.SMTP_SECURE === 'true',
      auth: {
        user: process.env.SMTP_USER,
        pass: process.env.SMTP_PASS
      }
    }
  },
  
  security: {
    bcryptRounds: parseInt(process.env.BCRYPT_ROUNDS) || 12,
    rateLimitWindow: parseInt(process.env.RATE_LIMIT_WINDOW) || 15 * 60 * 1000,
    rateLimitMax: parseInt(process.env.RATE_LIMIT_MAX) || 100
  },
  
  logging: {
    level: process.env.LOG_LEVEL || (process.env.NODE_ENV === 'production' ? 'info' : 'debug'),
    file: process.env.LOG_FILE !== 'false'
  }
};

// Validate on load
validateEnvironment();

module.exports = config;
```
