# NexusAI - Complete SaaS Platform Plan

## Context

Build a unified B2B SaaS platform combining two validated business ideas:
1. **DocAI** - AI-powered document processing (OCR + data extraction + export)
2. **FeedbackAI** - AI-powered survey & customer feedback analysis

Target: Production-ready, deployable on Vercel + Supabase, English UI, OpenAI GPT-4o.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14+ (App Router) + TypeScript |
| Styling | Tailwind CSS + shadcn/ui |
| Database | Supabase (PostgreSQL + Auth + Storage) |
| AI | OpenAI GPT-4o (Vision for OCR, Chat for analysis) |
| Payments | Stripe (Checkout + Webhooks) |
| Charts | Recharts |
| Forms | React Hook Form + Zod |
| Email | Resend (transactional emails) |
| Deploy | Vercel |

---

## Project Structure

```
nexus-ai/
├── .env.local.example
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── package.json
├── middleware.ts                    # Auth middleware
├── supabase/
│   └── migrations/
│       └── 001_initial_schema.sql  # Full DB schema + RLS
├── public/
│   ├── logo.svg
│   └── og-image.png
├── src/
│   ├── app/
│   │   ├── layout.tsx              # Root layout (fonts, providers)
│   │   ├── page.tsx                # Landing page
│   │   ├── pricing/page.tsx        # Pricing page
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   ├── signup/page.tsx
│   │   │   └── callback/route.ts   # OAuth callback
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx          # Dashboard shell (sidebar + topbar)
│   │   │   ├── dashboard/page.tsx  # Overview dashboard
│   │   │   ├── documents/
│   │   │   │   ├── page.tsx        # Document library
│   │   │   │   ├── upload/page.tsx # Upload & process
│   │   │   │   └── [id]/page.tsx   # Document detail/results
│   │   │   ├── surveys/
│   │   │   │   ├── page.tsx        # Survey list
│   │   │   │   ├── create/page.tsx # Survey builder
│   │   │   │   └── [id]/
│   │   │   │       ├── page.tsx    # Survey detail + results
│   │   │   │       ├── responses/page.tsx
│   │   │   │       └── analysis/page.tsx
│   │   │   ├── settings/
│   │   │   │   ├── page.tsx        # General settings
│   │   │   │   ├── team/page.tsx
│   │   │   │   ├── billing/page.tsx
│   │   │   │   └── api-keys/page.tsx
│   │   │   └── templates/page.tsx  # Extraction templates
│   │   ├── s/[surveyId]/page.tsx   # Public survey form
│   │   └── api/
│   │       ├── documents/
│   │       │   ├── upload/route.ts
│   │       │   ├── process/route.ts
│   │       │   └── export/route.ts
│   │       ├── surveys/
│   │       │   ├── route.ts        # CRUD surveys
│   │       │   ├── [id]/responses/route.ts
│   │       │   └── [id]/analyze/route.ts
│   │       ├── ai/
│   │       │   ├── extract/route.ts    # Document extraction
│   │       │   └── analyze/route.ts    # Feedback analysis
│   │       ├── stripe/
│   │       │   ├── checkout/route.ts
│   │       │   └── webhook/route.ts
│   │       └── health/route.ts
│   ├── components/
│   │   ├── ui/                     # shadcn/ui components
│   │   ├── layout/
│   │   │   ├── sidebar.tsx
│   │   │   ├── topbar.tsx
│   │   │   └── mobile-nav.tsx
│   │   ├── landing/
│   │   │   ├── hero.tsx
│   │   │   ├── features.tsx
│   │   │   ├── pricing-cards.tsx
│   │   │   ├── testimonials.tsx
│   │   │   └── footer.tsx
│   │   ├── documents/
│   │   │   ├── upload-zone.tsx
│   │   │   ├── document-card.tsx
│   │   │   ├── extraction-results.tsx
│   │   │   └── template-editor.tsx
│   │   ├── surveys/
│   │   │   ├── survey-builder.tsx
│   │   │   ├── question-types.tsx
│   │   │   ├── survey-preview.tsx
│   │   │   ├── response-chart.tsx
│   │   │   └── analysis-report.tsx
│   │   ├── dashboard/
│   │   │   ├── stats-cards.tsx
│   │   │   ├── recent-activity.tsx
│   │   │   └── usage-chart.tsx
│   │   └── shared/
│   │       ├── loading-spinner.tsx
│   │       ├── empty-state.tsx
│   │       ├── error-boundary.tsx
│   │       └── confirm-dialog.tsx
│   ├── lib/
│   │   ├── supabase/
│   │   │   ├── client.ts           # Browser client
│   │   │   ├── server.ts           # Server client
│   │   │   └── middleware.ts        # Auth helpers
│   │   ├── openai.ts               # OpenAI client + helpers
│   │   ├── stripe.ts               # Stripe client + helpers
│   │   ├── utils.ts                # General utilities
│   │   └── constants.ts            # App constants, plan limits
│   ├── hooks/
│   │   ├── use-user.ts
│   │   ├── use-subscription.ts
│   │   └── use-usage.ts
│   └── types/
│       ├── database.ts             # Supabase generated types
│       ├── api.ts                  # API request/response types
│       └── index.ts                # Shared types
```

---

## Database Schema

### Tables

