# Node.js Basics - Topic 02: Event Loop & Architecture

## Theory
Node.js uses a single-threaded event loop with callback queues to handle multiple operations concurrently without blocking the main thread. This architecture allows Node.js to handle thousands of concurrent connections efficiently.

### Event Loop Phases:
1. **Timers**: Executes callbacks scheduled by setTimeout() and setInterval()
2. **Pending Callbacks**: Executes I/O callbacks deferred to the next loop iteration
3. **Idle/Prepare**: Internal use only
4. **Poll**: Retrieves new I/O events and executes their callbacks
5. **Check**: Executes setImmediate() callbacks
6. **Close Callbacks**: Executes close event callbacks (e.g., socket.on('close'))

### Key Concepts:
- **Call Stack**: Tracks function execution
- **Callback Queue**: Holds callbacks waiting to be executed
- **Event Queue**: Holds events waiting to be processed
- **Microtask Queue**: Higher priority queue for Promises and process.nextTick()

## Code Examples

### 1. Event Loop Execution Order
```javascript
// event-loop-order.js
console.log('Start');

setTimeout(() => {
    console.log('Timer 1 fired (macrotask)');
}, 0);

setImmediate(() => {
    console.log('Immediate 1 fired (check phase)');
});

process.nextTick(() => {
    console.log('Next tick 1 fired (microtask)');
});

Promise.resolve().then(() => {
    console.log('Promise 1 resolved (microtask)');
});

console.log('End');

// Output order:
// Start
// End
// Next tick 1 fired (microtask)
// Promise 1 resolved (microtask)
// Immediate 1 fired (check phase) OR Timer 1 fired (depends on platform)
// Timer 1 fired (macrotask)
```

### 2. Understanding Blocking vs Non-blocking
```javascript
// blocking-vs-nonblocking.js
const fs = require('fs');

console.log('Start');

// Synchronous (Blocking)
const dataSync = fs.readFileSync('file.txt', 'utf8');
console.log('Sync read completed:', dataSync);

// Asynchronous (Non-blocking)
fs.readFile('file.txt', 'utf8', (err, data) => {
    console.log('Async read completed:', data);
});

console.log('End - Async operation still running');
```

### 3. Microtasks vs Macrotasks
```javascript
// microtasks-macrotasks.js
console.log('Script start');

setTimeout(() => {
    console.log('setTimeout 1');
    Promise.resolve().then(() => {
        console.log('Promise inside setTimeout');
    });
}, 0);

Promise.resolve().then(() => {
    console.log('Promise 1');
});

Promise.resolve().then(() => {
    console.log('Promise 2');
    setTimeout(() => {
        console.log('setTimeout inside Promise');
    }, 0);
});

console.log('Script end');

// Output:
// Script start
// Script end
// Promise 1
// Promise 2
// setTimeout 1
// Promise inside setTimeout
// setTimeout inside Promise
```

### 4. Event Loop with I/O Operations
```javascript
// event-loop-io.js
const http = require('http');
const fs = require('fs');

console.log('Starting server...');

const server = http.createServer((req, res) => {
    console.log('Request received:', req.url);
    
    // Simulate async file read
    fs.readFile('data.txt', 'utf8', (err, data) => {
        if (err) {
            res.writeHead(500);
            res.end('Error reading file');
            return;
        }
        
        res.writeHead(200, {'Content-Type': 'text/plain'});
        res.end(data);
        console.log('Response sent');
    });
});

server.listen(3000, () => {
    console.log('Server listening on port 3000');
});
```

### 5. Process.nextTick vs setImmediate
```javascript
// nexttick-vs-immediate.js
console.log('Start');

process.nextTick(() => {
    console.log('nextTick 1');
    process.nextTick(() => {
        console.log('nextTick 2 (nested)');
    });
});

setImmediate(() => {
    console.log('immediate 1');
    setImmediate(() => {
        console.log('immediate 2 (nested)');
    });
});

console.log('End');

// Output:
// Start
// End
// nextTick 1
// nextTick 2 (nested)
// immediate 1
// immediate 2 (nested)
```

## Real-time Use Cases

### 1. High-Concurrency Web Servers
- Handle thousands of simultaneous connections
- Real-time chat applications
- Live streaming platforms

### 2. API Gateways
- Route requests to multiple microservices
- Aggregate responses from different services
- Handle authentication and rate limiting

### 3. Real-time Data Processing
- Stock market data feeds
- IoT sensor data collection
- Social media activity streams

### 4. File Processing Systems
- Upload processing with progress tracking
- Image/video transcoding
- Batch data transformations

## Performance Considerations

### 1. Avoid Blocking the Event Loop
```javascript
// BAD: Blocking operation
function badExample() {
    const result = heavyComputation(); // Blocks event loop
    return result;
}

// GOOD: Non-blocking operation
function goodExample(callback) {
    setImmediate(() => {
        const result = heavyComputation();
        callback(null, result);
    });
}
```

### 2. Use Worker Threads for CPU-intensive Tasks
```javascript
// For CPU-heavy operations, use worker threads
const { Worker } = require('worker_threads');

function runWorker(script, data) {
    return new Promise((resolve, reject) => {
        const worker = new Worker(script, { workerData: data });
        worker.on('message', resolve);
        worker.on('error', reject);
    });
}
```

### 3. Proper Error Handling in Event Loop
```javascript
process.on('uncaughtException', (err) => {
    console.error('Uncaught Exception:', err);
    // Clean up and exit gracefully
    process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
    console.error('Unhandled Rejection at:', promise, 'reason:', reason);
});
```

## Practice Exercises

1. Create a program that demonstrates the difference between setTimeout and setImmediate
2. Build a simple event emitter that processes tasks in the correct order
3. Implement a non-blocking file processor that handles multiple files concurrently
4. Create a benchmark comparing blocking vs non-blocking operations

## Debugging Tips

1. Use `NODE_DEBUG=eventloop` to debug event loop behavior
2. Monitor event loop lag with `process.hrtime()`
3. Use Chrome DevTools for Node.js debugging
4. Profile your application with `--inspect` flag

## Next Steps
- Learn about Modules System (Topic 03)
- Explore File System operations (Topic 04)
- Understand Buffers and Streams (Topic 05)
