# ⚡ NeuralFlow — AI SaaS Boilerplate

> Production-ready, full-stack AI SaaS with **Next.js 14 App Router**, **PostgreSQL + Prisma**, **Stripe billing**, **Auth.js**, **OpenAI** and **Gemini** streaming.

---

## 🗂 Project Structure

```
src/
├── app/
│   ├── (root)                  # Landing page, pricing
│   ├── api/
│   │   ├── auth/[...nextauth]  # Auth.js route handler
│   │   ├── auth/register       # Email/password registration
│   │   ├── billing/checkout    # Stripe checkout session
│   │   ├── billing/portal      # Stripe billing portal
│   │   ├── billing/webhook     # Stripe webhook handler (idempotent)
│   │   ├── ai/chat             # SSE streaming AI responses
│   │   └── user/               # Profile, API keys, usage
│   ├── auth/                   # Login, register, error pages
│   ├── dashboard/              # Protected dashboard routes
│   │   ├── page.tsx            # Overview with stats
│   │   ├── chat/               # AI Workspace (streaming chat)
│   │   ├── usage/              # 30-day analytics charts
│   │   ├── api-keys/           # API key management
│   │   ├── billing/            # Plan comparison + Stripe
│   │   └── settings/           # Profile settings
│   └── pricing/                # Public pricing page
├── lib/
│   ├── auth.ts                 # NextAuth config (JWT + OAuth + Credentials)
│   ├── db.ts                   # Prisma singleton
│   ├── stripe.ts               # Stripe SDK + PLANS config
│   ├── ai.ts                   # OpenAI + Gemini SDK wrappers
│   └── utils.ts                # Helpers
├── components/
│   └── providers.tsx           # SessionProvider
├── middleware.ts               # Auth guard for /dashboard
└── types/
    └── next-auth.d.ts          # Session type augmentation
prisma/
├── schema.prisma               # Full DB schema
└── seed.ts                     # Demo data seeder
```

---

## 🚀 Quick Start

### 1. Prerequisites

- **Node.js 18+**
- **PostgreSQL** (local or cloud — Neon, Supabase, Railway)
- **Stripe** account (free test mode)
- **OpenAI** or **Gemini** API key

### 2. Install

```bash
npm install
```

### 3. Environment Variables

```bash
cp .env.example .env.local
```

Fill in `.env.local`:

```env
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=<run: openssl rand -base64 32>

DATABASE_URL=postgresql://user:pass@localhost:5432/neuralflow

GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
GITHUB_CLIENT_ID=...
GITHUB_CLIENT_SECRET=...

STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PRICE_PRO_MONTHLY=price_...
STRIPE_PRICE_PRO_YEARLY=price_...
STRIPE_PRICE_ENTERPRISE_MONTHLY=price_...
STRIPE_PRICE_ENTERPRISE_YEARLY=price_...

NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...

OPENAI_API_KEY=sk-proj-...
GEMINI_API_KEY=AIza...
```

### 4. Database

```bash
npx prisma generate         # Generate Prisma client
npx prisma db push          # Push schema to your DB
npx prisma db seed          # Seed demo user
```

### 5. Run

```bash
npm run dev
```

Open **http://localhost:3000**

Demo login: `demo@neuralflow.dev` / `password123`

---

## 💳 Stripe Setup

1. Create products in the **Stripe Dashboard**:
   - **NeuralFlow Pro** — $29/month, $290/year
   - **NeuralFlow Enterprise** — $199/month, $1990/year

2. Copy the **Price IDs** to `.env.local`

3. Listen to webhooks locally:
   ```bash
   npm run stripe:listen
   ```
   Copy the `whsec_...` to `STRIPE_WEBHOOK_SECRET`

---

## 🔐 Authentication

| Method       | Provider     | Notes                              |
|:-------------|:-------------|:-----------------------------------|
| Credentials  | Built-in     | Email + bcrypt password            |
| OAuth        | Google       | One-click sign-in                  |
| OAuth        | GitHub       | One-click sign-in                  |

Auto-provisions Stripe customer + free subscription on first sign-up.

---

## 🤖 AI Models

| Model ID           | Provider | Min Tier   | Cost / 1K tokens |
|:-------------------|:---------|:-----------|:-----------------|
| `gpt-4o-mini`      | OpenAI   | FREE       | $0.00015         |
| `gpt-4o`           | OpenAI   | PRO        | $0.005           |
| `gemini-1.5-flash` | Gemini   | PRO        | $0.0001          |
| `gemini-1.5-pro`   | Gemini   | ENTERPRISE | $0.0035          |

