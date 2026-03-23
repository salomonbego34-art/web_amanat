# Deploy Guide (Ubuntu + PM2 + Nginx)

## 1) Install dependencies on server

```bash
sudo apt update
sudo apt install -y nginx git curl
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm i -g pm2
```

## 2) Clone project and install packages

```bash
git clone <YOUR_REPO_URL> community-news-hub
cd community-news-hub
npm ci
```

## 3) Configure environment

```bash
cp .env.example .env
nano .env
```

Set these values in `.env`:

- `SQLITE_FILE=./data/app.db`
- `SESSION_SECRET=<random-long-secret>`

## 4) Build app

```bash
npm run build:fullstack
```

## 5) Run with PM2

```bash
set -a
source .env
set +a
pm2 start ecosystem.config.cjs --update-env
pm2 save
pm2 startup
```

## 6) Configure Nginx reverse proxy

```bash
sudo cp deploy/nginx/community-news-hub.conf /etc/nginx/sites-available/community-news-hub
sudo ln -s /etc/nginx/sites-available/community-news-hub /etc/nginx/sites-enabled/community-news-hub
sudo nginx -t
sudo systemctl reload nginx
```

Edit the nginx file and set `server_name` to your domain.

## 7) Enable HTTPS (Let's Encrypt)

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
```

## Useful commands

```bash
pm2 status
pm2 logs community-news-hub
pm2 restart community-news-hub
```
