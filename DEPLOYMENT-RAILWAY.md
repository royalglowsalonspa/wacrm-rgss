# Deployment Guide: Railway.app + Supabase

> **Stack:** wacrm-rgss (Next.js 16) deployed on Railway.app, backed by Supabase (free tier) for database, auth, storage, and realtime.
>
> **Monthly cost:** ~$5/month (Railway Hobby) + $0 (Supabase Free) + WhatsApp conversation fees.
>
> **Target user:** Royal Glow Salon & Spa — 2-3 staff (manager + receptionist), handling customer WhatsApp conversations.
>
> **Domain:** `wacrm-rgss.store`

---

## Table of Contents

1. [Why Railway (vs. Oracle Cloud)](#1-why-railway-vs-oracle-cloud)
2. [Railway Account Setup & Billing](#2-railway-account-setup--billing)
3. [Creating a Railway Project from GitHub](#3-creating-a-railway-project-from-github)
4. [Environment Variables Configuration](#4-environment-variables-configuration)
5. [Supabase Project Setup](#5-supabase-project-setup)
6. [Custom Domain Setup](#6-custom-domain-setup)
7. [Railway Cron Jobs Setup](#7-railway-cron-jobs-setup)
8. [WhatsApp Business API Setup](#8-whatsapp-business-api-setup)
9. [Connecting WhatsApp in the CRM](#9-connecting-whatsapp-in-the-crm)
10. [Post-Deployment Verification Checklist](#10-post-deployment-verification-checklist)
11. [Railway-Specific Considerations](#11-railway-specific-considerations)
12. [Cost Breakdown](#12-cost-breakdown)
13. [Comparison: Railway vs. Oracle Cloud + Coolify](#13-comparison-railway-vs-oracle-cloud--coolify)
14. [Troubleshooting Common Railway Issues](#14-troubleshooting-common-railway-issues)
15. [Security Considerations](#15-security-considerations)

---


## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│             YOUR CRM — ~$5/month total                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────┐  ┌────────────────────────────┐│
│  │   Railway.app (Hobby $5)    │  │   Supabase (Free Tier)     ││
│  │                             │  │                            ││
│  │  • Next.js 16 (wacrm-rgss) │  │  • Postgres database       ││
│  │  • Persistent Node.js      │◄►│  • Auth (login/signup)     ││
│  │  • Auto-deploy from GitHub │  │  • Realtime (live inbox)   ││
│  │  • Built-in cron jobs      │  │  • Storage (media files)   ││
│  │  • Custom domain + SSL     │  │  • Row Level Security      ││
│  │  • Always-on (Hobby plan)  │  │                            ││
│  └─────────────────────────────┘  └────────────────────────────┘│
│            ▲                                                     │
│            │ webhooks (HTTPS)                                    │
│  ┌─────────┴─────────────────────┐                              │
│  │   Meta WhatsApp Cloud API     │                              │
│  │   (official Business API)     │                              │
│  │                               │                              │
│  │  • 1000 free service          │                              │
│  │    conversations/month        │                              │
│  └───────────────────────────────┘                              │
│                                                                  │
│  ┌───────────────────────────────┐                              │
│  │   Custom Domain               │                              │
│  │   wacrm-rgss.store            │                              │
│  └───────────────────────────────┘                              │
└──────────────────────────────────────────────────────────────────┘
```

---


## 1. Why Railway (vs. Oracle Cloud)

### The Problem with Oracle Cloud Free Tier

Oracle Cloud offers an incredible Always Free ARM VPS (4 OCPU, 24GB RAM), but there's a critical issue: **ARM capacity is frequently unavailable**. When you try to create an Ampere A1 instance, you'll often see:

```
Error: Out of capacity for shape VM.Standard.A1.Flex in availability domain AD-1
```

This can persist for **days, weeks, or even months** depending on your region. There's no guaranteed timeline for availability.

### Why Railway is the Perfect Alternative

| Factor | Railway | Oracle Cloud |
|--------|---------|-------------|
| **Availability** | Instant — deploy in 2 minutes | May wait days/weeks for ARM capacity |
| **Setup complexity** | Click + connect GitHub = done | VPS + Coolify + Docker + firewall + SSL |
| **Maintenance** | Zero — fully managed | You manage OS updates, Docker, SSL renewal |
| **Cron jobs** | Built-in feature (dashboard UI) | Manual crontab setup via SSH |
| **Custom domains** | One-click + auto-SSL | Manual DNS + Let's Encrypt via Coolify |
| **Auto-deploy** | Automatic on git push | Requires Coolify GitHub App setup |
| **Cost** | $5/month (Hobby plan credit) | $0 (when capacity is available) |
| **Scaling** | Instant vertical scaling | Fixed resources (but generous) |
| **Uptime SLA** | 99.9% | No SLA on free tier |
| **Support** | Community + paid options | Community forums only |

### When to Choose Railway

- Oracle ARM capacity is unavailable in your region
- You want zero server management (no SSH, no Docker, no firewall rules)
- You value simplicity over cost savings ($5/mo is negligible for a business)
- You need to deploy immediately (client is waiting)
- You prefer a managed platform with a clean dashboard

### When to Choose Oracle Cloud + Coolify

- ARM capacity is available in your region
- You want $0/month infrastructure
- You're comfortable managing a Linux VPS
- You want unlimited resources (4 CPU + 24GB RAM vs. Railway's metered usage)
- You enjoy tinkering with server infrastructure

> **Recommendation for Royal Glow Salon & Spa:** Use Railway. The $5/month cost is trivial for a business, and zero maintenance means the salon owner never needs to worry about server updates, Docker crashes, or SSL renewal. Focus on serving customers, not managing infrastructure.

**Done when:** You've decided to use Railway and are ready to create an account.

---


## 2. Railway Account Setup & Billing

### 2.1 Create Your Railway Account

1. Go to **[railway.app](https://railway.app)**
2. Click **"Login"** in the top-right corner
3. Sign up with **GitHub** (recommended — makes deployment seamless)
   - Railway will ask permission to read your GitHub repos
   - Grant access to the `royalglowsalonspa` organization (or your personal account)
4. Complete email verification if prompted

### 2.2 Subscribe to the Hobby Plan ($5/month)

The **Trial plan** has limitations (500 execution hours, no custom domains). You need the **Hobby plan**:

1. After logging in, click your avatar (bottom-left) → **"Account Settings"**
2. Go to the **"Billing"** tab
3. Click **"Subscribe to Hobby"** (or "Upgrade to Hobby")
4. Enter your credit/debit card details
5. Confirm the $5/month subscription

**What the Hobby plan includes:**

| Feature | Limit |
|---------|-------|
| **Monthly credit** | $5 included (covers a small Next.js app 24/7) |
| **Execution hours** | Unlimited (no sleep) |
| **Custom domains** | Unlimited |
| **Team members** | 1 (just you) |
| **Deployments** | Unlimited |
| **Network egress** | 100 GB/month included |
| **Build minutes** | Included in $5 credit |
| **RAM** | Up to 32 GB (pay per use) |
| **vCPU** | Up to 32 cores (pay per use) |

**Estimated usage for wacrm-rgss:**
- A small Next.js app running 24/7 uses approximately **$3-4/month** in compute
- This fits well within the $5 monthly credit
- You'll likely have $1-2 leftover each month
- If you slightly exceed $5, Railway bills the overage to your card (usually pennies)

### 2.3 Verify Your Account is Active

After subscribing:
1. Go to **[railway.app/dashboard](https://railway.app/dashboard)**
2. You should see "Hobby" badge next to your account name
3. The billing page should show "$5.00 credit" for the current period

**Done when:** You see "Hobby" on your Railway dashboard and your billing is configured.

---


## 3. Creating a Railway Project from GitHub

### 3.1 Create a New Project

1. On your Railway dashboard, click **"+ New Project"** (top-right)
2. Select **"Deploy from GitHub Repo"**
3. If this is your first time, Railway will ask you to install the Railway GitHub App:
   - Click **"Configure GitHub App"**
   - Select the `royalglowsalonspa` organization
   - Under "Repository access", choose **"Only select repositories"**
   - Select `wacrm-rgss`
   - Click **"Install & Authorize"**
4. Back in Railway, you'll see your repos listed
5. Click **`royalglowsalonspa/wacrm-rgss`**

### 3.2 Initial Deployment Settings

Railway will auto-detect that this is a Next.js app and begin deploying immediately. However, **the first deploy will fail** because environment variables aren't set yet. That's fine — we'll fix it in the next section.

What Railway auto-detects:
- **Builder:** Nixpacks (auto-detects Node.js + Next.js)
- **Build command:** `npm run build`
- **Start command:** `npm start`
- **Port:** `3000` (auto-detected from Next.js)

### 3.3 Configure Service Settings

After the initial (failed) deploy, click on your service to open its settings:

1. Click the service card (named `wacrm-rgss` or similar)
2. Go to the **"Settings"** tab
3. Verify/adjust these settings:

| Setting | Value | Notes |
|---------|-------|-------|
| **Service name** | `wacrm-rgss` | Cosmetic name in dashboard |
| **Root directory** | `/` (blank/default) | The Next.js app is at repo root |
| **Build command** | `npm run build` | Auto-detected |
| **Start command** | `npm start` | Auto-detected |
| **Watch paths** | Leave empty | Deploy on any file change |
| **Restart policy** | `on-failure` | Auto-restart if crashes |
| **Region** | Choose closest to your customers | Singapore for SEA, US-West for Americas |

### 3.4 Set the Branch

Under **Settings → Source**:
- **Branch:** `main` (deploy from the main branch)
- **Auto-deploy:** Enabled (every push to `main` triggers a new deploy)

### 3.5 Understand the Railway Project Structure

Your Railway project now looks like:

```
Railway Project: "wacrm-rgss"
└── Service: "wacrm-rgss" (Next.js app)
    ├── Deployments (build + run)
    ├── Variables (env vars)
    ├── Settings (domain, build config)
    ├── Metrics (CPU, RAM, network)
    └── Logs (real-time stdout/stderr)
```

**Done when:** You have a Railway project with the service created (even if the first deploy failed due to missing env vars).

---


## 4. Environment Variables Configuration

### 4.1 Navigate to Variables

1. In your Railway project, click the **`wacrm-rgss`** service
2. Go to the **"Variables"** tab
3. You can add variables one-by-one or use **"RAW Editor"** to paste them all at once

### 4.2 All Required Variables

Click **"RAW Editor"** and paste the following (replacing placeholder values):

```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project-ref.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...your-anon-key
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...your-service-role-key
ENCRYPTION_KEY=your-64-character-hex-string-here
META_APP_SECRET=your-meta-app-secret-from-facebook-developers
NEXT_PUBLIC_SITE_URL=https://wacrm-rgss.store
AUTOMATION_CRON_SECRET=your-long-random-string-for-cron-auth
```

### 4.3 Detailed Explanation of Each Variable

#### `NEXT_PUBLIC_SUPABASE_URL`
- **What it is:** The URL of your Supabase project's API endpoint
- **Format:** `https://abcdefghijklmnop.supabase.co`
- **Where to find it:** Supabase Dashboard → Project Settings → API → "Project URL"
- **Public?** Yes — this is exposed to the browser (it's fine, RLS protects your data)
- **Example:** `https://xyzabc123def.supabase.co`

#### `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- **What it is:** The public anonymous key for client-side Supabase access
- **Format:** A long JWT token (starts with `eyJ...`)
- **Where to find it:** Supabase Dashboard → Project Settings → API → "anon public" key
- **Public?** Yes — this is safe to expose; RLS policies determine what data is accessible
- **Example:** `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InhtdX...`

#### `SUPABASE_SERVICE_ROLE_KEY`
- **What it is:** A privileged key that bypasses Row Level Security (RLS)
- **Format:** A long JWT token (starts with `eyJ...`)
- **Where to find it:** Supabase Dashboard → Project Settings → API → "service_role secret" key
- **Public?** **NO! NEVER expose this.** It grants full database access.
- **Used by:** Server-side API routes only (webhook handler, automation engine, cron endpoints)
- **Example:** `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InhtdX...`

#### `ENCRYPTION_KEY`
- **What it is:** A 32-byte (64 hex characters) key for AES-256-GCM encryption
- **Purpose:** Encrypts WhatsApp access tokens stored in the database
- **How to generate:**
  ```bash
  node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
  ```
  Or on any system:
  ```bash
  openssl rand -hex 32
  ```
- **Important:** If you rotate this key, all previously encrypted tokens become unreadable. Users will need to re-enter their WhatsApp credentials in Settings.
- **Example:** `a3f8b2c1d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1`

#### `META_APP_SECRET`
- **What it is:** Your Meta (Facebook) app's secret key
- **Purpose:** Verifies HMAC-SHA256 signatures on every inbound WhatsApp webhook POST
- **Where to find it:** [developers.facebook.com](https://developers.facebook.com) → Your App → App Settings → Basic → "App Secret" (click "Show")
- **Public?** **NO! Keep this secret.**
- **Example:** `a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6`

#### `NEXT_PUBLIC_SITE_URL`
- **What it is:** The canonical public URL of your deployed CRM
- **Purpose:** Used for OG images, sitemap, email links, and Supabase auth redirects
- **Format:** `https://your-domain.com` (no trailing slash)
- **Value for this deployment:** `https://wacrm-rgss.store`
- **Public?** Yes — it's the URL your users visit

#### `AUTOMATION_CRON_SECRET`
- **What it is:** A shared secret that protects the cron API endpoints
- **Purpose:** Prevents unauthorized access to `/api/automations/cron` and `/api/flows/cron`
- **How to generate:**
  ```bash
  openssl rand -hex 32
  ```
- **How it works:** The cron job sends this value in the `x-cron-secret` HTTP header. The endpoint rejects requests without a matching header.
- **Example:** `7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8`

### 4.4 Save and Redeploy

After pasting all variables:
1. Click **"Update Variables"** (or the save/apply button)
2. Railway will **automatically trigger a new deployment** with the updated variables
3. Wait 2-3 minutes for the build + deploy to complete
4. Check the **"Deployments"** tab — the latest deploy should show a green checkmark

### 4.5 Verify Variables Are Set

In the Variables tab, you should see all 7 variables listed. Railway masks secret values by default (shows `•••••`). You can click the eye icon to reveal them.

**Done when:** All 7 environment variables are saved in Railway and a new deployment is triggered.

---


## 5. Supabase Project Setup

### 5.1 Create a Supabase Account

1. Go to **[supabase.com](https://supabase.com)**
2. Click **"Start your project"** or **"Sign Up"**
3. Sign up with GitHub (recommended) or email
4. Complete email verification if prompted

### 5.2 Create a New Project

1. Click **"New Project"** on the Supabase dashboard
2. Fill in the project details:

| Field | Value | Notes |
|-------|-------|-------|
| **Organization** | Create new: "Royal Glow Salon" | Or select existing org |
| **Project name** | `wacrm-rgss` | Appears in your dashboard |
| **Database password** | Generate a strong one | **Save this password!** You'll need it for direct DB access |
| **Region** | Choose closest to your customers | `Southeast Asia (Singapore)` for PH/SEA |
| **Pricing plan** | Free | 500 MB database, unlimited auth |

3. Click **"Create new project"**
4. Wait 1-2 minutes for provisioning (you'll see a progress indicator)

### 5.3 Run Database Migrations (13 Files)

The database schema is defined in migration files. You must run them **in order**.

1. In your Supabase dashboard, go to **"SQL Editor"** (left sidebar, looks like a terminal icon)
2. Click **"New query"**

**Run each migration one at a time, in this exact order:**

#### Migration 1: `001_initial_schema.sql`
- Copy the entire contents of `supabase/migrations/001_initial_schema.sql` from the repository
- Paste into the SQL Editor
- Click **"Run"** (or press Ctrl+Enter / Cmd+Enter)
- Wait for "Success. No rows returned" message
- This creates: profiles, whatsapp_configs, contacts, conversations, messages, pipeline_stages, deals tables + RLS policies + realtime setup

#### Migration 2: `002_pipelines_enhancements.sql`
- Copy contents of `supabase/migrations/002_pipelines_enhancements.sql`
- Paste and Run
- This adds: pipeline stages ordering, deal value tracking

#### Migration 3: `003_broadcast_recipient_wamid.sql`
- Copy contents of `supabase/migrations/003_broadcast_recipient_wamid.sql`
- Paste and Run
- This adds: WhatsApp message ID tracking for broadcast recipients

#### Migration 4: `004_contact_delete_set_null.sql`
- Copy contents of `supabase/migrations/004_contact_delete_set_null.sql`
- Paste and Run
- This fixes: foreign key behavior when contacts are deleted

#### Migration 5: `005_broadcast_counts_incremental.sql`
- Copy contents of `supabase/migrations/005_broadcast_counts_incremental.sql`
- Paste and Run
- This adds: incremental broadcast sent/delivered/read counters

#### Migration 6: `006_automations.sql`
- Copy contents of `supabase/migrations/006_automations.sql`
- Paste and Run
- This creates: automations, automation_steps, automation_executions tables

#### Migration 7: `007_automations_increment_counter.sql`
- Copy contents of `supabase/migrations/007_automations_increment_counter.sql`
- Paste and Run
- This adds: trigger-based counter for automation execution stats

#### Migration 8: `008_profile_avatars_storage.sql`
- Copy contents of `supabase/migrations/008_profile_avatars_storage.sql`
- Paste and Run
- This creates: Supabase Storage bucket for profile avatars

#### Migration 9: `009_message_actions.sql`
- Copy contents of `supabase/migrations/009_message_actions.sql`
- Paste and Run
- This adds: message action tracking (starred, archived, etc.)

#### Migration 10: `010_flows.sql`
- Copy contents of `supabase/migrations/010_flows.sql`
- Paste and Run
- This creates: flows, flow_steps, flow_sessions tables for conversational flows

#### Migration 11: `011_profile_beta_features.sql`
- Copy contents of `supabase/migrations/011_profile_beta_features.sql`
- Paste and Run
- This adds: beta feature flags per user profile

#### Migration 12: `012_flows_increment_counter.sql`
- Copy contents of `supabase/migrations/012_flows_increment_counter.sql`
- Paste and Run
- This adds: trigger-based counter for flow session stats

#### Migration 13: `013_whatsapp_config_phone_number_id_unique.sql`
- Copy contents of `supabase/migrations/013_whatsapp_config_phone_number_id_unique.sql`
- Paste and Run
- This adds: unique constraint on phone_number_id to prevent duplicate WhatsApp configs

> **Important:** Run them in order (001 → 013). Each migration depends on previous ones. If one fails, fix the issue before proceeding.

> **Tip:** If a migration fails with "relation already exists", it means you've already run it. Skip to the next one.

### 5.4 Verify the Migration Results

After all 13 migrations:

1. Go to **"Table Editor"** (left sidebar)
2. You should see these tables:

| Table | Purpose |
|-------|---------|
| `profiles` | User accounts (staff members) |
| `whatsapp_configs` | WhatsApp connection settings per user |
| `contacts` | Customer phone numbers and details |
| `conversations` | Chat threads with customers |
| `messages` | Individual WhatsApp messages |
| `pipeline_stages` | Sales pipeline columns (Kanban) |
| `deals` | Sales deals/opportunities |
| `broadcasts` | Bulk message campaigns |
| `broadcast_recipients` | Individual recipients in a broadcast |
| `automations` | Automation rule definitions |
| `automation_steps` | Steps within an automation |
| `automation_executions` | Running/completed automation instances |
| `flows` | Conversational flow definitions |
| `flow_steps` | Steps within a flow |
| `flow_sessions` | Active flow sessions with contacts |

### 5.5 Verify Realtime is Enabled

1. Go to **"Database"** → **"Replication"** (or **"Realtime"** in newer UI)
2. Ensure these tables have realtime enabled (green toggle/checkmark):
   - `messages`
   - `conversations`
3. If not enabled, toggle them on

This is what powers the live inbox — new messages appear instantly without page refresh.

### 5.6 Configure Authentication Settings

1. Go to **"Authentication"** → **"URL Configuration"**
2. Set the following:

| Setting | Value |
|---------|-------|
| **Site URL** | `https://wacrm-rgss.store` |
| **Redirect URLs** | `https://wacrm-rgss.store/**` |

3. Click **"Save"**

This ensures login/signup redirects work correctly with your custom domain.

### 5.7 Collect Your API Credentials

Go to **"Project Settings"** (gear icon, bottom-left) → **"API"**:

| Credential | Label in Supabase | Copy to Railway as... |
|------------|-------------------|----------------------|
| Project URL | "Project URL" | `NEXT_PUBLIC_SUPABASE_URL` |
| anon public | "Project API keys → anon public" | `NEXT_PUBLIC_SUPABASE_ANON_KEY` |
| service_role | "Project API keys → service_role secret" | `SUPABASE_SERVICE_ROLE_KEY` |

Copy each value and paste it into your Railway environment variables (Section 4).

**Done when:** All 13 migrations ran successfully, realtime is enabled for messages/conversations, auth URLs are configured, and you've copied the 3 Supabase credentials to Railway.

---


## 6. Custom Domain Setup

### 6.1 Add Domain in Railway

1. In your Railway project, click the **`wacrm-rgss`** service
2. Go to the **"Settings"** tab
3. Scroll to **"Networking"** → **"Public Networking"**
4. Click **"+ Custom Domain"**
5. Enter: `wacrm-rgss.store`
6. Railway will display the DNS records you need to configure

Railway will show you something like:

```
Type: CNAME
Name: wacrm-rgss.store (or @)
Value: <your-service>.up.railway.app
```

### 6.2 Configure DNS Records

Go to your domain registrar's DNS settings (wherever you purchased `wacrm-rgss.store` — Namecheap, Cloudflare, GoDaddy, etc.):

#### Option A: Root Domain (`wacrm-rgss.store`) — Using CNAME Flattening

If your DNS provider supports **CNAME flattening** (Cloudflare does):

| Type | Name | Value | TTL |
|------|------|-------|-----|
| CNAME | `@` | `<your-service>.up.railway.app` | Auto |

#### Option B: Root Domain — Using A Records (if CNAME flattening not supported)

Railway doesn't provide static IPs for A records. If your registrar doesn't support CNAME flattening at the apex:

**Recommended:** Use Cloudflare as your DNS provider (free plan) which supports CNAME flattening at root.

To move to Cloudflare DNS:
1. Create a free Cloudflare account at [cloudflare.com](https://cloudflare.com)
2. Add your domain `wacrm-rgss.store`
3. Cloudflare will tell you to change nameservers at your registrar
4. Update nameservers at your registrar to the ones Cloudflare provides
5. Wait for propagation (can take 5 minutes to 24 hours)
6. Then add the CNAME record in Cloudflare DNS settings

#### Option C: Subdomain (`app.wacrm-rgss.store`)

If you prefer to use a subdomain (no CNAME flattening issues):

| Type | Name | Value | TTL |
|------|------|-------|-----|
| CNAME | `app` | `<your-service>.up.railway.app` | Auto |

Then update `NEXT_PUBLIC_SITE_URL` to `https://app.wacrm-rgss.store` in Railway.

### 6.3 Verify DNS Propagation

After adding DNS records, verify propagation:

**Method 1: Railway Dashboard**
- Go back to your service → Settings → Public Networking
- Railway shows a checkmark when DNS is verified and SSL is provisioned

**Method 2: Command Line (from any computer)**
```bash
# Check if DNS resolves
dig wacrm-rgss.store CNAME +short
# Should return: <something>.up.railway.app

# Or check A record resolution
dig wacrm-rgss.store A +short
# Should return an IP address (Cloudflare's if using CF)

# Test HTTPS
curl -I https://wacrm-rgss.store
# Should return: HTTP/2 200 (or 302 redirect to login)
```

**Method 3: Online Tool**
- Go to [dnschecker.org](https://dnschecker.org)
- Enter `wacrm-rgss.store`
- Verify it resolves from multiple locations worldwide

### 6.4 SSL Certificate (Automatic)

Railway automatically provisions and renews SSL certificates for custom domains via Let's Encrypt. You don't need to do anything — just wait 1-2 minutes after DNS verification.

Verify SSL:
- Visit `https://wacrm-rgss.store` in your browser
- You should see a padlock icon in the address bar
- Click the padlock to verify: "Certificate is valid" and issued by Let's Encrypt

### 6.5 Configure www Redirect (Optional)

If you want `www.wacrm-rgss.store` to redirect to `wacrm-rgss.store`:

1. Add another custom domain in Railway: `www.wacrm-rgss.store`
2. Add a DNS CNAME record:
   | Type | Name | Value | TTL |
   |------|------|-------|-----|
   | CNAME | `www` | `<your-service>.up.railway.app` | Auto |

Railway will handle the redirect automatically.

**Done when:** `https://wacrm-rgss.store` loads your CRM login page with a valid SSL certificate (padlock icon visible).

---


## 7. Railway Cron Jobs Setup

The wacrm-rgss app requires periodic cron triggers to process automation "Wait" steps and sweep abandoned flow sessions. Railway has a **built-in cron job feature** that eliminates the need for external cron services or a server crontab.

### 7.1 Understanding the Two Cron Endpoints

| Endpoint | Purpose | What it does |
|----------|---------|-------------|
| `GET /api/automations/cron` | Process automation Wait steps | Checks for automation executions that have completed their wait period and advances them to the next step |
| `GET /api/flows/cron` | Sweep abandoned flows | Marks flow sessions as abandoned if the contact hasn't responded within the timeout window |

Both endpoints require the `x-cron-secret` header with the value matching your `AUTOMATION_CRON_SECRET` environment variable.

### 7.2 Create Cron Job for Automations

1. In your Railway project dashboard, click **"+ New"** (or **"Add a Service"**)
2. Select **"Cron Job"**
3. Configure the cron job:

| Setting | Value |
|---------|-------|
| **Name** | `automations-cron` |
| **Schedule** | `*/2 * * * *` (every 2 minutes) |
| **Command** | See below |
| **Linked service** | `wacrm-rgss` |

**Command/Script:**
```bash
curl -sf -H "x-cron-secret: $AUTOMATION_CRON_SECRET" https://wacrm-rgss.store/api/automations/cron
```

> **Note:** Railway cron jobs can reference environment variables from linked services. Make sure `AUTOMATION_CRON_SECRET` is accessible.

**Alternative approach — If Railway's cron job UI requires a Docker image or build:**

Railway's cron jobs can also be configured as a lightweight service. Create it as:
- **Image:** `curlimages/curl:latest` (a tiny Docker image with curl)
- **Command:** `curl -sf -H "x-cron-secret: YOUR_ACTUAL_SECRET" https://wacrm-rgss.store/api/automations/cron`
- **Schedule:** `*/2 * * * *`

### 7.3 Create Cron Job for Flows

Repeat the process for the flows endpoint:

1. Click **"+ New"** → **"Cron Job"**
2. Configure:

| Setting | Value |
|---------|-------|
| **Name** | `flows-cron` |
| **Schedule** | `*/5 * * * *` (every 5 minutes) |
| **Command** | See below |

**Command/Script:**
```bash
curl -sf -H "x-cron-secret: $AUTOMATION_CRON_SECRET" https://wacrm-rgss.store/api/flows/cron
```

### 7.4 Alternative: Use an External Cron Service (Free)

If you prefer not to use Railway's built-in cron (to save a few cents of compute), use a free external cron service:

**Option A: cron-job.org (Recommended — Free)**

1. Go to [cron-job.org](https://cron-job.org) and create a free account
2. Create Job 1:
   - **Title:** `wacrm automations cron`
   - **URL:** `https://wacrm-rgss.store/api/automations/cron`
   - **Schedule:** Every 2 minutes
   - **Request method:** GET
   - **Headers:** Add custom header: `x-cron-secret: YOUR_AUTOMATION_CRON_SECRET`
3. Create Job 2:
   - **Title:** `wacrm flows cron`
   - **URL:** `https://wacrm-rgss.store/api/flows/cron`
   - **Schedule:** Every 5 minutes
   - **Request method:** GET
   - **Headers:** Add custom header: `x-cron-secret: YOUR_AUTOMATION_CRON_SECRET`

**Option B: EasyCron (Free tier — 1 job, 20-min minimum)**

Not ideal due to the 20-minute minimum interval on free tier.

**Option C: GitHub Actions (Free — but 5-min minimum)**

Create `.github/workflows/cron.yml`:

```yaml
name: Cron Jobs
on:
  schedule:
    - cron: '*/5 * * * *'  # Every 5 minutes (GitHub minimum)
jobs:
  trigger-crons:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger automations cron
        run: |
          curl -sf -H "x-cron-secret: ${{ secrets.AUTOMATION_CRON_SECRET }}" \
            https://wacrm-rgss.store/api/automations/cron
      - name: Trigger flows cron
        run: |
          curl -sf -H "x-cron-secret: ${{ secrets.AUTOMATION_CRON_SECRET }}" \
            https://wacrm-rgss.store/api/flows/cron
```

> **Note:** GitHub Actions cron has a minimum interval of 5 minutes and can be delayed by up to 15 minutes during high load.

### 7.5 Verify Cron Jobs are Working

Test each endpoint manually (from any terminal or browser-based API tool):

```bash
# Test automations cron
curl -v -H "x-cron-secret: YOUR_AUTOMATION_CRON_SECRET" \
  https://wacrm-rgss.store/api/automations/cron

# Expected response: {"processed":0} (or a count of processed executions)

# Test flows cron
curl -v -H "x-cron-secret: YOUR_AUTOMATION_CRON_SECRET" \
  https://wacrm-rgss.store/api/flows/cron

# Expected response: {"swept":0} (or a count of swept sessions)
```

**Verify unauthorized access is blocked:**
```bash
# Without the secret header — should return 401
curl -v https://wacrm-rgss.store/api/automations/cron
# Expected: 401 Unauthorized

# With wrong secret — should return 401
curl -v -H "x-cron-secret: wrong-secret" https://wacrm-rgss.store/api/automations/cron
# Expected: 401 Unauthorized
```

### 7.6 Monitor Cron Execution

- **Railway built-in cron:** Check the cron service's "Deployments" or "Logs" tab for execution history
- **cron-job.org:** Dashboard shows success/failure history with response codes
- **GitHub Actions:** Check the Actions tab in your repository for execution logs

**Done when:** Both cron endpoints respond with `200` when called with the correct secret, return `401` without it, and your chosen cron service is triggering them on schedule.

---


## 8. WhatsApp Business API Setup

### 8.1 Prerequisites

Before starting:
- A Facebook account (personal is fine — it's used for developer access)
- A phone number for WhatsApp Business (can't be the same number as a regular WhatsApp account)
- Your business is legitimate (Meta verifies this)

### 8.2 Create a Meta Developer Account

1. Go to **[developers.facebook.com](https://developers.facebook.com)**
2. Click **"Get Started"** (top-right)
3. If prompted, accept the Meta Platform Terms
4. Complete the developer registration:
   - Verify your email
   - Choose "Developer" role
   - Agree to terms

### 8.3 Create a Meta App

1. In the Meta Developer dashboard, click **"Create App"** (green button)
2. Select app type: **"Business"**
3. Fill in details:
   | Field | Value |
   |-------|-------|
   | **App name** | `Royal Glow Salon CRM` |
   | **App contact email** | Your business email |
   | **Business portfolio** | Create new or select existing |
4. Click **"Create App"**

### 8.4 Add WhatsApp Product

1. In your new app's dashboard, scroll down to **"Add Products"**
2. Find **"WhatsApp"** and click **"Set Up"**
3. This adds the WhatsApp section to your app's left sidebar

### 8.5 Configure a WhatsApp Business Phone Number

**Option A: Use the Test Number (for testing only)**

Meta provides a test phone number that you can use immediately:
- Go to WhatsApp → **"API Setup"** in your app
- You'll see a test phone number ID and a temporary token
- The test token expires in 24 hours — not suitable for production

**Option B: Add Your Own Business Number (for production)**

1. Go to WhatsApp → **"API Setup"** → **"Add phone number"**
2. Enter your business phone number (with country code, e.g., +63 for Philippines)
3. Verify via SMS or voice call
4. Choose a display name (this is what customers see — e.g., "Royal Glow Salon")
5. Wait for Meta to approve the display name (usually instant for simple names)

### 8.6 Get Your WhatsApp Credentials

In the WhatsApp API Setup page, note these values:

| Credential | Where to find it | Example |
|------------|-----------------|---------|
| **Phone Number ID** | WhatsApp → API Setup → "Phone number ID" shown under your number | `123456789012345` |
| **WhatsApp Business Account ID (WABA ID)** | WhatsApp → API Setup → "WhatsApp Business Account ID" | `987654321098765` |
| **Meta App Secret** | App Settings → Basic → "App Secret" (click "Show") | `a1b2c3d4e5f6...` |

### 8.7 Create a Permanent Access Token (System User)

The temporary token from API Setup expires in 24 hours. For production, create a permanent System User token:

1. Go to **[business.facebook.com](https://business.facebook.com)** (Meta Business Suite)
2. Click the gear icon → **"Business Settings"**
3. In the left sidebar: **"Users"** → **"System Users"**
4. Click **"Add"** button:
   | Field | Value |
   |-------|-------|
   | **System user name** | `wacrm-api` |
   | **Role** | Admin |
5. Click **"Create system user"**

6. Now assign your WhatsApp assets to this system user:
   - Click on the `wacrm-api` system user
   - Click **"Add Assets"**
   - Go to **"Apps"** tab → select your app (`Royal Glow Salon CRM`)
   - Toggle **"Full control"** → **"Save changes"**

7. Generate the token:
   - Click **"Generate New Token"**
   - Select your app (`Royal Glow Salon CRM`)
   - Set token expiration to **"Never"** (permanent)
   - Select permissions:
     - ✅ `whatsapp_business_management`
     - ✅ `whatsapp_business_messaging`
   - Click **"Generate Token"**
   - **COPY THIS TOKEN IMMEDIATELY** — you won't see it again!
   - Save it securely (password manager recommended)

### 8.8 Configure the Webhook in Meta Developer Dashboard

1. Go to your app → **WhatsApp** → **"Configuration"** (left sidebar)
2. Under **"Webhook"**, click **"Edit"**
3. Configure:

| Field | Value |
|-------|-------|
| **Callback URL** | `https://wacrm-rgss.store/api/whatsapp/webhook` |
| **Verify token** | Any string you choose (e.g., `royal-glow-verify-2026`) |

4. Click **"Verify and Save"**
   - Meta will send a GET request to your callback URL with the verify token
   - Your app must respond with the `hub.challenge` value
   - If your Railway deployment is running correctly, this will succeed automatically

5. After verification, **subscribe to webhook fields**:
   - Click **"Manage"** next to the webhook
   - Subscribe to: **`messages`** (toggle it on)
   - This ensures you receive all inbound messages, status updates, and errors

### 8.9 Verify Webhook is Receiving Events

1. In Meta Developer Dashboard → WhatsApp → Configuration
2. You should see a green "Active" status next to the webhook URL
3. Meta shows delivery statistics — you want to see successful deliveries (2xx responses)

**Done when:** You have a permanent System User token, webhook is verified (green/active status in Meta dashboard), and `messages` field is subscribed.

---


## 9. Connecting WhatsApp in the CRM

### 9.1 Create Your Staff Account

1. Open `https://wacrm-rgss.store` in your browser
2. Click **"Sign Up"** (or "Create Account")
3. Enter:
   - Email: Your business email
   - Password: A strong password
4. Check your email for a verification link
5. Click the verification link → you'll be logged in

### 9.2 Navigate to WhatsApp Settings

1. After logging in, you'll land on the Dashboard
2. Click **"Settings"** in the left sidebar (gear icon)
3. Look for the **"WhatsApp"** section or tab

### 9.3 Enter Your WhatsApp Configuration

Fill in the following fields:

| Field | Value | Where you got it |
|-------|-------|-----------------|
| **Phone Number ID** | `123456789012345` | Meta App → WhatsApp → API Setup |
| **Access Token** | `EAA...your-permanent-token` | System User token from Section 8.7 |
| **Verify Token** | `royal-glow-verify-2026` | The same string you entered in Meta webhook config (Section 8.8) |

### 9.4 Save and Verify Connection

1. Click **"Save"** or **"Connect"**
2. The CRM will:
   - Encrypt your access token using the `ENCRYPTION_KEY`
   - Store the encrypted token in Supabase
   - Test the connection by calling the WhatsApp Business API
3. Status should change to **"Connected"** (green indicator)

### 9.5 Send a Test Message

1. Go to **"Contacts"** in the left sidebar
2. Create a test contact (your personal WhatsApp number)
3. Click on the contact → open conversation
4. Type a message and send
5. Check your personal WhatsApp — you should receive the message

### 9.6 Receive a Test Message

1. From your personal WhatsApp, send a message to your business number
2. Go to **"Inbox"** in the CRM
3. The message should appear in real-time (no page refresh needed)
4. The contact should be auto-created if it didn't exist

**Done when:** You can send messages FROM the CRM and receive messages IN the CRM's inbox in real-time.

---


## 10. Post-Deployment Verification Checklist

Run through this complete checklist after deployment. Check each item:

### Application Health

- [ ] `https://wacrm-rgss.store` loads without errors
- [ ] SSL certificate is valid (padlock icon in browser)
- [ ] No console errors in browser DevTools (F12 → Console)
- [ ] Railway deployment shows green "Active" status
- [ ] Railway metrics show normal CPU/RAM usage

### Authentication

- [ ] Can create a new account (sign up)
- [ ] Verification email is received
- [ ] Can log in after verification
- [ ] Can log out
- [ ] Can reset password via "Forgot Password"
- [ ] Dashboard loads after login (no blank screen)

### WhatsApp Connection

- [ ] Settings → WhatsApp shows "Connected" status
- [ ] Can send a message from CRM inbox to a test phone number
- [ ] Message is received on the test phone's WhatsApp
- [ ] Can receive a message in the CRM inbox (send from test phone)
- [ ] Message appears in real-time (no page refresh)
- [ ] New contact is auto-created from inbound message
- [ ] Media messages (images, voice notes) are handled

### Core Features

- [ ] **Inbox:** Conversations load, can reply, real-time updates
- [ ] **Contacts:** Can create, edit, delete, search contacts
- [ ] **Pipelines:** Can create stages, add deals, drag between stages
- [ ] **Broadcasts:** Can create a broadcast, select recipients, send
- [ ] **Automations:** Can create a simple automation (e.g., auto-reply)
- [ ] **Flows:** Can create a conversational flow

### Cron Jobs

- [ ] `/api/automations/cron` responds 200 with correct secret
- [ ] `/api/automations/cron` responds 401 without secret
- [ ] `/api/flows/cron` responds 200 with correct secret
- [ ] `/api/flows/cron` responds 401 without secret
- [ ] Cron service (Railway/cron-job.org) shows successful executions

### Infrastructure

- [ ] Auto-deploy works: push a trivial commit → Railway rebuilds
- [ ] Railway logs show no errors (check Deployments → Logs)
- [ ] Custom domain resolves from multiple locations (use dnschecker.org)
- [ ] Supabase dashboard shows active connections
- [ ] Webhook deliveries show 200 status in Meta Developer Dashboard

### Performance Sanity Check

- [ ] Login page loads in < 3 seconds
- [ ] Inbox loads in < 2 seconds
- [ ] Sending a message responds in < 1 second
- [ ] Real-time updates arrive within 1-2 seconds

**Done when:** All checkboxes above are checked. Your CRM is production-ready.

---


## 11. Railway-Specific Considerations

### 11.1 Sleep Behavior (Important!)

**Hobby Plan: No Sleep** — Your app runs 24/7.

However, be aware of these Railway behaviors:

| Plan | Sleep behavior | Impact on wacrm |
|------|---------------|----------------|
| **Trial** | Sleeps after 30 min of no HTTP requests | Webhooks would be MISSED while sleeping |
| **Hobby ($5)** | Never sleeps | Webhooks always received — correct behavior |
| **Pro ($20)** | Never sleeps | Same as Hobby but with team features |

> **Critical:** The Trial plan is NOT suitable for wacrm because WhatsApp webhooks arrive at any time. If your app is sleeping, Meta will retry a few times and then stop — you'll lose messages.

### 11.2 Deployment Behavior

- **Zero-downtime deploys:** Railway runs the new container alongside the old one, then switches traffic. No messages are lost during deploys.
- **Build time:** Typically 1-3 minutes for Next.js apps
- **Deploy frequency:** No limit on deploys — push as often as you want
- **Rollback:** Click any previous deployment → "Redeploy" to roll back instantly

### 11.3 Usage Monitoring

Keep an eye on your Railway usage to stay within the $5 credit:

1. Click your avatar → **"Account Settings"** → **"Usage"**
2. Key metrics to watch:

| Resource | Approximate usage for wacrm | Included in $5 |
|----------|---------------------------|----------------|
| **Compute (vCPU-hours)** | ~0.1-0.2 vCPU avg | Well within limits |
| **Memory (GB-hours)** | ~256-512 MB avg | Well within limits |
| **Network egress** | < 5 GB/month typically | 100 GB included |
| **Build minutes** | ~2-3 min per deploy | Included |

3. Railway sends email alerts at 75% and 100% of your credit usage

### 11.4 When to Upgrade to Pro ($20/month)

Consider upgrading from Hobby to Pro when:

| Trigger | Why upgrade |
|---------|------------|
| Need team member access | Hobby is single-user; Pro supports teams |
| Exceeding $5 credit consistently | If bills exceed $8-10/month regularly |
| Need priority support | Pro includes priority Discord/email support |
| Running multiple services | More services = more compute usage |

For Royal Glow Salon with 2-3 staff and ~500 conversations/month, the Hobby plan should be sufficient for 1-2+ years.

### 11.5 Region Selection

Railway offers multiple regions. Choose based on your customer location:

| Your customers | Best Railway region |
|----------------|-------------------|
| Philippines / Southeast Asia | `us-west1` or `asia-southeast1` (if available) |
| India / South Asia | `asia-south1` (if available) |
| US / Americas | `us-west1` or `us-east4` |
| Europe | `europe-west4` |

> **Tip:** Also align your Supabase region with your Railway region for lowest latency between the app and database.

### 11.6 Persistent Storage

Railway services are **ephemeral** — the filesystem is wiped on each deploy. This is fine for wacrm because:
- All data is in Supabase (Postgres + Storage)
- No local file storage is needed
- Media files are stored in Supabase Storage buckets
- Session state is in the database, not on disk

### 11.7 Environment Variable Updates

When you update environment variables in Railway:
- Railway automatically triggers a **new deployment**
- The new deployment picks up the new values
- There's a brief period (~30 seconds) during the switchover
- No messages are lost (zero-downtime deploy)

### 11.8 Logs and Debugging

Access logs in Railway:
1. Click your service → **"Deployments"** tab
2. Click the active (green) deployment
3. Click **"View Logs"**
4. Logs stream in real-time — you'll see:
   - Application stdout/stderr
   - HTTP request logs
   - Next.js build output (during builds)
   - Any `console.log` / `console.error` in your code

**Log retention:** Railway keeps logs for the lifetime of a deployment. Old deployment logs are still accessible.

**Done when:** You understand Railway's behaviors and have verified your app runs 24/7 on the Hobby plan without sleeping.

---


## 12. Cost Breakdown

### Monthly Recurring Costs

| Service | Plan | Monthly Cost | What You Get |
|---------|------|-------------|--------------|
| Railway.app | Hobby | **$5.00** | Next.js app running 24/7, auto-deploy, custom domain, SSL, cron |
| Supabase | Free | **$0.00** | 500 MB Postgres, unlimited auth, 1 GB storage, 2 GB bandwidth, realtime |
| Domain (wacrm-rgss.store) | Annual | **~$1.00** | $10-12/year ÷ 12 months |
| Meta WhatsApp API | Free tier | **$0.00** | 1000 service conversations/month free |
| Cron service (cron-job.org) | Free | **$0.00** | Unlimited cron jobs, 1-min intervals |
| **TOTAL** | | **~$6/month** | Full production CRM |

### Variable Costs (WhatsApp Conversations)

WhatsApp charges per "conversation" (24-hour window), not per message:

| Conversation Category | First 1000/month | After 1000/month | When triggered |
|----------------------|------------------|-----------------|----------------|
| **Service** | FREE | ~$0.02/each | Customer messages you first |
| **Marketing** | ~$0.04-0.08/each | ~$0.04-0.08/each | Broadcast promotions |
| **Utility** | ~$0.02-0.04/each | ~$0.02-0.04/each | Appointment confirmations |
| **Authentication** | ~$0.02-0.04/each | ~$0.02-0.04/each | OTP/verification |

> Note: Exact pricing varies by country. See [Meta's WhatsApp pricing page](https://developers.facebook.com/docs/whatsapp/pricing) for current rates.

### Realistic Monthly Cost for Royal Glow Salon

Scenario: Salon handles ~200-400 customer conversations/month + occasional broadcasts:

| Item | Estimated Cost |
|------|---------------|
| Railway Hobby plan | $5.00 |
| Domain (amortized) | $1.00 |
| WhatsApp service conversations (200-400, within free 1000) | $0.00 |
| WhatsApp broadcast to 50-100 customers (marketing) | $2.00-$4.00 |
| **TOTAL** | **$6-10/month** |

### When Costs Could Increase

| Scenario | Additional cost | Solution |
|----------|----------------|----------|
| Database exceeds 500 MB | +$25/month (Supabase Pro) | Unlikely for 2-3 years with a small salon |
| App usage exceeds $5 Railway credit | +$1-5/month | Only if you add heavy features |
| 1000+ conversations/month | +$10-30/month | Growing business = more revenue anyway |
| Need team features on Railway | +$15/month (Pro plan) | Only if you need multi-user Railway access |

### Annual Cost Summary

| Plan | Monthly | Annual |
|------|---------|--------|
| **Current (just starting)** | ~$6 | ~$72/year |
| **Moderate usage** (500 convos + broadcasts) | ~$10 | ~$120/year |
| **Heavy usage** (1000+ convos, frequent broadcasts) | ~$25 | ~$300/year |

Compare this to commercial WhatsApp CRM solutions:
- Respond.io: $99-299/month
- WATI: $49-99/month
- Interakt: $59-149/month

**Your wacrm-rgss saves $500-3000+ per year vs. commercial alternatives.**

**Done when:** You understand the cost structure and are comfortable with the ~$6/month baseline cost.

---


## 13. Comparison: Railway vs. Oracle Cloud + Coolify

A detailed side-by-side for your decision:

| Aspect | Railway.app ($5/mo) | Oracle Cloud + Coolify ($0/mo) |
|--------|--------------------|---------------------------------|
| **Setup time** | 15-20 minutes | 60-90 minutes |
| **ARM capacity** | N/A (always available) | Often unavailable (days/weeks wait) |
| **Server maintenance** | Zero (fully managed) | You manage: OS updates, Docker, SSL, firewall |
| **Deploy method** | Git push → auto-deploy | Git push → Coolify auto-deploy |
| **Custom domain** | 1-click in dashboard | Manual DNS + Coolify config |
| **SSL certificates** | Automatic (zero config) | Automatic via Coolify/Caddy |
| **Cron jobs** | Built-in feature | Manual crontab via SSH |
| **Logs** | Dashboard (click to view) | SSH → docker logs |
| **Scaling** | Instant (slider in UI) | Fixed (but generous: 4 CPU/24 GB) |
| **Uptime SLA** | 99.9% | No SLA (free tier) |
| **Cold starts** | None (always running on Hobby) | None (always running) |
| **Monthly cost** | $5 | $0 |
| **Annual cost** | $60 | $0 |
| **Resources** | Metered (0.5 vCPU/512 MB typical) | Fixed (4 OCPU/24 GB RAM) |
| **Backup** | Supabase handles DB backups | Supabase + optional VPS snapshot |
| **Recovery from failure** | Railway restarts automatically | Manual SSH intervention possible |
| **Learning curve** | Almost zero | Moderate (Linux, Docker, networking) |
| **Risk of account closure** | Low (paid account) | Low (but Oracle has been known to close idle accounts) |
| **Support** | Community + paid options | Community only |
| **Multi-region** | Yes (select region) | Fixed to your VPS region |
| **Git integration** | Native (first-class GitHub) | Via Coolify GitHub App |

### Decision Matrix

Choose **Railway** if:
- ✅ Oracle ARM capacity is unavailable
- ✅ You want zero maintenance
- ✅ $5/month is acceptable
- ✅ You value simplicity and reliability
- ✅ You don't want to learn Linux/Docker/SSH

Choose **Oracle Cloud + Coolify** if:
- ✅ ARM capacity IS available in your region
- ✅ You absolutely need $0/month cost
- ✅ You enjoy managing servers
- ✅ You want 4 CPU + 24 GB RAM (massive overkill, but free)
- ✅ You plan to run additional services on the same VPS

### Migration Path

**From Railway → Oracle (later, when ARM capacity opens up):**
1. Set up Oracle VPS + Coolify (see `DEPLOYMENT-ORACLE-COOLIFY.md`)
2. Point your domain (`wacrm-rgss.store`) to the new VPS IP
3. Copy environment variables from Railway to Coolify
4. Delete the Railway project (stop billing)

**From Oracle → Railway (if VPS has issues):**
1. Create Railway project from same GitHub repo
2. Copy environment variables
3. Point domain to Railway
4. Done in 15 minutes

Both deployments use the **same Supabase project** — the database doesn't move. Switching hosting is just changing where the Next.js app runs.

**Done when:** You've made your platform decision and understand the tradeoffs.

---


## 14. Troubleshooting Common Railway Issues

### Build Failures

| Error | Cause | Fix |
|-------|-------|-----|
| `npm ERR! Could not resolve dependency` | npm version mismatch or lock file issue | Delete `package-lock.json`, run `npm install` locally, commit the new lock file |
| `error TS2xxx: Type error` | TypeScript compilation error | Fix the type error in your code and push again |
| `Error: Cannot find module 'xyz'` | Missing dependency | Run `npm install xyz` locally and commit updated `package.json` + `package-lock.json` |
| `ENOMEM: not enough memory` | Build uses too much RAM | In Railway Settings, increase build memory limit or add `NODE_OPTIONS=--max-old-space-size=2048` env var |
| `Nixpacks failed to detect` | Incorrect project structure | Ensure `package.json` is in the root directory, or set correct "Root directory" in Settings |
| Build timeout (>15 min) | Large dependencies or slow network | Railway auto-caches `node_modules` — second builds are much faster |

### Runtime Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `502 Bad Gateway` | App crashed or hasn't started yet | Check deployment logs — look for the error. Common: missing env var |
| `503 Service Unavailable` | Deploy in progress | Wait 1-2 minutes for deploy to complete |
| App starts then crashes immediately | Missing or invalid environment variable | Check logs for the specific error (e.g., "SUPABASE_SERVICE_ROLE_KEY is required") |
| `ECONNREFUSED` to Supabase | Wrong Supabase URL or network issue | Verify `NEXT_PUBLIC_SUPABASE_URL` is correct and Supabase project is active |
| Memory usage growing over time | Memory leak in application | Railway auto-restarts if OOM — check for large arrays or unbounded caches in code |
| `429 Too Many Requests` | Rate limiting (WhatsApp API or Supabase) | Add retry logic or reduce broadcast batch sizes |

### Domain/SSL Issues

| Error | Cause | Fix |
|-------|-------|-----|
| "DNS not verified" in Railway | DNS records not propagated yet | Wait 5-30 minutes, verify with `dig wacrm-rgss.store` |
| SSL certificate not issued | DNS still pointing elsewhere | Ensure CNAME points to `<service>.up.railway.app` |
| `ERR_SSL_VERSION_OR_CIPHER_MISMATCH` | Certificate still provisioning | Wait 1-2 minutes after DNS verification |
| Mixed content warnings | Hardcoded `http://` URLs in app | Ensure `NEXT_PUBLIC_SITE_URL` uses `https://` |
| Domain works but shows Railway's default page | Multiple services on same domain | Ensure only one service has the domain configured |

### Webhook Issues

| Error | Cause | Fix |
|-------|-------|-----|
| Meta webhook verification fails | App not running or wrong URL | Verify: `curl https://wacrm-rgss.store/api/whatsapp/webhook?hub.mode=subscribe&hub.challenge=test&hub.verify_token=YOUR_TOKEN` returns `test` |
| Webhook returns 401 | `META_APP_SECRET` env var is wrong | Verify it matches your app's secret in Meta Developer Dashboard |
| Webhook returns 500 | Application error processing the webhook | Check Railway logs for the full error stack trace |
| Messages received but not appearing in inbox | Supabase Realtime not enabled | Enable realtime for `messages` and `conversations` tables |
| Duplicate messages in inbox | Webhook being called multiple times | Check for multiple webhook subscriptions in Meta dashboard |

### Cron Job Issues

| Error | Cause | Fix |
|-------|-------|-----|
| Cron returns 401 | Wrong secret in header | Verify `x-cron-secret` header matches `AUTOMATION_CRON_SECRET` env var exactly |
| Cron returns 500 | Application error in cron handler | Check Railway logs at the time the cron fires |
| Automations not advancing | Cron not firing | Verify your cron service is active (check cron-job.org dashboard or Railway cron logs) |
| `{"error":"cron not configured"}` | `AUTOMATION_CRON_SECRET` env var missing | Add it to Railway variables and redeploy |

### Performance Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| Slow initial page load | Cold function compilation | Normal on first request after deploy; subsequent requests are fast |
| Inbox feels laggy | Too many conversations loaded | This is a UX issue — implement pagination (code change) |
| High memory usage | Large conversation history loaded | Railway handles this — monitor usage dashboard |

### General Debugging Steps

1. **Check Railway Logs:** Service → Deployments → Active deployment → "View Logs"
2. **Check Build Logs:** Service → Deployments → Latest → "Build Logs"
3. **Verify Environment Variables:** Service → Variables → confirm all 7 are set
4. **Test Endpoints Manually:**
   ```bash
   # Health check (any page)
   curl -I https://wacrm-rgss.store

   # Webhook endpoint
   curl "https://wacrm-rgss.store/api/whatsapp/webhook?hub.mode=subscribe&hub.challenge=test&hub.verify_token=YOUR_VERIFY_TOKEN"

   # Cron endpoint
   curl -H "x-cron-secret: YOUR_SECRET" https://wacrm-rgss.store/api/automations/cron
   ```
5. **Redeploy:** Sometimes a fresh deploy fixes transient issues — click "Redeploy" on the latest deployment

**Done when:** You can diagnose and fix common issues without external help.

---


## 15. Security Considerations

### 15.1 What's Already Secured (by the application code)

| Security measure | How it works |
|-----------------|-------------|
| **Row Level Security (RLS)** | Every Supabase table has RLS policies — users can only access their own organization's data |
| **Webhook HMAC verification** | Every inbound WhatsApp webhook is verified against `META_APP_SECRET` using HMAC-SHA256 |
| **Token encryption at rest** | WhatsApp access tokens are encrypted with AES-256-GCM before storing in database |
| **Cron endpoint protection** | `/api/automations/cron` and `/api/flows/cron` require `x-cron-secret` header |
| **Security headers** | CSP, HSTS, X-Frame-Options, X-Content-Type-Options configured in `next.config.ts` |
| **Input validation** | API routes validate request bodies before processing |
| **No SQL injection** | Supabase client uses parameterized queries |
| **Auth via Supabase** | Industry-standard JWT-based authentication with secure session management |

### 15.2 Railway-Specific Security

| Concern | Status |
|---------|--------|
| **HTTPS enforced** | Railway automatically redirects HTTP → HTTPS |
| **Environment variables** | Encrypted at rest in Railway's infrastructure |
| **Network isolation** | Your service is isolated in its own container |
| **No SSH access** | Railway doesn't expose SSH — reduces attack surface |
| **Build isolation** | Each build runs in an isolated environment |
| **Secrets in logs** | Railway automatically redacts known secret patterns from logs |

### 15.3 Things You Should Do

#### Keep Environment Variables Secure

- Never commit `.env` or `.env.local` to git (`.gitignore` already handles this)
- Never share your `SUPABASE_SERVICE_ROLE_KEY` or `META_APP_SECRET` publicly
- Store a backup of all secrets in a password manager (1Password, Bitwarden, etc.)
- Rotate `ENCRYPTION_KEY` only if you suspect compromise (it will invalidate all stored tokens)

#### Supabase Security Settings

1. **Disable email confirmations for testing, enable for production:**
   - Supabase → Authentication → Settings → "Enable email confirmations" → **ON**
   
2. **Set password requirements:**
   - Supabase → Authentication → Settings → Minimum password length: **8+**

3. **Review RLS policies periodically:**
   - Supabase → Database → Tables → click any table → "Policies" tab
   - Ensure policies are correct (the migrations set them up properly)

4. **Monitor auth activity:**
   - Supabase → Authentication → Users → check for suspicious signups

#### Meta App Security

1. **Enable two-factor authentication** on your Meta Developer account
2. **Restrict the System User token** to only the permissions it needs:
   - `whatsapp_business_management`
   - `whatsapp_business_messaging`
   - Nothing else
3. **Review app roles** — remove any test users or developers who no longer need access
4. **Set app mode to Live** when ready for production (removes rate limits)

#### Domain Security

1. **Enable DNSSEC** at your registrar (if supported) — prevents DNS spoofing
2. **Lock domain transfer** — prevent unauthorized domain transfers
3. **Use a registrar with 2FA** — protect your domain account

### 15.4 Incident Response Plan

If you suspect a security breach:

1. **Immediately:**
   - Rotate `ENCRYPTION_KEY` in Railway (users will need to re-enter WhatsApp credentials)
   - Rotate `AUTOMATION_CRON_SECRET`
   - Rotate `SUPABASE_SERVICE_ROLE_KEY` (regenerate in Supabase dashboard → Project Settings → API)
   - Rotate `META_APP_SECRET` (regenerate in Meta Developer Dashboard → App Settings)

2. **Within 1 hour:**
   - Check Supabase → Authentication → Users for unauthorized accounts
   - Review Railway deployment logs for suspicious activity
   - Check Meta Developer Dashboard for unusual API calls
   - Review Supabase → Database → SQL Editor → run: `SELECT * FROM auth.users ORDER BY created_at DESC LIMIT 20;`

3. **Within 24 hours:**
   - Change passwords on all connected accounts (Supabase, Railway, Meta, domain registrar)
   - Enable 2FA everywhere if not already done
   - Review and update RLS policies if data was accessed

### 15.5 Security Checklist

- [ ] All 7 environment variables are set (none empty or placeholder)
- [ ] `ENCRYPTION_KEY` is exactly 64 hex characters (32 bytes)
- [ ] `SUPABASE_SERVICE_ROLE_KEY` is NOT exposed in any client-side code
- [ ] `META_APP_SECRET` matches the value in Meta Developer Dashboard
- [ ] HTTPS works (padlock in browser)
- [ ] HTTP redirects to HTTPS
- [ ] Webhook HMAC verification is working (test with wrong signature → 401)
- [ ] Cron endpoints reject requests without correct header
- [ ] No test/default passwords are in use
- [ ] 2FA enabled on: GitHub, Meta Developer, Supabase, Railway, domain registrar
- [ ] `.env.local` and `.env` are in `.gitignore`
- [ ] System User token has minimal permissions (only WhatsApp-related)
- [ ] Supabase email confirmation is enabled

**Done when:** All security checklist items are verified and your incident response plan is documented.

---


## Quick-Start Summary (TL;DR)

For those who just want the fastest path to production:

```
1. Railway.app → Sign up → Hobby plan ($5/mo) → New Project → Deploy from GitHub
2. Connect repo: royalglowsalonspa/wacrm-rgss (branch: main)
3. Supabase.com → New Project → Run 13 migrations in SQL Editor
4. Railway Variables → Set all 7 env vars → Save (triggers redeploy)
5. Railway Settings → Add custom domain: wacrm-rgss.store
6. DNS → CNAME @ → <service>.up.railway.app
7. Meta Developer → WhatsApp → Webhook URL: https://wacrm-rgss.store/api/whatsapp/webhook
8. cron-job.org → Create 2 jobs hitting /api/automations/cron and /api/flows/cron
9. Log into CRM → Settings → WhatsApp → Connect
10. Send a test message → verify real-time inbox works
```

**Total time: ~30-45 minutes** (most of that is Meta Developer setup).

---

## Timeline Summary

| Phase | Time | Difficulty |
|-------|------|-----------|
| 1. Decide on Railway | 5 min | Easy (read this doc) |
| 2. Railway account + Hobby plan | 5 min | Easy (credit card required) |
| 3. Deploy from GitHub | 3 min | Easy (click + connect) |
| 4. Environment variables | 10 min | Easy (copy-paste) |
| 5. Supabase setup + migrations | 15 min | Easy (copy-paste SQL) |
| 6. Custom domain + SSL | 5 min | Easy (DNS record + wait) |
| 7. Cron jobs | 5 min | Easy (cron-job.org setup) |
| 8. WhatsApp Business API | 20 min | Medium (Meta's dashboard is complex) |
| 9. Connect WhatsApp in CRM | 3 min | Easy (fill form + save) |
| 10. Verification | 10 min | Easy (run through checklist) |
| **Total** | **~80 min** | |

---

## Maintenance Playbook

### Weekly (5 minutes)
- Check Railway usage dashboard (are you within $5 credit?)
- Glance at Supabase dashboard (database size, any errors?)

### Monthly (10 minutes)
- Review Meta Developer Dashboard for webhook delivery stats
- Check Supabase storage usage
- Review Railway billing (any unexpected charges?)

### When Things Break
1. Check Railway deployment logs (90% of issues are visible here)
2. Verify environment variables haven't been accidentally deleted
3. Check Supabase project is still running (not paused due to inactivity)
4. Test webhook manually with curl
5. If all else fails: redeploy from Railway dashboard

### Supabase Free Tier Pausing

> **Important:** Supabase pauses free-tier projects after **1 week of inactivity** (no API requests). Since your Railway app makes API calls on every webhook and cron job, this shouldn't happen. But if you take the app offline for an extended period, you'll need to "unpause" the Supabase project in the dashboard.

---

## Related Documentation

- **Oracle Cloud + Coolify deployment:** See [`DEPLOYMENT-ORACLE-COOLIFY.md`](./DEPLOYMENT-ORACLE-COOLIFY.md)
- **Environment variables reference:** See [`.env.local.example`](./.env.local.example)
- **Database migrations:** See [`supabase/migrations/`](./supabase/migrations/)
- **Application README:** See [`README.md`](./README.md)

---

*Last updated: May 2026*
*For Royal Glow Salon & Spa — wacrm-rgss.store*
