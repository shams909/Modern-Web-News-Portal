# Modern-Web-News-Portal

# 🏗️ System Architecture - Chottola News X

> Complete technical documentation of the bilingual news portal system architecture, data flow, and infrastructure.

[![Next.js](https://img.shields.io/badge/Next.js-16-black?logo=next.js)](https://nextjs.org/)
[![Fastify](https://img.shields.io/badge/Fastify-5-black?logo=fastify)](https://fastify.dev/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-blue?logo=postgresql)](https://www.postgresql.org/)
[![Vercel](https://img.shields.io/badge/Deployed%20on-Vercel-black?logo=vercel)](https://vercel.com)
[![Railway](https://img.shields.io/badge/Backend-Railway-purple?logo=railway)](https://railway.app)
[![Neon](https://img.shields.io/badge/Database-Neon-auroragreen?logo=neon)](https://neon.tech)
[![Cloudinary](https://img.shields.io/badge/Cloudinary-Cloudinary-blue?logo=cloudinary)](https://cloudinary.com)
[![Vercel Analytics](https://img.shields.io/badge/Analytics-Vercel-black?logo=vercel)](https://vercel.com)
---

## 📊 High-Level Architecture

```mermaid
graph TB
    subgraph "User's Browser"
        A[User Visits Site]
    end

    subgraph "Vercel CDN Edge Network"
        B[Vercel Edge Cache]
        C[Next.js Frontend]
        D[Vercel Analytics]
    end

    subgraph "Railway Backend"
        E[Fastify Server]
        F[Prisma Client Singleton]
        G[Auth Middleware]
        H[API Routes]
    end

    subgraph "Neon Database"
        I[PostgreSQL Pooler]
        J[Database Compute]
        K[Database Storage]
    end

    subgraph "Cloudinary CDN"
        L[Image Storage]
        M[Image Optimization]
    end

    A -->|HTTPS Request| B
    B -->|Cache Miss| C
    C -->|API Call| E
    E --> G
    G --> H
    H --> F
    F -->|Connection Pool| I
    I -->|Query| J
    J -->|Data| K
    E -->|Upload Image| L
    L --> M
    M -->|Optimized URL| E
    C -->|Track Event| D
    
    style A fill:#e1f5ff
    style C fill:#00d4ff
    style E fill:#7c3aed
    style J fill:#10b981
    style L fill:#f59e0b
```

---

## 🔄 Complete User Journey

### When a User Visits Your Site:

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Vercel
    participant Railway
    participant Neon
    participant Cloudinary
    participant Analytics

    User->>Browser: Types URL or clicks link
    Browser->>Vercel: HTTPS Request (GET /)
    
    Note over Vercel: Edge Network (Global CDN)
    Vercel->>Vercel: Check Edge Cache
    
    alt Cache Hit (Static Content)
        Vercel-->>Browser: Return Cached HTML/CSS/JS
    else Cache Miss
        Vercel->>Vercel: Server-Side Render (Next.js)
        Vercel->>Railway: API Call (GET /api/articles)
        
        Note over Railway: Fastify Backend
        Railway->>Railway: Auth Check (if needed)
        Railway->>Neon: Database Query
        
        Note over Neon: Auto-wake if suspended
        Neon->>Neon: Execute SQL Query
        Neon-->>Railway: Return Data (JSON)
        
        Railway-->>Vercel: API Response
        Vercel->>Vercel: Render Page with Data
        Vercel-->>Browser: Send HTML + Data
    end
    
    Browser->>Browser: Parse HTML, Load CSS/JS
    Browser->>Cloudinary: Load Images
    Cloudinary->>Cloudinary: Optimize & Resize
    Cloudinary-->>Browser: Optimized Images
    
    Browser->>Analytics: Track Page View
    Analytics->>Analytics: Record Metrics
    
    Browser->>User: Display Complete Page
```

---

## 🎯 Component Architecture

### Frontend (Next.js on Vercel)

```mermaid
graph LR
    subgraph "Next.js App"
        A[App Router] --> B[Pages]
        B --> C[Components]
        C --> D[UI Components]
        
        B --> E[API Client]
        E --> F[Fetch Calls]
        
        B --> G[Internationalization]
        G --> H[EN/BN Translations]
    end
    
    style A fill:#00d4ff
    style E fill:#7c3aed
    style G fill:#10b981
```

**Technologies:**
- **Next.js 16**: React framework with App Router
- **TypeScript**: Type-safe development
- **Tailwind CSS**: Utility-first styling
- **next-intl**: Internationalization (English/Bengali)

---

### Backend (Fastify on Railway)

```mermaid
graph TB
    subgraph "Fastify Server"
        A[Server Entry] --> B[Middleware]
        B --> C[CORS]
        B --> D[Helmet Security]
        B --> E[Rate Limiting]
        
        A --> F[Route Registration]
        F --> G[Articles Routes]
        F --> H[Auth Routes]
        F --> I[Complaints Routes]
        F --> J[Categories Routes]
        F --> K[Ticker Routes]
        F --> L[Notices Routes]
        F --> M[Resolved Complaints]
        
        G --> N[Prisma Singleton]
        H --> N
        I --> N
        J --> N
        K --> N
        L --> N
        M --> N
    end
    
    style A fill:#7c3aed
    style N fill:#10b981
```

**Technologies:**
- **Fastify**: High-performance Node.js framework
- **Prisma**: Type-safe ORM with singleton pattern
- **JWT**: Stateless authentication
- **bcrypt**: Password hashing

---

### Database (Neon Serverless PostgreSQL)

```mermaid
graph LR
    subgraph "Neon Serverless PostgreSQL"
        A[Connection Pooler] --> B[Compute Layer]
        B --> C[Storage Layer]
        
        B --> D[Auto-Suspend Logic]
        D -->|5min idle| E[Suspended State]
        D -->|Request comes| F[Active State]
    end
    
    style A fill:#10b981
    style B fill:#3b82f6
    style C fill:#6366f1
```

**Key Features:**
- **Auto-suspend**: Reduces costs by 60-70%
- **Connection pooling**: Optimized for serverless
- **Instant wake**: ~1-2 seconds from suspended state

---

## 🚀 Request Flow Example: Reading an Article

```mermaid
sequenceDiagram
    autonumber
    
    participant U as User
    participant B as Browser
    participant V as Vercel Edge
    participant N as Next.js
    participant R as Railway API
    participant P as Prisma
    participant D as Neon DB
    participant C as Cloudinary
    participant A as Analytics

    U->>B: Clicks article link
    B->>V: GET /en/news/article-123
    V->>N: Server-side render
    N->>R: GET /api/articles/article-123
    R->>P: prisma.article.findUnique()
    P->>D: SELECT * FROM Article WHERE id=?
    
    alt Database Suspended
        D->>D: Wake up (1-2s)
    end
    
    D-->>P: Article data
    P->>P: Increment view count
    P->>D: UPDATE Article SET views++
    D-->>P: Success
    P-->>R: Article object
    R-->>N: JSON response
    N->>N: Render HTML with data
    N-->>V: Rendered page
    V-->>B: HTML + CSS + JS
    
    B->>C: Load article image
    C->>C: Optimize & resize
    C-->>B: Optimized image
    
    B->>A: Track page view
    A->>A: Record metrics
    
    B->>U: Display article
```

---

## 📁 Project Structure

```
webportalnewslatest/
├── frontend/                    # Next.js Frontend (Vercel)
│   ├── src/
│   │   ├── app/
│   │   │   ├── [locale]/       # Internationalized routes
│   │   │   │   ├── page.tsx    # Homepage
│   │   │   │   ├── news/       # Article pages
│   │   │   │   ├── complaints/ # Complaint system
│   │   │   │   ├── admin/      # Admin dashboard
│   │   │   │   └── ...
│   │   │   ├── layout.tsx      # Root layout + Analytics
│   │   │   └── globals.css     # Global styles
│   │   ├── components/
│   │   │   ├── common/         # Navbar, Footer
│   │   │   └── ui/             # Reusable UI components
│   │   ├── lib/
│   │   │   ├── api.ts          # API client functions
│   │   │   └── types.ts        # TypeScript types
│   │   └── i18n/               # Translations (EN/BN)
│   └── package.json
│
└── backend/                     # Fastify Backend (Railway)
    ├── src/
    │   ├── server.ts           # Main server entry
    │   ├── lib/
    │   │   └── prisma.ts       # Prisma singleton ⭐
    │   ├── routes/
    │   │   ├── articles.ts     # Article CRUD
    │   │   ├── auth.ts         # Login/logout
    │   │   ├── complaints.ts   # Complaint handling
    │   │   └── ...
    │   ├── middleware/
    │   │   └── auth.ts         # JWT verification
    │   └── utils/
    │       └── cloudinary.ts   # Image upload
    ├── prisma/
    │   └── schema.prisma       # Database schema
    └── package.json
```

---

## 🔐 Security Architecture

```mermaid
graph TB
    A[User Login] --> B[Frontend Form]
    B --> C[POST /api/auth/login]
    C --> D[Backend: Find Admin]
    D --> E[bcrypt.compare password]
    
    E -->|Valid| F[Generate JWT Token]
    E -->|Invalid| G[Return 401 Error]
    
    F --> H[Return Token to Frontend]
    H --> I[Store in localStorage]
    
    I --> J[Future Requests]
    J --> K[Add Authorization Header]
    K --> L[Backend: Verify JWT]
    
    L -->|Valid| M[Allow Access]
    L -->|Invalid| N[Return 401]
    
    style F fill:#10b981
    style G fill:#ef4444
    style M fill:#10b981
    style N fill:#ef4444
```

**Security Features:**
- JWT tokens with 7-day expiry
- bcrypt password hashing (10 rounds)
- Rate limiting (500 requests/minute)
- Helmet security headers
- CORS configuration

---

## 💾 Data Flow: Creating an Article

```mermaid
sequenceDiagram
    participant Admin
    participant Browser
    participant Vercel
    participant Railway
    participant Cloudinary
    participant Neon

    Admin->>Browser: Fill article form
    Admin->>Browser: Upload image
    Browser->>Railway: POST /api/articles (with JWT)
    Railway->>Railway: Verify JWT token
    
    alt Image Upload
        Railway->>Cloudinary: Upload image
        Cloudinary-->>Railway: Image URL
    end
    
    Railway->>Neon: INSERT INTO Article
    Neon-->>Railway: Created article
    Railway-->>Browser: Success response
    Browser->>Admin: Show success message
    
    Note over Vercel: Next request will show new article
```

---

## 📊 Monitoring Stack

```mermaid
graph TB
    subgraph "Monitoring Tools"
        A[Vercel Analytics] --> B[Page Views]
        A --> C[Performance]
        A --> D[Geographic Data]
        
        E[Railway Logs] --> F[API Requests]
        E --> G[Error Tracking]
        E --> H[Performance]
        
        I[Neon Dashboard] --> J[DB Activity]
        I --> K[Compute Usage]
        I --> L[Auto-Suspend Status]
    end
    
    style A fill:#696969
    style E fill:#7c3aed
    style I fill:#10b981
```

---

## 🌍 Global Infrastructure

```mermaid
graph TB
    subgraph "User Locations"
        A[Bangladesh]
        B[India]
        C[USA]
        D[Europe]
    end
    
    subgraph "Vercel Edge Network"
        E[Dhaka Edge]
        F[Mumbai Edge]
        G[New York Edge]
        H[London Edge]
    end
    
    subgraph "Backend"
        I[Railway US-East]
    end
    
    subgraph "Database"
        J[Neon US-East]
    end
    
    A --> E
    B --> F
    C --> G
    D --> H
    
    E --> I
    F --> I
    G --> I
    H --> I
    
    I --> J
    
    style E fill:#b98922
    style I fill:#7c3aed
    style J fill:#10b981
```

**Infrastructure Details:**
- **Frontend**: Served from nearest Vercel edge (global CDN)
- **Backend**: Railway US-East (single region)
- **Database**: Neon US-East (co-located with backend)
- **Images**: Cloudinary global CDN

---

## 🎯 Key Optimizations

### 1. Prisma Singleton Pattern

**Before:**
```typescript
// ❌ Each route file created its own instance
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();
```

**After:**
```typescript
// ✅ Shared singleton across all routes
import prisma from '../lib/prisma';
```

**Impact:**
- Reduced from 8 instances to 1
- Reduced from 80 potential connections to 1
- Enabled Neon auto-suspend
- **Saved 60-70% on database costs**

### 2. Neon Auto-Suspend

```
Traffic Pattern:
├─ Peak Hours (10 AM - 10 PM): Active (12 hours)
└─ Off-Peak (10 PM - 10 AM): Suspended (12 hours)

Cost Reduction: 50%+ savings
```

### 3. Vercel Edge Caching

- **Static Assets**: Cached at edge (instant delivery)
- **Dynamic Pages**: Server-rendered, then cached
- **API Calls**: Not cached (always fresh data)

### 4. Cloudinary Optimization

```
Original Image: 2MB
↓ Cloudinary Processing
Optimized WebP: 200KB (90% smaller!)
↓ CDN Distribution
Delivered from nearest node
```

---

## 🚀 Deployment Pipeline

```mermaid
graph LR
    A[Code Change] --> B[Git Commit]
    B --> C[Git Push]
    
    C --> D[Vercel Detects Push]
    D --> E[Build Frontend]
    E --> F[Deploy to Edge]
    
    C --> G[Railway Detects Push]
    G --> H[Build Backend]
    H --> I[Deploy to Server]
    I --> J[Run Prisma Migrations]
    
    style D fill:#10b981
    style G fill:#7c3aed
```

**Automatic Deployment Features:**
- Zero-downtime deployments
- Automatic rollback on failure
- Environment-specific builds
- Database migration automation

---

## 📈 Scaling Strategy

### Current Setup (0-5K visitors/day)

```yaml
Frontend: Vercel (unlimited scale)
Backend: Railway Hobby ($5/mo)
Database: Neon Free (1 connection)
Total Cost: $5-10/month
```

### Medium Traffic (5K-20K visitors/day)

```yaml
Frontend: Vercel (unlimited scale)
Backend: Railway Pro ($20/mo)
Database: Neon Pro (10 connections)
Caching: Redis (optional)
Total Cost: $40-60/month
```

### High Traffic (20K-100K visitors/day)

```yaml
Frontend: Vercel (unlimited scale)
Backend: Railway Pro + Load Balancer
Database: Neon Scale (dedicated compute)
Caching: Redis + Read Replicas
Total Cost: $150-250/month
```

---

## 💰 Cost Breakdown

| Service | Tier | Monthly Cost | Purpose |
|---------|------|--------------|---------|
| **Vercel** | Hobby | Free | Frontend hosting & CDN |
| **Railway** | Hobby | $5 | Backend API server |
| **Neon** | Free | Free-$8 | PostgreSQL database |
| **Cloudinary** | Free | Free | Image storage & CDN |
| **Vercel Analytics** | Free | Free | Traffic analytics |

**Total: $5-15/month** for a production news portal! 🎉

---

## 🛠️ Tech Stack Summary

### Frontend
- **Framework**: Next.js 16 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **i18n**: next-intl (English/Bengali)
- **Deployment**: Vercel

### Backend
- **Framework**: Fastify
- **Language**: TypeScript
- **ORM**: Prisma (singleton pattern)
- **Auth**: JWT + bcrypt
- **Deployment**: Railway

### Database
- **Type**: PostgreSQL 15
- **Provider**: Neon (serverless)
- **Features**: Auto-suspend, connection pooling

### Infrastructure
- **CDN**: Vercel Edge Network
- **Images**: Cloudinary
- **Analytics**: Vercel Analytics
- **Monitoring**: Railway + Neon dashboards

---

## 📚 Additional Resources

- [Project Documentation](./project_documentation.md)
- [Neon Optimization Guide](./neon_optimization_guide.md)
- [Client Handover Guide](./client_handover.md)

---

## 📝 License

This project is proprietary and confidential.

---

**Built with ❤️ using Next.js, Fastify, and PostgreSQL**
