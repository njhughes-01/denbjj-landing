# DEN BJJ Tech Stack Plan

**Last Updated:** 2026-03-08  
**Status:** Planning Phase

## Verified Pricing (as of Jan 2026)

### Supabase
- **Free Tier:** $0/month
  - 500 MB database
  - 1 GB file storage
  - 50,000 MAUs
  - **Auto-pauses after 1 week inactivity** ⚠️
  - Limited to 2 projects
- **Pro Tier:** $25/month per project ✅ **This is what we need**
  - 8 GB database
  - 100 GB file storage
  - 100,000 MAUs
  - No auto-pause

### Other Costs
- **Stripe:** 2.9% + $0.30 per transaction (standard)
- **Resend (email):** Free tier 3,000 emails/month, then $20/mo for 50k
- **Domain + Email:** Already covered (Porkbun + MXRoute)
- **Cloudflare Pages:** Free (hosting landing page)

**Monthly Estimate:** $25-50/month for production

---

## Core Stack

### Backend: Supabase (PostgreSQL + Auth + Storage)
**Version:** Latest stable (track via supabase.com)  
**Why:** Open source, PostgreSQL-based, handles auth/database/storage/real-time  
**Security:** Row-level security (RLS) policies, API keys rotated regularly

### Frontend: Next.js 15
**Version:** 15.x (latest stable)  
**Why:** React framework, great DX, excellent for member portals  
**Hosting:** Vercel (free tier or $20/mo Pro)

### Payments: Stripe
**SDK:** `@stripe/stripe-js` + `stripe` (Node.js)  
**Why:** Industry standard, handles PCI compliance, subscription billing  
**Security:** Never store card data, use Stripe Elements, webhook signature verification

### Email: Resend
**SDK:** `resend` (Node.js)  
**Why:** Modern, developer-friendly, great deliverability  
**Use cases:** Transactional emails (receipts, password resets, class confirmations)

---

## Library Dependencies (Security-First)

### Core Dependencies

```json
{
  "dependencies": {
    "next": "^15.0.0",
    "@supabase/supabase-js": "^2.45.0",
    "@supabase/auth-helpers-nextjs": "^0.10.0",
    "@stripe/stripe-js": "^4.0.0",
    "stripe": "^16.0.0",
    "resend": "^4.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "zod": "^3.23.0",
    "@tanstack/react-query": "^5.50.0"
  },
  "devDependencies": {
    "typescript": "^5.5.0",
    "@types/react": "^19.0.0",
    "@types/node": "^22.0.0",
    "eslint": "^9.0.0",
    "prettier": "^3.3.0"
  }
}
```

### Dependency Security Policy

1. **Update Schedule:**
   - Weekly automated PR via Renovate or Dependabot
   - Security patches applied immediately (same day)
   - Minor/major updates reviewed within 7 days

2. **Version Pinning:**
   - Use `^` (caret) for minor updates (e.g., `^15.0.0` allows 15.x)
   - Pin exact versions for critical security libs (Stripe, Supabase auth)
   - Lock file (`package-lock.json`) committed to repo

3. **Vulnerability Scanning:**
   - GitHub Dependabot alerts enabled
   - Weekly `npm audit` checks
   - Snyk or Socket.dev integration (optional)

4. **Supply Chain Security:**
   - No packages with <100k weekly downloads (unless critical and vetted)
   - Check package maintainer history before adding new deps
   - Prefer official SDKs over community wrappers

---

## Architecture

### Member Portal Features (Phase 2)

**Authentication:**
- Supabase Auth (email/password)
- Email verification required
- Password reset flow
- Optional: Social login (Google) later

**Member Management:**
- Profile (name, email, photo, belt rank, emergency contact)
- Membership status (active/inactive/expired)
- Waiver signing (digital signature via canvas or third-party)
- Payment history

**Class Scheduling:**
- View class schedule
- RSVP for classes (track attendance)
- Waitlist for full classes
- Admin: create/edit/cancel classes

**Payments:**
- Stripe Checkout for initial signup
- Stripe Customer Portal for managing subscriptions
- Subscription tiers:
  - Monthly unlimited
  - Drop-in punch cards (10 classes, etc.)
  - Kids program
- Automated billing reminders

**Admin Dashboard:**
- View all members
- Attendance tracking
- Revenue reports (via Stripe data)
- Class roster management

### Database Schema (Rough)

