# Node.js Basics - Topic 03: Modules System

## Theory
Node.js uses the CommonJS modules system where each file is treated as a module with its own scope. Modules allow code organization, reusability, and encapsulation.

### Module Types:
1. **Core Modules**: Built-in modules (fs, http, path, etc.)
2. **Local Modules**: Custom modules created in your project
3. **Third-party Modules**: Installed via npm

### Module Resolution:
- Core modules are loaded first
- Local modules are resolved relative to the current file
- Third-party modules are searched in node_modules folders

## Code Examples

### 1. Creating and Exporting Modules
```javascript
// math.js - Multiple export methods

// Method 1: Export individual functions
function add(a, b) {
    return a + b;
}

function subtract(a, b) {
    return a - b;
}

function multiply(a, b) {
    return a * b;
}

// Export specific items
module.exports = {
    add,
    subtract,
    multiply
};

// Method 2: Export single function
// module.exports = function(a, b) { return a + b; };

// Method 3: Export class
// class Calculator { ... }
// module.exports = Calculator;
```

### 2. Importing Modules
```javascript
// app.js - Importing modules

// Destructuring import
const { add, subtract, multiply } = require('./math');

console.log(add(5, 3));      // 8
console.log(subtract(10, 4)); // 6
console.log(multiply(2, 7));  // 14

// Import entire module
const math = require('./math');
console.log(math.add(10, 5)); // 15

// Import with alias
const { add: sum } = require('./math');
console.log(sum(3, 4)); // 7
```

### 3. Module Caching
```javascript
// counter.js
let count = 0;

function increment() {
    count++;
    return count;
}

function getCount() {
    return count;
}

function reset() {
    count = 0;
}

module.exports = { increment, getCount, reset };

// cache-test.js
const counter1 = require('./counter');
const counter2 = require('./counter');

console.log(counter1.increment()); // 1
console.log(counter2.getCount());  // 1 (same instance due to caching)
console.log(counter1 === counter2); // true
```

### 4. Circular Dependencies
```javascript
// moduleA.js
const moduleB = require('./moduleB');

function funcA() {
    console.log('Function A called');
    moduleB.funcB();
}

module.exports = { funcA };

// moduleB.js
const moduleA = require('./moduleA');

function funcB() {
    console.log('Function B called');
    // Note: moduleA might be partially loaded here
}

module.exports = { funcB };

// main.js
const moduleA = require('./moduleA');
moduleA.funcA();
```

### 5. Dynamic Module Loading
```javascript
// dynamic-load.js
async function loadModule(moduleName) {
    try {
        const module = require(`./${moduleName}`);
        return module;
    } catch (error) {
        console.error(`Failed to load module: ${moduleName}`, error);
        throw error;
    }
}

// Conditional loading
function getHandler(type) {
    if (type === 'user') {
        return require('./handlers/userHandler');
    } else if (type === 'product') {
        return require('./handlers/productHandler');
    }
    throw new Error('Unknown handler type');
}
```

### 6. ES6 Modules in Node.js (Modern Approach)
```javascript
// package.json must have "type": "module" or use .mjs extension

// es6-module.mjs
export const PI = 3.14159;

export function calculateArea(radius) {
    return PI * radius * radius;
}

export default class Circle {
    constructor(radius) {
        this.radius = radius;
    }
    
    getArea() {
        return calculateArea(this.radius);
    }
}

// es6-app.mjs
import Circle, { PI, calculateArea } from './es6-module.mjs';

const circle = new Circle(5);
console.log(circle.getArea()); // 78.53975
```

### 7. Path Module for Module Resolution
```javascript
// path-examples.js
const path = require('path');

// Get absolute path to module
const modulePath = path.join(__dirname, 'modules', 'utils.js');
const utils = require(modulePath);

// Resolve module path
const resolvedPath = require.resolve('./math');
console.log('Resolved path:', resolvedPath);

// Directory name and filename
console.log('Current directory:', __dirname);
console.log('Current file:', __filename);
```

### 8. Creating a Utility Module
```javascript
// utils/stringUtils.js
function capitalize(str) {
    if (!str) return '';
    return str.charAt(0).toUpperCase() + str.slice(1);
}

function truncate(str, length) {
    if (str.length <= length) return str;
    return str.slice(0, length) + '...';
}

function slugify(str) {
    return str.toLowerCase()
        .replace(/[^\w\s-]/g, '')
        .replace(/[\s_-]+/g, '-')
        .replace(/^-+|-+$/g, '');
}

module.exports = {
    capitalize,
    truncate,
    slugify
};

// utils/index.js - Barrel export
const stringUtils = require('./stringUtils');
const numberUtils = require('./numberUtils');

module.exports = {
    stringUtils,
    numberUtils
};
```

## Real-time Use Cases

### 1. Project Structure Organization
```
project/
├── controllers/
│   ├── userController.js
│   └── productController.js
├── models/
│   ├── User.js
│   └── Product.js
├── services/
│   ├── emailService.js
│   └── paymentService.js
├── utils/
│   ├── validation.js
│   └── helpers.js
├── config/
│   └── database.js
└── app.js
```

### 2. Plugin Architecture
```javascript
// pluginManager.js
class PluginManager {
    constructor() {
        this.plugins = new Map();
    }

    register(name, plugin) {
        this.plugins.set(name, plugin);
    }

    get(name) {
        return this.plugins.get(name);
    }

    async loadPlugin(name, path) {
        const plugin = require(path);
        this.register(name, plugin);
        if (plugin.init) {
            await plugin.init();
        }
    }
}

module.exports = PluginManager;
```

### 3. Configuration Management
```javascript
// config/index.js
const path = require('path');
const env = process.env.NODE_ENV || 'development';

const config = {
    development: require('./config.development'),
    production: require('./config.production'),
    test: require('./config.test')
}[env];

module.exports = config;
```

## Best Practices

### 1. Module Design Principles
- Keep modules small and focused (Single Responsibility)
- Export only what's necessary
- Use meaningful names for exports
- Avoid circular dependencies

### 2. Error Handling in Modules
```javascript
// safe-module.js
function riskyOperation() {
    try {
        // Risky code
        return result;
    } catch (error) {
        console.error('Error in riskyOperation:', error);
        throw error; // Re-throw or handle appropriately
    }
}

module.exports = { riskyOperation };
```

### 3. Lazy Loading
```javascript
// lazy-loader.js
function getHeavyModule() {
    // Load only when needed
    return require('./heavyModule');
}

// Usage
app.get('/heavy', (req, res) => {
    const heavyModule = getHeavyModule();
    const result = heavyModule.process();
    res.json(result);
});
```

## Practice Exercises

1. Create a utility module with helper functions for string manipulation
2. Build a configuration module that loads different configs based on environment
3. Implement a plugin system that can dynamically load modules
4. Create a module that demonstrates proper error handling
5. Build a barrel export pattern for organizing related modules

## Common Pitfalls

1. **Circular Dependencies**: Can cause undefined exports
2. **Module Caching Issues**: Unexpected behavior when modules are modified at runtime
3. **Relative Path Confusion**: Using wrong paths for module imports
4. **Over-exporting**: Exposing internal implementation details

## Next Steps
- Explore File System operations (Topic 04)
- Understand Buffers and Streams (Topic 05)
- Learn Error Handling patterns (Topic 06)
