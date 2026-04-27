# Node.js Basics - Topic 05: Buffers and Streams

## Theory
Buffers and Streams are fundamental concepts in Node.js for handling binary data and efficient data processing.

### Buffers:
- Temporary storage for binary data
- Fixed-size memory allocation outside V8 heap
- Used for file operations, network communications, cryptography

### Streams:
- Collections of data that might not be available all at once
- Allow processing data chunk by chunk
- Memory efficient for large datasets
- Four types: Readable, Writable, Duplex, Transform

## Code Examples

### 1. Working with Buffers
```javascript
// buffers.js

// Create buffer from string
const buf1 = Buffer.from('Hello Node.js', 'utf8');
console.log(buf1.toString()); // 'Hello Node.js'
console.log(buf1.length); // 13 bytes

// Create buffer with size
const buf2 = Buffer.alloc(10);
buf2.write('ABC');
console.log(buf2.toString()); // 'ABC\u0000\u0000\u0000\u0000\u0000\u0000\u0000'

// Create buffer with data
const buf3 = Buffer.from([72, 101, 108, 108, 111]); // ASCII codes
console.log(buf3.toString()); // 'Hello'

// Buffer operations
const buf4 = Buffer.from('World');
const combined = Buffer.concat([buf3, Buffer.from(' '), buf4]);
console.log(combined.toString()); // 'Hello World'

// Compare buffers
console.log(Buffer.compare(buf1, buf2)); // Comparison result

// Slice buffer (no copy)
const sliced = buf1.slice(0, 5);
console.log(sliced.toString()); // 'Hello'
```

### 2. Readable Streams
```javascript
// readable-streams.js
const fs = require('fs');
const { Readable } = require('stream');

// Method 1: From file
const readStream = fs.createReadStream('large-file.txt', {
    highWaterMark: 1024, // 1KB chunks
    encoding: 'utf8'
});

readStream.on('data', (chunk) => {
    console.log('Received chunk:', chunk.length, 'bytes');
});

readStream.on('end', () => {
    console.log('Reading completed');
});

readStream.on('error', (err) => {
    console.error('Read error:', err);
});

// Method 2: Custom readable stream
class NumberStream extends Readable {
    constructor(options) {
        super(options);
        this.current = 0;
        this.max = options.max || 100;
    }

    _read(size) {
        if (this.current >= this.max) {
            this.push(null); // Signal end
            return;
        }
        
        const chunk = `${this.current++}\n`;
        this.push(chunk);
    }
}

const numStream = new NumberStream({ max: 10 });
numStream.pipe(process.stdout);
```

### 3. Writable Streams
```javascript
// writable-streams.js
const fs = require('fs');
const { Writable } = require('stream');

// Method 1: To file
const writeStream = fs.createWriteStream('output.txt');

for (let i = 0; i < 1000; i++) {
    const canContinue = writeStream.write(`Line ${i}\n`);
    
    if (!canContinue) {
        // Backpressure: wait for drain event
        writeStream.once('drain', () => {
            console.log('Ready to write more');
        });
    }
}

writeStream.end();
writeStream.on('finish', () => {
    console.log('Writing completed');
});

// Method 2: Custom writable stream
class LogStream extends Writable {
    _write(chunk, encoding, callback) {
        const timestamp = new Date().toISOString();
        const logEntry = `[${timestamp}] ${chunk.toString()}`;
        
        // Process log entry (e.g., save to database, send to service)
        console.log('Processing log:', logEntry.trim());
        
        callback(); // Signal completion
    }
}

const logStream = new LogStream();
logStream.write('Application started\n');
logStream.write('User logged in\n');
logStream.end();
```

### 4. Piping Streams
```javascript
// piping-streams.js
const fs = require('fs');
const zlib = require('zlib');

// Simple pipe: Read -> Write
const src = fs.createReadStream('input.txt');
const dest = fs.createWriteStream('output.txt');
src.pipe(dest);

// Chain pipes: Read -> Compress -> Write
const readStream = fs.createReadStream('large-file.txt');
const gzip = zlib.createGzip();
const writeStream = fs.createWriteStream('large-file.txt.gz');

readStream
    .pipe(gzip)
    .pipe(writeStream)
    .on('finish', () => {
        console.log('File compressed successfully');
    });

// Decompress
const gunzip = zlib.createGunzip();
const readGzip = fs.createReadStream('large-file.txt.gz');
const writeUnzip = fs.createWriteStream('uncompressed.txt');

readGzip
    .pipe(gunzip)
    .pipe(writeUnzip);
```