```sql
-- Users (extends Supabase auth.users)
CREATE TABLE public.profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT NOT NULL,
  full_name TEXT,
  avatar_url TEXT,
  organization_id UUID REFERENCES organizations(id),
  role TEXT DEFAULT 'member' CHECK (role IN ('owner', 'admin', 'member')),
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- Organizations (teams)
CREATE TABLE public.organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  stripe_customer_id TEXT,
  stripe_subscription_id TEXT,
  plan TEXT DEFAULT 'free' CHECK (plan IN ('free', 'pro', 'enterprise')),
  plan_period_end TIMESTAMPTZ,
  doc_quota INTEGER DEFAULT 10,
  survey_quota INTEGER DEFAULT 5,
  doc_used INTEGER DEFAULT 0,
  survey_used INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Documents
CREATE TABLE public.documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  uploaded_by UUID NOT NULL REFERENCES profiles(id),
  file_name TEXT NOT NULL,
  file_url TEXT NOT NULL,
  file_size INTEGER,
  file_type TEXT,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'processing', 'completed', 'failed')),
  extracted_data JSONB,
  template_id UUID REFERENCES extraction_templates(id),
  processing_time_ms INTEGER,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Extraction Templates
CREATE TABLE public.extraction_templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  description TEXT,
  fields JSONB NOT NULL, -- [{name, type, description}]
  category TEXT, -- 'invoice', 'contract', 'receipt', 'custom'
  is_default BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Surveys
CREATE TABLE public.surveys (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  created_by UUID NOT NULL REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  questions JSONB NOT NULL, -- [{id, type, text, options?, required}]
  status TEXT DEFAULT 'draft' CHECK (status IN ('draft', 'active', 'closed')),
  settings JSONB DEFAULT '{}', -- {anonymous, allowMultiple, closeDate}
  response_count INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- Survey Responses
CREATE TABLE public.survey_responses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  survey_id UUID NOT NULL REFERENCES surveys(id) ON DELETE CASCADE,
  answers JSONB NOT NULL, -- {questionId: answer}
  respondent_email TEXT,
  metadata JSONB, -- {userAgent, ip_country, etc.}
  created_at TIMESTAMPTZ DEFAULT now()
);

-- AI Analyses (cached feedback analyses)
CREATE TABLE public.analyses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  survey_id UUID NOT NULL REFERENCES surveys(id) ON DELETE CASCADE,
  analysis_type TEXT NOT NULL, -- 'themes', 'sentiment', 'summary', 'full'
  results JSONB NOT NULL,
  model_used TEXT DEFAULT 'gpt-4o',
  tokens_used INTEGER,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- API Keys
CREATE TABLE public.api_keys (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  key_hash TEXT NOT NULL,
  key_prefix TEXT NOT NULL, -- 'nxai_...' first 8 chars
  last_used_at TIMESTAMPTZ,
  expires_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Activity Log
CREATE TABLE public.activity_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id),
  action TEXT NOT NULL,
  resource_type TEXT, -- 'document', 'survey', 'template'
  resource_id UUID,
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### RLS Policies
- All tables: users can only access rows where `organization_id` matches their own
- `survey_responses`: public INSERT allowed (for respondents), SELECT only for org members
- `profiles`: users can read/update their own profile

---

## Stripe Pricing Plans

| Feature | Free | Pro ($29/mo) | Enterprise ($99/mo) |
|---------|------|-------------|-------------------|
| Documents/month | 10 | 200 | Unlimited |
| Surveys | 5 | 50 | Unlimited |
| AI analyses/month | 5 | 100 | Unlimited |
| Team members | 1 | 5 | 25 |
| Export formats | CSV | CSV, Excel, JSON | All + API |
| API access | No | Yes | Yes |
| Custom templates | 3 | 20 | Unlimited |
| Priority support | No | No | Yes |

---

## API Routes Design

| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/documents/upload` | Upload doc to Supabase Storage |
| POST | `/api/documents/process` | Trigger GPT-4o extraction |
| POST | `/api/documents/export` | Export results to CSV/Excel/JSON |
| GET/POST | `/api/surveys` | List/Create surveys |
| POST | `/api/surveys/[id]/responses` | Submit survey response (public) |
| POST | `/api/surveys/[id]/analyze` | Trigger AI analysis |
| POST | `/api/stripe/checkout` | Create Stripe checkout session |
| POST | `/api/stripe/webhook` | Stripe webhook handler |
| GET | `/api/health` | Health check |

---

## Implementation Order (13 Steps)

### Phase 1: Foundation (Steps 1-3)
1. **Project setup** - Next.js + TypeScript + Tailwind + shadcn/ui + ESLint
2. **Supabase setup** - Schema migration, RLS policies, Auth config
3. **Auth flow** - Login, signup, OAuth callback, middleware, user context

### Phase 2: Core Shell (Steps 4-5)
4. **Dashboard layout** - Sidebar, topbar, responsive shell, navigation
5. **Landing page** - Hero, features, pricing cards, footer

### Phase 3: DocAI Module (Steps 6-7)
6. **Document upload & library** - Upload zone, file storage, document list
7. **AI extraction & export** - GPT-4o Vision API, extraction templates, CSV/Excel/JSON export

### Phase 4: FeedbackAI Module (Steps 8-10)
8. **Survey builder** - Create surveys, question types, preview
9. **Public survey form & responses** - Public URL, response collection, response list
10. **AI analysis & reports** - Theme detection, sentiment, charts, summary reports

### Phase 5: Monetization & Polish (Steps 11-13)
11. **Stripe integration** - Checkout, webhook, plan enforcement, billing page
12. **Settings & team** - Profile, team invite, API keys, usage tracking
13. **Final polish** - Loading states, error boundaries, empty states, SEO meta tags

---

## Verification Plan

1. `npm run build` - Ensure zero build errors
2. `npm run lint` - Clean lint
3. Manual test: signup, login, upload a PDF, verify extraction
4. Manual test: create survey, submit response, run analysis
5. Stripe test mode: subscribe, verify webhook, check plan upgrade
6. Mobile responsive check on all pages
7. `.env.local.example` has all required variables documented
