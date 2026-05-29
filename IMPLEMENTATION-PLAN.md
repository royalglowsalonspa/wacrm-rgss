# Implementation Plan: wacrm-rgss

> **Project:** WhatsApp CRM for Royal Glow Salon & Spa
> **Repository:** [github.com/royalglowsalonspa/wacrm-rgss](https://github.com/royalglowsalonspa/wacrm-rgss)
> **Stack:** Oracle Cloud (Always Free) + Coolify + Supabase + Meta WhatsApp API
> **Timeline:** 3 weeks (Part-time, ~2-3 hours/day)
> **Team:** 1 person setting up + 2 end-users (Manager & Receptionist)

---

## Overview

This plan breaks the full deployment into **3 phases across 3 weeks**:

| Phase | Week | Focus | Outcome |
|-------|------|-------|---------|
| **Phase 1** | Week 1 | Infrastructure & Database | Server running, database ready |
| **Phase 2** | Week 2 | App Deployment & WhatsApp | CRM live, WhatsApp connected |
| **Phase 3** | Week 3 | Configuration & Go-Live | Team trained, automations active |

---

## Phase 1: Infrastructure & Database (Week 1)

### Day 1-2: Oracle Cloud VPS

| # | Task | Time | Done |
|---|------|------|------|
| 1.1 | Sign up at [cloud.oracle.com/free](https://cloud.oracle.com/free) | 10 min | [ ] |
| 1.2 | Verify identity (credit card — no charges) | 5 min | [ ] |
| 1.3 | Create ARM Compute Instance (Ampere A1, 2 OCPU, 12GB RAM, Ubuntu 22.04) | 10 min | [ ] |
| 1.4 | Note down the **Public IP Address** | 1 min | [ ] |
| 1.5 | Configure Security List: open ports 22, 80, 443, 8000 | 5 min | [ ] |
| 1.6 | SSH into instance and verify access | 5 min | [ ] |
| 1.7 | Open iptables firewall (ports 80, 443, 8000) | 5 min | [ ] |
| 1.8 | Run `sudo apt update && sudo apt upgrade -y && sudo reboot` | 10 min | [ ] |

**Checkpoint:** You can SSH into your Oracle VPS ✅

---

### Day 2-3: Install Coolify

| # | Task | Time | Done |
|---|------|------|------|
| 2.1 | SSH in and run: `curl -fsSL https://cdn.coollabs.io/coolify/install.sh \| sudo bash` | 5 min | [ ] |
| 2.2 | Wait for installation to complete | 3 min | [ ] |
| 2.3 | Open `http://<YOUR-IP>:8000` in browser | 1 min | [ ] |
| 2.4 | Create Coolify admin account (email + strong password) | 2 min | [ ] |
| 2.5 | Complete initial setup wizard | 3 min | [ ] |
| 2.6 | Go to **Sources → Add → GitHub App** | 5 min | [ ] |
| 2.7 | Connect your GitHub account and grant access to `royalglowsalonspa/wacrm-rgss` | 5 min | [ ] |

**Checkpoint:** Coolify dashboard is accessible and GitHub is connected ✅

---

### Day 3-4: Supabase Setup

| # | Task | Time | Done |
|---|------|------|------|
| 3.1 | Sign up at [supabase.com](https://supabase.com) | 5 min | [ ] |
| 3.2 | Create new project: name `wacrm-rgss`, choose region near your VPS | 3 min | [ ] |
| 3.3 | Wait for project to provision (~2 min) | 2 min | [ ] |
| 3.4 | Go to **SQL Editor** | 1 min | [ ] |
| 3.5 | Run migration `001_initial_schema.sql` — copy/paste from repo | 2 min | [ ] |
| 3.6 | Run migration `002_pipelines_enhancements.sql` | 1 min | [ ] |
| 3.7 | Run migration `003_broadcast_recipient_wamid.sql` | 1 min | [ ] |
| 3.8 | Run migration `004_contact_delete_set_null.sql` | 1 min | [ ] |
| 3.9 | Run migration `005_broadcast_counts_incremental.sql` | 1 min | [ ] |
| 3.10 | Run migration `006_automations.sql` | 1 min | [ ] |
| 3.11 | Run migration `007_automations_increment_counter.sql` | 1 min | [ ] |
| 3.12 | Run migration `008_profile_avatars_storage.sql` | 1 min | [ ] |
| 3.13 | Run migration `009_message_actions.sql` | 1 min | [ ] |
| 3.14 | Run migration `010_flows.sql` | 1 min | [ ] |
| 3.15 | Run migration `011_profile_beta_features.sql` | 1 min | [ ] |
| 3.16 | Run migration `012_flows_increment_counter.sql` | 1 min | [ ] |
| 3.17 | Run migration `013_whatsapp_config_phone_number_id_unique.sql` | 1 min | [ ] |
| 3.18 | Verify in **Database → Tables** that all tables exist | 2 min | [ ] |
| 3.19 | Verify in **Database → Replication** that `messages` and `conversations` have realtime enabled | 2 min | [ ] |
| 3.20 | Go to **Project Settings → API** and copy: Project URL, anon key, service_role key | 3 min | [ ] |

**Checkpoint:** All 13 migrations applied, credentials saved ✅

---

### Day 4-5: Generate Secrets & Domain Setup

| # | Task | Time | Done |
|---|------|------|------|
| 4.1 | Generate ENCRYPTION_KEY: `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"` | 1 min | [ ] |
| 4.2 | Generate AUTOMATION_CRON_SECRET: `openssl rand -hex 32` | 1 min | [ ] |
| 4.3 | Save both secrets in a password manager (1Password, Bitwarden, etc.) | 2 min | [ ] |
| 4.4 | **Option A:** Buy a domain (~$10/year from Cloudflare/Namecheap) | 10 min | [ ] |
| 4.5 | **Option B:** Get a free subdomain from [duckdns.org](https://duckdns.org) | 5 min | [ ] |
| 4.6 | Add DNS A record pointing to your Oracle VPS IP | 5 min | [ ] |
| 4.7 | Verify DNS propagation: `dig crm.yourdomain.com` (wait up to 10 min) | 10 min | [ ] |

**Checkpoint:** Domain resolves to your VPS IP, secrets generated and saved ✅

---

## Phase 2: App Deployment & WhatsApp (Week 2)

### Day 6-7: Deploy wacrm on Coolify

| # | Task | Time | Done |
|---|------|------|------|
| 5.1 | In Coolify → **Projects → Add → New Resource → Application** | 2 min | [ ] |
| 5.2 | Select GitHub source → pick `royalglowsalonspa/wacrm-rgss` | 2 min | [ ] |
| 5.3 | Set branch: `main` | 1 min | [ ] |
| 5.4 | Set Build Pack: **Nixpacks** | 1 min | [ ] |
| 5.5 | Configure port: `3000` | 1 min | [ ] |
| 5.6 | Add environment variables (all 7 — see below) | 5 min | [ ] |
| 5.7 | Set domain in Coolify → Settings → Domains | 2 min | [ ] |
| 5.8 | Click **Deploy** and wait for build (~3-5 min) | 5 min | [ ] |
| 5.9 | Verify: open `https://crm.yourdomain.com` → login page appears | 2 min | [ ] |
| 5.10 | Enable **Auto Deploy** in Settings → General | 1 min | [ ] |

**Environment variables to add in step 5.6:**
```
NEXT_PUBLIC_SUPABASE_URL = (from step 3.20)
NEXT_PUBLIC_SUPABASE_ANON_KEY = (from step 3.20)
SUPABASE_SERVICE_ROLE_KEY = (from step 3.20)
ENCRYPTION_KEY = (from step 4.1)
META_APP_SECRET = (from step 6.4 — add later)
NEXT_PUBLIC_SITE_URL = https://crm.yourdomain.com
AUTOMATION_CRON_SECRET = (from step 4.2)
```

**Checkpoint:** CRM loads at your domain with login page ✅

---

### Day 7-8: Create Staff Accounts

| # | Task | Time | Done |
|---|------|------|------|
| 5.11 | Open `https://crm.yourdomain.com/signup` | 1 min | [ ] |
| 5.12 | Create Manager account (your email + password) | 2 min | [ ] |
| 5.13 | Check email for verification link, click to verify | 2 min | [ ] |
| 5.14 | Log in → should redirect to `/dashboard` | 1 min | [ ] |
| 5.15 | Create Receptionist account (their email + password) | 2 min | [ ] |
| 5.16 | Verify receptionist email | 2 min | [ ] |

**Checkpoint:** Both staff can log in and see the dashboard ✅

---

### Day 8-10: WhatsApp Business API Setup

| # | Task | Time | Done |
|---|------|------|------|
| 6.1 | Go to [developers.facebook.com](https://developers.facebook.com) → Create App → Business type | 5 min | [ ] |
| 6.2 | Add **WhatsApp** product to your app | 2 min | [ ] |
| 6.3 | Note your **Phone Number ID** from WhatsApp → API Setup | 2 min | [ ] |
| 6.4 | Go to **App Settings → Basic** → copy **App Secret** (this is your META_APP_SECRET) | 2 min | [ ] |
| 6.5 | Update META_APP_SECRET in Coolify environment variables → redeploy | 3 min | [ ] |
| 6.6 | Go to [business.facebook.com](https://business.facebook.com) → Business Settings | 2 min | [ ] |
| 6.7 | Create System User → name: `wacrm-api` → role: Admin | 3 min | [ ] |
| 6.8 | Generate Token → select your app → permissions: `whatsapp_business_management`, `whatsapp_business_messaging` | 3 min | [ ] |
| 6.9 | Copy the permanent access token (save in password manager) | 2 min | [ ] |
| 6.10 | In Meta Dashboard → WhatsApp → Configuration → set Callback URL: `https://crm.yourdomain.com/api/whatsapp/webhook` | 3 min | [ ] |
| 6.11 | Set Verify Token to a random string (e.g., `rgss-verify-2026`) — remember it | 2 min | [ ] |
| 6.12 | Click **Verify and Save** — should succeed | 1 min | [ ] |
| 6.13 | Subscribe to webhook field: `messages` | 1 min | [ ] |
| 6.14 | In your CRM → **Settings → WhatsApp** → enter Phone Number ID, Access Token, Verify Token | 3 min | [ ] |
| 6.15 | Click **Connect** → status shows "Connected" | 1 min | [ ] |
| 6.16 | Send a test WhatsApp message to your business number | 1 min | [ ] |
| 6.17 | Verify message appears in **Inbox** in real-time | 1 min | [ ] |
| 6.18 | Reply from Inbox → verify customer receives the reply | 1 min | [ ] |

**Checkpoint:** WhatsApp connected, messages flowing both ways ✅

---

### Day 10: Cron Jobs Setup

| # | Task | Time | Done |
|---|------|------|------|
| 7.1 | SSH into your Oracle VPS | 1 min | [ ] |
| 7.2 | Create cron file: `sudo nano /etc/cron.d/wacrm-crons` | 1 min | [ ] |
| 7.3 | Paste cron config (see below) | 2 min | [ ] |
| 7.4 | Save and set permissions: `sudo chmod 644 /etc/cron.d/wacrm-crons` | 1 min | [ ] |
| 7.5 | Test automation cron: `curl -s -H "x-cron-secret: YOUR_SECRET" https://crm.yourdomain.com/api/automations/cron` | 1 min | [ ] |
| 7.6 | Test flows cron: `curl -s -H "x-cron-secret: YOUR_SECRET" https://crm.yourdomain.com/api/flows/cron` | 1 min | [ ] |
| 7.7 | Verify both return `{"processed":0}` or `{"swept":0}` | 1 min | [ ] |

**Cron file content for step 7.3:**
```cron
# wacrm-rgss automation wait-step processor (every 2 min)
*/2 * * * * root curl -sf -H "x-cron-secret: YOUR_AUTOMATION_CRON_SECRET" https://crm.yourdomain.com/api/automations/cron > /dev/null 2>&1

# wacrm-rgss flow timeout sweep (every 5 min)
*/5 * * * * root curl -sf -H "x-cron-secret: YOUR_AUTOMATION_CRON_SECRET" https://crm.yourdomain.com/api/flows/cron > /dev/null 2>&1
```

**Checkpoint:** Cron jobs running, automation Wait steps will process every 2 minutes ✅

---

## Phase 3: Configuration & Go-Live (Week 3)

### Day 11-12: Security Hardening

| # | Task | Time | Done |
|---|------|------|------|
| 8.1 | Lock Coolify dashboard to your IP only (iptables rule) | 3 min | [ ] |
| 8.2 | Disable password SSH: edit `/etc/ssh/sshd_config` → `PasswordAuthentication no` | 3 min | [ ] |
| 8.3 | Restart SSH: `sudo systemctl restart sshd` | 1 min | [ ] |
| 8.4 | Set server timezone: `sudo timedatectl set-timezone Asia/Manila` (or your timezone) | 1 min | [ ] |
| 8.5 | Enable auto-updates: `sudo apt install unattended-upgrades -y` | 2 min | [ ] |
| 8.6 | Configure Supabase Auth → URL Configuration → set Site URL and Redirect URLs | 3 min | [ ] |
| 8.7 | Sign up for [UptimeRobot](https://uptimerobot.com) (free) → add monitor for your CRM URL | 5 min | [ ] |

**Checkpoint:** Server hardened, monitoring active ✅

---

### Day 12-13: Create Message Templates (Meta Approval Required)

| # | Task | Time | Done |
|---|------|------|------|
| 9.1 | In CRM → **Settings → Templates** → Create "welcome_message" | 5 min | [ ] |
| 9.2 | Create "appointment_reminder" template | 5 min | [ ] |
| 9.3 | Create "promotion" template (for broadcasts) | 5 min | [ ] |
| 9.4 | Create "feedback_request" template | 5 min | [ ] |
| 9.5 | Submit all for Meta approval | 5 min | [ ] |
| 9.6 | Wait for approval (typically 24-48 hours) | — | [ ] |

**Suggested templates for a salon:**

| Template Name | Category | Body Text Example |
|---|---|---|
| `welcome_message` | Marketing | "Hi {{1}}! Welcome to Royal Glow Salon & Spa. How can we help you today?" |
| `appointment_reminder` | Utility | "Hi {{1}}, this is a reminder for your {{2}} appointment tomorrow at {{3}}. See you soon!" |
| `promo_offer` | Marketing | "Hi {{1}}! This week only: {{2}}. Book now and treat yourself! Reply BOOK to schedule." |
| `feedback_request` | Marketing | "Hi {{1}}, thank you for visiting us! We'd love your feedback. How was your {{2}} experience? Reply 1-5." |
| `rebooking_reminder` | Marketing | "Hi {{1}}, it's been a while since your last visit! Ready for another {{2}}? Reply YES to book." |

**Checkpoint:** Templates submitted, waiting for Meta approval ✅

---

### Day 13-14: Build Automations

| # | Task | Time | Done |
|---|------|------|------|
| 10.1 | Go to **Automations → New** | 1 min | [ ] |
| 10.2 | Create: **After-Hours Auto-Reply** | 10 min | [ ] |
| 10.3 | Create: **New Customer Welcome** | 10 min | [ ] |
| 10.4 | Create: **Keyword "book" → Send booking info** | 10 min | [ ] |
| 10.5 | Create: **Keyword "price" → Send price list** | 10 min | [ ] |
| 10.6 | Test each automation by sending WhatsApp messages | 10 min | [ ] |
| 10.7 | Verify automation logs show successful execution | 5 min | [ ] |

**Automation recipes:**

**After-Hours Auto-Reply:**
```
Trigger: New message received
Condition: Time is between 18:00 - 09:00
→ Send message: "Thank you for contacting Royal Glow! We're currently closed.
   Our hours are 9 AM - 6 PM. We'll reply as soon as we open! 💆‍♀️"
```

**New Customer Welcome:**
```
Trigger: New contact created
→ Send message: "Hi {{contact.name}}! Welcome to Royal Glow Salon & Spa ✨
   How can we help you today?"
→ Add tag: "new-customer"
```

**Checkpoint:** Core automations active and tested ✅

---

### Day 14-15: Set Up Pipeline & Import Contacts

| # | Task | Time | Done |
|---|------|------|------|
| 11.1 | Go to **Pipelines** → Create pipeline "Salon Bookings" | 3 min | [ ] |
| 11.2 | Add stages: Inquiry → Consultation → Booked → Completed → Follow-up | 5 min | [ ] |
| 11.3 | Set stage colors for visual clarity | 2 min | [ ] |
| 11.4 | Go to **Contacts** → Import → upload your customer CSV | 5 min | [ ] |
| 11.5 | Map CSV columns to contact fields (name, phone, email) | 3 min | [ ] |
| 11.6 | Verify imported contacts appear in the list | 2 min | [ ] |
| 11.7 | Create tags: "VIP", "regular", "new-customer", "facial", "massage", "hair" | 5 min | [ ] |

**CSV format for import:**
```csv
name,phone,email
Sarah Johnson,+63917XXXXXXX,sarah@email.com
Maria Santos,+63918XXXXXXX,maria@email.com
```

**Checkpoint:** Pipeline ready, contacts imported, tags created ✅

---

### Day 15-16: Team Training & Go-Live

| # | Task | Time | Done |
|---|------|------|------|
| 12.1 | Walk Manager through: Dashboard, Automations, Broadcasts, Settings | 30 min | [ ] |
| 12.2 | Walk Receptionist through: Inbox, Contacts, Pipelines | 30 min | [ ] |
| 12.3 | Practice: Receptionist replies to a test message | 5 min | [ ] |
| 12.4 | Practice: Manager creates and sends a test broadcast (1 recipient) | 5 min | [ ] |
| 12.5 | Practice: Receptionist creates a deal and moves through pipeline | 5 min | [ ] |
| 12.6 | Remove test data (if any) | 5 min | [ ] |
| 12.7 | **GO LIVE** — Start using with real customers! | — | [ ] |

**Checkpoint:** Team trained, system live with real customers ✅

---

## Post-Launch: Week 4+ (Ongoing)

### Routine Tasks (Weekly)

| Task | Who | When |
|------|-----|------|
| Check UptimeRobot for any downtime alerts | Manager | Monday morning |
| Review automation logs for failures | Manager | Monday morning |
| Send weekly broadcast (promotions/tips) | Manager | Wednesday |
| Review pipeline — follow up on stale deals | Receptionist | Friday |
| Check Supabase dashboard — DB size | Manager | Monthly |

### Optional Enhancements (When Ready)

| Enhancement | Effort | Value |
|-------------|--------|-------|
| Add more automation recipes (birthday wishes, rebooking reminders) | 30 min each | High |
| Create interactive booking flow (buttons + list menus) | 1-2 hours | High |
| Set up Oracle Cloud VPS backup schedule | 15 min | Medium |
| Add custom fields (e.g., "preferred service", "last visit date") | 10 min | Medium |
| Create a second pipeline for "Membership Renewals" | 15 min | Low |

---

## Quick Reference: All Credentials You'll Need

Save these in a password manager:

| Credential | Where to get it | Used for |
|---|---|---|
| Oracle Cloud SSH key | Generated during VPS creation | SSH access to server |
| Oracle VPS Public IP | Oracle Cloud Console → Instances | DNS, SSH |
| Coolify admin email + password | You create during setup | Managing deployments |
| Supabase Project URL | Project Settings → API | App config |
| Supabase anon key | Project Settings → API | App config |
| Supabase service_role key | Project Settings → API | App config (keep secret!) |
| ENCRYPTION_KEY | You generate (64 hex chars) | Encrypts WhatsApp tokens |
| AUTOMATION_CRON_SECRET | You generate (64 hex chars) | Protects cron endpoints |
| Meta App Secret | developers.facebook.com → App Settings | Webhook verification |
| WhatsApp Phone Number ID | Meta Dashboard → WhatsApp → API Setup | API calls |
| WhatsApp Access Token | System User → Generate Token | API calls |
| Verify Token | You choose (any string) | Webhook setup |
| Domain registrar login | Cloudflare/Namecheap/DuckDNS | DNS management |
| CRM Manager account | You create in the app | Daily use |
| CRM Receptionist account | You create in the app | Daily use |

---

## Risk Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Oracle "Out of capacity" error | Medium | Blocks Day 1 | Try different availability domain; retry at off-peak hours |
| Meta template rejection | Low | Delays broadcasts | Follow [Meta's template guidelines](https://developers.facebook.com/docs/whatsapp/message-templates/guidelines/); resubmit with edits |
| DNS propagation delay | Low | Delays webhook setup | Wait up to 24h; use `dig` to check progress |
| Webhook verification fails | Medium | Blocks WhatsApp | Double-check verify token matches in both Meta dashboard AND CRM settings |
| Build fails in Coolify | Low | Delays deployment | Check logs; usually a Node version issue — set to 20+ |
| Supabase free tier limits | Very Low | Not for 1-2 years | Monitor DB size monthly; upgrade only if hitting 500MB |

---

## Decision Points (Where You Need to Choose)

| Decision | Options | Recommendation |
|----------|---------|----------------|
| **Domain** | A) Buy custom domain (~$10/yr) <br> B) Free DuckDNS subdomain | **A** — looks professional for customers |
| **VPS region** | Singapore, Mumbai, Tokyo, Frankfurt, etc. | **Closest to your customers** |
| **Supabase region** | Same options | **Same as VPS region** |
| **Server timezone** | Your local timezone | **Your salon's operating timezone** |
| **Staff accounts** | Shared login vs individual | **Individual** — audit trail shows who replied |

---

## How to Proceed

### Right Now (Today):
1. **Sign up for Oracle Cloud** — [cloud.oracle.com/free](https://cloud.oracle.com/free)
2. **Sign up for Supabase** — [supabase.com](https://supabase.com)
3. **Decide on domain** — buy one or use DuckDNS

### This Week:
4. Follow **Phase 1** (Day 1-5) step by step
5. Reference `DEPLOYMENT-ORACLE-COOLIFY.md` for detailed commands

### Next Week:
6. Follow **Phase 2** (Day 6-10) — app goes live
7. Create Meta Developer account if you don't have one

### Week After:
8. Follow **Phase 3** (Day 11-16) — automations, team training, go-live

---

## Need Help?

- **Deployment details:** See [DEPLOYMENT-ORACLE-COOLIFY.md](./DEPLOYMENT-ORACLE-COOLIFY.md)
- **wacrm docs:** [wacrm.tech/docs](https://wacrm.tech/docs)
- **Coolify docs:** [coolify.io/docs](https://coolify.io/docs)
- **Supabase docs:** [supabase.com/docs](https://supabase.com/docs)
- **Oracle Cloud:** [docs.oracle.com](https://docs.oracle.com/en-us/iaas/Content/home.htm)
- **WhatsApp Business API:** [developers.facebook.com/docs/whatsapp](https://developers.facebook.com/docs/whatsapp)

---

*Created: May 2026*
*For Royal Glow Salon & Spa*