### 5. Transform Streams
```javascript
// transform-streams.js
const { Transform } = require('stream');
const fs = require('fs');

// Create uppercase transform stream
class UpperCaseTransform extends Transform {
    _transform(chunk, encoding, callback) {
        const upperCaseData = chunk.toString().toUpperCase();
        this.push(upperCaseData);
        callback();
    }
}

// Use transform stream
const readStream = fs.createReadStream('input.txt', 'utf8');
const upperTransform = new UpperCaseTransform();
const writeStream = fs.createWriteStream('output-upper.txt', 'utf8');

readStream
    .pipe(upperTransform)
    .pipe(writeStream)
    .on('finish', () => {
        console.log('Transformation completed');
    });

// Line-by-line processor
class LineProcessor extends Transform {
    constructor(options) {
        super({ ...options, objectMode: true });
        this.buffer = '';
    }

    _transform(chunk, encoding, callback) {
        this.buffer += chunk.toString();
        const lines = this.buffer.split('\n');
        this.buffer = lines.pop(); // Keep incomplete line
        
        lines.forEach(line => {
            this.push(this.processLine(line));
        });
        
        callback();
    }

    _flush(callback) {
        if (this.buffer) {
            this.push(this.processLine(this.buffer));
        }
        callback();
    }

    processLine(line) {
        return {
            original: line,
            length: line.length,
            words: line.split(' ').length,
            timestamp: new Date().toISOString()
        };
    }
}
```

### 6. Stream Events and Error Handling
```javascript
// stream-events.js
const fs = require('fs');

const readStream = fs.createReadStream('nonexistent.txt');

// Handle all stream events
readStream.on('open', () => {
    console.log('Stream opened');
});

readStream.on('data', (chunk) => {
    console.log('Data received');
});

readStream.on('end', () => {
    console.log('Stream ended');
});

readStream.on('close', () => {
    console.log('Stream closed');
});

readStream.on('error', (err) => {
    console.error('Stream error:', err.message);
});

// Proper error handling with pipe
const src = fs.createReadStream('source.txt');
const dest = fs.createWriteStream('dest.txt');

src.on('error', (err) => {
    console.error('Read error:', err);
    dest.end();
});

dest.on('error', (err) => {
    console.error('Write error:', err);
    src.destroy();
});

dest.on('finish', () => {
    console.log('Copy completed successfully');
});

src.pipe(dest);
```

### 7. Advanced: File Upload with Progress
```javascript
// upload-progress.js
const fs = require('fs');
const { Transform } = require('stream');

class ProgressTracker extends Transform {
    constructor(totalSize, options) {
        super(options);
        this.totalSize = totalSize;
        this.received = 0;
        this.lastProgress = 0;
    }

    _transform(chunk, encoding, callback) {
        this.received += chunk.length;
        
        const progress = Math.round((this.received / this.totalSize) * 100);
        
        // Report progress every 10%
        if (progress >= this.lastProgress + 10) {
            this.lastProgress = progress;
            console.log(`Upload progress: ${progress}%`);
        }
        
        this.push(chunk);
        callback();
    }
}

async function uploadWithProgress(filePath) {
    const stats = await fs.promises.stat(filePath);
    const fileSize = stats.size;
    
    const readStream = fs.createReadStream(filePath);
    const progressTracker = new ProgressTracker(fileSize);
    
    // Simulate upload destination
    const uploadStream = fs.createWriteStream('uploaded.tmp');
    
    return new Promise((resolve, reject) => {
        readStream
            .pipe(progressTracker)
            .pipe(uploadStream)
            .on('finish', () => {
                console.log('Upload completed!');
                resolve();
            })
            .on('error', reject);
    });
}
```

