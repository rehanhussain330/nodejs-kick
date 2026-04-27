# Node.js Basics - Topic 06: Error Handling

## Theory
Proper error handling is crucial for building robust Node.js applications. Node.js provides multiple patterns for handling errors including callbacks, promises, async/await, and event emitters.

### Error Types:
- **Operational Errors**: Expected errors (file not found, network timeout)
- **Programmer Errors**: Bugs in code (undefined variable, wrong type)
- **System Errors**: OS-level errors (out of memory, disk full)

### Error Handling Patterns:
1. Error-first callbacks
2. Try-catch with async/await
3. Promise .catch()
4. Event emitter error events
5. Global error handlers

## Code Examples

### 1. Error-First Callbacks
```javascript
// callback-errors.js
const fs = require('fs');

// Standard Node.js callback pattern
fs.readFile('file.txt', 'utf8', (err, data) => {
    if (err) {
        // Handle error
        console.error('Failed to read file:', err.message);
        return;
    }
    // Use data
    console.log(data);
});

// Custom function with error-first callback
function divide(a, b, callback) {
    if (b === 0) {
        callback(new Error('Division by zero'), null);
        return;
    }
    callback(null, a / b);
}

divide(10, 2, (err, result) => {
    if (err) {
        console.error('Error:', err.message);
    } else {
        console.log('Result:', result);
    }
});
```

### 2. Try-Catch with Async/Await
```javascript
// async-await-errors.js
const fs = require('fs').promises;

async function readFileAsync(filePath) {
    try {
        const data = await fs.readFile(filePath, 'utf8');
        return data;
    } catch (err) {
        if (err.code === 'ENOENT') {
            console.error('File not found:', filePath);
            return null;
        }
        console.error('Read error:', err.message);
        throw err; // Re-throw if can't handle
    }
}

// Multiple async operations
async function processFiles(files) {
    const results = [];
    
    for (const file of files) {
        try {
            const data = await readFileAsync(file);
            if (data) {
                results.push({ file, success: true, data });
            }
        } catch (err) {
            results.push({ file, success: false, error: err.message });
        }
    }
    
    return results;
}

// Using Promise.all with error handling
async function processFilesParallel(files) {
    try {
        const promises = files.map(file => 
            fs.readFile(file, 'utf8')
                .then(data => ({ file, success: true, data }))
                .catch(err => ({ file, success: false, error: err.message }))
        );
        
        const results = await Promise.all(promises);
        return results;
    } catch (err) {
        console.error('Unexpected error:', err);
        throw err;
    }
}
```

### 3. Promise Error Handling
```javascript
// promise-errors.js
const fs = require('fs').promises;

// Method 1: .catch()
fs.readFile('file.txt', 'utf8')
    .then(data => {
        console.log(data);
    })
    .catch(err => {
        console.error('Promise error:', err.message);
    });

// Method 2: Multiple catches
fs.readFile('file.txt', 'utf8')
    .then(data => JSON.parse(data))
    .then(json => {
        console.log('Parsed JSON:', json);
    })
    .catch(SyntaxError, err => {
        console.error('Invalid JSON:', err.message);
    })
    .catch(err => {
        console.error('Other error:', err.message);
    });

// Method 3: Finally block
fs.readFile('file.txt', 'utf8')
    .then(data => console.log(data))
    .catch(err => console.error(err))
    .finally(() => {
        console.log('Operation completed');
    });
```

### 4. Custom Error Classes
```javascript
// custom-errors.js

// Base application error
class AppError extends Error {
    constructor(message, statusCode, code) {
        super(message);
        this.statusCode = statusCode;
        this.code = code;
        this.isOperational = true;
        
        Error.captureStackTrace(this, this.constructor);
    }
}

// Specific error types
class ValidationError extends AppError {
    constructor(message, field) {
        super(message, 400, 'VALIDATION_ERROR');
        this.field = field;
    }
}

class NotFoundError extends AppError {
    constructor(resource) {
        super(`${resource} not found`, 404, 'NOT_FOUND');
        this.resource = resource;
    }
}

class AuthenticationError extends AppError {
    constructor(message = 'Authentication failed') {
        super(message, 401, 'AUTH_ERROR');
    }
}

class DatabaseError extends AppError {
    constructor(message) {
        super(message, 500, 'DATABASE_ERROR');
        this.isOperational = false; // Programmer/system error
    }
}

// Usage
function validateUser(user) {
    if (!user.email) {
        throw new ValidationError('Email is required', 'email');
    }
    if (!user.email.includes('@')) {
        throw new ValidationError('Invalid email format', 'email');
    }
    if (!user.password || user.password.length < 8) {
        throw new ValidationError('Password must be at least 8 characters', 'password');
    }
}

try {
    validateUser({ email: 'invalid', password: '123' });
} catch (err) {
    if (err instanceof ValidationError) {
        console.error(`Validation failed for ${err.field}: ${err.message}`);
    } else {
        console.error('Unexpected error:', err);
    }
}
```

