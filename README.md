# StoryLoom

A full-stack serialized fiction and short video platform — think Wattpad meets TikTok.

## Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14 (App Router) |
| Styling | Tailwind CSS (custom design system) |
| Auth | Clerk |
| Database | Supabase (PostgreSQL + RLS) |
| Storage | Cloudinary (video + images) |
| Payments | Paystack |
| Deployment | Vercel |

## Features

- **TikTok-style Shorts Feed** — vertical scroll, auto-play, mute/unmute, like, bookmark, share
- **Novel Reader** — chapter-by-chapter with word count and read time
- **Premium Locking** — blurred preview + paywall for locked chapters and shorts
- **Subscription System** — Paystack-powered Basic (₦1,500/mo) and Premium (₦3,500/mo) plans
- **Creator Dashboard** — upload shorts, create novels, write chapters, track analytics
- **Admin Dashboard** — manage users, content moderation, view payments
- **Comments & Likes** — threaded comments on any content type
- **Bookmarks** — save novels and shorts to read/watch later
- **Infinite Scroll** — paginated feeds with intersection observer
- **SEO Optimized** — dynamic metadata, OG tags, sitemap-ready
- **Mobile-First** — bottom navigation on mobile, responsive grid

## Quick Start

```bash
git clone <repo>
cd storyloom
npm install
cp .env.example .env.local
# Fill in .env.local with your service credentials
npm run dev
```

See **DEPLOYMENT.md** for full step-by-step setup of all services.

## Project Structure

```
src/
├── app/
│   ├── page.tsx                    # Landing page
│   ├── shorts/                     # TikTok-style feed
│   ├── novels/                     # Novel listing + reader
│   ├── pricing/                    # Subscription plans
│   ├── dashboard/
│   │   ├── creator/                # Creator studio
│   │   └── admin/                  # Admin panel
│   └── api/                        # All API routes
├── components/
│   ├── shared/                     # Navbar, PricingCards, CommentsSection
│   ├── shorts/                     # ShortsFeed, ShortItem, ShortsNav
│   └── admin/                      # AdminContentTable, AdminUsersTable
├── lib/
│   ├── supabase/                   # Client + server clients
│   ├── paystack.ts                 # Payment helpers
│   ├── cloudinary.ts               # Upload helpers
│   └── utils.ts                    # Shared utilities
└── types/index.ts                  # All TypeScript types
```

## Database

Two SQL migrations in `supabase/migrations/`:
- `001_initial_schema.sql` — all tables, RLS policies, triggers
- `002_rpc_functions.sql` — RPCs for view/like counts, search, analytics

## API Routes

| Route | Method | Description |
|---|---|---|
| `/api/shorts` | GET | Paginated shorts feed |
| `/api/likes` | POST/DELETE | Toggle like |
| `/api/bookmarks` | GET/POST/DELETE | Manage bookmarks |
| `/api/comments` | GET/POST/DELETE | Comments |
| `/api/search` | GET | Full-text search |
| `/api/upload/video` | POST | Cloudinary video upload |
| `/api/upload/image` | POST | Cloudinary image upload |
| `/api/creator/shorts` | GET/POST | Creator shorts management |
| `/api/creator/novels` | GET/POST | Creator novels management |
| `/api/creator/novels/[id]/chapters` | GET/POST | Chapter management |
| `/api/creator/apply` | POST | Activate creator account |
| `/api/admin/users/[id]` | PATCH/DELETE | Admin user management |
| `/api/admin/novels/[id]` | PATCH/DELETE | Admin content moderation |
| `/api/admin/shorts/[id]` | PATCH/DELETE | Admin content moderation |
| `/api/subscriptions/initialize` | POST | Create Paystack transaction |
| `/api/subscriptions/verify` | GET | Verify payment callback |
| `/api/webhooks/paystack` | POST | Paystack webhook handler |
| `/api/webhooks/clerk` | POST | Clerk user sync webhook |

## Design System

Custom Tailwind theme with:
- **Fonts**: Playfair Display (headings) + DM Sans (body) + DM Mono
- **Colors**: `ink` (dark browns), `parchment` (warm gold), `crimson` (accents)
- **Components**: `.btn-primary`, `.btn-ghost`, `.card-novel`, `.glass`, `.prose-story`

## License

MIT
