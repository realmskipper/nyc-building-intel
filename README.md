# NYC Building Intel

> Building health scores, violation alerts, and permit tracking for NYC property professionals.

A fast, mobile-first alternative to D.O.B. Guard and ViolationWatch — built on NYC Open Data (Socrata API) with a cleaner UX and lower price point.

---

## What It Does

Search any NYC address and instantly see:

- **Building Health Score** — composite score from violations, complaints, and inspection history
- **DOB Violations** — open and resolved Department of Buildings violations
- **HPD Complaints** — housing maintenance complaints (heat, water, pests, etc.)
- **311 Complaints** — noise, garbage, conditions over time
- **Active Permits** — construction, renovation, and alteration permits
- **Inspection Grades** — DOHMH and other agency inspection history
- **Alerts** — push/email notifications when anything changes on a watched building

---

## Target Audience

B2B — property managers, landlords, real estate attorneys, brokers, and building inspectors who need to monitor multiple properties and stay ahead of violations.

---

## Business Model

| Tier | Price | Includes |
|------|-------|----------|
| Free | $0 | 3 address searches/month |
| Pro | $19-29/month | Unlimited searches + violation alerts |
| Team | $79-99/month | Multiple users + API access |

---

## Data Sources

All free via [NYC Open Data (Socrata API)](https://data.cityofnewyork.us/):

- **DOB NOW**: Violations, permits, complaints
- **HPD**: Housing maintenance code violations, complaints
- **311 Service Requests**: All complaint types by address
- **DOHMH**: Inspection grades
- **PLUTO**: Property attributes, zoning, building age, tax data

---

## Tech Stack

- **Backend**: FastAPI + PostgreSQL + asyncpg
- **Frontend**: iOS (SwiftUI) + Web
- **Data**: NYC Open Data Socrata API (no scraping required)
- **Alerts**: Push notifications + email

---

## Competitive Landscape

| Product | Price | Mobile | Target |
|---------|-------|--------|--------|
| D.O.B. Guard | $50/month | Web only | Landlords |
| ViolationWatch | $40/month | Web only | Property managers |
| Augrented | Freemium | Limited | Renters |
| **NYC Building Intel** | $19-29/month | iOS + Web | Property professionals |

---

## Status

🚧 Pre-development — spec and architecture in progress.