## Real-time Use Cases

### 1. Video Streaming Server
```javascript
// video-streaming.js
const fs = require('fs');
const http = require('http');
const path = require('path');

http.createServer((req, res) => {
    const videoPath = path.join(__dirname, 'video.mp4');
    const stat = fs.statSync(videoPath);
    const fileSize = stat.size;
    const range = req.headers.range;

    if (range) {
        const parts = range.replace(/bytes=/, '').split('-');
        const start = parseInt(parts[0], 10);
        const end = parts[1] ? parseInt(parts[1], 10) : fileSize - 1;
        const chunksize = (end - start) + 1;
        
        const file = fs.createReadStream(videoPath, { start, end });
        
        res.writeHead(206, {
            'Content-Range': `bytes ${start}-${end}/${fileSize}`,
            'Accept-Ranges': 'bytes',
            'Content-Length': chunksize,
            'Content-Type': 'video/mp4'
        });
        
        file.pipe(res);
    } else {
        const file = fs.createReadStream(videoPath);
        res.writeHead(200, {
            'Content-Length': fileSize,
            'Content-Type': 'video/mp4'
        });
        file.pipe(res);
    }
}).listen(3000);
```

### 2. Data Processing Pipeline
```javascript
// data-pipeline.js
const { Transform } = require('stream');
const fs = require('fs');

// CSV Parser Transform
class CSVParser extends Transform {
    constructor(options) {
        super({ ...options, objectMode: true });
        this.headers = null;
        this.buffer = '';
    }

    _transform(chunk, encoding, callback) {
        this.buffer += chunk.toString();
        const lines = this.buffer.split('\n');
        this.buffer = lines.pop();

        if (!this.headers && lines.length > 0) {
            this.headers = lines.shift().split(',');
        }

        lines.forEach(line => {
            if (line.trim()) {
                const values = line.split(',');
                const obj = {};
                this.headers.forEach((header, i) => {
                    obj[header.trim()] = values[i]?.trim();
                });
                this.push(obj);
            }
        });

        callback();
    }
}

// Data Validator Transform
class DataValidator extends Transform {
    constructor(options) {
        super({ ...options, objectMode: true });
    }

    _transform(obj, encoding, callback) {
        if (obj.email && obj.email.includes('@')) {
            this.push({ ...obj, valid: true });
        } else {
            this.push({ ...obj, valid: false, error: 'Invalid email' });
        }
        callback();
    }
}

// Usage
const readStream = fs.createReadStream('data.csv', 'utf8');
const parser = new CSVParser();
const validator = new DataValidator();
const output = fs.createWriteStream('validated.json', 'utf8');

let first = true;
output.write('[');

validator.on('data', (obj) => {
    if (!first) output.write(',');
    output.write(JSON.stringify(obj));
    first = false;
});

validator.on('end', () => {
    output.write(']');
    output.end();
});

readStream.pipe(parser).pipe(validator);
```

## Best Practices

### 1. Handle Backpressure
```javascript
// Good: Respect backpressure
async function writeFile(data) {
    const stream = fs.createWriteStream('output.txt');
    
    for (const chunk of data) {
        if (!stream.write(chunk)) {
            await new Promise(resolve => stream.once('drain', resolve));
        }
    }
    
    stream.end();
}
```

### 2. Always Handle Errors
```javascript
stream.on('error', (err) => {
    console.error('Stream error:', err);
    // Clean up resources
});
```

### 3. Use Pipeline for Better Error Handling
```javascript
const { pipeline } = require('stream');
const { promisify } = require('util');
const pump = promisify(pipeline);

await pump(
    fs.createReadStream('input.txt'),
    zlib.createGzip(),
    fs.createWriteStream('output.txt.gz')
);
```

## Practice Exercises

1. Create a stream that encrypts data on-the-fly
2. Build a log aggregator that processes multiple log files
3. Implement a file splitter that divides large files into chunks
4. Create a real-time data transformer for sensor readings

## Next Steps
- Learn Error Handling patterns (Topic 06)
- Explore Events and EventEmitter (Intermediate Topic 07)
- Master HTTP and APIs (Intermediate Topic 08)
