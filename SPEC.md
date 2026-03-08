# NYC Building Intel — Product Spec

## Vision

A fast, mobile-first building intelligence tool for NYC property professionals. Search any address, get an instant building health score, and get alerted when anything changes. Simpler and cheaper than the competition, better on mobile than anyone.

---

## MVP Scope (v1)

### In
- Address search → building profile page
- Building Health Score (composite, 0–100)
- DOB violations (open + resolved)
- HPD complaints (heat, water, pests, structural)
- 311 complaints (noise, garbage, conditions)
- Active permits (construction, alteration, renovation)
- Save/watch up to 5 buildings (free tier)
- Email alerts when violations or permits change on watched buildings
- Web app
- iOS app (SwiftUI, mirrors web functionality)

### Out (v2+)
- PLUTO property attributes (zoning, tax assessment, lot size)
- Bedbug report history
- Push notifications (iOS only, v2)
- Team/multi-user accounts
- API access tier
- Map view / neighborhood heat map
- Historical score trending

---

## Data Sources (all free via Socrata API)

| Dataset | Agency | Key Fields |
|---------|--------|------------|
| DOB Violations | Dept. of Buildings | violation type, date, status, disposition |
| DOB Permits | Dept. of Buildings | permit type, filing date, work description |
| HPD Violations | Housing Preservation | class (A/B/C), description, open/closed |
| HPD Complaints | Housing Preservation | complaint type, status, date |
| 311 Service Requests | NYC311 | complaint type, date, resolution |
| DOHMH Inspections | Health Dept. | grade, score, date |

Base URL: `https://data.cityofnewyork.us/resource/{dataset-id}.json`
Auth: App token (free, 1000 req/hour unauthenticated, higher with token)

---

## Building Health Score

Composite 0–100 score. Lower = more problems.

### Formula (v1)

```
score = 100
  - open_dob_violations * 8        (max -40)
  - open_hpd_class_c * 10          (serious: heat/water/structural, max -30)
  - open_hpb_class_b * 4           (moderate, max -20)
  - active_311_last_90d * 2        (max -10)
  - active_permits_major * 5       (major construction, max -15)
  + clamp to 0 minimum
```

### Grade Bands
- 80–100: 🟢 Good
- 60–79: 🟡 Fair
- 40–59: 🟠 Concerning
- 0–39: 🔴 Poor

*Formula will be tuned after seeing real data distribution.*

---

## Data Model (PostgreSQL)

```sql
-- Core building record (cached from Socrata + geocoding)
buildings (
  id UUID PRIMARY KEY,
  address TEXT NOT NULL,
  borough TEXT,
  bin TEXT,           -- Building Identification Number (DOB key)
  bbl TEXT,           -- Borough-Block-Lot (HPD/PLUTO key)
  latitude FLOAT,
  longitude FLOAT,
  building_class TEXT,
  year_built INT,
  num_units INT,
  health_score INT,
  score_updated_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
)

-- Violations, complaints, permits (normalized)
violations (
  id UUID PRIMARY KEY,
  building_id UUID REFERENCES buildings(id),
  source TEXT,        -- 'DOB' | 'HPD'
  violation_type TEXT,
  severity TEXT,      -- 'A' | 'B' | 'C' | null
  description TEXT,
  status TEXT,        -- 'OPEN' | 'CLOSED'
  issued_date DATE,
  closed_date DATE,
  raw JSONB           -- original Socrata record
)

complaints (
  id UUID PRIMARY KEY,
  building_id UUID REFERENCES buildings(id),
  source TEXT,        -- 'HPD' | '311'
  complaint_type TEXT,
  status TEXT,
  filed_date DATE,
  closed_date DATE,
  raw JSONB
)

permits (
  id UUID PRIMARY KEY,
  building_id UUID REFERENCES buildings(id),
  permit_type TEXT,
  description TEXT,
  status TEXT,
  filed_date DATE,
  expiration_date DATE,
  raw JSONB
)

-- User watchlist
watched_buildings (
  id UUID PRIMARY KEY,
  user_id UUID,
  building_id UUID REFERENCES buildings(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
)

-- Alert log
alerts (
  id UUID PRIMARY KEY,
  building_id UUID REFERENCES buildings(id),
  alert_type TEXT,    -- 'new_violation' | 'new_permit' | 'score_change'
  detail TEXT,
  sent_at TIMESTAMPTZ
)
```

