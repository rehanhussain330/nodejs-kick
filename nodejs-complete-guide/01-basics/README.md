# Part 1: Node.js Basics

## Table of Contents
1. [Introduction to Node.js](#1-introduction-to-nodejs)
2. [Setting Up Development Environment](#2-setting-up-development-environment)
3. [Node.js Architecture & Event Loop](#3-nodejs-architecture--event-loop)
4. [Modules System](#4-modules-system)
5. [File System Operations](#5-file-system-operations)
6. [Buffers and Streams](#6-buffers-and-streams)
7. [Error Handling](#7-error-handling)
8. [NPM Package Manager](#8-npm-package-manager)

---

## 1. Introduction to Node.js

### What is Node.js?

Node.js is an open-source, cross-platform JavaScript runtime environment that executes JavaScript code outside a web browser. It was created by Ryan Dahl in 2009 and is built on Chrome's V8 JavaScript engine.

### Key Features

- **Asynchronous & Event-Driven**: Non-blocking I/O operations
- **Fast Execution**: Built on Chrome's V8 engine
- **Single-Threaded**: Uses event loop for concurrency
- **Scalable**: Handles thousands of concurrent connections
- **Cross-Platform**: Runs on Windows, Linux, macOS
- **Rich Ecosystem**: NPM has over 2 million packages

### Use Cases

- Web servers and APIs
- Real-time applications (chat, gaming)
- Microservices architecture
- Command-line tools
- IoT applications
- Data streaming applications

### When NOT to Use Node.js

- CPU-intensive tasks (video encoding, complex calculations)
- Heavy server-side computation
- Relational database operations (though possible, not ideal)

---

## 2. Setting Up Development Environment

### Installation Steps

#### Windows/macOS
1. Download from [nodejs.org](https://nodejs.org/)
2. Run the installer
3. Verify installation:
```bash
node --version
npm --version
```

#### Linux (Ubuntu/Debian)
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### Hello World Example

**File: `examples/01-hello-world.js`**
```javascript
// Traditional way
console.log("Hello, World!");

// With variables
const greeting = "Hello";
const target = "Node.js Developer";
console.log(`${greeting}, ${target}!`);

// Create a simple server
const http = require('http');

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000/');
});
```

**Run it:**
```bash
node examples/01-hello-world.js
```

---

## 3. Node.js Architecture & Event Loop

### Architecture Overview

```
┌─────────────────────┐
│   Your Code         │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   Node.js API       │
│   (Libuv, V8, etc.) │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   Operating System  │
└─────────────────────┘
```

### The Event Loop

The event loop is what allows Node.js to perform non-blocking I/O operations despite being single-threaded.

#### Phases of Event Loop

1. **Timers**: Executes callbacks scheduled by setTimeout() and setInterval()
2. **Pending Callbacks**: Executes I/O callbacks deferred to the next loop iteration
3. **Idle, Prepare**: Internal use only
4. **Poll**: Retrieves new I/O events; executes I/O callbacks
5. **Check**: Executes setImmediate() callbacks
6. **Close Callbacks**: Executes close event callbacks

### Visual Example

**File: `examples/02-event-loop.js`**
```javascript
const fs = require('fs');

console.log('1. Starting');

setTimeout(() => {
  console.log('2. Timer callback (timers phase)');
}, 0);

setImmediate(() => {
  console.log('3. Immediate callback (check phase)');
});

fs.readFile('test.txt', (err, data) => {
  console.log('4. File read callback (poll phase)');
  
  process.nextTick(() => {
    console.log('5. Next tick callback');
  });
  
  Promise.resolve().then(() => {
    console.log('6. Promise callback');
  });
});

console.log('7. Ending');

// Output order:
// 1. Starting
// 7. Ending
// 4. File read callback (poll phase)
// 5. Next tick callback
// 6. Promise callback
// 2. Timer callback (timers phase)
// 3. Immediate callback (check phase)
```

### Real-World Example: API Request Handler

**File: `examples/03-api-handler.js`**
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  console.log(`Request received: ${req.method} ${req.url}`);
  
  // Simulate database query (async operation)
  setTimeout(() => {
    const userData = {
      id: 1,
      name: 'John Doe',
      email: 'john@example.com'
    };
    
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify(userData));
    console.log('Response sent');
  }, 1000);
  
  // Event loop continues handling other requests
  console.log('Continuing to handle other requests...');
});

server.listen(3001, () => {
  console.log('API Server running on port 3001');
});
```

---

## 4. Modules System

### CommonJS Modules (Traditional)

**File: `examples/04-modules/math.js`**
```javascript
// math.js - Module definition

const add = (a, b) => a + b;
const subtract = (a, b) => a - b;
const multiply = (a, b) => a * b;
const divide = (a, b) => a / b;

// Export individual functions
module.exports.add = add;
module.exports.subtract = subtract;

// Or export everything at once
module.exports = {
  add,
  subtract,
  multiply,
  divide
};
```

**File: `examples/04-modules/app.js`**
```javascript
// app.js - Using the module

const math = require('./math');

console.log(math.add(5, 3));        // 8
console.log(math.subtract(10, 4));  // 6
console.log(math.multiply(2, 7));   // 14
console.log(math.divide(20, 4));    // 5
```

### ES6 Modules (Modern)

**File: `examples/05-es6-modules/math.mjs`**
```javascript
// Named exports
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// Default export
const calculate = (a, b, operation) => {
  switch(operation) {
    case 'add': return a + b;
    case 'subtract': return a - b;
    default: return null;
  }
};

export default calculate;
```

**File: `examples/05-es6-modules/app.mjs`**
```javascript
import calculate, { add, subtract } from './math.mjs';

console.log(add(5, 3));              // 8
console.log(calculate(10, 5, 'add')); // 15
```

### Built-in Modules

**File: `examples/06-built-in-modules.js`**
```javascript
// Path module
const path = require('path');

console.log(path.join('/users', 'john', 'file.txt'));
// Output: /users/john/file.txt (or \users\john\file.txt on Windows)

console.log(path.basename('/users/john/file.txt'));
// Output: file.txt

console.log(path.extname('file.txt'));
// Output: .txt

// OS module
const os = require('os');

console.log(`Platform: ${os.platform()}`);
console.log(`Architecture: ${os.arch()}`);
console.log(`Total Memory: ${os.totalmem()} bytes`);
console.log(`Free Memory: ${os.freemem()} bytes`);
console.log(`CPU Cores: ${os.cpus().length}`);

// Util module
const util = require('util');

const debugLog = util.debuglog('myapp');
debugLog('This is a debug message');
```

### Creating Custom Modules - Real Example

**File: `examples/07-custom-module/logger.js`**
```javascript
const fs = require('fs');
const path = require('path');

class Logger {
  constructor(logFile = 'app.log') {
    this.logFile = path.join(__dirname, logFile);
  }

  info(message) {
    this._write('INFO', message);
  }

  error(message) {
    this._write('ERROR', message);
  }

  warn(message) {
    this._write('WARN', message);
  }

  _write(level, message) {
    const timestamp = new Date().toISOString();
    const logEntry = `[${timestamp}] [${level}] ${message}\n`;
    
    console.log(logEntry.trim());
    
    fs.appendFile(this.logFile, logEntry, (err) => {
      if (err) console.error('Failed to write to log file:', err);
    });
  }
}

module.exports = Logger;
```

**File: `examples/07-custom-module/app.js`**
```javascript
const Logger = require('./logger');

const logger = new Logger();

logger.info('Application started');
logger.warn('Low memory warning');
logger.error('Database connection failed');
```

---

## 5. File System Operations

### Synchronous vs Asynchronous

**File: `examples/08-file-sync.js`**
```javascript
const fs = require('fs');

// Synchronous (blocks execution)
try {
  const data = fs.readFileSync('example.txt', 'utf8');
  console.log('Sync Read:', data);
} catch (err) {
  console.error('Sync Error:', err.message);
}

console.log('This waits for file read to complete');
```

**File: `examples/09-file-async.js`**
```javascript
const fs = require('fs');

// Asynchronous (non-blocking)
fs.readFile('example.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Async Error:', err.message);
    return;
  }
  console.log('Async Read:', data);
});

console.log('This executes immediately, without waiting');
```

### Promises API (Modern Approach)

**File: `examples/10-file-promises.js`**
```javascript
const fs = require('fs').promises;
const path = require('path');

async function fileOperations() {
  try {
    // Create directory
    await fs.mkdir(path.join(__dirname, 'test-dir'), { recursive: true });
    console.log('Directory created');

    // Write file
    await fs.writeFile(
      path.join(__dirname, 'test-dir', 'test.txt'),
      'Hello from Node.js!',
      'utf8'
    );
    console.log('File written');

    // Read file
    const data = await fs.readFile(
      path.join(__dirname, 'test-dir', 'test.txt'),
      'utf8'
    );
    console.log('File content:', data);

    // Append to file
    await fs.appendFile(
      path.join(__dirname, 'test-dir', 'test.txt'),
      '\nAppended content',
      'utf8'
    );

    // Rename file
    await fs.rename(
      path.join(__dirname, 'test-dir', 'test.txt'),
      path.join(__dirname, 'test-dir', 'renamed.txt')
    );

    // Get file stats
    const stats = await fs.stat(path.join(__dirname, 'test-dir', 'renamed.txt'));
    console.log('File size:', stats.size, 'bytes');
    console.log('Created:', stats.birthtime);

    // List directory contents
    const files = await fs.readdir(path.join(__dirname, 'test-dir'));
    console.log('Directory contents:', files);

    // Delete file
    await fs.unlink(path.join(__dirname, 'test-dir', 'renamed.txt'));
    
    // Delete directory
    await fs.rmdir(path.join(__dirname, 'test-dir'));
    
    console.log('Cleanup complete');
  } catch (err) {
    console.error('Error:', err);
  }
}

fileOperations();
```

### Real-World Example: Configuration Manager

**File: `examples/11-config-manager.js`**
```javascript
const fs = require('fs').promises;
const path = require('path');

class ConfigManager {
  constructor(configPath = './config.json') {
    this.configPath = path.join(__dirname, configPath);
  }

  async load() {
    try {
      const data = await fs.readFile(this.configPath, 'utf8');
      return JSON.parse(data);
    } catch (err) {
      if (err.code === 'ENOENT') {
        console.log('Config file not found, creating default...');
        const defaultConfig = {
          port: 3000,
          database: 'mongodb://localhost:27017/mydb',
          debug: false
        };
        await this.save(defaultConfig);
        return defaultConfig;
      }
      throw err;
    }
  }

  async save(config) {
    await fs.writeFile(
      this.configPath,
      JSON.stringify(config, null, 2),
      'utf8'
    );
    console.log('Configuration saved');
  }

  async update(updates) {
    const currentConfig = await this.load();
    const newConfig = { ...currentConfig, ...updates };
    await this.save(newConfig);
    return newConfig;
  }
}

// Usage
(async () => {
  const configManager = new ConfigManager();
  
  // Load or create config
  const config = await configManager.load();
  console.log('Current config:', config);
  
  // Update config
  const updatedConfig = await configManager.update({
    port: 8080,
    debug: true
  });
  console.log('Updated config:', updatedConfig);
})();
```

---

## 6. Buffers and Streams

### Understanding Buffers

**File: `examples/12-buffers.js`**
```javascript
// Creating buffers
const buf1 = Buffer.alloc(10); // Allocates 10 bytes filled with 0
const buf2 = Buffer.from([1, 2, 3, 4, 5]); // From array
const buf3 = Buffer.from('Hello Node.js', 'utf8'); // From string

console.log('Buffer 1:', buf1);
console.log('Buffer 2:', buf2);
console.log('Buffer 3:', buf3);
console.log('Buffer 3 as string:', buf3.toString());

// Writing to buffer
buf1.write('Hi', 0);
console.log('After write:', buf1.toString());

// Concatenating buffers
const combined = Buffer.concat([buf2, buf3]);
console.log('Combined:', combined.toString());

// Buffer slicing
const sliced = buf3.slice(0, 5);
console.log('Sliced:', sliced.toString());

// Real-world: Reading binary file
const fs = require('fs');

const imageBuffer = fs.readFileSync('examples/sample-image.png');
console.log('Image size:', imageBuffer.length, 'bytes');
console.log('First 10 bytes:', imageBuffer.slice(0, 10));
```

### Streams - Types and Usage

Streams are collections of data that might not be available all at once and don't have to fit in memory.

#### Four Types of Streams:
1. **Readable**: Can read data (fs.createReadStream)
2. **Writable**: Can write data (fs.createWriteStream)
3. **Duplex**: Both readable and writable (net.Socket)
4. **Transform**: Modify data as it passes through (zlib.createGzip)

**File: `examples/13-streams-basic.js`**
```javascript
const fs = require('fs');
const path = require('path');

// Readable Stream
const readStream = fs.createReadStream(
  path.join(__dirname, 'large-file.txt'),
  { highWaterMark: 1024, encoding: 'utf8' }
);

readStream.on('data', (chunk) => {
  console.log('Received chunk:', chunk.length, 'bytes');
  console.log('Chunk:', chunk);
});

readStream.on('end', () => {
  console.log('Reading complete');
});

readStream.on('error', (err) => {
  console.error('Read error:', err);
});

// Writable Stream
const writeStream = fs.createWriteStream(
  path.join(__dirname, 'output.txt')
);

for (let i = 1; i <= 100; i++) {
  writeStream.write(`Line ${i}: This is test data\n`);
}

writeStream.end();
writeStream.on('finish', () => {
  console.log('Writing complete');
});
```

### Piping Streams

**File: `examples/14-stream-pipe.js`**
```javascript
const fs = require('fs');
const zlib = require('zlib');
const path = require('path');

// Pipe example: Read -> Compress -> Write
const input = fs.createReadStream(path.join(__dirname, 'large-file.txt'));
const gzip = zlib.createGzip();
const output = fs.createWriteStream(path.join(__dirname, 'large-file.txt.gz'));

input
  .pipe(gzip)
  .pipe(output)
  .on('finish', () => {
    console.log('File compressed successfully');
  });

// Decompress
const decompressInput = fs.createReadStream(path.join(__dirname, 'large-file.txt.gz'));
const gunzip = zlib.createGunzip();
const decompressOutput = fs.createWriteStream(path.join(__dirname, 'large-file-decompressed.txt'));

decompressInput
  .pipe(gunzip)
  .pipe(decompressOutput)
  .on('finish', () => {
    console.log('File decompressed successfully');
  });
```

### Real-World Example: File Upload Handler

**File: `examples/15-file-upload-stream.js`**
```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

const server = http.createServer((req, res) => {
  if (req.method === 'POST' && req.url === '/upload') {
    const fileName = `${Date.now()}-${crypto.randomBytes(4).toString('hex')}.dat`;
    const filePath = path.join(__dirname, 'uploads', fileName);
    
    // Ensure uploads directory exists
    if (!fs.existsSync(path.join(__dirname, 'uploads'))) {
      fs.mkdirSync(path.join(__dirname, 'uploads'));
    }

    const writeStream = fs.createWriteStream(filePath);
    let totalBytes = 0;

    req.on('data', (chunk) => {
      totalBytes += chunk.length;
      console.log(`Received ${totalBytes} bytes so far...`);
    });

    req.pipe(writeStream);

    writeStream.on('finish', () => {
      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({
        success: true,
        message: 'File uploaded successfully',
        fileName: fileName,
        size: totalBytes
      }));
    });

    writeStream.on('error', (err) => {
      res.writeHead(500, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({
        success: false,
        message: 'Upload failed',
        error: err.message
      }));
    });
  } else {
    res.writeHead(404);
    res.end('Not Found');
  }
});

server.listen(3002, () => {
  console.log('Upload server running on port 3002');
  console.log('Test with: curl -X POST --data-binary @yourfile.txt http://localhost:3002/upload');
});
```

---

## 7. Error Handling

### Types of Errors

**File: `examples/16-error-types.js`**
```javascript
// Operational Errors (expected)
const fs = require('fs');

fs.readFile('nonexistent.txt', (err, data) => {
  if (err) {
    console.log('Operational Error:', err.code, err.message);
  }
});

// Programmer Errors (bugs)
const x = null;
// x.property; // TypeError: Cannot read property 'property' of null

// Try-catch for synchronous code
try {
  JSON.parse('{invalid json}');
} catch (err) {
  console.log('Caught error:', err.message);
}
```

### Error Handling with Async/Await

**File: `examples/17-error-async-await.js`**
```javascript
const fs = require('fs').promises;

async function readFileSafe(filePath) {
  try {
    const data = await fs.readFile(filePath, 'utf8');
    return { success: true, data };
  } catch (err) {
    return { 
      success: false, 
      error: err.message,
      code: err.code
    };
  }
}

// Multiple async operations with error handling
async function processFiles(files) {
  const results = [];
  
  for (const file of files) {
    try {
      const data = await fs.readFile(file, 'utf8');
      results.push({ file, success: true, data });
    } catch (err) {
      results.push({ file, success: false, error: err.message });
    }
  }
  
  return results;
}

// Using Promise.allSettled (ES2020)
async function processFilesParallel(files) {
  const promises = files.map(file => 
    fs.readFile(file, 'utf8')
      .then(data => ({ file, success: true, data }))
      .catch(err => ({ file, success: false, error: err.message }))
  );
  
  return Promise.allSettled(promises);
}

// Usage
(async () => {
  const result = await readFileSafe('nonexistent.txt');
  console.log(result);
  
  const results = await processFiles(['file1.txt', 'file2.txt']);
  console.log(results);
})();
```

### Custom Error Classes

**File: `examples/18-custom-errors.js`**
```javascript
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
    
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message, field) {
    super(message, 400, 'VALIDATION_ERROR');
    this.field = field;
  }
}

