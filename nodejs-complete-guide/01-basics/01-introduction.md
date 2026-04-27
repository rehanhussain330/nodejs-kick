# Node.js Basics - Topic 01: Introduction to Node.js

## Theory
Node.js is a JavaScript runtime built on Chrome's V8 engine that allows executing JavaScript on the server-side. It enables building scalable network applications using an event-driven, non-blocking I/O model.

### Key Features:
- **Event-driven**: Uses events and callbacks for handling operations
- **Non-blocking I/O**: Operations don't block the execution thread
- **Single-threaded**: Uses a single thread with event loop
- **Fast execution**: Built on Chrome's V8 JavaScript engine
- **NPM**: Largest ecosystem of open-source libraries

## Code Examples

### 1. Hello World
```javascript
// hello.js
console.log("Hello from Node.js!");
```

### 2. Simple HTTP Server
```javascript
// server.js
const http = require('http');

const server = http.createServer((req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World from Node.js!');
});

server.listen(3000, () => {
    console.log('Server running at http://localhost:3000/');
});
```

### 3. Reading Command Line Arguments
```javascript
// args.js
console.log('Process ID:', process.pid);
console.log('Node version:', process.version);
console.log('Arguments:', process.argv);

// Run with: node args.js arg1 arg2
```

### 4. Global Objects
```javascript
// globals.js
console.log('__dirname:', __dirname);
console.log('__filename:', __filename);
console.log('setTimeout:', typeof setTimeout);
console.log('setInterval:', typeof setInterval);
console.log('Buffer:', typeof Buffer);
```

### 5. Process Object
```javascript
// process-info.js
console.log('Platform:', process.platform);
console.log('Architecture:', process.arch);
console.log('Memory Usage:', process.memoryUsage());
console.log('Uptime:', process.uptime(), 'seconds');
console.log('Environment:', process.env.NODE_ENV || 'development');
```

## Real-time Use Cases

### 1. Web Servers
- Building REST APIs
- Serving static files
- Handling HTTP requests/responses

### 2. Command Line Tools
- Build tools (Webpack, Gulp)
- Development utilities
- Automation scripts

### 3. Real-time Applications
- Chat applications
- Live notifications
- Collaborative tools

## Practice Exercises

1. Create a simple HTTP server that returns current time
2. Build a CLI tool that reads file size
3. Create a server that serves different responses based on URL path
4. Log all environment variables to a file

## Next Steps
- Learn about Event Loop (Topic 02)
- Understand Modules System (Topic 03)
- Explore File System operations (Topic 04)
