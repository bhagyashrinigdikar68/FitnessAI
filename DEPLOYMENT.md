# FITNOVA Deployment Guide

This document provides comprehensive instructions for deploying the FITNOVA fitness app across various platforms and environments.

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Development Setup](#development-setup)
- [Production Build](#production-build)
- [Deployment Options](#deployment-options)
- [Environment Configuration](#environment-configuration)
- [Performance Optimization](#performance-optimization)
- [Monitoring & Analytics](#monitoring--analytics)
- [Troubleshooting](#troubleshooting)

## Prerequisites

Before deploying FITNOVA, ensure you have:

- Node.js (v16 or higher)
- npm or yarn package manager
- Git for version control
- A code editor (VS Code recommended)
- Basic knowledge of HTML, CSS, and JavaScript

## Development Setup

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/fitnova-app.git
cd fitnova-app
```

### 2. Install Dependencies

```bash
npm install
# or
yarn install
```

### 3. Start Development Server

```bash
npm run dev
# or
yarn dev
```

The app will be available at `http://localhost:3000`

## Production Build

### 1. Build for Production

```bash
npm run build
# or
yarn build
```

### 2. Preview Production Build

```bash
npm run preview
# or
yarn preview
```

## Deployment Options

### Option 1: Vercel (Recommended)

Vercel provides seamless deployment with automatic CI/CD.

#### Quick Deploy

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/your-username/fitnova-app)

#### Manual Deployment

1. Install Vercel CLI:
```bash
npm install -g vercel
```

2. Login to Vercel:
```bash
vercel login
```

3. Deploy:
```bash
vercel --prod
```

#### Vercel Configuration (`vercel.json`)

```json
{
  "version": 2,
  "name": "fitnova-app",
  "builds": [
    {
      "src": "*.html",
      "use": "@vercel/static"
    },
    {
      "src": "assets/**",
      "use": "@vercel/static"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/index.html"
    }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    }
  ]
}
```

### Option 2: Netlify

1. Build the project:
```bash
npm run build
```

2. Drag and drop the `dist` folder to [Netlify Drop](https://app.netlify.com/drop)

Or use Netlify CLI:

```bash
npm install -g netlify-cli
netlify deploy --prod --dir=dist
```

#### Netlify Configuration (`netlify.toml`)

```toml
[build]
  publish = "dist"
  command = "npm run build"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[build.environment]
  NODE_VERSION = "18"

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"
```

### Option 3: Firebase Hosting

1. Install Firebase CLI:
```bash
npm install -g firebase-tools
```

2. Login to Firebase:
```bash
firebase login
```

3. Initialize Firebase:
```bash
firebase init hosting
```

4. Deploy:
```bash
firebase deploy
```

#### Firebase Configuration (`firebase.json`)

```json
{
  "hosting": {
    "public": "dist",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "**/*.@(js|css)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=31536000"
          }
        ]
      }
    ]
  }
}
```

### Option 4: GitHub Pages

1. Build the project:
```bash
npm run build
```

2. Install gh-pages:
```bash
npm install --save-dev gh-pages
```

3. Add deploy script to `package.json`:
```json
{
  "scripts": {
    "deploy": "gh-pages -d dist"
  }
}
```

4. Deploy:
```bash
npm run deploy
```

### Option 5: Docker Deployment

#### Dockerfile

```dockerfile
# Build stage
FROM node:18-alpine as build-stage

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine as production-stage

COPY --from=build-stage /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

#### Nginx Configuration (`nginx.conf`)

```nginx
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    server {
        listen 80;
        server_name localhost;
        root /usr/share/nginx/html;
        index index.html;

        # Gzip compression
        gzip on;
        gzip_types text/css application/javascript text/javascript application/json;

        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }

        # Handle client-side routing
        location / {
            try_files $uri $uri/ /index.html;
        }

        # Security headers
        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
    }
}
```

#### Build and Run Docker Container

```bash
# Build image
docker build -t fitnova-app .

# Run container
docker run -p 8080:80 fitnova-app
```

## Environment Configuration

### Environment Variables

Create `.env` files for different environments:

#### `.env.development`
```env
NODE_ENV=development
VITE_API_BASE_URL=http://localhost:3001/api
VITE_ANALYTICS_ID=dev-analytics-id
VITE_ENABLE_ANALYTICS=false
```

#### `.env.production`
```env
NODE_ENV=production
VITE_API_BASE_URL=https://api.fitnova.com
VITE_ANALYTICS_ID=prod-analytics-id
VITE_ENABLE_ANALYTICS=true
VITE_SENTRY_DSN=your-sentry-dsn
```

### Build Scripts

Update `package.json` with environment-specific scripts:

```json
{
  "scripts": {
    "dev": "vite --mode development",
    "build:dev": "vite build --mode development",
    "build:staging": "vite build --mode staging",
    "build:prod": "vite build --mode production",
    "preview": "vite preview",
    "deploy:dev": "npm run build:dev && vercel --target development",
    "deploy:staging": "npm run build:staging && vercel --target staging",
    "deploy:prod": "npm run build:prod && vercel --prod"
  }
}
```

## Performance Optimization

### 1. Asset Optimization

```bash
# Install optimization tools
npm install --save-dev imagemin imagemin-webp imagemin-mozjpeg imagemin-pngquant
```

### 2. Code Splitting

Implement dynamic imports for large features:

```javascript
// Lazy load heavy components
const MeditationModal = () => import('./components/MeditationModal.js');
const ProgressChart = () => import('./components/ProgressChart.js');
```

### 3. Service Worker for Caching

```javascript
// sw.js
const CACHE_NAME = 'fitnova-v1.0.0';
const urlsToCache = [
  '/',
  '/styles.css',
  '/script.js',
  '/assets/images/logo.png'
];

self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(urlsToCache))
  );
});
```

### 4. CDN Configuration

Use CDN for static assets:

```html
<link rel="preload" href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.0.0/css/all.min.css" as="style">
<link rel="preload" href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap" as="style">
```

## Monitoring & Analytics

### 1. Performance Monitoring

```javascript
// Performance tracking
if ('performance' in window) {
  window.addEventListener('load', () => {
    const perfData = performance.getEntriesByType('navigation')[0];
    console.log('Load time:', perfData.loadEventEnd - perfData.loadEventStart);
  });
}
```

### 2. Error Tracking

```javascript
// Error monitoring
window.addEventListener('error', (event) => {
  // Send to error tracking service
  console.error('Application error:', event.error);
});
```

### 3. Analytics Integration

```javascript
// Google Analytics 4
gtag('config', 'GA_MEASUREMENT_ID', {
  page_title: 'FitNova App',
  page_location: window.location.href
});
```

## Troubleshooting

### Common Issues

#### 1. Build Failures
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
npm run build
```

#### 2. Memory Issues
```bash
# Increase Node.js memory limit
NODE_OPTIONS=--max-old-space-size=4096 npm run build
```

#### 3. CORS Issues
Add proper CORS headers in your server configuration:

```javascript
// Express.js example
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept');
  next();
});
```

### Performance Issues

1. **Slow loading**: Enable gzip compression and use CDN
2. **Large bundle size**: Implement code splitting and tree shaking
3. **Memory leaks**: Clean up event listeners and intervals

### Debug Mode

Enable debug mode for troubleshooting:

```javascript
// Add to your HTML
<script>
  window.DEBUG = true;
  if (window.DEBUG) {
    console.log('Debug mode enabled');
  }
</script>
```

## Security Considerations

### 1. Content Security Policy

```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'unsafe-inline' https://cdnjs.cloudflare.com; 
               style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; 
               font-src 'self' https://fonts.gstatic.com;">
```

### 2. HTTPS Enforcement

Always deploy with HTTPS enabled. Most hosting platforms provide free SSL certificates.

### 3. Environment Variables

Never commit sensitive data to version control. Use environment variables for:
- API keys
- Database connection strings
- Third-party service credentials

## Deployment Checklist

- [ ] Environment variables configured
- [ ] Build process tested
- [ ] Performance optimized
- [ ] Security headers added
- [ ] Analytics implemented
- [ ] Error tracking enabled
- [ ] HTTPS enabled
- [ ] CDN configured
- [ ] Cache headers set
- [ ] Monitoring setup
- [ ] Backup strategy planned

---

For additional support, please refer to the [README.md](./README.md) or create an issue in the repository.
