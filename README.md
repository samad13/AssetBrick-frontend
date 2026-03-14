# AssetBrick-frontend
# 🖥️ Real Estate Fractional Ownership Platform — Frontend

![Next.js](https://img.shields.io/badge/Framework-Next.js-black?logo=nextdotjs)
![TypeScript](https://img.shields.io/badge/Language-TypeScript-blue?logo=typescript)
![MUI](https://img.shields.io/badge/UI-MUI-007FFF?logo=mui)
![License](https://img.shields.io/badge/License-MIT-green)

> The investor-facing web application for the fractional real estate platform. Browse properties, manage your portfolio, trade shares, and track dividend income — all in one place.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Features](#features)
- [How It Works With the Backend](#how-it-works-with-the-backend)
- [Application Architecture](#application-architecture)
- [Project Structure](#project-structure)
- [Page Breakdown](#page-breakdown)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Development Phases](#development-phases)
- [Design Principles](#design-principles)

---

## Overview

This is the investor-facing Next.js web app for the fractional real estate platform. It connects to the NestJS backend via REST API and WebSockets to provide a real-time investment experience — from browsing properties to executing trades and receiving dividend payouts.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript |
| UI Components | MUI (Material UI) |
| Data Fetching | React Query (TanStack) |
| State Management | Zustand |
| Forms | React Hook Form + Zod |
| Charts | Recharts |
| Auth | JWT (stored in httpOnly cookie) |
| Real-time | WebSocket (Socket.io client) |

---

## Features

### For Investors
- **Portfolio Dashboard** — Live view of all owned shares, current value, and performance
- **Property Marketplace** — Browse available properties, view financials, buy fractional shares
- **Secondary Market** — Order book UI for buying/selling shares peer-to-peer
- **Dividend History** — Full log of all USDC dividend payouts received
- **Wallet Management** — Deposit and withdraw USDC, view Stellar wallet address
- **KYC Onboarding** — Guided identity verification before investing

### For Admins *(separate route group)*
- Property listing management
- Investor management
- Dividend trigger and audit log
- Platform analytics

---

## How It Works With the Backend

The frontend is a **pure client** — it holds no financial logic. All calculations, ownership records, and Stellar transactions are handled entirely by the backend.

```
┌─────────────────────────────────┐
│         Next.js Frontend        │
│                                 │
│  Pages → Service Layer → API    │
│                  │              │
│          React Query Cache      │
└──────────────────┼──────────────┘
                   │ REST + WebSocket
┌──────────────────▼──────────────┐
│         NestJS Backend          │
│  Auth │ Trading │ Dividends...  │
└─────────────────────────────────┘
```

**How data flows:**

1. Pages call **service functions** (e.g., `portfolioService.getHoldings()`)
2. Services use **axios** to call the backend REST API with the JWT token
3. Responses are **cached and managed by React Query**
4. Real-time updates (order book, trade confirmations) arrive via **WebSocket**
5. Forms submit via service functions — never directly to blockchain or Stellar

**Protected Routes:**
All routes under `/dashboard`, `/marketplace`, `/wallet`, and `/portfolio` require an authenticated JWT. The Next.js middleware checks for a valid token and redirects to `/login` if absent.

---

## Application Architecture

```
Next.js App Router
│
├── Middleware (auth guard)
│
├── Public Routes
│   ├── /                 → Landing page
│   ├── /login            → Auth
│   ├── /register         → Sign up + KYC start
│   └── /properties/[id]  → Public property view
│
└── Protected Routes
    ├── /dashboard         → Portfolio overview
    ├── /marketplace       → Browse + buy properties
    ├── /portfolio         → Holdings + performance
    ├── /trade             → Secondary market order book
    ├── /dividends         → Payout history
    └── /wallet            → USDC wallet management
```

---

## Project Structure

```
realestate-platform-frontend/
│
├── app/
│   ├── (public)/
│   │   ├── page.tsx           ← Landing page
│   │   ├── login/
│   │   └── register/
│   │
│   ├── (protected)/
│   │   ├── dashboard/
│   │   ├── marketplace/
│   │   ├── portfolio/
│   │   ├── trade/
│   │   ├── dividends/
│   │   └── wallet/
│   │
│   └── layout.tsx
│
├── components/
│   ├── ui/                    ← Reusable base components
│   ├── property/              ← Property card, detail view
│   ├── portfolio/             ← Holdings table, performance chart
│   ├── trading/               ← Order book, trade form
│   ├── dividends/             ← Payout table, timeline
│   └── wallet/                ← Balance display, transfer form
│
├── hooks/
│   ├── useAuth.ts
│   ├── usePortfolio.ts
│   ├── useOrderBook.ts
│   └── useWallet.ts
│
├── services/
│   ├── api.ts                 ← Axios instance + interceptors
│   ├── auth.service.ts
│   ├── property.service.ts
│   ├── trading.service.ts
│   ├── dividend.service.ts
│   └── wallet.service.ts
│
├── store/
│   └── auth.store.ts          ← Zustand global auth state
│
├── types/
│   └── index.ts               ← Shared TypeScript interfaces
│
├── lib/
│   └── queryClient.ts         ← React Query config
│
└── middleware.ts              ← Auth route guard
```

---

## Page Breakdown

### `/dashboard`
- Portfolio value summary card
- Shares owned per property
- Recent dividend payouts (last 3)
- Recent trade activity

### `/marketplace`
- Property listing cards with projected yield, location, share price
- Filter by city, yield range, availability
- Property detail modal with full financials
- "Buy Shares" flow with quantity selector and USDC cost preview

### `/trade` *(Phase 2+)*
- Live order book (bids/asks) per property
- Place limit or market orders
- Open orders management
- Trade history table

### `/dividends`
- Monthly payout history table
- Per-property breakdown
- USDC amounts with Stellar transaction links
- Cumulative earnings chart

### `/wallet`
- USDC balance (internal ledger + Stellar wallet)
- Deposit instructions (Stellar address QR code)
- Withdraw form (to external Stellar wallet)
- Transaction history

---

## Getting Started

### Prerequisites

- Node.js >= 18
- Backend API running locally (see [Backend README](../backend/README.md))

### Installation

```bash
git clone https://github.com/your-org/realestate-platform-frontend.git
cd realestate-platform-frontend

cp .env.local.example .env.local
# Set NEXT_PUBLIC_API_URL to your backend URL

npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

---

## Environment Variables

```env
# Backend API
NEXT_PUBLIC_API_URL=http://localhost:3001

# WebSocket
NEXT_PUBLIC_WS_URL=ws://localhost:3001

# App
NEXT_PUBLIC_APP_NAME=PropShare
NEXT_PUBLIC_STELLAR_NETWORK=testnet
```

---

## Development Phases

| Phase | Frontend Scope |
|---|---|
| **MVP** | Auth flow, property listings, share purchase, portfolio view, admin dashboard |
| **Marketplace** | Order book UI, trade form, open orders, trade history |
| **Stellar** | Wallet page, deposit/withdraw, USDC balance, Stellar tx links in dividend history |
| **Production** | Performance optimization, accessibility audit, mobile responsiveness |

---

## Design Principles

- **Data lives in the backend** — the frontend never performs financial calculations
- **Optimistic UI** — React Query updates the UI instantly, rolls back on error
- **Protected by default** — all routes require auth unless explicitly public
- **Real-time where it matters** — order book and trade confirmation via WebSocket
- **Type-safe** — all API responses typed end-to-end with TypeScript interfaces

---

> See [`CONTRIBUTING.md`](../CONTRIBUTING.md) for how to contribute to this project.
