# Deployment Guide

## Requirements

### Server
- Ubuntu 20.04+ or similar Linux
- Node.js 20 LTS
- MySQL 8+
- PM2 (process manager)
- Nginx (reverse proxy)

### Resources
- CPU: 2+ cores
- RAM: 4GB minimum
- Storage: 20GB SSD

## Backend Deployment

### 1. Clone and Install

```bash
git clone <repo> /var/www/crypto-yield-maxi
cd /var/www/crypto-yield-maxi/backend
pnpm install
```

### 2. Environment Variables

Create `.env`:

```env
NODE_ENV=production
PORT=1000

# Database
DB_HOST=localhost
DB_PORT=3306
DB_NAME=crypto_yield_maxi
DB_USER=app_user
DB_PASSWORD=<secure_password>

# Optional
LOG_LEVEL=info
```

### 3. Database Setup

```bash
mysql -u root -p
CREATE DATABASE crypto_yield_maxi;
CREATE USER 'app_user'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON crypto_yield_maxi.* TO 'app_user'@'localhost';
FLUSH PRIVILEGES;
```

### 4. PM2 Configuration

The repository already comes with a production-ready `ecosystem.config.cjs` that defines both `YIELD-MAXI-API` and `YIELD-MAXI-CRON` processes. It automatically loads variables from your `.env` file.

### 5. Build and Start

```bash
pnpm build
pm2 start ecosystem.config.cjs --env production
pm2 save
pm2 startup
```

We also highly recommend installing `pm2-logrotate` to prevent PM2 log files from consuming all server disk space:
```bash
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 50M
pm2 set pm2-logrotate:retain 10
pm2 set pm2-logrotate:compress true
```

## Frontend Deployment

### Option 1: Vercel (Recommended)

1. Connect GitHub repo to Vercel
2. Set environment variable:
   ```
   NEXT_PUBLIC_API_BASE_URL=https://api.yourdomain.com
   ```
3. Deploy

### Option 2: Self-hosted (PM2)

```bash
cd frontend
pnpm install
pnpm build

# Serve with provided PM2 ecosystem
pm2 start ecosystem.config.cjs --env production
pm2 save
```

## Nginx Configuration

### API Reverse Proxy

```nginx
server {
    listen 80;
    server_name api.yourdomain.com;

    location / {
        proxy_pass http://localhost:1000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### SSL with Let's Encrypt

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d api.yourdomain.com
```

## Monitoring

### PM2 Logs

```bash
pm2 logs api
pm2 logs cron
```

### Health Check

```bash
curl https://api.yourdomain.com/api/health
```

### Restart Services

```bash
pm2 restart all
pm2 restart api
pm2 restart cron
```

## Manual Sync Commands

```bash
cd /var/www/crypto-yield-maxi/backend

# Individual syncs
pnpm sync:lending     # Lending pools
pnpm sync:vault       # Vaults
pnpm sync:arbitrage   # Arbitrage opportunities
pnpm sync:delta       # Delta-neutral opportunities

# Check status
pnpm sync:check       # Verify all syncs
```

## Cron Schedule Summary

| Job | Schedule | Command |
|-----|----------|---------|
| Delta-Neutral | 1 * * * * | Auto via worker |
| Arbitrage | 3 * * * * | Auto via worker |
| Lending | 7 * * * * | Auto via worker |
| Vaults | 9 * * * * | Auto via worker |
| Telegram Summary | 15 * * * * | Auto via worker |
| Exchange Meta | 0 1 * * * | Auto via worker |

## Troubleshooting

### Database Connection Issues
```bash
mysql -u app_user -p crypto_yield_maxi
# Check if tables exist
SHOW TABLES;
```

### API Not Responding
```bash
pm2 status
pm2 logs api --lines 50
```

### Cron Not Running
```bash
pm2 logs cron --lines 50
# Check last sync times via API
curl http://localhost:1000/api/health
```

## Security Checklist

- [ ] MySQL password is strong
- [ ] `.env` is not in git
- [ ] CORS configured for frontend domain only
- [ ] Rate limiting on API
- [ ] HTTPS enabled
- [ ] Firewall configured (only 80, 443, 22)
