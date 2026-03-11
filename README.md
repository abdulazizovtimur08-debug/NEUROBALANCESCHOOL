# NeuroBalance School Platform

> A secure, multilingual school mental health and counseling platform for private schools.

---

## Overview

NeuroBalance is a production-ready web platform that enables:

- **Students & Staff** – complete psychological screenings, submit confidential help requests, attend online/offline counseling sessions
- **Psychologists** – manage cases, schedule appointments, write confidential session notes, generate reports
- **Directors** – view anonymized KPI dashboards and school-wide mental health analytics
- **Admins** – manage users, permissions, and school settings

**Supported Languages:** O'zbek (default) · Русский · English

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS + custom design tokens |
| Components | shadcn/ui + Radix UI primitives |
| Icons | lucide-react |
| Auth | Firebase Authentication |
| Database | Cloud Firestore |
| Push Notifications | Firebase Cloud Messaging |
| Charts | Recharts |
| Forms | react-hook-form + Zod |
| State | React Context + Zustand |
| Deployment | Vercel + Firebase |

---

## Project Structure

```
neurobalance/
├── app/                        # Next.js App Router
│   ├── auth/sign-in/           # Sign-in page
│   ├── dashboard/              # Role-aware dashboard
│   ├── screening/              # 3-stage psychological screening
│   ├── help-request/           # Confidential help request form
│   ├── schedule/               # Appointment calendar
│   ├── workspace/              # Psychologist case management
│   ├── notes/                  # Private session notes (psychologist only)
│   ├── feedback/               # Post-session rating & feedback
│   ├── director/               # KPI dashboard (director only)
│   ├── ai-advisor/             # AI advisor chat (with escalation)
│   ├── profile/                # User profile & settings
│   └── globals.css             # Design tokens + global styles
├── components/
│   ├── shared/                 # AppShell, Sidebar, etc.
│   ├── screening/              # Screening-specific components
│   ├── dashboard/              # Dashboard widgets
│   └── ...
├── lib/
│   ├── firebase.ts             # Firebase initialization
│   ├── auth-context.tsx        # Auth React context
│   ├── i18n.tsx                # i18n context + hook
│   ├── rbac.ts                 # Role-Based Access Control
│   ├── scoring-engine.ts       # Screening score calculation
│   ├── screening-questions.ts  # All questions (3 languages)
│   └── seed-data.ts            # Demo data seeder
├── services/
│   └── notifications.ts        # Notification abstraction
├── types/
│   └── index.ts                # All TypeScript types
├── locales/
│   ├── uz/common.json          # Uzbek translations
│   ├── ru/common.json          # Russian translations
│   └── en/common.json          # English translations
├── firestore.rules             # Firestore security rules
└── .env.local.example          # Environment variable template
```

---

## Quick Start

### 1. Clone and install

```bash
git clone https://github.com/yourorg/neurobalance.git
cd neurobalance
npm install
```

### 2. Configure Firebase

1. Go to [Firebase Console](https://console.firebase.google.com)
2. Create a new project
3. Enable **Authentication** → Email/Password
4. Create a **Firestore** database (start in production mode)
5. Copy your config values:

```bash
cp .env.local.example .env.local
# Edit .env.local with your Firebase credentials
```

### 3. Deploy Firestore Security Rules

```bash
npm install -g firebase-tools
firebase login
firebase init firestore
firebase deploy --only firestore:rules
```

### 4. Seed demo data (optional)

```typescript
// In your browser console or a seed script:
import { seedDemoData } from '@/lib/seed-data';
await seedDemoData();
```

Demo accounts after seeding:
| Email | Password | Role |
|---|---|---|
| student@demo.school.uz | demo123 | Student |
| psixolog@demo.school.uz | demo123 | Psychologist |
| direktor@demo.school.uz | demo123 | Director |
| admin@demo.school.uz | demo123 | Admin |

### 5. Run development server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

---

## Role Permissions Matrix

| Feature | Student | Staff | Psychologist | Director | Admin |
|---|:---:|:---:|:---:|:---:|:---:|
| Take screening | ✓ | ✓ | ✓ | – | ✓ |
| View own screening | ✓ | ✓ | ✓ | – | ✓ |
| View all screenings | – | – | ✓ | aggregate | ✓ |
| Submit help request | ✓ | ✓ | – | – | ✓ |
| View own requests | ✓ | ✓ | – | – | ✓ |
| Manage all requests | – | – | ✓ | – | ✓ |
| Create appointment | – | – | ✓ | – | ✓ |
| View own appointments | ✓ | ✓ | ✓ | – | ✓ |
| Write session notes | – | – | ✓ | – | ✓ |
| Read session notes | – | – | own only | ❌ | ✓ |
| Leave feedback | ✓ | ✓ | – | – | ✓ |
| View KPI dashboard | – | – | – | ✓ | ✓ |
| Manage users | – | – | – | – | ✓ |
| AI Advisor | ✓ | ✓ | ✓ | – | ✓ |

---

## Firestore Collections

| Collection | Purpose | Who can read |
|---|---|---|
| `/schools/{id}` | School config | Same school members |
| `/users/{uid}` | User profiles | Self + psych/admin |
| `/screeningResults/{id}` | Screening submissions | Owner + psychologist |
| `/helpRequests/{id}` | Help requests | Owner + psychologist + admin |
| `/appointments/{id}` | Scheduled sessions | Student + psychologist |
| `/sessionNotes/{id}` | **PRIVATE** notes | Psychologist only |
| `/feedback/{id}` | Session ratings | Psychologist (own) |
| `/notifications/{id}` | In-app notifications | Recipient only |
| `/schoolKPIs/{id}` | Aggregated analytics | Director + admin |

---

## i18n Setup

All UI strings are stored in `locales/{lang}/common.json`.

```typescript
// Usage in any component:
import { useI18n } from '@/lib/i18n';

function MyComponent() {
  const { t, language, setLanguage } = useI18n();
  return <h1>{t('dashboard.title')}</h1>;
}
```

Translation keys follow dot notation: `section.key.subkey`

---

## Screening Scoring Engine

The scoring engine in `lib/scoring-engine.ts`:

1. Takes Likert 1–5 answers
2. Normalizes each to 0–100 (reverse-scored questions are flipped)
3. Groups by indicator (mood, stress, social, sleep, relationship)
4. Calculates per-indicator average
5. Computes total score (average of all indicators)
6. Maps to risk level: `none (≥75)` · `low (≥55)` · `moderate (≥40)` · `high (≥25)` · `critical (<25)`

---

## Deployment

### Vercel

```bash
npm install -g vercel
vercel login
vercel --prod
```

Add all `NEXT_PUBLIC_*` environment variables in your Vercel project settings.

### Firebase

```bash
# Deploy Firestore rules
firebase deploy --only firestore:rules

# Deploy Cloud Functions (for KPI aggregation, FCM triggers)
firebase deploy --only functions
```

---

## Safety & Privacy

- **Session notes** are never exposed to directors, students, or staff (enforced at Firestore rules level)
- **Director dashboard** shows only anonymized aggregate data
- **AI Advisor** detects high-risk language and shows escalation guidance instead of normal responses
- **Anonymous submissions** are supported for screenings and help requests
- All data access is school-scoped (multi-tenancy ready)

---

## Roadmap

- [ ] Firebase Cloud Functions for KPI aggregation
- [ ] FCM push notification service worker
- [ ] Google Calendar integration for appointments
- [ ] PDF report generation for psychologists
- [ ] Multi-school admin portal
- [ ] Parent/guardian portal
- [ ] Mobile app (React Native)

---

## License

MIT – see LICENSE file