### 5. Global Error Handlers
```javascript
// global-errors.js

// Uncaught exceptions
process.on('uncaughtException', (err) => {
    console.error('Uncaught Exception:', err);
    console.error('Stack:', err.stack);
    
    // Log to file or monitoring service
    // cleanup();
    
    // Exit process (recommended)
    process.exit(1);
});

// Unhandled promise rejections
process.on('unhandledRejection', (reason, promise) => {
    console.error('Unhandled Rejection at:', promise);
    console.error('Reason:', reason);
    
    // Log to monitoring service
});

// Warning events
process.on('warning', (warning) => {
    console.warn('Process Warning:', warning);
});

// Exit handlers
process.on('exit', (code) => {
    console.log(`Process exiting with code: ${code}`);
});

// SIGTERM for graceful shutdown
process.on('SIGTERM', () => {
    console.log('SIGTERM received, shutting down gracefully');
    // Cleanup resources
    process.exit(0);
});
```

### 6. Express.js Error Handling
```javascript
// express-errors.js
const express = require('express');
const app = express();

// Async wrapper for route handlers
const asyncHandler = (fn) => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

// Route with error handling
app.get('/users/:id', asyncHandler(async (req, res) => {
    const user = await getUserById(req.params.id);
    if (!user) {
        throw new NotFoundError('User');
    }
    res.json(user);
}));

// Error handling middleware (must have 4 parameters)
app.use((err, req, res, next) => {
    console.error('Error:', err);
    
    // Don't leak error details in production
    const statusCode = err.statusCode || 500;
    const message = process.env.NODE_ENV === 'production' 
        ? 'Internal server error' 
        : err.message;
    
    res.status(statusCode).json({
        error: {
            message,
            code: err.code,
            stack: process.env.NODE_ENV === 'development' ? err.stack : undefined
        }
    });
});

// 404 handler
app.use((req, res) => {
    res.status(404).json({ error: 'Not found' });
});
```

### 7. Retry Logic with Exponential Backoff
```javascript
// retry-logic.js

async function retry(fn, options = {}) {
    const {
        maxRetries = 3,
        delay = 1000,
        backoff = 2,
        shouldRetry = (err) => true
    } = options;
    
    let lastError;
    let currentDelay = delay;
    
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            return await fn();
        } catch (err) {
            lastError = err;
            
            if (!shouldRetry(err) || attempt === maxRetries) {
                break;
            }
            
            console.log(`Attempt ${attempt} failed, retrying in ${currentDelay}ms...`);
            await new Promise(resolve => setTimeout(resolve, currentDelay));
            currentDelay *= backoff;
        }
    }
    
    throw lastError;
}

// Usage
async function fetchWithRetry(url) {
    return retry(
        async () => {
            const response = await fetch(url);
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }
            return response.json();
        },
        {
            maxRetries: 5,
            delay: 1000,
            shouldRetry: (err) => err.message.includes('500') || err.message.includes('timeout')
        }
    );
}
```

### 8. Error Logging Service
```javascript
// error-logger.js
const fs = require('fs').promises;
const path = require('path');

class ErrorLogger {
    constructor(logDir = 'logs') {
        this.logDir = path.join(process.cwd(), logDir);
        this.init();
    }
    
    async init() {
        try {
            await fs.mkdir(this.logDir, { recursive: true });
        } catch (err) {
            console.error('Failed to create log directory:', err);
        }
    }
    
    async log(error, context = {}) {
        const logEntry = {
            timestamp: new Date().toISOString(),
            message: error.message,
            stack: error.stack,
            name: error.name,
            code: error.code,
            context,
            environment: {
                nodeVersion: process.version,
                platform: process.platform,
                pid: process.pid
            }
        };
        
        const date = new Date().toISOString().split('T')[0];
        const logFile = path.join(this.logDir, `error-${date}.log`);
        
        try {
            await fs.appendFile(
                logFile,
                JSON.stringify(logEntry) + '\n',
                'utf8'
            );
        } catch (err) {
            console.error('Failed to write error log:', err);
        }
        
        // Also log to console in development
        if (process.env.NODE_ENV !== 'production') {
            console.error('Error logged:', logEntry);
        }
    }
    
    async getErrors(date) {
        const logFile = path.join(this.logDir, `error-${date}.log`);
        try {
            const content = await fs.readFile(logFile, 'utf8');
            return content.split('\n')
                .filter(line => line.trim())
                .map(line => JSON.parse(line));
        } catch (err) {
            return [];
        }
    }
}

// Usage
const logger = new ErrorLogger();

async function riskyOperation() {
    try {
        // Risky code
        throw new Error('Something went wrong');
    } catch (err) {
        await logger.log(err, {
            userId: 123,
            action: 'update_profile'
        });
        throw err;
    }
}
```