```sql
-- Users (via Supabase Auth, extended with profiles)
profiles:
  - id (uuid, FK to auth.users)
  - full_name
  - belt_rank
  - profile_photo_url
  - emergency_contact_name
  - emergency_contact_phone
  - created_at
  - updated_at

-- Memberships
memberships:
  - id (uuid)
  - user_id (FK profiles)
  - stripe_customer_id
  - stripe_subscription_id
  - plan_type (monthly, punch_card, etc.)
  - status (active, paused, canceled)
  - start_date
  - end_date
  - created_at

-- Classes
classes:
  - id (uuid)
  - title (e.g., "All Levels Gi")
  - description
  - instructor_id (FK profiles)
  - day_of_week
  - start_time
  - end_time
  - max_capacity
  - created_at

-- Attendance
attendance:
  - id (uuid)
  - class_id (FK classes)
  - user_id (FK profiles)
  - status (rsvp, attended, no-show)
  - checked_in_at
  - created_at

-- Waivers
waivers:
  - id (uuid)
  - user_id (FK profiles)
  - signed_at
  - signature_data (base64 canvas or file URL)
  - ip_address
  - created_at
```

---

## Security Checklist

### Authentication
- ✅ Email verification required before first login
- ✅ Password strength requirements (min 12 chars, enforced by Supabase)
- ✅ Rate limiting on login attempts (Supabase built-in)
- ✅ Secure session management (httpOnly cookies)
- ✅ CSRF protection (Next.js built-in)

### Database
- ✅ Row-level security (RLS) policies on all tables
- ✅ API keys stored in environment variables (never in code)
- ✅ Separate read/write permissions for service role vs anon key
- ✅ SQL injection prevention (use Supabase client, never raw SQL from user input)

### Payments
- ✅ Never store card numbers (Stripe Elements handles this)
- ✅ Webhook signature verification (reject unsigned requests)
- ✅ Idempotency keys for payment operations
- ✅ Audit log for all payment events

### File Uploads
- ✅ Validate file types (images only for profile photos)
- ✅ File size limits (max 5MB per upload)
- ✅ Scan for malware (optional: ClamAV or VirusTotal API)
- ✅ Serve files from CDN, not directly from storage bucket

### General
- ✅ HTTPS only (Cloudflare + Vercel enforce this)
- ✅ Environment variables for all secrets
- ✅ Content Security Policy (CSP) headers
- ✅ Regular security audits (quarterly npm audit, Snyk scans)
- ✅ Backup strategy (Supabase daily backups on Pro plan)

---

## Development Workflow

### Local Development
```bash
npm install
npm run dev  # Runs Next.js on localhost:3000
```

### Environment Variables (.env.local)
```bash
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=xxx
SUPABASE_SERVICE_ROLE_KEY=xxx  # Server-side only
STRIPE_PUBLIC_KEY=pk_test_xxx
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
RESEND_API_KEY=re_xxx
```

### Deployment
- **Hosting:** Vercel (connected to GitHub repo)
- **Auto-deploy:** Every push to `main` branch
- **Preview deploys:** Every PR gets a unique URL
- **Environment:** Production env vars set in Vercel dashboard

### Testing
- **Unit tests:** Vitest (Next.js compatible)
- **Integration tests:** Playwright (for critical flows like signup/payment)
- **Pre-commit hooks:** ESLint + Prettier (via Husky)

---

## Migration Path

### Phase 1: Email Capture (This Week)
- Add simple form to landing page
- Store emails in Supabase free tier table
- No auth required yet
- Netlify/Cloudflare Forms as fallback

### Phase 2: MVP Member Portal (When facility secured)
- Supabase Pro ($25/mo)
- Next.js app with:
  - User signup/login
  - Profile management
  - View class schedule
  - Stripe payment integration (one-time or subscription)
  - Basic admin panel

### Phase 3: Full Features (After launch)
- Class RSVP system
- Attendance tracking
- Waiver signing
- Email automations (welcome series, billing reminders)
- Mobile app (optional, using React Native + Supabase)

---

## Self-Hosting Option (Future)

If costs grow or you want full control:
- **Supabase:** Self-host on your VPS (Docker Compose setup)
- **Next.js:** Deploy on your VPS via PM2 or Docker
- **Database:** Run your own PostgreSQL (backup strategy critical)
- **Estimated Savings:** $25/mo, but adds maintenance burden

**Recommendation:** Stay on managed Supabase Pro until you hit 500+ members or $100+/mo in costs.

---

## Next Steps

1. ✅ Verify this tech stack with Nate
2. ⬜ Set up Supabase project (free tier for email capture)
3. ⬜ Add email capture form to landing page
4. ⬜ Draft social media content
5. ⬜ When facility secured: Build MVP member portal

---

## Questions / Decisions Needed

- **Email capture:** Use Supabase free tier or simple Cloudflare Workers endpoint?
- **Social media:** Create accounts now or wait until closer to launch?
- **Timeline:** What's realistic launch date for first classes?
- **Pricing model:** Monthly unlimited, punch cards, or both?