Responses stream via **Server-Sent Events (SSE)** for zero-latency UX.

---

## 📊 Subscription Tiers

| Feature            | Free       | Pro        | Enterprise   |
|:-------------------|:-----------|:-----------|:-------------|
| Tokens/month       | 10,000     | 500,000    | 5,000,000    |
| API Keys           | 5          | 50         | Unlimited    |
| Models             | GPT-4o Mini| GPT-4o + Gemini | All     |
| Price/month        | $0         | $29        | $199         |

Token limits auto-reset on each Stripe billing cycle via `invoice.paid` webhook.

---

## 🗄️ Database Schema

| Table              | Purpose                                    |
|:-------------------|:-------------------------------------------|
| `users`            | Core user records                          |
| `accounts`         | OAuth provider links                       |
| `sessions`         | JWT session records                        |
| `subscriptions`    | Stripe customer + tier + token usage       |
| `conversations`    | Chat session metadata                      |
| `messages`         | Individual chat messages with token counts |
| `usage_records`    | Daily token/request/cost aggregates        |
| `api_keys`         | SHA-256 hashed API keys                    |
| `stripe_events`    | Processed Stripe event IDs (idempotency)   |

---

## 🔑 API Endpoints

| Method | Endpoint                    | Auth | Description                        |
|:-------|:----------------------------|:-----|:-----------------------------------|
| POST   | `/api/auth/register`        | No   | Email registration + Stripe setup  |
| GET    | `/api/auth/[...nextauth]`   | —    | NextAuth handler                   |
| POST   | `/api/ai/chat`              | Yes  | SSE streaming chat (multi-model)   |
| POST   | `/api/billing/checkout`     | Yes  | Create Stripe Checkout session     |
| POST   | `/api/billing/portal`       | Yes  | Open Stripe billing portal         |
| POST   | `/api/billing/webhook`      | —    | Stripe webhook (idempotent)        |
| GET    | `/api/user/api-keys`        | Yes  | List API keys                      |
| POST   | `/api/user/api-keys`        | Yes  | Create API key (SHA-256 hashed)    |
| DELETE | `/api/user/api-keys`        | Yes  | Revoke API key                     |
| GET    | `/api/user/usage`           | Yes  | Usage stats + subscription info    |
| PATCH  | `/api/user/profile`         | Yes  | Update display name                |

---

## 🛡️ Security Notes

- Passwords hashed with **bcrypt** (12 rounds)
- API keys stored as **SHA-256 hashes** — raw key shown only once
- Stripe webhooks verified with **signature validation** + **idempotency keys**
- All dashboard routes protected via **Next.js middleware**
- Environment secrets never exposed to client bundle

---

## 🧪 Scripts

```bash
npm run dev              # Development server
npm run build            # Production build
npm run start            # Production server
npm run db:push          # Apply schema changes
npm run db:migrate       # Create migration files
npm run db:studio        # Prisma Studio GUI
npm run db:seed          # Seed demo data
npm run stripe:listen    # Forward Stripe webhooks locally
npm run type-check       # TypeScript check
```

---

## 📦 Tech Stack

| Layer         | Technology                      |
|:--------------|:--------------------------------|
| Framework     | Next.js 14 (App Router)         |
| Language      | TypeScript (strict)             |
| Styling       | Tailwind CSS v3 + Syne font     |
| Auth          | Auth.js v5 (NextAuth)           |
| Database      | PostgreSQL + Prisma ORM         |
| Payments      | Stripe (checkout + webhooks)    |
| AI (OpenAI)   | GPT-4o, GPT-4o Mini             |
| AI (Google)   | Gemini 1.5 Flash, Gemini 1.5 Pro|
| Charts        | Recharts                        |
| Validation    | Zod                             |
| Deployment    | Vercel / Railway / any Node host|

---

## 🚢 Deploy to Vercel

```bash
npm i -g vercel
vercel
```

Set all `.env.example` variables in the Vercel dashboard.  
Set `NEXTAUTH_URL` to your production URL.  
Set `STRIPE_WEBHOOK_SECRET` from a **live** Stripe webhook endpoint.

---

## 📄 License

MIT — free for personal and commercial use.
