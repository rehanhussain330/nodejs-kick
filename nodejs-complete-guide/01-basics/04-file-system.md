# Node.js Basics - Topic 04: File System Operations

## Theory
Node.js provides the `fs` module for reading, writing, and manipulating files. It supports both synchronous and asynchronous operations, allowing efficient file handling without blocking the event loop.

### Key Concepts:
- **Synchronous Operations**: Block execution until completion (e.g., `readFileSync`)
- **Asynchronous Operations**: Non-blocking with callbacks (e.g., `readFile`)
- **Promise-based API**: Modern approach using `fs.promises`
- **Streams**: Efficient handling of large files
- **Watchers**: Monitor file changes in real-time

## Code Examples

### 1. Reading Files
```javascript
// read-files.js
const fs = require('fs');
const path = require('path');

// Asynchronous read with callback
fs.readFile(path.join(__dirname, 'data.txt'), 'utf8', (err, data) => {
    if (err) {
        console.error('Error reading file:', err);
        return;
    }
    console.log('Async read:', data);
});

// Synchronous read
try {
    const data = fs.readFileSync(path.join(__dirname, 'data.txt'), 'utf8');
    console.log('Sync read:', data);
} catch (err) {
    console.error('Sync error:', err);
}

// Promise-based read (Node.js 10+)
async function readWithPromises() {
    try {
        const data = await fs.promises.readFile(
            path.join(__dirname, 'data.txt'), 
            'utf8'
        );
        console.log('Promise read:', data);
    } catch (err) {
        console.error('Promise error:', err);
    }
}

readWithPromises();
```

### 2. Writing Files
```javascript
// write-files.js
const fs = require('fs');
const path = require('path');

const content = 'Hello, Node.js File System!';

// Asynchronous write
fs.writeFile(
    path.join(__dirname, 'output.txt'),
    content,
    'utf8',
    (err) => {
        if (err) {
            console.error('Write error:', err);
            return;
        }
        console.log('File written successfully');
    }
);

// Synchronous write
try {
    fs.writeFileSync(
        path.join(__dirname, 'output-sync.txt'),
        content,
        'utf8'
    );
    console.log('Sync write completed');
} catch (err) {
    console.error('Sync write error:', err);
}

// Append to file
fs.appendFile(
    path.join(__dirname, 'output.txt'),
    '\nAdditional content',
    (err) => {
        if (!err) console.log('Content appended');
    }
);
```

### 3. File Information and Stats
```javascript
// file-stats.js
const fs = require('fs');
const path = require('path');

// Get file statistics
fs.stat(path.join(__dirname, 'data.txt'), (err, stats) => {
    if (err) {
        console.error('Stat error:', err);
        return;
    }
    
    console.log('File size:', stats.size, 'bytes');
    console.log('Is file?', stats.isFile());
    console.log('Is directory?', stats.isDirectory());
    console.log('Created:', stats.birthtime);
    console.log('Modified:', stats.mtime);
    console.log('Permissions:', stats.mode.toString(8));
});

// Check if path exists
fs.exists(path.join(__dirname, 'data.txt'), (exists) => {
    console.log('File exists:', exists);
});

// Modern exists check
async function checkExists(filePath) {
    try {
        await fs.promises.access(filePath);
        console.log('File is accessible');
        return true;
    } catch {
        console.log('File not accessible');
        return false;
    }
}
```

### 4. Directory Operations
```javascript
// directory-ops.js
const fs = require('fs');
const path = require('path');

// Create directory
fs.mkdir(path.join(__dirname, 'newDir'), { recursive: true }, (err) => {
    if (!err) console.log('Directory created');
});

// Read directory contents
fs.readdir(path.join(__dirname), (err, files) => {
    if (err) {
        console.error('Readdir error:', err);
        return;
    }
    console.log('Directory contents:', files);
    
    // Filter only files
    const jsFiles = files.filter(file => file.endsWith('.js'));
    console.log('JS files:', jsFiles);
});

// Remove directory (must be empty)
fs.rmdir(path.join(__dirname, 'emptyDir'), (err) => {
    if (!err) console.log('Directory removed');
});

// Remove directory recursively (Node.js 14.14+)
fs.rm(path.join(__dirname, 'dirToRemove'), { recursive: true, force: true }, (err) => {
    if (!err) console.log('Directory recursively removed');
});
```

