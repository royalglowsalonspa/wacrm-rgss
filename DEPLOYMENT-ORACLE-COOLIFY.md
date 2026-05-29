# Deployment Plan: Oracle Cloud + Coolify + Supabase

> **Stack:** wacrm (Next.js 16) deployed on Oracle Cloud Always Free VPS with Coolify PaaS, backed by Supabase (free tier) for database, auth, storage, and realtime.
>
> **Monthly cost:** $0 infrastructure + WhatsApp conversation fees only.
>
> **Target user:** Royal Glow Salon & Spa — 2-3 staff (manager + receptionist), handling customer WhatsApp conversations.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Prerequisites](#prerequisites)
3. [Phase 1: Oracle Cloud VPS Setup](#phase-1-oracle-cloud-vps-setup)
4. [Phase 2: Install Coolify](#phase-2-install-coolify)
5. [Phase 3: Supabase Project Setup](#phase-3-supabase-project-setup)
6. [Phase 4: Deploy wacrm via Coolify](#phase-4-deploy-wacrm-via-coolify)
7. [Phase 5: WhatsApp Business API Setup](#phase-5-whatsapp-business-api-setup)
8. [Phase 6: Cron Jobs for Automations](#phase-6-cron-jobs-for-automations)
9. [Phase 7: Domain & SSL](#phase-7-domain--ssl)
10. [Phase 8: Post-Deployment Verification](#phase-8-post-deployment-verification)
11. [Maintenance & Monitoring](#maintenance--monitoring)
12. [Troubleshooting](#troubleshooting)
13. [Cost Breakdown](#cost-breakdown)

---

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│                    YOUR CRM — $0/month                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────┐   ┌──────────────────────────┐ │
│  │   Oracle Cloud VPS       │   │    Supabase (Free)       │ │
│  │   (Always Free ARM)      │   │                          │ │
│  │                          │   │  • Postgres database     │ │
│  │  • Ubuntu 22.04/24.04   │◄─►│  • Auth (login/signup)   │ │
│  │  • 4 OCPU / 24 GB RAM   │   │  • Realtime (live inbox) │ │
│  │  • Coolify (PaaS)       │   │  • Storage (media files) │ │
│  │  • Next.js (wacrm)      │   │                          │ │
│  │  • System crontab       │   │                          │ │
│  │  • Auto SSL via Caddy   │   │                          │ │
│  │  • Always on (no sleep) │   │                          │ │
│  └─────────────────────────┘   └──────────────────────────┘ │
│            ▲                                                 │
│            │ webhooks (HTTPS)                                │
│  ┌─────────┴───────────────┐                                │
│  │   Meta WhatsApp Cloud   │                                │
│  │   API (official)        │                                │
│  │                          │                                │
│  │  • 1000 free service    │                                │
│  │    conversations/month  │                                │
│  └─────────────────────────┘                                │
│                                                              │
│  ┌─────────────────────────┐                                │
│  │   Your Domain           │                                │
│  │   wacrm-rgss.store      │                                │
│  └─────────────────────────┘                                │
└──────────────────────────────────────────────────────────────┘
```

### Why This Stack Works

| Concern | How it's handled |
|---------|-----------------|
| **Always-on server** | Oracle VPS runs 24/7 — webhooks never miss a message |
| **No cold starts** | Persistent Node.js process, instant response |
| **Rate limiting works** | In-memory Map stays alive (single process) |
| **No function timeout** | Broadcasts to 500+ contacts run without limit |
| **Cron jobs** | System crontab, runs every minute, free |
| **Auto-deploy** | Coolify watches GitHub, deploys on push |
| **Free SSL** | Coolify auto-provisions Let's Encrypt certificates |
| **Database + Auth** | Supabase free tier (500MB, unlimited auth users) |
| **Realtime inbox** | Supabase Realtime (200 concurrent connections) |
| **Commercial use** | Allowed on all services |

---

## Prerequisites

Before starting, you'll need:

- [ ] A GitHub account (to host the code — this repo!)
- [ ] An Oracle Cloud account (free — no credit card charge, though card is required for verification)
- [ ] A Supabase account (free tier)
- [ ] A Meta Developer account + WhatsApp Business Account
- [ ] A domain name (optional but recommended — $10-15/year from Namecheap/Cloudflare)
- [ ] 45-60 minutes of uninterrupted setup time

---

## Phase 1: Oracle Cloud VPS Setup

### 1.1 Create Oracle Cloud Account

1. Go to [cloud.oracle.com/free](https://cloud.oracle.com/free)
2. Sign up for an **Always Free** account
3. Choose your home region (pick the closest to your customers):
   - Singapore, Mumbai, Tokyo for Asia
   - Frankfurt, London for Europe
   - Ashburn, Phoenix for Americas
4. Complete identity verification (credit card is used for verification only — you won't be charged)

> **Important:** The Always Free tier is permanent. You'll never be billed unless you manually upgrade.

### 1.2 Create an ARM Compute Instance

1. Go to **Compute → Instances → Create Instance**
2. Configure:

| Setting | Value |
|---------|-------|
| **Name** | `wacrm-server` |
| **Image** | Ubuntu 22.04 (or 24.04) — Canonical |
| **Shape** | Ampere A1 Flex (ARM) |
| **OCPUs** | 2 (of your 4 free) |
| **Memory** | 12 GB (of your 24 free) |
| **Boot volume** | 50 GB (of your 200 free) |
| **Networking** | Create new VCN + public subnet |
| **SSH key** | Upload your public key or let Oracle generate one |

3. Click **Create** and wait 2-3 minutes for provisioning

> **Tip:** If instance creation fails with "Out of capacity", try a different availability domain or try again later. ARM instances are popular on free tier.

### 1.3 Configure Networking (Security List)

After the instance is running, open the required ports:

1. Go to **Networking → Virtual Cloud Networks → your VCN → Security Lists → Default**
2. Add **Ingress Rules**:

| Port | Protocol | Source | Purpose |
|------|----------|--------|---------|
| 22 | TCP | 0.0.0.0/0 | SSH access |
| 80 | TCP | 0.0.0.0/0 | HTTP (redirect to HTTPS) |
| 443 | TCP | 0.0.0.0/0 | HTTPS (your CRM) |
| 8000 | TCP | 0.0.0.0/0 | Coolify dashboard (temporary — lock down later) |

### 1.4 SSH Into Your Instance

```bash
ssh -i ~/.ssh/your-key ubuntu@<your-instance-public-ip>
```

### 1.5 Open Firewall Ports on Ubuntu (iptables)

Oracle's Ubuntu images have iptables rules that block traffic even after security list changes:

```bash
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 443 -j ACCEPT
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 8000 -j ACCEPT
sudo netfilter-persistent save
```

### 1.6 Update System

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

---

## Phase 2: Install Coolify

### 2.1 One-Line Install

SSH back in after reboot, then:

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | sudo bash
```

This installs:
- Docker
- Coolify (with its own Postgres for internal state)
- Traefik (reverse proxy with auto-SSL)

Installation takes 3-5 minutes.

### 2.2 Access Coolify Dashboard

1. Open `http://<your-vps-ip>:8000` in your browser
2. Create your admin account (email + password)
3. Complete the initial setup wizard

### 2.3 Connect Your GitHub

1. In Coolify, go to **Sources → Add → GitHub App**
2. Follow the OAuth flow to connect your GitHub account
3. Grant access to `royalglowsalonspa/wacrm-rgss`

> **Alternative:** You can also use a deploy key or the public repository URL if you prefer not to connect your full GitHub account.

---

## Phase 3: Supabase Project Setup

### 3.1 Create Supabase Project

1. Go to [supabase.com](https://supabase.com) → **New Project**
2. Configure:

| Setting | Value |
|---------|-------|
| **Organization** | Create one (e.g., "Royal Glow") |
| **Project name** | `wacrm-rgss` |
| **Database password** | Generate a strong one (save it!) |
| **Region** | Same region as your Oracle VPS |

3. Wait ~2 minutes for provisioning

### 3.2 Run Database Migrations

1. Go to **SQL Editor** in your Supabase dashboard
2. Run each migration file **in order** from the `supabase/migrations/` folder:

```
001_initial_schema.sql
002_pipelines_enhancements.sql
003_broadcast_recipient_wamid.sql
004_contact_delete_set_null.sql
005_broadcast_counts_incremental.sql
006_automations.sql
007_automations_increment_counter.sql
008_profile_avatars_storage.sql
009_message_actions.sql
010_flows.sql
011_profile_beta_features.sql
012_flows_increment_counter.sql
013_whatsapp_config_phone_number_id_unique.sql
```

> **Tip:** Copy each file's contents and paste into the SQL Editor → Run. They're idempotent (safe to run multiple times if something fails midway).

### 3.3 Collect Your Supabase Credentials

Go to **Project Settings → API** and note:

| Key | Where to find it |
|-----|-----------------|
| `NEXT_PUBLIC_SUPABASE_URL` | Project URL (e.g., `https://xxxxx.supabase.co`) |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | anon / public key |
| `SUPABASE_SERVICE_ROLE_KEY` | service_role key (keep secret!) |

### 3.4 Verify Realtime

The migration already enables realtime for `messages` and `conversations` tables. Verify in **Database → Replication** that both tables show a green checkmark.

### 3.5 Configure Auth

1. Go to **Authentication → URL Configuration**
2. Set **Site URL** to `https://wacrm-rgss.store`
3. Set **Redirect URLs** to `https://wacrm-rgss.store/**`

---

## Phase 4: Deploy wacrm via Coolify

### 4.1 Create New Application in Coolify

1. In Coolify dashboard → **Projects → Add → New Resource → Application**
2. Select **GitHub** as the source
3. Pick `royalglowsalonspa/wacrm-rgss` repository
4. Branch: `main`
5. Build pack: **Nixpacks** (auto-detects Next.js)

### 4.2 Configure Build Settings

| Setting | Value |
|---------|-------|
| **Build command** | `npm run build` |
| **Start command** | `npm start` |
| **Install command** | `npm install` |
| **Port** | `3000` |
| **Node version** | `20` (or `22`) |

### 4.3 Set Environment Variables

In Coolify → your app → **Environment Variables**, add:

```env
# === REQUIRED ===
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
ENCRYPTION_KEY=<see below>
META_APP_SECRET=your-meta-app-secret

# === RECOMMENDED ===
NEXT_PUBLIC_SITE_URL=https://wacrm-rgss.store

# === OPTIONAL (required for automation Wait steps) ===
AUTOMATION_CRON_SECRET=<see below>
```

**Generate your ENCRYPTION_KEY** (run on any machine with Node.js):
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

**Generate your AUTOMATION_CRON_SECRET:**
```bash
openssl rand -hex 32
```

### 4.4 Configure Domain in Coolify

1. Go to your app → **Settings → Domains**
2. Add your domain: `wacrm-rgss.store`
3. Coolify will auto-provision a Let's Encrypt SSL certificate
4. If no custom domain yet, use the VPS IP with port: `http://<vps-ip>:3000`

### 4.5 Deploy

Click **Deploy** in Coolify. First build takes 3-5 minutes. Subsequent deploys take ~2 minutes.

### 4.6 Enable Auto-Deploy

In Coolify → your app → **Settings → General**:
- Enable **Auto Deploy** — pushes to `main` trigger automatic redeploys

### 4.7 Verify Deployment

Open `https://wacrm-rgss.store` — you should see the login page.

---

## Phase 5: WhatsApp Business API Setup

### 5.1 Create Meta Developer Account

1. Go to [developers.facebook.com](https://developers.facebook.com)
2. Create a new app → **Business** type
3. Add the **WhatsApp** product

### 5.2 Get WhatsApp Business API Credentials

In your Meta app dashboard:

| Credential | Where to find |
|------------|--------------|
| **Phone Number ID** | WhatsApp → API Setup → Phone number ID |
| **WhatsApp Business Account ID** | WhatsApp → API Setup → WABA ID |
| **Permanent Access Token** | System user token (see below) |
| **Meta App Secret** | App Settings → Basic → App Secret |

### 5.3 Create a System User Token (Permanent)

The test token expires in 24h. For production:

1. Go to [business.facebook.com](https://business.facebook.com) → **Business Settings**
2. **System Users → Add** → name it `wacrm-api` → role: Admin
3. **Generate Token** → select your app → permissions:
   - `whatsapp_business_management`
   - `whatsapp_business_messaging`
4. Copy the token (it won't expire)

### 5.4 Configure Webhook in Meta Dashboard

1. In Meta Developer Dashboard → WhatsApp → Configuration
2. Set **Callback URL** to: `https://wacrm-rgss.store/api/whatsapp/webhook`
3. Set **Verify Token** to any string you choose (you'll save this in the CRM settings)
4. Subscribe to webhook fields: `messages`

### 5.5 Connect in wacrm

1. Log into your CRM at `https://crm.yourdomain.com`
2. Go to **Settings → WhatsApp**
3. Enter your Phone Number ID, Access Token, and Verify Token
4. Click **Connect** — status should show "Connected"

---

## Phase 6: Cron Jobs for Automations

The wacrm app has two cron endpoints that need periodic triggering:

| Endpoint | Purpose | Recommended interval |
|----------|---------|---------------------|
| `/api/automations/cron` | Processes automation "Wait" steps | Every 2 minutes |
| `/api/flows/cron` | Sweeps abandoned flow conversations | Every 5 minutes |

### 6.1 Set Up System Crontab (on Oracle VPS)

SSH into your VPS and create a cron file:

```bash
sudo nano /etc/cron.d/wacrm-crons
```

Paste (replace `YOUR_CRON_SECRET`):

```cron
# wacrm automation cron - every 2 minutes
*/2 * * * * root curl -sf -H "x-cron-secret: YOUR_CRON_SECRET" https://wacrm-rgss.store/api/automations/cron > /dev/null 2>&1

# wacrm flows cron - every 5 minutes
*/5 * * * * root curl -sf -H "x-cron-secret: YOUR_CRON_SECRET" https://wacrm-rgss.store/api/flows/cron > /dev/null 2>&1
```

Set permissions:
```bash
sudo chmod 644 /etc/cron.d/wacrm-crons
```

### 6.2 Verify Cron is Working

```bash
# Test manually
curl -s -H "x-cron-secret: YOUR_CRON_SECRET" https://wacrm-rgss.store/api/automations/cron
# Expected: {"processed":0}

curl -s -H "x-cron-secret: YOUR_CRON_SECRET" https://wacrm-rgss.store/api/flows/cron
# Expected: {"swept":0}
```

---

## Phase 7: Domain & SSL

### Option A: Custom Domain (Recommended — ~$10/year)

1. Buy a domain (e.g., from Cloudflare Registrar — cheapest)
2. Add a DNS **A record**:
   - **Name:** `crm` (or `@` for root)
   - **Value:** `<your-oracle-vps-public-ip>`
   - **TTL:** Auto
3. In Coolify, add the domain to your app settings
4. Coolify auto-provisions SSL via Let's Encrypt (~30 seconds)

### Option B: Free Subdomain (No Purchase Needed)

Use a free DNS service:
- **DuckDNS** (duckdns.org) — free dynamic DNS subdomain
- **Afraid.org FreeDNS** — free subdomains
- **nip.io** — `<your-ip>.nip.io` resolves automatically

### Option C: IP-Only Access (Not Recommended for Production)

Access via `http://<vps-ip>:3000` — works but no SSL, which means:
- WhatsApp webhooks require HTTPS (won't work without a domain/SSL)
- Passwords sent in cleartext

> **Bottom line:** You need at minimum a free subdomain (Option B) for WhatsApp webhooks to work.

---

## Phase 8: Post-Deployment Verification

Run through this checklist after deployment:

### Functional Tests

- [ ] **Signup/Login** — Create an account, verify email, log in
- [ ] **Dashboard loads** — No console errors, stats display correctly
- [ ] **WhatsApp connected** — Settings → WhatsApp shows "Connected"
- [ ] **Receive message** — Send a test WhatsApp message to your business number
- [ ] **Message appears in inbox** — Real-time, no page refresh needed
- [ ] **Send reply** — Reply from inbox, verify customer receives it
- [ ] **Contacts created** — New contact auto-created from inbound message
- [ ] **Broadcast** — Send a test template message to one contact
- [ ] **Automation** — Create a simple auto-reply automation, test it
- [ ] **Pipeline** — Create a deal, move between Kanban stages

### Infrastructure Tests

- [ ] **Uptime** — VPS stays running after SSH disconnect
- [ ] **Auto-deploy** — Push a commit to `main`, verify Coolify rebuilds
- [ ] **SSL valid** — `https://wacrm-rgss.store` shows padlock in browser
- [ ] **Cron working** — `grep wacrm /var/log/syslog` shows executions
- [ ] **Webhook health** — Meta dashboard shows successful deliveries (green)

---

## Maintenance & Monitoring

### Auto-Updates

| What | How |
|------|-----|
| **App code** | Push to `main` → Coolify auto-deploys |
| **Coolify** | Dashboard → Settings → Update (one click) |
| **OS security patches** | Enable unattended-upgrades (below) |

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

### Monitoring (Free)

| Tool | Purpose | Cost |
|------|---------|------|
| **UptimeRobot** | Ping every 5min, alert if site is down | Free (50 monitors) |
| **Coolify dashboard** | Container health, logs, CPU/RAM usage | Built-in |
| **Supabase dashboard** | DB size, auth usage, API request count | Built-in |

### Backup Strategy

| What | How | Frequency |
|------|-----|-----------|
| **Database** | Supabase auto-backups (free tier: 7 days retention) | Daily (automatic) |
| **Code** | This GitHub repository | Every push |
| **Environment vars** | Export from Coolify + save in password manager | After any change |
| **VPS boot volume** | Oracle Cloud backup (free: 5 manual backups) | Weekly/monthly |

### Log Access

```bash
# Application logs via Coolify dashboard → your app → Logs
# Or via SSH:
docker logs $(docker ps -q --filter "name=wacrm") --tail 100 -f

# Cron execution logs
grep cron /var/log/syslog | tail -20
```

---

## Troubleshooting

### Common Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| Can't create Oracle ARM instance | Free tier capacity shortage | Try different availability domain, or retry at off-peak hours (early morning local time) |
| Coolify build fails | Node version too old | Set Node 20+ in Coolify build settings → Environment |
| Webhook returns 401 | Wrong Meta App Secret | Verify `META_APP_SECRET` env var matches your app's App Secret exactly |
| Messages not appearing in real-time | Supabase realtime not enabled | Check Database → Replication — `messages` and `conversations` must be enabled |
| Cron returns `{"error":"cron not configured"}` | Missing env var | Add `AUTOMATION_CRON_SECRET` in Coolify environment variables and redeploy |
| SSL cert not issuing | DNS not propagated yet | Wait 5-10 min, verify A record points to VPS IP with `dig wacrm-rgss.store` |
| App shows 502 Bad Gateway | Container crashed or still starting | Check Coolify logs, wait for build to finish, or restart the app |
| Rate limiting not working | Multiple container replicas | Ensure Coolify runs only 1 instance (default setting) |
| "Out of capacity" on Oracle | High demand in your region | Try again later, or try a different availability domain |
| WhatsApp "verify token mismatch" | Token in Meta dashboard ≠ token saved in CRM settings | Make sure both match exactly (case-sensitive) |

### Useful Commands (SSH into VPS)

```bash
# Check if containers are running
docker ps

# View app logs (last 50 lines)
docker logs $(docker ps -q --filter "name=wacrm") --tail 50

# Check disk space
df -h

# Check memory usage
free -h

# Restart Coolify
sudo systemctl restart coolify

# Test webhook endpoint manually
curl -X GET "https://wacrm-rgss.store/api/whatsapp/webhook?hub.mode=subscribe&hub.challenge=test123&hub.verify_token=YOUR_VERIFY_TOKEN"
```

---

## Cost Breakdown

### Monthly Recurring: $0

| Service | Tier | Monthly Cost | What You Get |
|---------|------|-------------|--------------|
| Oracle Cloud VPS | Always Free | **$0** | 4 OCPU, 24GB RAM, 200GB disk, 10TB bandwidth |
| Coolify | Self-hosted OSS | **$0** | PaaS dashboard, auto-deploy, SSL |
| Supabase | Free | **$0** | 500MB DB, unlimited auth users, 1GB storage, realtime |
| WhatsApp (service convos) | First 1000/month | **$0** | Customer-initiated conversations |
| UptimeRobot | Free | **$0** | Uptime monitoring with email alerts |
| **TOTAL INFRASTRUCTURE** | | **$0/month** | |

### Variable Costs (WhatsApp Conversations Only)

| Conversation Type | Cost (approx.) | When it applies |
|-------------------|----------------|-----------------|
| Service (customer messages you first) | First 1000/month **FREE**, then ~$0.02 each | Customer inquiries, bookings |
| Marketing (you broadcast to customer) | ~$0.04-0.08 each | Promotions, announcements |
| Utility (transactional updates) | ~$0.02-0.04 each | Appointment reminders, confirmations |

**Realistic monthly cost for Royal Glow Salon:**
- ~200-500 customer conversations (mostly free tier)
- Occasional broadcast promotions (~50-100 recipients)
- **Total: $0-15/month** (only WhatsApp fees when exceeding 1000 free conversations)

### When You'd Need to Upgrade

| Trigger | Upgrade to | Cost |
|---------|-----------|------|
| Database exceeds 500MB | Supabase Pro | $25/month |
| Need more than 4 CPU / 24GB RAM | Oracle paid tier (unlikely!) | ~$10-20/month |
| Want pro support for Coolify | Coolify Cloud | $5/month |

For a salon with 2-3 staff and hundreds of customers, **you probably won't need any upgrades for 1-2+ years**.

---

## Security Checklist

After deployment, verify these are in place:

- [x] **RLS enabled** — Every Supabase table has Row Level Security (handled by migrations)
- [x] **Webhook HMAC verification** — `META_APP_SECRET` verifies every inbound webhook signature
- [x] **Token encryption** — WhatsApp access tokens encrypted at rest (AES-256-GCM)
- [x] **HTTPS enforced** — Coolify auto-provisions and renews SSL certificates
- [x] **Cron endpoint protected** — Requires `x-cron-secret` header
- [x] **Security headers** — CSP, HSTS, X-Frame-Options, etc. (configured in `next.config.ts`)
- [ ] **Lock Coolify dashboard** — Restrict port 8000 to your IP only (see below)
- [ ] **SSH key-only auth** — Disable password SSH login
- [ ] **Minimal open ports** — Only 22, 80, 443 exposed to public

### Lock Down Coolify Dashboard (Do This After Initial Setup)

```bash
# Remove the open rule
sudo iptables -D INPUT -m state --state NEW -p tcp --dport 8000 -j ACCEPT
# Only allow YOUR IP to access Coolify
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 8000 -s YOUR.HOME.IP.ADDRESS -j ACCEPT
sudo netfilter-persistent save
```

### Disable Password SSH (Key-Only)

```bash
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
sudo systemctl restart sshd
```

---

## Timeline Summary

| Phase | Time | Difficulty |
|-------|------|-----------|
| 1. Oracle Cloud VPS | 15 min | Easy (clicking through wizard) |
| 2. Install Coolify | 5 min | Easy (one command) |
| 3. Supabase Setup | 15 min | Easy (copy-paste SQL migrations) |
| 4. Deploy wacrm | 10 min | Easy (fill form in Coolify) |
| 5. WhatsApp API | 20 min | Medium (Meta's dashboard is complex) |
| 6. Cron Jobs | 5 min | Easy (one config file) |
| 7. Domain & SSL | 5 min | Easy (DNS record + Coolify auto-SSL) |
| 8. Verification | 10 min | Easy (run through checklist) |
| **Total** | **~85 min** | |

---

## Next Steps After Deployment

1. **Create your staff accounts** — Sign up via the login page for manager + receptionist
2. **Import your contacts** — Use CSV import in the Contacts page
3. **Create message templates** — Get them approved by Meta (takes 24-48 hours)
4. **Set up automations** — Auto-reply for after-hours, welcome message for new contacts
5. **Build your sales pipeline** — Create stages: Inquiry → Consultation → Booked → Completed → Follow-up
6. **Train your team** — Walk manager and receptionist through inbox, contacts, and broadcasts
7. **Set up UptimeRobot** — Monitor `https://crm.yourdomain.com` with email alerts

---

*Last updated: May 2026*
*For Royal Glow Salon & Spa*
