# StoryLoom — Full Deployment Guide

## Stack
- **Next.js 14** (App Router)
- **Tailwind CSS** (custom design system)
- **Supabase** (database + auth RLS)
- **Clerk** (authentication)
- **Cloudinary** (video + image storage)
- **Paystack** (payments + subscriptions)
- **Vercel** (deployment)

---

## Folder Structure

```
storyloom/
├── middleware.ts                    # Clerk route protection
├── next.config.js
├── tailwind.config.js
├── .env.example
├── supabase/
│   └── migrations/
│       └── 001_initial_schema.sql   # Full DB schema
├── src/
│   ├── app/
│   │   ├── layout.tsx               # Root layout (Clerk, fonts)
│   │   ├── globals.css
│   │   ├── page.tsx                 # Homepage
│   │   ├── shorts/
│   │   │   └── page.tsx             # TikTok-style shorts feed
│   │   ├── novels/
│   │   │   ├── page.tsx             # Novel listing
│   │   │   └── [slug]/
│   │   │       ├── page.tsx         # Novel detail + chapters
│   │   │       └── chapter/[number]/page.tsx  # Reader + premium lock
│   │   ├── pricing/
│   │   │   └── page.tsx             # Subscription plans
│   │   ├── dashboard/
│   │   │   ├── creator/
│   │   │   │   ├── page.tsx         # Creator dashboard
│   │   │   │   └── shorts/new/page.tsx  # Upload short
│   │   │   └── admin/
│   │   │       └── page.tsx         # Admin dashboard
│   │   └── api/
│   │       ├── shorts/route.ts
│   │       ├── likes/route.ts
│   │       ├── creator/shorts/route.ts
│   │       ├── upload/video/route.ts
│   │       ├── subscriptions/
│   │       │   ├── initialize/route.ts
│   │       │   └── verify/route.ts
│   │       └── webhooks/
│   │           ├── paystack/route.ts
│   │           └── clerk/route.ts
│   ├── components/
│   │   ├── shared/
│   │   │   ├── Navbar.tsx
│   │   │   └── PricingCards.tsx
│   │   └── shorts/
│   │       ├── ShortsFeed.tsx
│   │       └── ShortItem.tsx
│   ├── lib/
│   │   ├── supabase/
│   │   │   ├── client.ts
│   │   │   └── server.ts
│   │   ├── paystack.ts
│   │   ├── cloudinary.ts
│   │   └── utils.ts
│   └── types/
│       └── index.ts
```

---

## Step 1: Clone & Install

```bash
git clone <your-repo>
cd storyloom
npm install
cp .env.example .env.local
```

---

## Step 2: Supabase Setup

1. Create project at https://supabase.com
2. Go to **SQL Editor** → paste contents of `supabase/migrations/001_initial_schema.sql` → Run
3. In **Project Settings → API**: copy `URL` and `anon key`
4. In **Authentication → URL Configuration**: add your Vercel URL
5. Add to `.env.local`:
   ```
   NEXT_PUBLIC_SUPABASE_URL=https://xxxx.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
   SUPABASE_SERVICE_ROLE_KEY=eyJ...
   ```

---

## Step 3: Clerk Authentication

1. Create app at https://clerk.com
2. Enable Email/Password + Google/GitHub OAuth (optional)
3. In **Webhooks**: add endpoint `https://yourdomain.com/api/webhooks/clerk`
   - Subscribe to: `user.created`, `user.updated`, `user.deleted`
   - Copy the webhook signing secret → `CLERK_WEBHOOK_SECRET`
4. Add to `.env.local`:
   ```
   NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_live_...
   CLERK_SECRET_KEY=sk_live_...
   CLERK_WEBHOOK_SECRET=whsec_...
   ```

---

## Step 4: Cloudinary Setup

1. Create account at https://cloudinary.com
2. Go to **Settings → Upload**: create an unsigned upload preset named `storyloom_unsigned`
3. In **Dashboard**: copy Cloud Name, API Key, API Secret
4. Add to `.env.local`:
   ```
   NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME=your_cloud_name
   CLOUDINARY_API_KEY=123456789
   CLOUDINARY_API_SECRET=xxxxx
   CLOUDINARY_UPLOAD_PRESET=storyloom_unsigned
   ```

---

## Step 5: Paystack Setup