---

## Backend API (FastAPI)

```
GET  /buildings/search?q={address}        Search by address (geocode + lookup)
GET  /buildings/{id}                      Full building profile
GET  /buildings/{id}/score                Health score + breakdown
GET  /buildings/{id}/violations           Violations list
GET  /buildings/{id}/complaints           Complaints list
GET  /buildings/{id}/permits              Active permits
POST /watch                               Add building to watchlist
GET  /watch                               Get watchlist
DELETE /watch/{building_id}               Remove from watchlist
GET  /alerts                              Alert history for watchlist
```

---

## Screens

### Web + iOS (same structure)

1. **Home / Search** — address search bar, recent searches, watchlist summary
2. **Building Profile** — health score badge, address details, tab bar (Overview / Violations / Complaints / Permits)
3. **Overview Tab** — score breakdown, key stats at a glance
4. **Violations Tab** — filterable list (DOB/HPD, open/closed, severity)
5. **Complaints Tab** — filterable list (source, type, date range)
6. **Permits Tab** — active permits with descriptions
7. **Watchlist** — list of saved buildings with scores + change indicators
8. **Alerts** — chronological feed of changes across watchlist

---

## Monetization (v1)

| Tier | Price | Limits |
|------|-------|--------|
| Free | $0 | 3 searches/month, 1 watched building, email alerts weekly |
| Pro | $24/month | Unlimited searches, 25 watched buildings, daily alerts |
| Team | $89/month | 5 users, 100 watched buildings, instant alerts + API access |

Stripe for billing. Simple paywall — no complex feature gating in v1.

---

## Technical Architecture

```
iOS App (SwiftUI)
     ↕
FastAPI Backend (Python)
     ↕                    ↕
PostgreSQL DB      NYC Open Data
(cache layer)      (Socrata API)
```

- Backend caches Socrata data in Postgres (TTL: 24h for scores, 1h for live checks)
- Address geocoding via NYC GeoSearch API (free, no key required)
- BIN lookup via DOB BIS API (free)
- BBL lookup via PLUTO / GeoClient API
- Background job (cron) refreshes watched buildings every 6 hours, triggers alerts on changes

---

## Build Order

1. **Data layer** — Socrata API wrappers for DOB, HPD, 311
2. **Address → BIN/BBL resolution** (geocoding pipeline)
3. **Score calculation** — build + test formula against real buildings
4. **Backend API** — search, profile, score endpoints
5. **Web frontend** — search + building profile (React or simple HTML/JS)
6. **Watchlist + alerts** — email alerts via SendGrid or Resend
7. **iOS app** — SwiftUI mirrors web, push notifications
8. **Stripe billing** — paywall on Pro/Team features
9. **Launch** — Product Hunt, NYC real estate communities, Reddit r/NYCapartments

---

## Estimated Build Time

| Phase | Time |
|-------|------|
| Data layer + scoring | 1 week |
| Backend API | 1 week |
| Web frontend | 1 week |
| Watchlist + alerts | 3-4 days |
| iOS app | 1-2 weeks |
| Billing + polish | 3-4 days |
| **Total** | **~5-6 weeks** |

---

## Open Questions

- [ ] NYC GeoClient API key needed? (free but requires registration)
- [ ] Email provider for alerts (SendGrid / Resend / Postmark)?
- [ ] Web framework preference (React / Next.js / plain HTML)?
- [ ] App name / branding (nyc-building-intel is a working title)
- [ ] Stripe account setup
