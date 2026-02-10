# 📘 Developer Documentation — Lost & Found Tracking System

> **Version**: Sprint 1
> **Last Updated**: 2026-02-10
> **Repository**: [Lost-and-Found-Tracking-System/lost-and-found-system](https://github.com/Lost-and-Found-Tracking-System/lost-and-found-system)

---

## Table of Contents

- [1. Architecture Overview](#1-architecture-overview)
- [2. Tech Stack Details](#2-tech-stack-details)
- [3. Backend Architecture](#3-backend-architecture)
  - [3.1 Entry Point & Server Bootstrap](#31-entry-point--server-bootstrap)
  - [3.2 Configuration Layer](#32-configuration-layer)
  - [3.3 Models & Database Schema](#33-models--database-schema)
  - [3.4 Middleware Pipeline](#34-middleware-pipeline)
  - [3.5 Routes & API Endpoints](#35-routes--api-endpoints)
  - [3.6 Services (Business Logic)](#36-services-business-logic)
  - [3.7 Utilities](#37-utilities)
  - [3.8 Schemas (Validation)](#38-schemas-validation)
- [4. Frontend Architecture](#4-frontend-architecture)
  - [4.1 Routing](#41-routing)
  - [4.2 Pages](#42-pages)
  - [4.3 Components](#43-components)
  - [4.4 Context (State Management)](#44-context-state-management)
  - [4.5 Services (API Layer)](#45-services-api-layer)
  - [4.6 Hooks](#46-hooks)
  - [4.7 Effects & Animations](#47-effects--animations)
- [5. AI/ML Module](#5-aiml-module)
- [6. Authentication Flow](#6-authentication-flow)
- [7. Database Design](#7-database-design)
- [8. Testing Strategy](#8-testing-strategy)
  - [8.1 Backend Unit Testing](#81-backend-unit-testing)
  - [8.2 Backend Integration Testing](#82-backend-integration-testing)
  - [8.3 Frontend Unit Testing](#83-frontend-unit-testing)
  - [8.4 Frontend Integration Testing](#84-frontend-integration-testing)
  - [8.5 Frontend–Backend Integration Testing](#85-frontendbackend-integration-testing)
- [9. Environment Configuration](#9-environment-configuration)
- [10. Coding Conventions](#10-coding-conventions)
- [11. Scripts Reference](#11-scripts-reference)
- [12. Sprint Status & Integration Map](#12-sprint-status--integration-map)

---

## 1. Architecture Overview

The system follows a **three-tier architecture** with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                     PRESENTATION TIER                       │
│  React 18 + Vite + Tailwind CSS + GSAP + Framer Motion     │
│  Pages: Landing, Login, Register, Dashboard, Report Item,   │
│         Item Details, Inventory, Claims, Profile,           │
│         Notifications, Admin Dashboard, Visitor Register    │
└──────────────────────────┬──────────────────────────────────┘
                           │ Axios HTTP (REST JSON)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                     APPLICATION TIER                        │
│  Node.js + Express 4 + TypeScript                          │
│  ┌─────────┐ ┌────────────┐ ┌──────────┐ ┌──────────────┐ │
│  │ Routes  │→│ Middleware  │→│ Services │→│   Models     │ │
│  │ (v1/*)  │ │ Auth/Valid  │ │ Business │ │  Mongoose    │ │
│  └─────────┘ └────────────┘ └──────────┘ └──────────────┘ │
└──────────────────────────┬──────────────────────────────────┘
                           │ Mongoose ODM
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                       DATA TIER                             │
│  MongoDB (15+ collections) + Redis (OTP/Cache/Queues)       │
│  Cloudinary (images) + HuggingFace (AI inference)          │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Tech Stack Details

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| **Runtime** | Node.js | 18+ | Server runtime |
| **Backend Framework** | Express | 4.21.2 | REST API framework |
| **Language** | TypeScript | 5.9.3 | Type-safe backend |
| **Database** | MongoDB + Mongoose | 8.9.5 | Document database + ODM |
| **Cache/Queue** | Redis + ioredis + Bull | 5.9.2 / 4.16.5 | OTP storage, job queues |
| **Auth** | jsonwebtoken + Argon2 | 9.0.2 / 0.31.2 | JWT tokens + password hashing |
| **Validation** | Zod | 3.23.10 | Request schema validation |
| **Email** | @sendgrid/mail | 8.1.6 | Transactional email |
| **SMS** | Fast2SMS API | REST | OTP SMS (India) |
| **Images** | Cloudinary | 2.9.0 | Image upload/CDN |
| **PDF** | PDFKit | 0.17.2 | Handover letter generation |
| **Scheduler** | node-cron | 4.2.1 | Background cron jobs |
| **Upload** | Multer | 2.0.2 | Multipart file handling |
| **Frontend** | React | 18.2.0 | UI framework |
| **Build Tool** | Vite | 5.0.0 | Dev server + bundler |
| **Styling** | Tailwind CSS | 3.4.1 | Utility-first CSS |
| **Animation** | GSAP + Framer Motion | 3.14.2 / 11.0.0 | UI animations |
| **HTTP Client** | Axios | 1.6.0 | API requests |
| **Routing** | React Router | 6.22.0 | Client-side routing |
| **Icons** | Lucide React | 0.363.0 | Icon library |
| **Backend Testing** | Vitest + Supertest + mongodb-memory-server | 4.0.18 / 7.2.2 / 11.0.1 | Unit & integration tests |
| **Frontend Testing** | Vitest + React Testing Library + jsdom | 4.0.18 / 16.3.2 / 28.0.0 | Component tests |
| **E2E Testing** | Playwright | 1.58.2 | Browser automation tests |

---

## 3. Backend Architecture

### 3.1 Entry Point & Server Bootstrap

**`backend/src/server.ts`** — The server entry point. Bootstraps in this order:
1. Loads environment variables via `config/env.ts`
2. Connects to MongoDB via `config/database.ts`
3. Initializes Redis client via `config/redis.ts` (with in-memory fallback)
4. Sets up Bull job queues via `config/queue.ts`
5. Starts background schedulers (cron jobs)
6. Creates Express app via `app.ts` and starts listening

**`backend/src/app.ts`** — Express app factory. Configures:
- CORS (allowing `CLIENT_URL` origin with credentials)
- JSON body parsing
- Cookie parser (for refresh tokens)
- API routes mounted at `/api/v1/*`
- Global error handler middleware

### 3.2 Configuration Layer

| File | Purpose |
|------|---------|
| `config/env.ts` | Centralized env variable management with `assertRequiredEnv()` and `isProductionServicesEnabled()` helpers |
| `config/database.ts` | MongoDB connection via Mongoose with retry logic |
| `config/redis.ts` | Redis client (ioredis) with graceful in-memory Map fallback for OTP storage |
| `config/queue.ts` | Bull queue setup (data retention, reminders, escalation, email, SMS) |
| `config/cloudinary.ts` | Cloudinary SDK configuration with `isCloudinaryConfigured()` guard |

### 3.3 Models & Database Schema

All models are defined in `backend/src/models/` and re-exported via `models/index.ts`:

| File | Collections | Key Fields |
|------|-------------|------------|
| `identity.ts` | `users`, `login_sessions`, `login_activity_logs` | role (student/faculty/visitor/admin/delegated_admin), credentials (passwordHash, failedAttempts), profile (fullName, email, phone), visitorMetadata |
| `locations.ts` | `campus_zones` | zoneName, geoBoundary (GeoJSON Polygon, 2dsphere index), isActive |
| `items.ts` | `items`, `draft_submissions` | trackingId (unique), submissionType (lost/found), itemAttributes (category, color, material, size, description), location (GeoJSON Point), timeMetadata, aiMetadata, images[], status |
| `claims.ts` | `claims`, `claim_conflicts`, `claim_decisions`, `archived_claims` | itemId, claimantId, ownershipProofs[], proofScore, status (pending/approved/rejected/withdrawn/conflict), confidenceTier, aiConfidenceScore |
| `ai.ts` | `ai_matches`, `ai_configurations`, `ai_decision_versions` | similarityScore, featureBreakdown (text/image/location/time weights), thresholds (autoApprove/partialMatch) |
| `notifications.ts` | `notifications`, `notification_preferences`, `announcements` | type (12 notification types), channel (email/push/sms/in_app), priority (low/normal/high/urgent), isRead |
| `audit.ts` | `audit_logs`, `data_retention_policies` | actorId, action, targetEntity, targetId, metadata, timestamp |

### 3.4 Middleware Pipeline

| File | Functions | Purpose |
|------|-----------|---------|
| `auth.ts` | `authMiddleware`, `optionalAuthMiddleware` | JWT Bearer token verification. Extracts `userId` and `role` from token payload and attaches to `req.user` |
| `roleAuth.ts` | `requireRole()`, `requireAdmin`, `requireFullAdmin`, `requireScope()`, `excludeVisitors`, `requireOwnerOrAdmin()` | Role-based access control with 5 roles and scope-based delegation |
| `validation.ts` | `validateRequest()`, `validateQuery()` | Zod schema validation for request body and query parameters |
| `errorHandler.ts` | `createApiError()`, `errorHandler` | Centralized error handling with status codes, dev-mode stack traces |

**Request flow:**
```
Request → CORS → JSON Parse → Cookie Parse → Route Match →
  [authMiddleware] → [validateRequest] → [requireRole] →
  Route Handler → Service Layer → Model Layer → Response
  └── On Error → errorHandler → JSON Error Response
```

### 3.5 Routes & API Endpoints

All routes are versioned under `/api/v1/` and defined in `backend/src/routes/v1/`:

| File | Base Path | Key Endpoints |
|------|-----------|---------------|
| `auth.ts` | `/api/v1/auth` | `POST /register`, `POST /login`, `POST /refresh`, `POST /logout`, `POST /visitor/request-otp`, `POST /visitor/verify-otp`, `POST /forgot-password`, `POST /reset-password` |
| `users.ts` | `/api/v1/users` | `GET /profile`, `PUT /profile`, `GET /login-activity`, `GET /my-items`, `GET /notification-preferences`, `PUT /notification-preferences` |
| `items.ts` | `/api/v1/items` | `POST /` (submit item), `GET /` (search/browse), `GET /:id`, `POST /organization`, `POST /drafts`, `GET /drafts/me`, `DELETE /drafts/me` |
| `claims.ts` | `/api/v1/claims` | `POST /` (submit claim), `GET /:id`, `GET /user/my-claims`, `POST /:id/withdraw` |
| `notifications.ts` | `/api/v1/notifications` | `GET /`, `PUT /:id/read`, `PUT /read-all`, `DELETE /:id` |
| `zones.ts` | `/api/v1/zones` | `GET /`, `GET /:id`, `POST /` (admin), `PUT /:id` (admin) |
| `uploads.ts` | `/api/v1/uploads` | `POST /image`, `POST /images`, `DELETE /:publicId` |
| `admin.ts` | `/api/v1/admin` | `GET /stats`, `GET /activity`, user management, role changes, AI config, announcements |
| `dashboard.ts` | `/api/v1/dashboard` | `GET /stats` (per-user dashboard statistics) |

### 3.6 Services (Business Logic)

Services are split into two categories based on current integration status:

#### Fully Integrated Services (Route → Service → Model → Frontend)

| Service | File | Responsibilities |
|---------|------|-----------------|
| **AuthService** | `authService.ts` | User registration (with Argon2 hashing), login (with session creation), token generation |
| **ItemService** | `itemService.ts` | Item submission, organization submissions, draft CRUD, search with filters, zone validation |
| **OTPService** | `otpService.ts` | OTP generation (6-digit), Redis storage with TTL, rate limiting, verification, password reset flow, visitor registration flow |
| **EmailService** | `emailService.ts` | SendGrid integration for OTP emails, notification emails, security alerts. Logs to console in dev mode |
| **SMSService** | `smsService.ts` | Fast2SMS integration for OTP SMS, claim status alerts, security alerts. Indian phone validation & normalization. Logs to console in dev mode |
| **NotificationService** | `notificationService.ts` | In-app notification creation, email/SMS dispatch based on user preferences and priority, bulk notifications, security alerts |
| **UploadService** | `uploadService.ts` | Cloudinary image upload/delete, multi-image upload, URL transformations |
| **Schedulers** | `schedulers.ts` | Bull queue job processors + cron fallback: archive old claims, cleanup expired visitors, cleanup old drafts, admin pending reminders, email/SMS queue processing |

#### Backend-Only Services (Service + Model built, no API route exposure yet — Sprint 2)

| Service | File | Responsibilities | Sprint 2 Integration Plan |
|---------|------|-----------------|---------------------------|
| **EmbeddingService** | `embeddingService.ts` | YOLO object detection (4 models via Roboflow & HuggingFace), OpenCLIP image embeddings, TF-IDF text embeddings, FAISS-like in-memory index, weighted similarity scoring, `findSimilarItems()`, `getTopMatches()`, `batchProcessItems()` | Expose via `/api/v1/ai/match` routes, call on item submission, show matches in frontend |
| **FindImagePairsService** | `findImagePairs.ts` | Lost-to-found item matching using cosine similarity (50% embedding + 30% TF-IDF + 20% YOLO class), `matchAllMissingToFoundItems()`, `scoreClaimsForFoundItem()`, `runCompletePipeline()`, AI performance analytics | Wire into admin dashboard, automated matching scheduler |
| **FraudDetectionService** | `fraudDetectionService.ts` | Claim confidence scoring (40% confidence + 35% proof quality + 25% history), competing claim evaluation, flag generation (REPEAT_OFFENDER, WEAK_PROOF, etc.), suspicion thresholds | Call in claim submission route, display fraud flags in admin Claims page |
| **LetterService** | `letterService.ts` | Professional PDF generation via PDFKit: handover confirmation letters (A4 with header, item/recipient details boxes, signature lines), claim confirmation PDFs (with status badge), text fallback for dev | Add `/api/v1/letters/handover/:id` and `/api/v1/letters/claim/:id` download routes, add download buttons in admin and MyClaims pages |

### 3.7 Utilities

| File | Functions | Purpose |
|------|-----------|---------|
| `jwt.ts` | `signAccessToken()`, `signRefreshToken()`, `verifyAccessToken()`, `verifyRefreshToken()` | JWT sign/verify with configurable expiry |
| `password.ts` | `hashPassword()`, `verifyPassword()` | Argon2id password hashing |

### 3.8 Schemas (Validation)

Zod schemas in `backend/src/schemas/` validate all incoming request bodies:
- `registerSchema` — email, fullName, password (min 8 chars)
- `loginSchema` — email, password
- `submitItemSchema` — submissionType, itemAttributes, location (with coordinates & zoneId), timeMetadata
- `organizationSubmissionSchema` — extends submitItemSchema with organizationId
- `submitClaimSchema` — itemId, ownershipProofs array
- `draftSaveSchema` — flexible partial data
- `updateProfileSchema` — fullName, phone, affiliation
- `updateNotificationPrefsSchema` — channel & category preferences

---

## 4. Frontend Architecture

### 4.1 Routing

`App.jsx` uses React Router v6 with the following route structure:

| Route | Page Component | Auth Required |
|-------|---------------|---------------|
| `/` | `LandingPage` | No |
| `/login` | `Login` | No |
| `/register` | `Register` | No |
| `/visitor-register` | `VisitorRegister` | No |
| `/dashboard` | `Dashboard` | Yes |
| `/report` | `ReportItem` | Yes |
| `/items` | `ItemInventory` | No |
| `/items/:id` | `ItemDetails` | No |
| `/claims/:itemId` | `SubmitClaim` | Yes |
| `/my-claims` | `MyClaims` | Yes |
| `/notifications` | `Notifications` | Yes |
| `/profile` | `Profile` | Yes |
| `/admin` | `AdminDashboard` | Admin |

### 4.2 Pages

| Page | File | Description |
|------|------|-------------|
| Landing | `LandingPage.jsx` | Public homepage with 39+ GSAP effects, feature showcase, scrolling animations |
| Login | `Login.jsx` | Email/password login with 3D card tilt, spotlight effects |
| Register | `Register.jsx` | User registration with step animations |
| Visitor Register | `VisitorRegister.jsx` | Multi-step OTP-based visitor registration |
| Dashboard | `Dashboard.jsx` | Personalized stats (totalReported, totalClaims, pendingClaims, resolvedItems), quick actions |
| Report Item | `ReportItem.jsx` | Multi-step form with image upload, zone selection, draft auto-save |
| Item Inventory | `ItemInventory.jsx` | Searchable/filterable grid/list view with pagination |
| Item Details | `ItemDetails.jsx` | Full item view with image gallery, claim button, inline claim form |
| Submit Claim | `SubmitClaim.jsx` | Ownership proof submission with image evidence upload (max 3 images) |
| My Claims | `MyClaims.jsx` | User's claim history with status badges, rejection reasons |
| Notifications | `Notifications.jsx` | Filterable notification list (all/unread/read) with mark read/delete, type-specific icons, GSAP animations |
| Profile | `Profile.jsx` | User profile editing, login activity history |
| Admin Dashboard | `admin/AdminDashboard.jsx` | Admin statistics, live activity feed, user/claim management |
| Admin Claims | `admin/Claims.jsx` | Claim review with confidence tiers, proof display, approve/reject workflow |

### 4.3 Components

Reusable components in `frontend/src/components/`:
- `Sidebar` — Unified navigation sidebar with role-based menu items (different links for student vs admin)
- `Navbar` — Top navigation bar
- Other shared UI components (buttons, modals, cards, loading spinners)

### 4.4 Context (State Management)

| Context | File | State Managed |
|---------|------|---------------|
| `AuthContext` | `context/AuthContext.jsx` | User authentication state, login/logout/register functions, access token management, auto-refresh on 401 |

### 4.5 Services (API Layer)

| File | Purpose |
|------|---------|
| `services/api.js` | Axios instance with base URL (`VITE_API_URL`), request interceptor (auto-attaches Bearer token), response interceptor (auto-refresh on 401, retry failed request) |

### 4.6 Hooks

| Hook | File | Purpose |
|------|------|---------|
| `useGSAPAnimations` | `hooks/useGSAPAnimations.js` | Reusable GSAP animation hooks: `use3DTilt`, `useMagneticHover`, `useStaggerIn`, `useScrollReveal` |

### 4.7 Effects & Animations

The frontend uses **39+ custom animation effects** split across two files:

**`effects/PremiumEffects.jsx`**: GlitchText, ParticleCursor, MorphingBlob, WaveText, AuroraBackground, NoiseOverlay, SpotlightCursor, ElasticButton, ScrambleLink, FloatingElement, GradientFlowText, Typewriter, InfiniteMarquee, RippleButton

**`effects/AdvancedEffects.jsx`**: MeteorShower, HolographicCard, CyberpunkGrid, NeonText, PulseRings, GradientBorderCard, ScrollProgress, TiltCard, AnimatedShapes, ParticleExplosion, GlowingOrb, BreathingCircle

---

## 5. AI/ML Module

### Current Status

The AI/ML module consists of **three backend services** that are fully coded and unit-tested, but **not yet exposed via API routes** and therefore not yet callable from the frontend. Integration is planned for Sprint 2.

### Processing Pipeline (Backend Services)

```
Item Submitted
       │
       ▼
┌──────────────────────────────────────────────────┐
│         embeddingService.processItemImage()       │
│                                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────┐ │
│  │ YOLO Detection│  │ OpenCLIP     │  │ TF-IDF  │ │
│  │ (4 models in  │  │ Image        │  │ Text    │ │
│  │  parallel via │  │ Embeddings   │  │ Embed.  │ │
│  │  Roboflow &   │  │ (HuggingFace │  │         │ │
│  │  HuggingFace) │  │  Inference)  │  │         │ │
│  └──────┬───────┘  └──────┬───────┘  └────┬────┘ │
│         │                  │               │       │
│         ▼                  ▼               ▼       │
│   detectedObjects     embedding[]     textEmbedding│
│   primaryClass        (512-d vector)  (TF-IDF vec) │
│         │                  │               │       │
│         └────────┬─────────┴───────────────┘       │
│                  ▼                                  │
│        Saved to item.aiMetadata in MongoDB         │
└──────────────────────────────────────────────────┘
                   │
                   ▼ (when matching is triggered)
┌──────────────────────────────────────────────────┐
│     findImagePairs.matchAllMissingToFoundItems()  │
│                                                    │
│   Weighted Similarity Scoring:                     │
│   Score = 0.5 × embedding + 0.3 × TF-IDF          │
│         + 0.2 × YOLO class overlap                 │
│                                                    │
│   embeddingService.calculateItemSimilarity():      │
│   Score = w₁·text + w₂·image + w₃·location        │
│         + w₄·time  (configurable weights)          │
│                                                    │
│   Thresholds:                                      │
│   ├── Score ≥ 85  → Auto-approve match             │
│   ├── Score ≥ 50  → Partial match (manual review)  │
│   └── Score < 50  → No match                       │
└──────────────────────────────────────────────────┘
                   │
                   ▼ (when claims exist on found items)
┌───────────────────────────────────���──────────────┐
│     fraudDetectionService.evaluateCompetingClaims()│
│                                                    │
│   Per-claim scoring:                               │
│   ├── 40% Confidence (proof score + image count)   │
│   ├── 35% Proof quality (keyword, length, image)   │
│   └── 25% Claimant history (past claims/rejections)│
│                                                    │
│   Flags generated:                                 │
│   ├── REPEAT_OFFENDER (≥5 past rejections)         │
│   ├── WEAK_PROOF (score < 30)                      │
│   ├── MULTIPLE_ACTIVE_CLAIMS (≥3 pending)          │
│   └── SUSPICION_THRESHOLD (score ≥ 40 suspicious)  │
│                                                    │
│   Output: winner claim + manual review flag        │
└──────────────────────────────────────────────────┘
```

### YOLO Models

| Model | Source | Detects |
|-------|--------|---------|
| `IndUSV/yoloV8_SE_3` | HuggingFace | Mobile phones, suitcases, handbags |
| `IndUSV/Yolov8_Screen_Detection` | HuggingFace | Electronic gadgets & screens |
| `IndUSV/computerApparatus-detector` | HuggingFace | Keyboards, mice, monitors |
| Stationery Classifier | Roboflow | Pens, pencils, notebooks |

> **Testing Status:** YOLO models have been tested on custom campus data (photos taken inside Amrita) with appreciably good results. One model can count objects per class across different categories within the same photo. Fine-tuning notebooks with evaluation metrics are in `YOLO_Finetuning/`.

### Configurable Parameters (via `ai_configurations` collection)

- **Weights**: text, image, location, time (sum to 1.0, default: 0.30 / 0.35 / 0.20 / 0.15)
- **Thresholds**: autoApprove (default 85), partialMatch (default 50)
- **Version tracking**: Every config change creates a new version for auditability and rollback

### Sprint 2 Integration Plan

1. **API Routes**: Create `/api/v1/ai/` routes to expose `processItemImage()`, `findSimilarItems()`, `getTopMatches()`, `batchProcessItems()`
2. **Auto-trigger**: Call `processItemImage()` automatically when an item is submitted via `POST /api/v1/items`
3. **Claim scoring**: Call `assessClaimRisk()` when a claim is submitted via `POST /api/v1/claims`
4. **Frontend**: AI match results display on Item Details page, fraud flags on admin Claims page
5. **PDF downloads**: `/api/v1/letters/handover/:id` and `/api/v1/letters/claim/:id` with download buttons

---

## 6. Authentication Flow

```
┌─────────┐          ┌─────────┐          ┌─────────┐
│ Client  │          │ Server  │          │   DB    │
└────┬────┘          └────┬────┘          └────┬────┘
     │   POST /register    │                    │
     │────────────────────►│  Hash password      │
     │                     │  (Argon2id)        │
     │                     │───────────────────►│ Save user
     │                     │◄───────────────────│
     │  { accessToken,     │                    │
     │    refreshToken }   │                    │
     │◄────────────────────│                    │
     │                     │                    │
     │   POST /login       │                    │
     │────────────────────►│  Verify password    │
     │                     │───────────────────►│ Find user
     │                     │◄───────────────────│
     │                     │  Create session     │
     │                     │───────────────────►│ Save session
     │  { accessToken }    │                    │
     │  Cookie: refresh    │                    │
     │◄────────────────────│                    │
     │                     │                    │
     │   GET /api/v1/*     │                    │
     │  Authorization:     │                    │
     │  Bearer <token>     │                    │
     │────────────────────►│  Verify JWT        │
     │                     │  Extract userId    │
     │  { data }           │  + role            │
     │◄────────────────────│                    │
     │                     │                    │
     │  Token Expired      │                    │
     │   POST /refresh     │                    │
     │  Cookie: refresh    │                    │
     │────────────────────►│  Verify refresh    │
     │                     │  Issue new access  │
     │  { accessToken }    │                    │
     │◄────────────────────│                    │
```

**Token Details:**
- Access Token: 15 minutes expiry, stored in memory (frontend)
- Refresh Token: 7 days expiry, httpOnly secure cookie
- Password: Argon2id hash (memory-hard, timing-attack resistant)
- Sessions: SHA-256 hashed token stored in `login_sessions` with device info
- Failed Login Protection: Security alert triggered after 3 failed attempts (via `notificationService.sendSecurityAlert()`)

---

## 7. Database Design

### Entity Relationships

```
users ──┬──< login_sessions
        ├──< login_activity_logs
        ├──< items (submittedBy)
        ├──< claims (claimantId)
        ├──< notifications
        ├──< notification_preferences
        ├──< audit_logs (actorId)
        └──< draft_submissions

items ──┬──< claims (itemId)
        ├──< ai_matches (lostItemId / foundItemId)
        └──> campus_zones (location.zoneId)

claims ──┬──< claim_decisions
         ├──< claim_conflicts
         └──< archived_claims

ai_matches ──< ai_decision_versions
```

### Indexes

| Collection | Index | Type |
|-----------|-------|------|
| `users` | `institutionalId` | Unique, sparse |
| `users` | `visitorId` | Unique, sparse |
| `users` | `visitorMetadata.expiresAt` | TTL (auto-expire visitors) |
| `items` | `trackingId` | Unique |
| `items` | `itemAttributes.category` | Standard |
| `items` | `location` | 2dsphere (geospatial) |
| `campus_zones` | `geoBoundary` | 2dsphere |
| `claims` | `itemId`, `claimantId`, `status` | Standard |
| `ai_matches` | `lostItemId + foundItemId` | Unique compound |
| `notifications` | `userId + createdAt` | Compound (desc) |
| `audit_logs` | `timestamp` | Descending |

---

## 8. Testing Strategy

Our testing strategy covers **five layers** of testing to ensure comprehensive quality assurance:

### 8.1 Backend Unit Testing

**Purpose**: Test individual functions and utilities in isolation, without database or external service dependencies.

**Tool**: Vitest
**Location**: `backend/tests/unit/`
**What's tested**:
- JWT sign/verify functions (`utils/jwt.ts`)
- Password hash/verify functions (`utils/password.ts`)
- OTP generation (`services/otpService.ts` — `generateOtp()`)
- Phone number validation & normalization (`services/smsService.ts`)
- Error creation helpers (`middleware/errorHandler.ts`)
- Zod schema validation (schemas)
- Letter service data structures and date formatting (`services/letterService.ts`)

**Run**: `cd backend && npm test`

### 8.2 Backend Integration Testing

**Purpose**: Test backend components integrated together — controllers + services + database — while mocking external APIs (SendGrid, Fast2SMS, Cloudinary, HuggingFace).

**Tools**: Vitest + Supertest + mongodb-memory-server
**Location**: `backend/tests/integration/`
**What's tested**:
- Full auth flow: register → login → token refresh → protected route access
- Item CRUD: submit → search → get by ID → draft save/load
- Claim flow: submit claim → get claims → withdraw
- Zone CRUD: create → list → update (admin only)
- Notification flow: create → list → mark read → delete, email/SMS dispatch with mocked services
- Role-based access: student vs admin vs visitor permissions
- Error handling: invalid input → 400, missing auth → 401, wrong role → 403

**How it works**:
- `mongodb-memory-server` spins up an in-memory MongoDB instance per test suite
- No real MongoDB, Redis, or external services needed
- Tests use Supertest to send real HTTP requests to the Express app
- Environment variables are mocked in `backend/tests/setup.ts`

### 8.3 Frontend Unit Testing

**Purpose**: Test individual React components in isolation with mocked props and context.

**Tools**: Vitest + React Testing Library + jsdom
**Location**: `frontend/src/tests/`
**What's tested**:
- Component rendering (does it render without crashing?)
- User interactions (clicks, form inputs, navigation)
- Conditional rendering (loading states, error states, empty states)
- Props validation (correct data display)

### 8.4 Frontend Integration Testing

**Purpose**: Test multiple frontend components integrated together — pages with context providers, routing, and API service layer — while **mocking the backend** API responses.

**Tools**: Vitest + React Testing Library + jsdom
**Location**: `frontend/src/tests/`
**What's tested**:
- Full page rendering with `AuthContext` provider
- Navigation between pages via React Router
- Form submission → API call → state update → UI feedback
- Protected route redirection (unauthenticated → login)
- Error handling from mocked API failures

**How it works**:
- API module (`services/api.js`) is mocked using `vi.mock()`
- `AuthContext` is wrapped around components with test user data
- Router is wrapped with `MemoryRouter` for route testing

### 8.5 Frontend–Backend Integration Testing

**Purpose**: Verify that the real frontend communicates correctly with the real backend over actual HTTP APIs. Tests the **contract** between frontend and backend — endpoint URLs, request formats, response schemas, auth headers, error handling.

**Tools**: Playwright (E2E)
**Location**: `e2e/tests/`
**What's tested**:
- Login form submits to `POST /api/v1/auth/login` and receives valid token
- Report item form submits to `POST /api/v1/items` with correct payload
- Dashboard fetches from `GET /api/v1/dashboard/stats` with auth header
- Protected pages redirect to login when token is missing/expired
- Error responses from backend display correctly in frontend UI
- Full user journeys: register → login → report item → view inventory → submit claim

**Test Suite**: E2E test specs across authentication, landing page, dashboard, item reporting, inventory browsing, claim submission, notifications, profile management, and admin features.

### Testing Summary

```
┌───────────────────────────────────────────────────────┐
│                    TESTING PYRAMID                    │
├───────────────────────────────────────────────────────┤
│                                                       │
│              ┌──────────────┐                        │
│              │   E2E Tests  │  Playwright             │
│              │  (FE ↔ BE)   │  Real browser + API     │
│              └──────┬───────┘                        │
│                     │                                 │
│           ┌─────────┴─────────┐                      │
│           │  Integration Tests │  Vitest + Supertest  │
│           │  BE: controllers+  ���  mongodb-memory-     │
│           │  services+models   │  server              │
│           │  FE: pages+context │  Mocked API          │
│           │  +routing          │                      │
│           └─────────┬─────────┘                      │
│                     │                                 │
│        ┌────────────┴────────────┐                   │
│        │      Unit Tests         │  Vitest            │
│        │  BE: utils, validators  │  Isolated functions │
│        │  FE: components         │  Mocked deps       │
│        └─────────────────────────┘                   │
│                                                       │
└───────────────────────────────────────────────────────┘
```

---

## 9. Environment Configuration

All services degrade gracefully when API keys are not configured:

| Service | Env Variable | Fallback Behavior |
|---------|-------------|-------------------|
| MongoDB | `MONGODB_URI` | **Required** — no fallback |
| JWT Secrets | `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET` | **Required** — no fallback |
| Redis | `REDIS_URL` | In-memory Map-based fallback (OTP storage, direct email/SMS send instead of queuing) |
| SendGrid | `SENDGRID_API_KEY` | Logs email content to console |
| Fast2SMS | `FAST2SMS_API_KEY` | Logs OTP/SMS to console |
| Cloudinary | `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` | Returns 503 on upload routes |
| HuggingFace | `HUGGINGFACE_API_KEY` | AI embedding/detection features disabled |

---

## 10. Coding Conventions

| Area | Convention |
|------|-----------|
| **Backend Language** | TypeScript strict mode, ESNext modules (`"type": "module"`) |
| **Frontend Language** | JavaScript (JSX), functional components + hooks only |
| **Styling** | Tailwind CSS utility classes, no custom CSS except `landing.css` |
| **State Management** | React Context API (no Redux) |
| **API Communication** | Axios with centralized interceptors |
| **Validation** | Zod schemas for all request bodies |
| **Error Handling** | Centralized `errorHandler` middleware, `createApiError()` factory |
| **Authentication** | Bearer token in Authorization header, refresh token in httpOnly cookie |
| **Database** | Mongoose schemas with TypeScript type inference (`InferSchemaType`) |
| **Code Comments** | TSDoc/JSDoc style (`/** */`) on all exported functions, interfaces, and file headers |
| **Git** | Feature branches, meaningful commit messages |
| **Linting** | ESLint with TypeScript plugin (backend), React plugin (frontend) |

---

## 11. Scripts Reference

### Backend (`cd backend`)

| Script | Command | Purpose |
|--------|---------|---------|
| `npm run dev` | `vite-node src/server.ts` | Start dev server with hot reload |
| `npm run build` | `tsc` | Compile TypeScript |
| `npm start` | `node dist/server.js` | Start production server |
| `npm test` | `vitest run` | Run all tests once |
| `npm run test:watch` | `vitest` | Run tests in watch mode |
| `npm run test:coverage` | `vitest run --coverage` | Generate coverage report |
| `npm run lint` | `eslint .` | Lint codebase |

### Frontend (`cd frontend`)

| Script | Command | Purpose |
|--------|---------|---------|
| `npm run dev` | `vite` | Start dev server (port 5173) |
| `npm run build` | `vite build` | Production build |
| `npm run preview` | `vite preview` | Preview production build |
| `npm test` | `vitest` | Run component tests |
| `npm run lint` | `eslint .` | Lint codebase |

### E2E (`cd e2e`)

| Script | Command | Purpose |
|--------|---------|---------|
| `npm run e2e` | `playwright test` | Run all E2E tests |
| `npm run e2e:ui` | `playwright test --ui` | Interactive test UI |
| `npm run e2e:headed` | `playwright test --headed` | Run with visible browser |
| `npm run e2e:report` | `playwright show-report` | View HTML report |

---

## 12. Sprint Status & Integration Map