1. Create account at https://paystack.com
2. **Plans → Create Plan**:
   - Reader Plan: ₦1,500/month → copy plan code → `PAYSTACK_BASIC_PLAN_CODE`
   - Unlimited Plan: ₦3,500/month → copy plan code → `PAYSTACK_PREMIUM_PLAN_CODE`
3. In **Settings → API Keys**: copy public + secret keys
4. In **Settings → Webhooks**: add `https://yourdomain.com/api/webhooks/paystack`
   - Subscribe to: `charge.success`, `subscription.disable`, `invoice.payment_failed`
   - Copy webhook secret
5. Add to `.env.local`:
   ```
   NEXT_PUBLIC_PAYSTACK_PUBLIC_KEY=pk_live_...
   PAYSTACK_SECRET_KEY=sk_live_...
   PAYSTACK_WEBHOOK_SECRET=xxx
   PAYSTACK_BASIC_PLAN_CODE=PLN_xxx
   PAYSTACK_PREMIUM_PLAN_CODE=PLN_xxx
   ```

---

## Step 6: Supabase RPC Function

Run this in Supabase SQL Editor to support like counting:

```sql
CREATE OR REPLACE FUNCTION increment_like_count(row_id UUID, table_name TEXT)
RETURNS void AS $$
BEGIN
  IF table_name = 'shorts' THEN
    UPDATE shorts SET like_count = like_count + 1 WHERE id = row_id;
  ELSIF table_name = 'novels' THEN
    UPDATE novels SET like_count = like_count + 1 WHERE id = row_id;
  ELSIF table_name = 'chapters' THEN
    UPDATE chapters SET like_count = COALESCE(like_count, 0) + 1 WHERE id = row_id;
  END IF;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

---

## Step 7: Make First User an Admin + Creator

After signing up, run in Supabase SQL Editor:

```sql
UPDATE users 
SET is_admin = TRUE, is_creator = TRUE 
WHERE email = 'your@email.com';
```

---

## Step 8: Vercel Deployment

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Add all environment variables in Vercel dashboard
# Settings → Environment Variables → add each from .env.local
```

Or connect your GitHub repo to Vercel and it auto-deploys on push.

**Vercel Settings to configure:**
- Framework: Next.js
- Build Command: `next build`
- Output Directory: `.next`
- Node.js Version: 18.x or 20.x

---

## Step 9: Post-Deployment Checks

- [ ] Sign up and verify Clerk webhook syncs user to Supabase
- [ ] Upload a short as a creator
- [ ] Subscribe via Paystack test mode (use test keys first)
- [ ] Verify webhook fires and subscription_status updates in Supabase
- [ ] Check premium chapter locking works
- [ ] Confirm admin dashboard accessible for admin user

---

## Environment Variables — Complete List

| Variable | Where to get |
|---|---|
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk Dashboard → API Keys |
| `CLERK_SECRET_KEY` | Clerk Dashboard → API Keys |
| `CLERK_WEBHOOK_SECRET` | Clerk Dashboard → Webhooks |
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase → Settings → API |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase → Settings → API |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase → Settings → API |
| `NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME` | Cloudinary Dashboard |
| `CLOUDINARY_API_KEY` | Cloudinary Dashboard |
| `CLOUDINARY_API_SECRET` | Cloudinary Dashboard |
| `CLOUDINARY_UPLOAD_PRESET` | Cloudinary → Upload Presets |
| `NEXT_PUBLIC_PAYSTACK_PUBLIC_KEY` | Paystack → Settings → API |
| `PAYSTACK_SECRET_KEY` | Paystack → Settings → API |
| `PAYSTACK_WEBHOOK_SECRET` | Paystack → Settings → Webhooks |
| `PAYSTACK_BASIC_PLAN_CODE` | Paystack → Plans |
| `PAYSTACK_PREMIUM_PLAN_CODE` | Paystack → Plans |
| `NEXT_PUBLIC_APP_URL` | Your Vercel deployment URL |

---

## Notes

- **Paystack test mode**: Use `pk_test_` / `sk_test_` keys during development
- **Video uploads**: Limited to 200MB per short; Cloudinary auto-generates thumbnails
- **Premium chapters**: Access is checked server-side; client never receives locked content
- **RLS**: All database policies are row-level; the service role key is only used in API routes
- **Shorts feed**: Infinite scroll loads 10 at a time; scroll-snap provides the TikTok feel
