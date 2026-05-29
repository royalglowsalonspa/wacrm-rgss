# wacrm-rgss — WhatsApp CRM for Royal Glow Salon & Spa

> Internal WhatsApp CRM for managing customer conversations, bookings, and promotions — powered by the official WhatsApp Business API.

[![License: MIT](https://img.shields.io/badge/License-MIT-violet.svg)](./LICENSE)
[![Next.js 16](https://img.shields.io/badge/Next.js-16-black?logo=nextdotjs)](https://nextjs.org)
[![Supabase](https://img.shields.io/badge/Supabase-Postgres%20%2B%20Auth-3ecf8e?logo=supabase)](https://supabase.com)
[![Oracle Cloud](https://img.shields.io/badge/Oracle_Cloud-Always_Free-F80000?logo=oracle)](https://cloud.oracle.com/free)
[![Coolify](https://img.shields.io/badge/Coolify-Self--Hosted_PaaS-6C3BF5)](https://coolify.io)

---

## About

This is **Royal Glow Salon & Spa's** internal WhatsApp CRM system. It enables our manager and receptionist to:

- Respond to customer inquiries via WhatsApp from a shared inbox
- Manage contacts, tags, and customer notes
- Track sales through visual pipelines (Kanban boards)
- Send promotional broadcasts (offers, seasonal campaigns)
- Automate responses (welcome messages, after-hours replies, booking confirmations)

The system runs on a **$0/month infrastructure stack** using Oracle Cloud's Always Free tier.

---

## Our Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **App** | Next.js 16, React 19, TypeScript, Tailwind v4 | Frontend + API routes |
| **Database** | Supabase (Postgres + RLS) | Contacts, conversations, pipelines, automations |
| **Auth** | Supabase Auth | Staff login (email + password) |
| **Realtime** | Supabase Realtime | Live inbox — new messages appear instantly |
| **Storage** | Supabase Storage | Avatars, media files |
| **Hosting** | Oracle Cloud VPS (Always Free ARM) | 4 OCPU, 24 GB RAM, always-on |
| **PaaS** | Coolify (self-hosted) | Git-push deploys, auto-SSL, container management |
| **WhatsApp** | Meta Cloud API (official) | Send/receive messages, templates, broadcasts |
| **Cron** | System crontab (on VPS) | Automation wait-steps, flow timeout sweeps |

---

## Architecture

```
┌──────────────────────────────────────────────────────┐
│              ROYAL GLOW SALON & SPA CRM              │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────────┐    ┌────────────────────────┐ │
│  │ Oracle Cloud VPS  │    │   Supabase (Free)      │ │
│  │ (Always Free)     │    │                        │ │
│  │                   │    │  • Postgres database   │ │
│  │ • Ubuntu + Docker │◄──►│  • Auth (staff login)  │ │
│  │ • Coolify PaaS    │    │  • Realtime (inbox)    │ │
│  │ • Next.js app     │    │  • Storage (media)     │ │
│  │ • Crontab         │    │                        │ │
│  │ • Auto-SSL        │    │                        │ │
│  └──────────────────┘    └────────────────────────┘ │
│           ▲                                          │
│           │ webhooks                                 │
│  ┌────────┴─────────┐                               │
│  │  Meta WhatsApp    │                               │
│  │  Cloud API        │                               │
│  └──────────────────┘                               │
└──────────────────────────────────────────────────────┘
```

---

## Features

### Shared Inbox
- Multiple agents (manager + receptionist) on one WhatsApp number
- Per-conversation assignment, status tracking, and internal notes
- Real-time message delivery — no page refresh needed
- Media support (images, documents, audio, video, stickers)
- Message reactions and reply-to-message (swipe quotes)

### Contacts & Tags
- Auto-created from inbound messages
- Custom fields, tags with colors
- CSV import with deduplication
- Contact notes and conversation history

### Sales Pipelines
- Kanban board with drag-and-drop deals
- Stages: Inquiry → Consultation → Booked → Completed → Follow-up
- Deal values, expected close dates, linked conversations
- Pipeline analytics

### Broadcasts
- Meta-approved message templates
- Per-recipient variable substitution (personalized messages)
- Delivery, read, and reply tracking per recipient
- Audience filtering by tags

### No-Code Automations
- Triggers: new message, new contact, keyword match, schedule
- Actions: send message, add tag, assign conversation, webhook, wait
- Conditional branches (if/else logic)
- Visual builder — no coding required

### Conversational Flows (Bot Builder)
- Multi-step interactive flows with buttons and list menus
- Customer responses advance the flow automatically
- Timeout detection and fallback messages
- One active flow per contact (prevents message conflicts)

### Dashboard
- Response time metrics
- Daily conversation volume chart
- Pipeline value overview
- Cross-module activity feed

---

## Deployment

See **[DEPLOYMENT-ORACLE-COOLIFY.md](./DEPLOYMENT-ORACLE-COOLIFY.md)** for the complete step-by-step guide.

### Quick Summary

| Phase | What | Time |
|-------|------|------|
| 1 | Create Oracle Cloud VPS (Always Free) | 15 min |
| 2 | Install Coolify (one command) | 5 min |
| 3 | Set up Supabase + run migrations | 15 min |
| 4 | Deploy this repo via Coolify | 10 min |
| 5 | Configure WhatsApp Business API | 20 min |
| 6 | Set up cron jobs | 5 min |
| 7 | Domain + SSL | 5 min |
| 8 | Verify everything works | 10 min |
| **Total** | | **~85 min** |

### Monthly Cost: $0 (infrastructure)

| Service | Cost |
|---------|------|
| Oracle Cloud VPS | Free forever |
| Supabase | Free tier |
| Coolify | Free (open source) |
| SSL certificates | Free (Let's Encrypt via Coolify) |
| WhatsApp conversations | 1,000 free/month (customer-initiated) |

---

## Environment Variables

Copy `.env.local.example` to `.env.local` and fill in:

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# WhatsApp token encryption (AES-256-GCM)
ENCRYPTION_KEY=your-64-char-hex-key

# Meta webhook verification
META_APP_SECRET=your-meta-app-secret

# Site URL
NEXT_PUBLIC_SITE_URL=https://crm.yourdomain.com

# Cron endpoint protection
AUTOMATION_CRON_SECRET=your-cron-secret
```

---

## Database Migrations

Run these in order in Supabase SQL Editor:

```
supabase/migrations/001_initial_schema.sql
supabase/migrations/002_pipelines_enhancements.sql
supabase/migrations/003_broadcast_recipient_wamid.sql
supabase/migrations/004_contact_delete_set_null.sql
supabase/migrations/005_broadcast_counts_incremental.sql
supabase/migrations/006_automations.sql
supabase/migrations/007_automations_increment_counter.sql
supabase/migrations/008_profile_avatars_storage.sql
supabase/migrations/009_message_actions.sql
supabase/migrations/010_flows.sql
supabase/migrations/011_profile_beta_features.sql
supabase/migrations/012_flows_increment_counter.sql
supabase/migrations/013_whatsapp_config_phone_number_id_unique.sql
```

All migrations are idempotent — safe to run multiple times.

---

## Local Development

```bash
git clone https://github.com/royalglowsalonspa/wacrm-rgss.git
cd wacrm-rgss
npm install
cp .env.local.example .env.local   # fill in your credentials
npm run dev
```

Open http://localhost:3000

### Available Scripts

| Script | Purpose |
|--------|---------|
| `npm run dev` | Start development server |
| `npm run build` | Production build |
| `npm start` | Start production server |
| `npm run lint` | Run ESLint |
| `npm run typecheck` | TypeScript type checking |
| `npm test` | Run tests (Vitest) |
| `npm run format` | Format with Prettier |

---

## Security

- **Row Level Security (RLS)** on every database table
- **AES-256-GCM encryption** for WhatsApp access tokens at rest
- **HMAC-SHA256 verification** on every inbound webhook from Meta
- **Security headers**: HSTS, CSP, X-Frame-Options, X-Content-Type-Options
- **Rate limiting** on send, broadcast, and reaction endpoints
- **Cron endpoint protection** via shared secret header

---

## Team

| Role | Responsibilities |
|------|-----------------|
| Manager | Broadcasts, automations, pipeline management, settings |
| Receptionist | Inbox conversations, contact management, deal updates |

---

## License

[MIT](./LICENSE)

---

## Credits & Attribution

This project is built upon the open-source **wacrm** template by Arnas Donauskas.

- **Original repository:** [github.com/ArnasDon/wacrm](https://github.com/ArnasDon/wacrm)
- **Original README:** [github.com/ArnasDon/wacrm/blob/main/README.md](https://github.com/ArnasDon/wacrm/blob/main/README.md)
- **Documentation:** [wacrm.tech/docs](https://wacrm.tech/docs)
- **License:** MIT — permits commercial use, modification, and distribution

We have customized this template for Royal Glow Salon & Spa's specific needs, including:
- Oracle Cloud + Coolify deployment (instead of Hostinger)
- Salon-specific pipeline stages and automations
- Team structure optimized for salon operations

Thank you to the wacrm project for providing an excellent, well-architected foundation.