### 5. File Renaming and Copying
```javascript
// file-operations.js
const fs = require('fs');
const path = require('path');

// Rename file
fs.rename(
    path.join(__dirname, 'oldName.txt'),
    path.join(__dirname, 'newName.txt'),
    (err) => {
        if (!err) console.log('File renamed');
    }
);

// Copy file (Node.js 14+)
fs.copyFile(
    path.join(__dirname, 'source.txt'),
    path.join(__dirname, 'destination.txt'),
    (err) => {
        if (!err) console.log('File copied');
    }
);

// Copy with promises
async function copyFile(src, dest) {
    try {
        await fs.promises.copyFile(src, dest);
        console.log('File copied with promises');
    } catch (err) {
        console.error('Copy error:', err);
    }
}
```

### 6. Working with Streams
```javascript
// file-streams.js
const fs = require('fs');
const path = require('path');

// Read stream for large files
const readStream = fs.createReadStream(
    path.join(__dirname, 'largeFile.txt'),
    { encoding: 'utf8', highWaterMark: 1024 }
);

readStream.on('data', (chunk) => {
    console.log('Received chunk:', chunk.length, 'bytes');
});

readStream.on('end', () => {
    console.log('Read stream completed');
});

readStream.on('error', (err) => {
    console.error('Read stream error:', err);
});

// Write stream
const writeStream = fs.createWriteStream(
    path.join(__dirname, 'stream-output.txt')
);

for (let i = 0; i < 1000; i++) {
    writeStream.write(`Line ${i}\n`);
}

writeStream.end();
writeStream.on('finish', () => {
    console.log('Write stream completed');
});

// Pipe streams (efficient file copying)
const src = fs.createReadStream(path.join(__dirname, 'source.txt'));
const dest = fs.createWriteStream(path.join(__dirname, 'copy.txt'));
src.pipe(dest);
```

### 7. Watching Files for Changes
```javascript
// file-watcher.js
const fs = require('fs');
const path = require('path');

// Watch a file
const watcher = fs.watch(
    path.join(__dirname, 'data.txt'),
    { persistent: true, recursive: false },
    (eventType, filename) => {
        console.log(`Event: ${eventType}`);
        console.log(`Filename: ${filename}`);
        
        if (eventType === 'change') {
            fs.readFile(path.join(__dirname, filename), 'utf8', (err, data) => {
                if (!err) {
                    console.log('New content:', data);
                }
            });
        }
    }
);

// Watcher events
watcher.on('error', (err) => {
    console.error('Watcher error:', err);
});

// Stop watching after 30 seconds
setTimeout(() => {
    watcher.close();
    console.log('Stopped watching');
}, 30000);
```

### 8. Advanced: File Upload Handler
```javascript
// file-upload-handler.js
const fs = require('fs');
const path = require('path');
const { pipeline } = require('stream');
const { promisify } = require('util');

const pump = promisify(pipeline);

async function handleFileUpload(uploadStream, filename) {
    const uploadDir = path.join(__dirname, 'uploads');
    
    // Ensure upload directory exists
    await fs.promises.mkdir(uploadDir, { recursive: true });
    
    const filePath = path.join(uploadDir, filename);
    const writeStream = fs.createWriteStream(filePath);
    
    try {
        await pump(uploadStream, writeStream);
        console.log('File uploaded successfully:', filePath);
        
        // Get file stats
        const stats = await fs.promises.stat(filePath);
        return {
            success: true,
            path: filePath,
            size: stats.size,
            filename: filename
        };
    } catch (err) {
        console.error('Upload error:', err);
        // Clean up partial file
        await fs.promises.unlink(filePath).catch(() => {});
        throw err;
    }
}

module.exports = { handleFileUpload };
```

## Real-time Use Cases