## Real-time Use Cases

### 1. API Error Response Standardization
```javascript
// api-error-handler.js
class APIError extends Error {
    constructor(status, code, message, details = {}) {
        super(message);
        this.status = status;
        this.code = code;
        this.details = details;
    }
    
    toJSON() {
        return {
            success: false,
            error: {
                code: this.code,
                message: this.message,
                details: this.details
            }
        };
    }
}

// Error codes
const ErrorCodes = {
    VALIDATION: 'VALIDATION_ERROR',
    NOT_FOUND: 'NOT_FOUND',
    UNAUTHORIZED: 'UNAUTHORIZED',
    FORBIDDEN: 'FORBIDDEN',
    SERVER: 'INTERNAL_SERVER_ERROR'
};

// Middleware
function errorHandler(err, req, res, next) {
    if (err instanceof APIError) {
        return res.status(err.status).json(err.toJSON());
    }
    
    // Unknown error
    console.error('Unexpected error:', err);
    return res.status(500).json({
        success: false,
        error: {
            code: ErrorCodes.SERVER,
            message: 'Internal server error'
        }
    });
}
```

### 2. Database Transaction Error Handling
```javascript
// db-transaction-errors.js
async function transferMoney(fromId, toId, amount) {
    const client = await pool.connect();
    
    try {
        await client.query('BEGIN');
        
        // Check balance
        const balanceResult = await client.query(
            'SELECT balance FROM accounts WHERE id = $1',
            [fromId]
        );
        
        if (balanceResult.rows[0].balance < amount) {
            throw new ValidationError('Insufficient funds', 'balance');
        }
        
        // Deduct from sender
        await client.query(
            'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
            [amount, fromId]
        );
        
        // Add to receiver
        await client.query(
            'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
            [amount, toId]
        );
        
        // Log transaction
        await client.query(
            'INSERT INTO transactions (from_id, to_id, amount) VALUES ($1, $2, $3)',
            [fromId, toId, amount]
        );
        
        await client.query('COMMIT');
        return { success: true };
        
    } catch (err) {
        await client.query('ROLLBACK');
        
        if (err instanceof ValidationError) {
            throw err;
        }
        
        throw new DatabaseError('Transaction failed');
    } finally {
        client.release();
    }
}
```

## Best Practices

### 1. Always Handle Errors
```javascript
// Bad: Ignoring errors
fs.readFile('file.txt', (err, data) => {
    // No error handling!
    console.log(data);
});

// Good: Proper error handling
fs.readFile('file.txt', (err, data) => {
    if (err) {
        console.error('Failed to read file:', err);
        return;
    }
    console.log(data);
});
```

### 2. Don't Swallow Errors
```javascript
// Bad: Silently swallowing errors
try {
    riskyOperation();
} catch (err) {
    // Do nothing - dangerous!
}

// Good: Log and handle
try {
    riskyOperation();
} catch (err) {
    logger.error(err);
    throw err; // Or handle appropriately
}
```

### 3. Use Specific Error Types
```javascript
// Good: Specific error handling
try {
    await operation();
} catch (err) {
    if (err.code === 'ENOENT') {
        // Handle file not found
    } else if (err.code === 'EACCES') {
        // Handle permission error
    } else {
        // Handle other errors
        throw err;
    }
}
```

## Practice Exercises

1. Create a custom error class hierarchy for an e-commerce application
2. Implement a retry mechanism for API calls with exponential backoff
3. Build an error logging service that stores errors in a database
4. Create middleware for validating API request parameters

## Next Steps
- Explore Events and EventEmitter (Intermediate Topic 07)
- Master HTTP and APIs (Intermediate Topic 08)
- Learn Express.js fundamentals (Section 04)
