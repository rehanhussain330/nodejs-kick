# Part 9: Deployment Roadmap

## Table of Contents
1. [Preparation for Production](#1-preparation-for-production)
2. [Environment Variables & Configuration](#2-environment-variables--configuration)
3. [Process Managers (PM2)](#3-process-managers-pm2)
4. [Docker & Containerization](#4-docker--containerization)
5. [CI/CD Pipelines](#5-cicd-pipelines)
6. [Cloud Platforms](#6-cloud-platforms)
7. [Load Balancing](#7-load-balancing)
8. [Monitoring & Logging](#8-monitoring--logging)
9. [SSL/HTTPS Setup](#9-sslhttps-setup)
10. [Database Deployment Strategies](#10-database-deployment-strategies)
11. [Scaling Strategies](#11-scaling-strategies)
12. [Backup & Recovery](#12-backup--recovery)
13. [DevOps Best Practices](#13-devops-best-practices)

---

## 1. Preparation for Production

### Pre-Deployment Checklist

#### Code Quality
- [ ] Run linters (ESLint)
- [ ] Format code (Prettier)
- [ ] Remove console.logs and debug statements
- [ ] Remove unused dependencies
- [ ] Update package.json with correct versions

#### Security
- [ ] Audit dependencies (`npm audit`)
- [ ] Update vulnerable packages
- [ ] Remove sensitive data from code
- [ ] Set up environment variables
- [ ] Configure CORS properly
- [ ] Add security headers (Helmet)

#### Performance
- [ ] Enable compression
- [ ] Implement caching strategies
- [ ] Optimize database queries
- [ ] Add database indexes
- [ ] Minify assets (if serving frontend)

#### Testing
- [ ] Run unit tests
- [ ] Run integration tests
- [ ] Run end-to-end tests
- [ ] Load testing
- [ ] Security testing

### Example: Production Build Script

**File: `examples/package.json`**
```json
{
  "name": "production-app",
  "version": "1.0.0",
  "scripts": {
    "lint": "eslint . --ext .js",
    "format": "prettier --write \"src/**/*.js\"",
    "test": "jest --coverage",
    "test:watch": "jest --watch",
    "build": "npm run lint && npm run test",
    "start": "node src/index.js",
    "dev": "nodemon src/index.js",
    "prod": "NODE_ENV=production node src/index.js",
    "pm2:start": "pm2 start ecosystem.config.js",
    "pm2:stop": "pm2 stop ecosystem.config.js",
    "pm2:restart": "pm2 restart ecosystem.config.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0",
    "dotenv": "^16.0.3",
    "helmet": "^6.0.1",
    "compression": "^1.7.4",
    "winston": "^3.8.2"
  },
  "devDependencies": {
    "eslint": "^8.40.0",
    "prettier": "^2.8.8",
    "jest": "^29.5.0",
    "nodemon": "^2.0.22"
  }
}
```

---

## 2. Environment Variables & Configuration

### Using dotenv

**File: `examples/.env.example`**
```bash
# Application
NODE_ENV=production
PORT=3000
API_VERSION=v1

# Database - MongoDB
MONGODB_URI=mongodb://localhost:27017/myapp
MONGODB_POOL_SIZE=10

# Database - MySQL
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_USER=root
MYSQL_PASSWORD=your_password
MYSQL_DATABASE=myapp

# Authentication
JWT_SECRET=your-super-secret-jwt-key-change-in-production
JWT_EXPIRE=7d
JWT_COOKIE_EXPIRE=7

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# Third-party APIs
STRIPE_SECRET_KEY=sk_test_xxx
PAYPAL_CLIENT_ID=xxx
GOOGLE_MAPS_API_KEY=xxx

# File Upload
MAX_FILE_SIZE=10485760
UPLOAD_PATH=./uploads

# Logging
LOG_LEVEL=info
LOG_FILE_PATH=./logs/app.log

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# CORS
ALLOWED_ORIGINS=https://yourdomain.com,https://www.yourdomain.com
```

### Configuration Manager

**File: `examples/config/index.js`**
```javascript
require('dotenv').config();

const config = {
  env: process.env.NODE_ENV || 'development',
  
  app: {
    port: parseInt(process.env.PORT, 10) || 3000,
    apiVersion: process.env.API_VERSION || 'v1',
    name: process.env.APP_NAME || 'MyApp'
  },
  
  mongoDB: {
    uri: process.env.MONGODB_URI || 'mongodb://localhost:27017/myapp',
    options: {
      poolSize: parseInt(process.env.MONGODB_POOL_SIZE, 10) || 10,
      useNewUrlParser: true,
      useUnifiedTopology: true
    }
  },
  
  mysql: {
    host: process.env.MYSQL_HOST || 'localhost',
    port: parseInt(process.env.MYSQL_PORT, 10) || 3306,
    user: process.env.MYSQL_USER || 'root',
    password: process.env.MYSQL_PASSWORD || '',
    database: process.env.MYSQL_DATABASE || 'myapp',
    connectionLimit: parseInt(process.env.MYSQL_CONNECTION_LIMIT, 10) || 10
  },
  
  jwt: {
    secret: process.env.JWT_SECRET,
    expire: process.env.JWT_EXPIRE || '7d',
    cookieExpire: parseInt(process.env.JWT_COOKIE_EXPIRE, 10) || 7
  },
  
  email: {
    host: process.env.SMTP_HOST,
    port: parseInt(process.env.SMTP_PORT, 10) || 587,
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS
  },
  
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT, 10) || 6379,
    password: process.env.REDIS_PASSWORD || undefined
  },
  
  stripe: {
    secretKey: process.env.STRIPE_SECRET_KEY
  },
  
  upload: {
    maxSize: parseInt(process.env.MAX_FILE_SIZE, 10) || 10 * 1024 * 1024, // 10MB
    path: process.env.UPLOAD_PATH || './uploads'
  },
  
  rateLimit: {
    windowMs: parseInt(process.env.RATE_LIMIT_WINDOW_MS, 10) || 15 * 60 * 1000, // 15 minutes
    maxRequests: parseInt(process.env.RATE_LIMIT_MAX_REQUESTS, 10) || 100
  },
  
  cors: {
    origins: process.env.ALLOWED_ORIGINS ? process.env.ALLOWED_ORIGINS.split(',') : ['http://localhost:3000']
  },
  
  logging: {
    level: process.env.LOG_LEVEL || 'info',
    filePath: process.env.LOG_FILE_PATH || './logs/app.log'
  }
};

// Validate required environment variables
const requiredEnvVars = [
  'JWT_SECRET'
];

if (config.env === 'production') {
  requiredEnvVars.push('MONGODB_URI', 'STRIPE_SECRET_KEY');
}

const missingVars = requiredEnvVars.filter(varName => !process.env[varName]);

if (missingVars.length > 0) {
  console.error(`❌ Missing required environment variables: ${missingVars.join(', ')}`);
  process.exit(1);
}

module.exports = config;
```

### Environment-Specific Configurations

**File: `examples/config/environments.js`**
```javascript
const baseConfig = require('./index');

const environments = {
  development: {
    ...baseConfig,
    logging: {
      ...baseConfig.logging,
      level: 'debug'
    },
    rateLimit: {
      ...baseConfig.rateLimit,
      maxRequests: 1000 // Higher limit for development
    }
  },
  
  test: {
    ...baseConfig,
    mongoDB: {
      ...baseConfig.mongoDB,
      uri: 'mongodb://localhost:27017/myapp_test'
    },
    mysql: {
      ...baseConfig.mysql,
      database: 'myapp_test'
    }
  },
  
  production: {
    ...baseConfig,
    logging: {
      ...baseConfig.logging,
      level: 'warn' // Only log warnings and errors in production
    }
  },
  
  staging: {
    ...baseConfig,
    logging: {
      ...baseConfig.logging,
      level: 'info'
    }
  }
};

const currentEnv = process.env.NODE_ENV || 'development';
module.exports = environments[currentEnv] || environments.development;
```

---

## 3. Process Managers (PM2)

### PM2 Basics

```bash
# Install PM2 globally
npm install -g pm2

# Start application
pm2 start app.js

# Start with custom name
pm2 start app.js --name my-api

# Start with specific Node version
pm2 start app.js --interpreter=node@18

# Start in cluster mode (uses all CPU cores)
pm2 start app.js -i max

# Start with ecosystem file
pm2 start ecosystem.config.js

# View running processes
pm2 list

# View logs
pm2 logs

# Monitor in real-time
pm2 monit

# Restart application
pm2 restart my-api

# Stop application
pm2 stop my-api

# Delete from PM2
pm2 delete my-api

# Save process list
pm2 save

# Resurrect saved processes
pm2 resurrect

# Startup script (runs on system boot)
pm2 startup
```

### Ecosystem Configuration File

**File: `examples/ecosystem.config.js`**
```javascript
module.exports = {
  apps: [
    {
      name: 'api-server',
      script: './src/index.js',
      instances: 'max', // Use all CPU cores
      exec_mode: 'cluster',
      
      // Environment variables
      env: {
        NODE_ENV: 'development',
        PORT: 3000
      },
      env_production: {
        NODE_ENV: 'production',
        PORT: 80
      },
      env_staging: {
        NODE_ENV: 'staging',
        PORT: 8080
      },
      
      // Error handling
      error_file: './logs/pm2-error.log',
      out_file: './logs/pm2-out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      merge_logs: true,
      
      // Auto restart settings
      autorestart: true,
      watch: false, // Disable watch in production
      max_memory_restart: '500M', // Restart if memory exceeds 500MB
      
      // Graceful shutdown
      kill_timeout: 3000,
      wait_ready: true,
      listen_timeout: 10000,
      
      // Advanced settings
      min_uptime: '10s',
      max_restarts: 10,
      restart_delay: 4000
    },
    
    {
      name: 'worker',
      script: './src/workers/background.js',
      instances: 2,
      exec_mode: 'fork',
      env: {
        NODE_ENV: 'production'
      }
    },
    
    {
      name: 'scheduler',
      script: './src/workers/scheduler.js',
      instances: 1,
      exec_mode: 'fork',
      cron_restart: '0 */6 * * *' // Restart every 6 hours
    }
  ],
  
  // Deployment configuration
  deploy: {
    production: {
      user: 'deploy',
      host: 'your-server-ip',
      ref: 'origin/main',
      repo: 'git@github.com:username/repo.git',
      path: '/var/www/myapp',
      'post-deploy': 'npm install && pm2 restart ecosystem.config.js --env production'
    },
    staging: {
      user: 'deploy',
      host: 'staging-server-ip',
      ref: 'origin/develop',
      repo: 'git@github.com:username/repo.git',
      path: '/var/www/myapp-staging',
      'post-deploy': 'npm install && pm2 restart ecosystem.config.js --env staging'
    }
  }
};
```

### PM2 with Docker

**File: `examples/Dockerfile.pm2`**
```dockerfile
FROM node:18-alpine

# Install PM2 globally
RUN npm install -g pm2

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Create logs directory
RUN mkdir -p logs

# Expose port
EXPOSE 3000

# Start with PM2
CMD ["pm2-runtime", "ecosystem.config.js"]
```

---

## 4. Docker & Containerization

### Basic Dockerfile

**File: `examples/Dockerfile`**
```dockerfile
# Use official Node.js runtime as base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apk add --no-cache libc6-compat

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Change ownership of app directory
RUN chown -R nodejs:nodejs /app
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# Start application
CMD ["node", "src/index.js"]
```

### Multi-Stage Build (Optimized)

**File: `examples/Dockerfile.multi-stage`**
```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

# Run tests and build
RUN npm run build

# Production stage
FROM node:18-alpine AS production

# Install security updates
RUN apk add --no-cache libc6-compat && \
    apk upgrade

WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy only necessary files from builder
COPY --from=builder --chown=nodejs:nodejs /app/package*.json ./
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/src ./src

USER nodejs

EXPOSE 3000

ENV NODE_ENV=production

CMD ["node", "dist/index.js"]
```

### Docker Compose

**File: `examples/docker-compose.yml`**
```yaml
version: '3.8'

services:
  # API Server
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongo:27017/myapp
      - MYSQL_HOST=mysql
      - REDIS_HOST=redis
    depends_on:
      - mongo
      - mysql
      - redis
    volumes:
      - ./logs:/app/logs
      - ./uploads:/app/uploads
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "node", "healthcheck.js"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Frontend (React)
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "80:80"
    depends_on:
      - api
    networks:
      - app-network
    restart: unless-stopped

  # MongoDB
  mongo:
    image: mongo:6
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
      - ./backups/mongo:/backups
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_PASSWORD}
    networks:
      - app-network
    restart: unless-stopped

  # MySQL
  mysql:
    image: mysql:8
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./init-scripts:/docker-entrypoint-initdb.d
      - ./backups/mysql:/backups
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=myapp
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes
    networks:
      - app-network
    restart: unless-stopped

  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - api
      - frontend
    networks:
      - app-network
    restart: unless-stopped

  # Monitoring (Prometheus)
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    networks:
      - app-network
    restart: unless-stopped

  # Grafana
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
    networks:
      - app-network
    restart: unless-stopped

networks:
  app-network:
    driver: bridge

volumes:
  mongo-data:
  mysql-data:
  redis-data:
  prometheus-data:
  grafana-data:
```

### Docker Commands

```bash
# Build image
docker build -t my-node-app .

# Run container
docker run -d -p 3000:3000 --name my-app my-node-app

# Run with environment variables
docker run -d -p 3000:3000 \
  -e NODE_ENV=production \
  -e MONGODB_URI=mongodb://mongo:27017/myapp \
  --name my-app my-node-app

# View logs
docker logs my-app
docker logs -f my-app  # Follow logs

# Execute command in running container
docker exec -it my-app sh
docker exec -it my-app npm run test

# Stop and remove container
docker stop my-app
docker rm my-app

# Docker Compose
docker-compose up -d              # Start all services
docker-compose down               # Stop all services
docker-compose logs -f            # View logs
docker-compose ps                 # List containers
docker-compose restart api        # Restart specific service
docker-compose build --no-cache   # Rebuild images
```

---

## 5. CI/CD Pipelines

### GitHub Actions

**File: `examples/.github/workflows/deploy.yml`**
```yaml
name: Deploy to Production

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      mongodb:
        image: mongo:6
        ports:
          - 27017:27017
      mysql:
        image: mysql:8
        env:
          MYSQL_ROOT_PASSWORD: testpassword
          MYSQL_DATABASE: testdb
        ports:
          - 3306:3306
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]

    steps:
      - uses: actions/checkout@v3

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test
        env:
          MONGODB_URI: mongodb://localhost:27017/testdb
          MYSQL_HOST: localhost
          MYSQL_PASSWORD: testpassword
          REDIS_HOST: localhost

      - name: Build
        run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3

      - name: Deploy to Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/myapp
            git pull origin main
            npm ci --production
            npm run build
            pm2 restart ecosystem.config.js --env production

      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        if: always()

  docker-build:
    needs: deploy
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.DOCKER_USERNAME }}/myapp:latest
            ${{ secrets.DOCKER_USERNAME }}/myapp:${{ github.sha }}
          cache-from: type=registry,ref=${{ secrets.DOCKER_USERNAME }}/myapp:buildcache
          cache-to: type=registry,ref=${{ secrets.DOCKER_USERNAME }}/myapp:buildcache,mode=max
```

### Jenkins Pipeline

**File: `examples/Jenkinsfile`**
```groovy
pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS 18'
    }
    
    environment {
        DOCKER_REGISTRY = credentials('docker-registry')
        SERVER_HOST = credentials('server-host')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Code Quality') {
            parallel {
                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
                stage('Format Check') {
                    steps {
                        sh 'npx prettier --check "src/**/*.js"'
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    docker-compose -f docker-compose.test.yml up -d
                    sleep 10
                    npm test
                    docker-compose -f docker-compose.test.yml down
                '''
            }
            post {
                always {
                    junit 'reports/*.xml'
                    publishCoverage adapters: [coberturaAdapter('coverage/cobertura-coverage.xml')]
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/myapp:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_REGISTRY) {
                        docker.image("${DOCKER_REGISTRY}/myapp:${BUILD_NUMBER}").push()
                        docker.image("${DOCKER_REGISTRY}/myapp:${BUILD_NUMBER}").push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    ssh ${SERVER_HOST} << EOF
                    cd /var/www/myapp
                    git pull origin main
                    npm ci --production
                    npm run build
                    pm2 restart ecosystem.config.js --env production
                    EOF
                '''
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Deployment successful! 🎉'
        }
        failure {
            echo 'Deployment failed! ❌'
            // Send notification
            mail to: 'team@example.com',
                 subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Something is wrong with ${BUILD_URL}"
        }
    }
}
```

---

## 6. Cloud Platforms

### AWS Deployment

#### EC2 Deployment Script

**File: `examples/deploy-aws.sh`**
```bash
#!/bin/bash

# AWS EC2 Deployment Script

INSTANCE_ID="i-0123456789abcdef0"
KEY_PAIR="my-key-pair"
SECURITY_GROUP="my-sg"
REGION="us-east-1"

# Create security group rules
aws ec2 authorize-security-group-ingress \
    --group-id $SECURITY_GROUP \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-id $SECURITY_GROUP \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-id $SECURITY_GROUP \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-id $SECURITY_GROUP \
    --protocol tcp \
    --port 3000 \
    --cidr 0.0.0.0/0

# Deploy application
scp -i ~/.ssh/${KEY_PAIR}.pem \
    -r . ec2-user@YOUR_EC2_IP:/var/www/myapp

ssh -i ~/.ssh/${KEY_PAIR}.pem ec2-user@YOUR_EC2_IP << 'EOF'
    cd /var/www/myapp
    npm ci --production
    npm run build
    pm2 restart ecosystem.config.js --env production
EOF
```

### AWS Elastic Beanstalk

**File: `examples/.elasticbeanstalk/config.yml`**
```yaml
branch-defaults:
  main:
    environment: myapp-production
    group_suffix: null
global:
  application_name: myapp
  branch: null
  default_ec2_keyname: my-key-pair
  default_platform: Node.js 18 running on 64bit Amazon Linux 2
  default_region: us-east-1
  include_git_submodules: true
  instance_profile: null
  platform_name: null
  platform_version: null
  profile: eb-cli
  repository: null
  sc: git
  workspace_type: Application
```

**File: `examples/.ebextensions/nodecommand.config`**
```yaml
option_settings:
  aws:elasticbeanstalk:application:environment:
    NODE_ENV: production
    NPM_USE_PRODUCTION: true
  aws:elasticbeanstalk:container:nodejs:
    NodeCommand: "npm run start:prod"
```

### Heroku Deployment

**File: `examples/Procfile`**
```
web: node src/index.js
worker: node src/workers/background.js
```

```bash
# Heroku CLI commands
heroku login
heroku create myapp-name
heroku config:set NODE_ENV=production
heroku config:set MONGODB_URI=mongodb+srv://...
heroku config:set JWT_SECRET=your-secret
git push heroku main
heroku open
heroku logs --tail
```

### Google Cloud Platform (GCP)

**File: `examples/cloudbuild.yaml`**
```yaml
steps:
  - name: 'gcr.io/cloud-builders/npm'
    args: ['ci']
  
  - name: 'gcr.io/cloud-builders/npm'
    args: ['run', 'build']
  
  - name: 'gcr.io/cloud-builders/npm'
    args: ['test']
  
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/myapp:$SHORT_SHA', '.']
  
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/myapp:$SHORT_SHA']
  
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args:
      - 'run'
      - 'deploy'
      - 'myapp'
      - '--image'
      - 'gcr.io/$PROJECT_ID/myapp:$SHORT_SHA'
      - '--platform'
      - 'managed'
      - '--region'
      - 'us-central1'
      - '--allow-unauthenticated'

images:
  - 'gcr.io/$PROJECT_ID/myapp:$SHORT_SHA'
```

---

## 7. Load Balancing

### Nginx Configuration

**File: `examples/nginx/nginx.conf`**
```nginx
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 10M;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml application/json application/javascript application/xml;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
    limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

    # Upstream servers
    upstream api_servers {
        least_conn;
        server api1:3000 weight=3;
        server api2:3000 weight=3;
        server api3:3000 weight=2;
        server api4:3000 backup;
        keepalive 32;
    }

    upstream frontend_servers {
        server frontend1:80;
        server frontend2:80;
        keepalive 32;
    }

    # HTTP Server - Redirect to HTTPS
    server {
        listen 80;
        server_name example.com www.example.com;
        return 301 https://$server_name$request_uri;
    }

    # HTTPS Server
    server {
        listen 443 ssl http2;
        server_name example.com www.example.com;

        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 10m;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

        # API routes
        location /api/ {
            limit_req zone=api_limit burst=20 nodelay;
            limit_conn conn_limit 10;

            proxy_pass http://api_servers;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
            proxy_read_timeout 90;
        }

        # WebSocket support
        location /ws/ {
            proxy_pass http://api_servers;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        # Static files
        location /static/ {
            alias /var/www/static/;
            expires 30d;
            add_header Cache-Control "public, immutable";
        }

        # Frontend
        location / {
            proxy_pass http://frontend_servers;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        # Health check endpoint
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
}
```

### HAProxy Configuration

**File: `examples/haproxy.cfg`**
```haproxy
global
    log stdout format raw local0
    maxconn 4096
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms
    retries 3

frontend http_front
    bind *:80
    default_backend api_servers

backend api_servers
    balance roundrobin
    option httpchk GET /health
    server api1 api1:3000 check
    server api2 api2:3000 check
    server api3 api3:3000 check
```

---

## 8. Monitoring & Logging

### Winston Logger Setup

**File: `examples/src/utils/logger.js`**
```javascript
const winston = require('winston');
const path = require('path');

const logDir = path.join(__dirname, '../../logs');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
    winston.format.errors({ stack: true }),
    winston.format.splat(),
    winston.format.json()
  ),
  defaultMeta: { service: 'myapp' },
  transports: [
    // Console output
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    
    // Error logs
    new winston.transports.File({
      filename: path.join(logDir, 'error.log'),
      level: 'error',
      maxsize: 5242880, // 5MB
      maxFiles: 5
    }),
    
    // Combined logs
    new winston.transports.File({
      filename: path.join(logDir, 'combined.log'),
      maxsize: 5242880,
      maxFiles: 5
    })
  ]
});

// Stream for Morgan HTTP logger
logger.stream = {
  write: (message) => {
    logger.info(message.trim());
  }
};

module.exports = logger;
```

### Prometheus Metrics

**File: `examples/src/metrics/prometheus.js`**
```javascript
const client = require('prom-client');

const register = new client.Registry();
client.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

const activeConnections = new client.Gauge({
  name: 'active_connections',
  help: 'Number of active connections'
});

const databaseQueryDuration = new client.Histogram({
  name: 'database_query_duration_seconds',
  help: 'Duration of database queries',
  labelNames: ['query_type', 'table'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 2, 5]
});

register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);
register.registerMetric(activeConnections);
register.registerMetric(databaseQueryDuration);

// Middleware to collect metrics
function metricsMiddleware(req, res, next) {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    const route = req.route?.path || req.path;
    
    httpRequestDuration
      .labels(req.method, route, res.statusCode)
      .observe(duration);
    
    httpRequestTotal
      .labels(req.method, route, res.statusCode)
      .inc();
  });
  
  next();
}

// Metrics endpoint
function metricsHandler(req, res) {
  res.set('Content-Type', register.contentType);
  res.end(register.metrics());
}

module.exports = {
  register,
  metricsMiddleware,
  metricsHandler,
  httpRequestDuration,
  httpRequestTotal,
  activeConnections,
  databaseQueryDuration
};
```

### Grafana Dashboard JSON

**File: `examples/monitoring/grafana-dashboard.json`**
```json
{
  "dashboard": {
    "title": "Node.js Application Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{route}}"
          }
        ]
      },
      {
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "singlestat",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status_code=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100"
          }
        ]
      },
      {
        "title": "Active Connections",
        "type": "gauge",
        "targets": [
          {
            "expr": "active_connections"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "nodejs_heap_size_used_bytes",
            "legendFormat": "Heap Used"
          },
          {
            "expr": "nodejs_heap_size_total_bytes",
            "legendFormat": "Heap Total"
          }
        ]
      }
    ]
  }
}
```

---

## 9. SSL/HTTPS Setup

### Let's Encrypt with Certbot

```bash
# Install Certbot
sudo apt-get update
sudo apt-get install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d example.com -d www.example.com

# Auto-renewal (already set up by certbot)
sudo certbot renew --dry-run
```

### Manual SSL Certificate Setup

**File: `examples/ssl/generate-self-signed.sh`**
```bash
#!/bin/bash

# Generate private key
openssl genrsa -out privkey.pem 2048

# Generate CSR
openssl req -new -key privkey.pem -out csr.pem \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=example.com"

# Generate self-signed certificate (valid for 365 days)
openssl x509 -req -days 365 -in csr.pem -signkey privkey.pem -out fullchain.pem

# Combine for Nginx
cat privkey.pem fullchain.pem > combined.pem
```

### HTTPS Express Server

**File: `examples/src/https-server.js`**
```javascript
const https = require('https');
const fs = require('fs');
const express = require('express');
const path = require('path');

const app = express();

// SSL options
const options = {
  key: fs.readFileSync(path.join(__dirname, '../ssl/privkey.pem')),
  cert: fs.readFileSync(path.join(__dirname, '../ssl/fullchain.pem'))
};

// Force HTTPS middleware
app.use((req, res, next) => {
  if (!req.secure && process.env.NODE_ENV === 'production') {
    return res.redirect(`https://${req.headers.host}${req.url}`);
  }
  next();
});

app.get('/', (req, res) => {
  res.json({ message: 'Secure connection established!' });
});

// Create HTTPS server
const httpsServer = https.createServer(options, app);

httpsServer.listen(443, () => {
  console.log('HTTPS Server running on port 443');
});
```

---

## 10. Database Deployment Strategies

### MongoDB Atlas Setup

```javascript
// Connection string format
const uri = "mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/myapp?retryWrites=true&w=majority";

// Mongoose connection with retry logic
const mongoose = require('mongoose');

async function connectWithRetry() {
  const maxRetries = 5;
  let retries = 0;
  
  while (retries < maxRetries) {
    try {
      await mongoose.connect(uri, {
        useNewUrlParser: true,
        useUnifiedTopology: true,
        serverSelectionTimeoutMS: 5000,
        socketTimeoutMS: 45000,
      });
      console.log('MongoDB connected successfully');
      return;
    } catch (err) {
      retries++;
      console.error(`Connection attempt ${retries} failed:`, err.message);
      if (retries < maxRetries) {
        await new Promise(resolve => setTimeout(resolve, 5000 * retries));
      }
    }
  }
  
  console.error('Failed to connect to MongoDB after multiple attempts');
  process.exit(1);
}

connectWithRetry();
```

### MySQL Production Setup

```javascript
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: process.env.MYSQL_HOST,
  user: process.env.MYSQL_USER,
  password: process.env.MYSQL_PASSWORD,
  database: process.env.MYSQL_DATABASE,
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0,
  enableKeepAlive: true,
  keepAliveInitialDelay: 0
});

// Test connection
async function testConnection() {
  try {
    const connection = await pool.getConnection();
    console.log('MySQL connection successful');
    connection.release();
  } catch (err) {
    console.error('MySQL connection failed:', err);
    process.exit(1);
  }
}

testConnection();
```

### Database Migration Strategy

**File: `examples/migrations/001-create-users.js`**
```javascript
exports.up = async (knex) => {
  await knex.schema.createTable('users', (table) => {
    table.increments('id').primary();
    table.string('email').unique().notNullable();
    table.string('password').notNullable();
    table.string('name');
    table.timestamps(true, true);
  });
};

exports.down = async (knex) => {
  await knex.schema.dropTable('users');
};
```

```bash
# Run migrations
npx knex migrate:latest

# Rollback last migration
npx knex migrate:rollback

# Create new migration
npx knex migrate:make create_users_table
```

---

## 11. Scaling Strategies

### Horizontal Scaling

```javascript
// Cluster mode with PM2
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'api',
    script: 'src/index.js',
    instances: 'max', // Use all CPU cores
    exec_mode: 'cluster'
  }]
};
```

### Vertical Scaling

```javascript
// Increase Node.js memory limit
// package.json
{
  "scripts": {
    "start": "node --max-old-space-size=4096 src/index.js"
  }
}
```

### Auto-scaling with AWS

**File: `examples/aws-autoscaling-config.json`**
```json
{
  "AutoScalingGroupName": "myapp-asg",
  "MinSize": 2,
  "MaxSize": 10,
  "DesiredCapacity": 3,
  "TargetTrackingConfiguration": {
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "ScaleInCooldown": 300,
    "ScaleOutCooldown": 60
  }
}
```

---

## 12. Backup & Recovery

### Automated Backup Script

**File: `examples/backup/backup.sh`**
```bash
#!/bin/bash

BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# MongoDB backup
mongodump --uri="$MONGODB_URI" --out="$BACKUP_DIR/mongo_$DATE"

# MySQL backup
mysqldump -h $MYSQL_HOST -u $MYSQL_USER -p$MYSQL_PASSWORD \
  $MYSQL_DATABASE > "$BACKUP_DIR/mysql_$DATE.sql"

# Compress backups
tar -czf "$BACKUP_DIR/mongo_$DATE.tar.gz" -C "$BACKUP_DIR" "mongo_$DATE"
rm -rf "$BACKUP_DIR/mongo_$DATE"

gzip "$BACKUP_DIR/mysql_$DATE.sql"

# Upload to S3
aws s3 cp "$BACKUP_DIR/mongo_$DATE.tar.gz" s3://myapp-backups/mongo/
aws s3 cp "$BACKUP_DIR/mysql_$DATE.sql.gz" s3://myapp-backups/mysql/

# Clean old backups
find "$BACKUP_DIR" -type f -mtime +$RETENTION_DAYS -delete

echo "Backup completed: $DATE"
```

### Disaster Recovery Plan

1. **Data Backup**: Daily automated backups to cloud storage
2. **Infrastructure as Code**: Terraform/CloudFormation templates
3. **Documentation**: Runbooks for common issues
4. **Testing**: Regular disaster recovery drills
5. **Monitoring**: Alert systems for early detection

---

## 13. DevOps Best Practices

### Infrastructure as Code (Terraform)

**File: `examples/terraform/main.tf`**
```terraform
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "app_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "myapp-server"
  }
  
  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y nodejs npm
              cd /var/www/myapp
              npm install
              npm run build
              pm2 start src/index.js
              EOF
}

resource "aws_security_group" "app_sg" {
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Production Checklist

✅ **Security**
- [ ] All dependencies updated
- [ ] No hardcoded secrets
- [ ] SSL/TLS enabled
- [ ] Security headers configured
- [ ] Rate limiting implemented
- [ ] Input validation on all endpoints

✅ **Performance**
- [ ] Caching implemented
- [ ] Database queries optimized
- [ ] Compression enabled
- [ ] CDN for static assets
- [ ] Load balancing configured

✅ **Reliability**
- [ ] Health checks implemented
- [ ] Auto-restart configured
- [ ] Backups automated
- [ ] Monitoring in place
- [ ] Alert notifications set up

✅ **Scalability**
- [ ] Horizontal scaling ready
- [ ] Database replication configured
- [ ] Session management stateless
- [ ] Auto-scaling policies defined

✅ **Documentation**
- [ ] API documentation complete
- [ ] Deployment procedures documented
- [ ] Runbooks created
- [ ] Team trained on procedures

---

## Conclusion

Deployment is not just about pushing code to a server. It involves:

1. **Preparation**: Code quality, security, testing
2. **Infrastructure**: Containers, orchestration, networking
3. **Automation**: CI/CD pipelines, infrastructure as code
4. **Monitoring**: Logging, metrics, alerting
5. **Maintenance**: Backups, updates, scaling

Remember: **Automate everything, monitor everything, document everything.**

---

## Additional Resources

- [PM2 Documentation](https://pm2.keymetrics.io/docs/)
- [Docker Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [The Twelve-Factor App](https://12factor.net/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