### 1. Log File Management
```javascript
// logger.js
const fs = require('fs');
const path = require('path');

class FileLogger {
    constructor(logFile) {
        this.logFile = path.join(__dirname, logFile);
        this.stream = fs.createWriteStream(this.logFile, { flags: 'a' });
    }
    
    log(message, level = 'INFO') {
        const timestamp = new Date().toISOString();
        const logEntry = `[${timestamp}] [${level}] ${message}\n`;
        this.stream.write(logEntry);
    }
    
    error(message) {
        this.log(message, 'ERROR');
    }
    
    warn(message) {
        this.log(message, 'WARN');
    }
    
    close() {
        this.stream.end();
    }
}

// Usage
const logger = new FileLogger('app.log');
logger.log('Application started');
logger.error('Something went wrong');
```

### 2. Configuration File Manager
```javascript
// config-manager.js
const fs = require('fs').promises;
const path = require('path');

class ConfigManager {
    constructor(configFile) {
        this.configFile = path.join(__dirname, configFile);
    }
    
    async load() {
        try {
            const data = await fs.readFile(this.configFile, 'utf8');
            return JSON.parse(data);
        } catch (err) {
            if (err.code === 'ENOENT') {
                return {}; // Return empty config if file doesn't exist
            }
            throw err;
        }
    }
    
    async save(config) {
        const data = JSON.stringify(config, null, 2);
        await fs.writeFile(this.configFile, data, 'utf8');
    }
    
    async update(updates) {
        const config = await this.load();
        const updatedConfig = { ...config, ...updates };
        await this.save(updatedConfig);
        return updatedConfig;
    }
}
```

### 3. Batch File Processor
```javascript
// batch-processor.js
const fs = require('fs').promises;
const path = require('path');

async function processDirectory(dir, processorFn) {
    const files = await fs.readdir(dir);
    const results = [];
    
    for (const file of files) {
        const filePath = path.join(dir, file);
        const stat = await fs.stat(filePath);
        
        if (stat.isFile()) {
            try {
                const result = await processorFn(filePath, file);
                results.push({ file, success: true, result });
            } catch (err) {
                results.push({ file, success: false, error: err.message });
            }
        }
    }
    
    return results;
}

// Example: Convert all text files to uppercase
async function convertToUppercase(filePath) {
    const content = await fs.readFile(filePath, 'utf8');
    const upperContent = content.toUpperCase();
    await fs.writeFile(filePath, upperContent, 'utf8');
    return 'Converted to uppercase';
}
```

## Best Practices

### 1. Always Handle Errors
```javascript
// Good error handling
async function safeReadFile(filePath) {
    try {
        return await fs.promises.readFile(filePath, 'utf8');
    } catch (err) {
        if (err.code === 'ENOENT') {
            console.log('File not found');
            return null;
        }
        throw err; // Re-throw unexpected errors
    }
}
```

### 2. Use Streams for Large Files
```javascript
// Bad: Loading entire file into memory
const data = fs.readFileSync('huge-file.txt');

// Good: Using streams
const stream = fs.createReadStream('huge-file.txt');
stream.on('data', chunk => processChunk(chunk));
```

### 3. Clean Up Temporary Files
```javascript
async function processWithTempFile(data) {
    const tempFile = path.join(__dirname, `temp-${Date.now()}.tmp`);
    
    try {
        await fs.promises.writeFile(tempFile, data);
        // Process file
        const result = await processFile(tempFile);
        return result;
    } finally {
        // Always clean up
        await fs.promises.unlink(tempFile).catch(() => {});
    }
}
```

## Practice Exercises

1. Create a file backup utility that copies files with timestamps
2. Build a log rotation system that archives old logs
3. Implement a directory size calculator
4. Create a file search tool that finds files by pattern
5. Build a simple file watcher that triggers actions on changes

## Security Considerations

1. **Path Traversal**: Validate file paths to prevent accessing unauthorized files
2. **File Permissions**: Set appropriate permissions for created files
3. **Input Validation**: Sanitize filenames from user input
4. **Resource Limits**: Limit file sizes to prevent DoS attacks

```javascript
// Prevent path traversal
function sanitizePath(userPath) {
    const safePath = path.normalize(userPath).replace(/^(\.\.(\/|\\|$))+/, '');
    return path.join(__dirname, 'uploads', safePath);
}
```

## Next Steps
- Understand Buffers and Streams (Topic 05)
- Learn Error Handling patterns (Topic 06)
- Explore Events and EventEmitter (Intermediate Topic 07)