class NotFoundError extends AppError {
  constructor(resource) {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

class AuthenticationError extends AppError {
  constructor(message = 'Authentication failed') {
    super(message, 401, 'AUTHENTICATION_ERROR');
  }
}

class DatabaseError extends AppError {
  constructor(message) {
    super(message, 500, 'DATABASE_ERROR');
  }
}

// Usage
function validateUser(user) {
  if (!user.email) {
    throw new ValidationError('Email is required', 'email');
  }
  if (!user.password || user.password.length < 8) {
    throw new ValidationError('Password must be at least 8 characters', 'password');
  }
  return true;
}

try {
  validateUser({ email: 'test@example.com' });
} catch (err) {
  if (err instanceof ValidationError) {
    console.log(`Validation failed for ${err.field}: ${err.message}`);
  }
}
```

### Global Error Handler

**File: `examples/19-global-error-handler.js`**
```javascript
const EventEmitter = require('events');

class ErrorHandler extends EventEmitter {
  constructor() {
    super();
    this.errors = [];
    
    // Listen for errors
    this.on('error', (err) => {
      this.logError(err);
      this.notifyTeam(err);
    });
  }

  logError(err) {
    const timestamp = new Date().toISOString();
    const errorLog = {
      timestamp,
      message: err.message,
      stack: err.stack,
      code: err.code,
      statusCode: err.statusCode
    };
    
    this.errors.push(errorLog);
    console.error(`[${timestamp}] Error:`, err.message);
    
    // In production, write to file or logging service
  }

  notifyTeam(err) {
    // Send email, Slack notification, etc.
    if (err.statusCode >= 500) {
      console.log('🚨 Critical error - notifying team!');
      // Implement actual notification logic here
    }
  }

  getErrors(limit = 10) {
    return this.errors.slice(-limit);
  }
}

const errorHandler = new ErrorHandler();

// Handle uncaught exceptions
process.on('uncaughtException', (err) => {
  errorHandler.emit('error', err);
  process.exit(1);
});

// Handle unhandled promise rejections
process.on('unhandledRejection', (reason, promise) => {
  errorHandler.emit('error', reason);
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM received. Shutting down gracefully...');
  process.exit(0);
});

// Test errors
setTimeout(() => {
  throw new Error('Test uncaught exception');
}, 5000);
```

---

## 8. NPM Package Manager

### Basic Commands

```bash
# Initialize a new project
npm init
npm init -y  # Quick initialization with defaults

# Install packages
npm install package-name
npm install package-name@version
npm install package-name --save-dev  # Development dependency

# Global installation
npm install -g package-name

# List installed packages
npm list
npm list --depth=0  # Top-level only

# Update packages
npm update
npm update package-name

# Uninstall packages
npm uninstall package-name

# Clear cache
npm cache clean --force

# Audit for vulnerabilities
npm audit
npm audit fix

# Run scripts
npm run script-name
```

### package.json Structure

**File: `examples/package.json`**
```json
{
  "name": "nodejs-complete-guide",
  "version": "1.0.0",
  "description": "Complete Node.js learning guide",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest",
    "lint": "eslint .",
    "build": "babel src -d dist"
  },
  "keywords": [
    "nodejs",
    "tutorial",
    "learning"
  ],
  "author": "Your Name",
  "license": "MIT",
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.22",
    "jest": "^29.5.0",
    "eslint": "^8.40.0"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

### Popular NPM Packages

#### Essential Packages
- **express**: Web framework
- **mongoose**: MongoDB ODM
- **sequelize**: SQL ORM
- **dotenv**: Environment variables
- **cors**: Cross-origin resource sharing
- **helmet**: Security headers
- **morgan**: HTTP request logger
- **winston**: Logging library
- **jsonwebtoken**: JWT authentication
- **bcryptjs**: Password hashing

#### Development Tools
- **nodemon**: Auto-restart on file changes
- **jest**: Testing framework
- **eslint**: Code linting
- **prettier**: Code formatting
- **supertest**: API testing

### Creating Your Own NPM Package

**File: `examples/my-npm-package/index.js`**
```javascript
/**
 * A simple utility package
 * @module my-utils
 */

/**
 * Capitalize first letter of string
 * @param {string} str - Input string
 * @returns {string} Capitalized string
 */
function capitalize(str) {
  if (!str || typeof str !== 'string') {
    throw new Error('Input must be a non-empty string');
  }
  return str.charAt(0).toUpperCase() + str.slice(1);
}

/**
 * Generate random ID
 * @param {number} length - Length of ID
 * @returns {string} Random ID
 */
function generateId(length = 10) {
  const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  let result = '';
  for (let i = 0; i < length; i++) {
    result += chars.charAt(Math.floor(Math.random() * chars.length));
  }
  return result;
}

/**
 * Format date
 * @param {Date} date - Date object
 * @param {string} format - Format string
 * @returns {string} Formatted date
 */
function formatDate(date, format = 'YYYY-MM-DD') {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  
  return format
    .replace('YYYY', year)
    .replace('MM', month)
    .replace('DD', day);
}

module.exports = {
  capitalize,
  generateId,
  formatDate
};
```

**File: `examples/my-npm-package/package.json`**
```json
{
  "name": "@yourusername/my-utils",
  "version": "1.0.0",
  "description": "Utility functions for Node.js projects",
  "main": "index.js",
  "scripts": {
    "test": "jest",
    "publish": "npm publish"
  },
  "keywords": [
    "utils",
    "helpers",
    "utilities"
  ],
  "author": "Your Name",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/yourusername/my-utils.git"
  }
}
```

### Real-World Project Setup

**File: `examples/20-project-setup.sh`**
```bash
#!/bin/bash

# Create project structure
mkdir -p my-node-app/{src,tests,config,logs,public}
cd my-node-app

# Initialize npm
npm init -y

# Install production dependencies
npm install express mongoose dotenv cors helmet morgan winston bcryptjs jsonwebtoken

# Install development dependencies
npm install --save-dev nodemon jest eslint prettier supertest

# Create .env file
cat > .env << EOF
PORT=3000
NODE_ENV=development
MONGODB_URI=mongodb://localhost:27017/myapp
JWT_SECRET=your-secret-key
EOF

# Create .gitignore
cat > .gitignore << EOF
node_modules/
.env
logs/
*.log
.DS_Store
EOF

echo "Project setup complete!"
```

---

## Exercises

### Exercise 1: File Processor
Create a script that reads a text file, counts word frequency, and writes results to a new file.

### Exercise 2: Simple HTTP Server
Build an HTTP server that serves different responses based on the URL path.

### Exercise 3: Event Emitter Chat
Implement a simple chat system using Event Emitter pattern.

### Exercise 4: Stream Transformer
Create a transform stream that converts text to uppercase while reading from a file.

### Exercise 5: Error Handler Middleware
Build a reusable error handling middleware for Express applications.

---

## Quiz

1. What makes Node.js non-blocking?
2. Explain the difference between CommonJS and ES6 modules.
3. When would you use streams instead of reading entire files?
4. What's the purpose of the event loop?
5. How do you handle asynchronous errors in Node.js?

---

## Next Steps

After mastering these basics, move to **Part 2: Intermediate Node.js** where you'll learn:
- Advanced asynchronous patterns
- Building REST APIs
- Authentication & Authorization
- Database integration fundamentals

---

## Additional Resources

- [Official Node.js Documentation](https://nodejs.org/docs/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [Express.js Guide](https://expressjs.com/en/guide/routing.html)
- [NPM Documentation](https://docs.npmjs.com/)
