# Deployment Guide

This guide covers deploying the AI Fitness Trainer to production on Vercel (frontend) and Render (backend).

## Frontend Deployment (Vercel)

### 1. Prepare for Deployment

Create a `vercel.json` configuration:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "client/package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "dist"
      }
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "https://your-backend-url.onrender.com/api/$1"
    },
    {
      "src": "/(.*)",
      "dest": "/index.html"
    }
  ]
}
```

### 2. Deploy to Vercel

```bash
# Install Vercel CLI
npm i -g vercel

# Login to Vercel
vercel login

# Deploy
vercel --prod
```

### 3. Environment Variables

Set in Vercel dashboard:
- `VITE_API_URL`: Your backend URL (e.g., `https://your-app.onrender.com`)

## Backend Deployment (Render)

### 1. Create Render Service

1. Go to [Render Dashboard](https://dashboard.render.com)
2. Click "New +" → "Web Service"
3. Connect your GitHub repository
4. Configure:
   - **Name**: `ai-fitness-trainer-api`
   - **Environment**: `Node`
   - **Build Command**: `npm install`
   - **Start Command**: `npm start`

### 2. Environment Variables

Set in Render dashboard:
- `NODE_ENV`: `production`
- `PORT`: `10000` (Render default)

### 3. Health Check

Render will automatically check `/health` endpoint. The app includes basic health monitoring.

## Database Setup (Optional)

For persistent storage beyond in-memory:

### PostgreSQL on Render

1. Create PostgreSQL database on Render
2. Get connection string
3. Update environment variables:
   - `DATABASE_URL`: Your PostgreSQL connection string

### Update Configuration

Modify `drizzle.config.ts` for production database:

```typescript
export default {
  schema: "./shared/schema.ts",
  out: "./drizzle",
  driver: "pg",
  dbCredentials: {
    connectionString: process.env.DATABASE_URL!,
  },
} satisfies Config;
```

## GitHub Actions (Optional)

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm run build
      - run: npm run test
```

## Performance Optimization

### Frontend Optimizations

1. **Bundle Analysis**
```bash
npm run build -- --analyze
```

2. **Image Optimization**
- Use WebP format for images
- Implement lazy loading

3. **Code Splitting**
- Already configured with Vite
- Automatic route-based splitting

### Backend Optimizations

1. **Caching**
- Implement Redis for session caching
- Add HTTP caching headers

2. **CDN Setup**
- Use Vercel CDN for static assets
- Configure cache policies

## Monitoring & Analytics

### Error Tracking

Add Sentry for error monitoring:

```bash
npm install @sentry/react @sentry/node
```

### Performance Monitoring

- Vercel Analytics (automatic)
- Google Analytics for user tracking
- Core Web Vitals monitoring

## Security Considerations

### Content Security Policy

Add to `index.html`:

```html
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self';
  script-src 'self' 'unsafe-inline' 'unsafe-eval' https://cdn.jsdelivr.net;
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  font-src 'self' https://fonts.gstatic.com;
  img-src 'self' data: blob:;
  media-src 'self' blob:;
  connect-src 'self' https://your-api-url.onrender.com;
">
```

### HTTPS Enforcement

Both Vercel and Render provide automatic HTTPS. Ensure all API calls use HTTPS in production.

## Custom Domain Setup

### Vercel Domain

1. Go to Project Settings → Domains
2. Add your custom domain
3. Configure DNS records as instructed

### Render Domain

1. Go to Service Settings → Custom Domains
2. Add your domain
3. Update DNS CNAME record

## Troubleshooting

### Common Issues

1. **Camera Access**: Ensure HTTPS for camera permissions
2. **CORS Errors**: Configure CORS in Express server
3. **Build Failures**: Check Node.js version compatibility

### Debug Commands

```bash
# Check build output
npm run build

# Test production build locally
npm run preview

# Check type errors
npm run type-check
```

## Post-Deployment Checklist

- [ ] Frontend loads correctly
- [ ] Camera access works on HTTPS
- [ ] API endpoints respond
- [ ] Database connections work
- [ ] Error tracking configured
- [ ] Analytics set up
- [ ] Custom domain configured
- [ ] SSL certificate active
- [ ] Performance metrics baseline established

## Maintenance

### Regular Tasks

1. **Dependency Updates**
```bash
npm audit
npm update
```

2. **Performance Monitoring**
- Check Lighthouse scores monthly
- Monitor Core Web Vitals
- Review error rates

3. **Security Updates**
- Monitor vulnerability reports
- Update dependencies regularly
- Review access logs

### Backup Strategy

- Database backups (if using persistent storage)
- Environment variable documentation
- Configuration file versioning

## Support

For deployment issues:
1. Check service logs in respective dashboards
2. Review GitHub Issues for known problems
3. Contact support through platform channels
