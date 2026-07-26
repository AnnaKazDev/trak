# TREK - Service Overview
## Kompleksowy przewodnik po aplikacji

> **Wersja:** 3.4.1  
> **Licencja:** AGPL v3  
> **Typ:** Self-hosted collaborative travel planner  
> **Status:** Production-ready

---

## 🎯 Co to jest TREK?

**TREK** to **self-hosted, real-time collaborative travel planner** — kompleksowa aplikacja do planowania i zarządzania podróżami, z wbudowanymi mapami, budżetami, listami pakowania, dziennikiem podróży i integracją AI.

### Tagline
> *Your trips. Your plan. Your server.*

TREK to **all-in-one rozwiązanie** dla osób i grup planujących podróże, które łączy:
- 🗺️ Interaktywne mapy z wizualizacją tras
- 📅 Planer dnia z drag & drop
- 💰 Zarządzanie budżetem i rozliczanie kosztów (Splitwise-style)
- 🧳 Listy pakowania z trackingiem wagi
- 📸 Dziennik podróży z integracją zdjęć (Immich/Synology)
- 👥 Real-time współpraca (WebSocket)
- 🤖 AI/LLM integration (MCP - Model Context Protocol)

### Kluczowe Cechy

✅ **Self-hosted** - Pełna kontrola nad danymi, brak cloud lock-in  
✅ **Real-time collaboration** - Zmiany widoczne natychmiast dla wszystkich użytkowników  
✅ **PWA** - Instalowalna jako aplikacja mobilna (iOS/Android) bez App Store  
✅ **Offline support** - Service Worker + IndexedDB  
✅ **Multi-language** - 20 języków (w tym AR z RTL, PL, i inne)  
✅ **Modern UX** - Dark mode, responsive, touch-optimized  
✅ **Extensible** - System pluginów, MCP dla AI, addons

---

## 📦 Główne Funkcjonalności

### 🧭 1. Trip Planning (Planowanie Wycieczek)

#### Interaktywny Planer
- **Drag & drop interface** - Przeciąganie miejsc między dniami, reordering
- **Day plans** - Organizacja wycieczki w plany dzienne
- **Interactive map** - Leaflet lub Mapbox GL z:
  - Budynkami 3D
  - Terenem
  - Markerami zdjęć
  - Klastrowaniem pinów
  - Wizualizacją tras
- **Place search** - Google Places (zdjęcia, oceny, godziny) lub OpenStreetMap (free, bez API key)
- **Weather forecasts** - 16-dniowa prognoza via Open-Meteo (bez klucza) + historyczne dane klimatyczne

#### Import i Export
- **Place import**:
  - Google Maps shared lists
  - Naver Maps lists
  - GPX files
  - KML/KMZ/GeoJSON
- **Route optimization** - Auto-sort miejsc i export do Google Maps
- **PDF export** - Pełny plan wycieczki z cover page, obrazkami, notatkami

#### Organizacja
- **Day notes** - Timestamped, icon-tagged notes z drag-and-drop
- **Category filter** - Filtrowanie pinów na mapie według kategorii
- **Place details** - Kompletne info o miejscach z Google/OSM

---

### 🧳 2. Travel Management (Zarządzanie Podróżą)

#### Reservations (Rezerwacje)
- **Typy rezerwacji**:
  - ✈️ Flights (Loty)
  - 🏨 Accommodations (Noclegi)
  - 🍽️ Restaurants (Restauracje)
  - 🚗 Transportation (Transport)
- **Tracking**:
  - Status rezerwacji
  - Numery potwierdzenia
  - Załączniki (pliki, PDF-y)
- **Smart import** - Import z booking confirmation emails i PDF-ów ([KDE Itinerary](https://invent.kde.org/pim/kitinerary))

#### Costs & Budget (Koszty i Budżet)
- **Expense tracking** - Śledzenie wydatków z podziałem:
  - Per-person breakdown
  - Per-day breakdown
  - Multi-currency support
- **Splitwise-style splitting** - Rozliczanie kosztów między uczestnikami
- **Settle-up** - "Kto komu ile jest winien"
- **Real-time calculation** - Automatyczne przeliczenia

#### Packing Lists (Listy Pakowania)
- **Kategorie** - Organizacja według kategorii (odzież, elektronika, dokumenty, etc.)
- **Templates** - Szablony list do wielokrotnego użycia
- **User assignment** - Przypisanie itemów do konkretnych osób
- **Progress tracking** - Tracking postępu pakowania
- **Bag tracking** (opcjonalne) - Śledzenie wagi z iOS-style distribution

#### Document Manager (Menedżer Dokumentów)
- **Attachments** - Załączniki do:
  - Trips (Wycieczki)
  - Places (Miejsca)
  - Reservations (Rezerwacje)
- **Limit** - Do 50 MB per plik
- **Typy** - Dokumenty, bilety, PDF-y, zdjęcia

---

### 👥 3. Collaboration (Współpraca)

#### Real-time Synchronization
- **WebSocket** - Natychmiastowa synchronizacja zmian
- **Live updates** - Zmiany widoczne dla wszystkich użytkowników w czasie rzeczywistym
- **Conflict resolution** - Inteligentne rozwiązywanie konfliktów

#### Multi-user Trips
- **Invite system**:
  - One-time links
  - Reusable links
  - Expiry dates
- **Role-based access** - Różne poziomy uprawnień
- **Member management** - Zarządzanie uczestnikami wycieczki

#### Collab Suite
- **Group chat** - Czat grupowy dla uczestników
- **Shared notes** - Współdzielone notatki
- **Polls** - Ankiety / głosowania
- **Day check-ins** - Dzienne check-iny / obecność

---

### 🔐 4. Authentication & Security

#### Authentication Methods
- **Standard** - Email/password z validation
- **SSO (OIDC)** - Single Sign-On:
  - Google
  - Apple
  - Authentik
  - Keycloak
  - Dowolny OIDC provider
- **Passkeys** (WebAuthn) - Passwordless login:
  - Fingerprint
  - Face ID / Face recognition
  - PIN
  - Security keys (YubiKey, etc.)
- **2FA** - Two-Factor Authentication:
  - TOTP (Time-based One-Time Password)
  - Backup codes

#### Security Features
- **Encryption** - At-rest encryption dla secrets (API keys, MFA, SMTP, OIDC)
- **JWT tokens** - Secure session management
- **OAuth 2.1** - Modern auth dla MCP integration
- **SSRF protection** - Ochrona przed Server-Side Request Forgery
- **Admin-toggleable features** - Kontrola nad dostępnymi funkcjami

---

### 📱 5. Mobile & PWA

#### Progressive Web App
- **Installable** - iOS i Android, bez App Store
- **Offline support** - Service Worker cache:
  - Map tiles
  - API responses
  - Uploads (queue)
- **Native feel**:
  - Fullscreen standalone mode
  - Themed status bar
  - Splash screen
  - Safe-area handling
- **Touch optimized** - Mobile-specific layouts i gestures

---

### 🧩 6. Addons (Admin-toggleable)

System modułów, które admin może włączać/wyłączać:

#### Lists
Listy pakowania + to-do z templates, przypisaniami i opcjonalnym bag trackingiem

#### Costs
Expense tracker z splits i settle-up (kto komu ile jest winien), multi-currency

#### Documents
File attachments na trips, places i reservations

#### Collab
Chat, notes, polls, day-by-day attendance

#### Vacay
Personal vacation planner z:
- Calendar view
- 100+ country holidays
- Carry-over tracking (niewykorzystane dni)
- Annual leave management

#### Atlas
World map visited countries z:
- Bucket list
- Travel stats
- Streak tracking
- Liquid-glass UI
- Sub-national boundaries (prowincje, stany)

#### Journey
Magazine-style travel journal z:
- Entries (wpisy)
- Photos (Immich/Synology integration)
- Maps
- Moods / emotions
- Public sharing

#### AirTrail Integration
Connect self-hosted AirTrail instance - import i sync lotów do reservations

#### MCP (Model Context Protocol)
Expose TREK do AI assistants via OAuth 2.1

---

### 🤖 7. AI / MCP Integration

TREK ma **wbudowany MCP server** - pierwszorzędną integrację z AI assistants (Claude, Cursor, etc.).

#### Capabilities
- **150+ tools** - Kompletny zestaw operacji
- **30 resources** - Dostęp do danych
- **OAuth 2.1 authenticated** - Secure, granular access
- **27 OAuth scopes** - Across 13 permission groups
- **Full automation** - AI może:
  - Tworzyć trips
  - Planować dni
  - Budować packing lists
  - Zarządzać budżetami
  - Oznaczać odwiedzone kraje
  - I wiele więcej...

#### Pre-built Prompts
- `trip-summary` - Podsumowanie wycieczki
- `packing-list` - Sugestie do pakowania
- `budget-overview` - Przegląd budżetu

#### Addon-aware
MCP exposes funkcjonalności Atlas, Collab, Vacay gdy te addony są włączone

---

### ⚙️ 8. Admin & Customization

#### Admin Panel
- **User management** - Zarządzanie użytkownikami
- **Invites** - System zaproszeń
- **Packing templates** - Szablony list pakowania
- **Categories** - Zarządzanie kategoriami miejsc
- **Addons** - Włączanie/wyłączanie modułów
- **API keys** - Zarządzanie kluczami API (Google, Unsplash, etc.)
- **Backups** - System backup/restore
- **GitHub history** - Historia commitów/releases

#### Customization
- **Dashboard views** - Card grid lub compact list
- **Dark mode** - Pełny theme z matching status bar
- **20 languages**:
  - EN, DE, ES, FR, IT, NL, HU, RU
  - ZH, ZH-TW (Chiński tradycyjny/uproszczony)
  - PL, CS (Polski, Czeski)
  - AR (RTL support), BR, ID
  - TR, JA, KO, UK, GR
- **Units** - °C/°F, 12h/24h
- **Map settings** - Tile sources, default coordinates

#### Notifications
- **Channels**:
  - 📧 Email (SMTP)
  - 🔔 Webhook
  - 📱 ntfy.sh
  - 🔕 In-app notification center
- **Per-user preferences** - Każdy user może skonfigurować swoje preferencje

#### Backups
- **Manual backups** - On-demand via Admin Panel
- **Auto-backups** - Scheduled z configurable retention
- **Restore** - Pełny restore z backup file

---

## 🏗️ Tech Stack

### Frontend

#### Core Framework
- **React 19** - Latest React z nowymi features (automatic batching, transitions, etc.)
- **TypeScript** - Type safety
- **Vite 8** - Ultrafast build tool z HMR

#### State Management
- **Zustand** - Lightweight state management
- **Dexie** - IndexedDB wrapper dla offline storage

#### UI & Styling
- **Tailwind CSS** - Utility-first CSS framework
- **Lucide React** - Icon library (344+ icons)
- **Drag-drop-touch** - Touch support dla drag & drop

#### Maps & Geo
- **Leaflet** - Open-source map library
- **Mapbox GL** - 3D maps z terrain i buildings
- **MapLibre GL** - Open-source Mapbox GL fork
- **react-leaflet** - React bindings
- **topojson-client** - TopoJSON dla Atlas boundaries
- **tz-lookup** - Timezone lookup

#### Rich Content
- **@react-pdf/renderer** - PDF generation
- **react-markdown** - Markdown rendering
- **marked** - Markdown parser
- **Plyr** - Video player
- **heic-to** - HEIC image conversion

#### Forms & Input
- **react-dropzone** - File uploads z drag & drop
- **Zod** - Schema validation

#### Auth
- **@simplewebauthn/browser** - WebAuthn/Passkeys client

#### Other
- **Axios** - HTTP client
- **react-router-dom** - Routing
- **react-window** - Virtualized lists
- **iso-3166-2** - Country/region codes

---

### Backend

#### Core Framework
- **Node.js 22** - Latest LTS
- **NestJS 11** - Progressive Node.js framework
- **Express** - Web framework (pod NestJS)
- **TypeScript 6** - Type safety

#### Database
- **SQLite** - Embedded database
- **better-sqlite3** - Fast SQLite driver
- **Migrations** - Custom migration system

#### Real-time
- **ws** - WebSocket library
- **WebSocket server** - Real-time sync across users

#### Authentication
- **jsonwebtoken** - JWT tokens
- **bcryptjs** - Password hashing
- **@simplewebauthn/server** - WebAuthn/Passkeys server
- **otplib** - TOTP dla 2FA
- **qrcode** - QR code generation (2FA setup)

#### External Integrations
- **@modelcontextprotocol/sdk** - MCP (AI assistant integration)
- **nodemailer** - Email sending (SMTP)
- **undici** - Modern HTTP client
- **fast-xml-parser** - XML parsing (KML, GPX)

#### File Processing
- **Jimp** - Image processing
- **archiver** - ZIP creation
- **unzipper** - ZIP extraction
- **pdf-parse** - PDF parsing
- **multer** - File uploads

#### API & Docs
- **@nestjs/swagger** - OpenAPI/Swagger documentation
- **Swagger UI** - Interactive API docs

#### Security
- **helmet** - Security headers
- **cors** - CORS handling
- **cookie-parser** - Cookie parsing
- **compression** - Response compression

#### Utilities
- **node-cron** - Scheduled jobs (auto-backups, cleanup)
- **semver** - Version comparison
- **uuid** - UUID generation
- **dotenv** - Environment variables
- **tz-lookup** - Timezone lookup
- **tsx** - TypeScript execution

---

### Shared

#### Monorepo Structure
- **npm workspaces** - Monorepo management
- **@trek/shared** - Shared types, validators, utils między client i server

#### Validation
- **Zod** - Runtime schema validation (shared między frontend i backend)

---

### DevOps & Infrastructure

#### Containerization
- **Docker** - Containerization
  - Production-ready Dockerfile
  - Multi-stage build
  - Security-hardened (read-only, no-new-privileges, dropped capabilities)
- **Docker Compose** - Local development i production deployment

#### Orchestration
- **Kubernetes** - Production orchestration
- **Helm** - Kubernetes package manager
  - Official Helm chart at `https://chart.liketrek.com`

#### Reverse Proxy Support
- **Nginx** - Configuration examples
- **Caddy** - Auto HTTPS
- **Traefik** - Compatible
- **WebSocket upgrade** - Full support

---

### Testing

#### Testing Frameworks
- **Vitest** - Unit i integration tests
  - Coverage: v8 i Istanbul
  - Watch mode
  - Parallel execution
- **Playwright** - E2E tests + screenshot testing
- **@testing-library/react** - React component testing
- **Supertest** - HTTP API testing
- **MSW** (Mock Service Worker) - API mocking

#### Quality Tools
- **ESLint** - Linting (TypeScript ESLint)
- **Prettier** - Code formatting
  - `prettier-plugin-tailwindcss` - Tailwind class sorting
  - `prettier-plugin-organize-imports` - Import organization
- **TypeScript** - Type checking

---

### Maps & Geo Data

#### Map Providers
- **Leaflet** - Default, open-source
- **Mapbox GL** - Premium (API key required)
- **MapLibre GL** - Open-source Mapbox alternative
- **OpenStreetMap** - Tile source (free)

#### Place Data
- **Google Places API** - Rich place data (photos, ratings, hours)
- **OpenStreetMap Nominatim** - Free geocoding (no API key)

#### Weather
- **Open-Meteo** - Weather forecasts (no API key required!)
  - 16-day forecasts
  - Historical climate data

#### Geo Boundaries
- **geoBoundaries** - Country i sub-national boundaries (CC BY 4.0)

---

### External Services (Optional)

#### Photo Management
- **Immich** - Self-hosted photo management
- **Synology Photos** - NAS photo integration

#### Flight Tracking
- **AirTrail** - Self-hosted flight tracker

#### Notifications
- **SMTP** - Email notifications
- **ntfy.sh** - Push notifications
- **Webhooks** - Custom integrations

#### SSO Providers
- **Google** - OAuth/OIDC
- **Apple** - OAuth/OIDC
- **Authentik** - Self-hosted IdP
- **Keycloak** - Self-hosted IdP
- **Any OIDC provider** - Standard compliance

#### AI/LLM
- **Claude** (Anthropic) - Via MCP
- **Cursor** - Via MCP
- **Other MCP clients** - Any MCP-compatible AI assistant

---

## 🏛️ Architektura

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Client (React 19)                     │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │   Pages     │  │  Components  │  │  Zustand      │  │
│  │  (Routing)  │  │   (UI/UX)    │  │  (State)      │  │
│  └─────────────┘  └──────────────┘  └───────────────┘  │
│         │                 │                  │           │
│         └─────────────────┴──────────────────┘           │
│                          │                               │
│                   ┌──────▼──────┐                        │
│                   │  API Client │                        │
│                   │  (Axios)    │                        │
│                   └──────┬──────┘                        │
└──────────────────────────┼──────────────────────────────┘
                           │ HTTP/WebSocket
                           │
┌──────────────────────────▼──────────────────────────────┐
│              Server (NestJS 11 + Express)                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
│  │  Routes  │  │ Services │  │   Auth   │  │  WebSocket│
│  │  (REST)  │  │(Business)│  │(JWT/OIDC)│  │  (ws)    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬────┘ │
│       │             │              │             │       │
│       └─────────────┴──────────────┴─────────────┘       │
│                          │                               │
│                   ┌──────▼──────┐                        │
│                   │  Database   │                        │
│                   │  (SQLite)   │                        │
│                   └─────────────┘                        │
└─────────────────────────────────────────────────────────┘
                           │
                           │ External APIs
                           │
┌──────────────────────────▼──────────────────────────────┐
│              External Services (Optional)                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
│  │  Google  │  │  Immich  │  │   SMTP   │  │   MCP   │ │
│  │  Places  │  │ (Photos) │  │  (Email) │  │  (AI)   │ │
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Monorepo Structure

```
TREK/
├── client/              # React frontend
│   ├── src/
│   │   ├── pages/       # Page components (routing)
│   │   ├── components/  # Reusable UI components
│   │   ├── store/       # Zustand state management
│   │   ├── api/         # API client (Axios)
│   │   ├── hooks/       # Custom React hooks
│   │   ├── utils/       # Utility functions
│   │   ├── types.ts     # TypeScript types
│   │   └── i18n/        # Internationalization
│   ├── public/          # Static assets
│   └── package.json
│
├── server/              # NestJS backend
│   ├── src/
│   │   ├── nest/        # NestJS modules (controllers, plugins)
│   │   ├── routes/      # Legacy Express routes (migration in progress)
│   │   ├── services/    # Business logic
│   │   ├── middleware/  # Express middleware
│   │   ├── db/          # Database (schema, migrations, seeds)
│   │   ├── mcp/         # Model Context Protocol (AI)
│   │   ├── websocket.ts # WebSocket server
│   │   ├── scheduler.ts # Cron jobs
│   │   └── index.ts     # Entry point
│   ├── tests/           # Tests (unit, integration, e2e)
│   └── package.json
│
├── shared/              # Shared code (types, validators)
│   ├── src/
│   │   ├── types/       # Shared TypeScript types
│   │   ├── validators/  # Zod schemas
│   │   └── utils/       # Shared utilities
│   └── package.json
│
├── docs/                # Documentation
├── wiki/                # In-app help pages
├── charts/              # Helm charts (Kubernetes)
├── docker-compose.yml   # Docker Compose config
├── Dockerfile           # Production Docker image
└── package.json         # Root workspace config
```

### Database Schema

**SQLite database** w `data/travel.db` z tabelami:

**Core:**
- `users` - User accounts
- `trips` - Wycieczki
- `trip_members` - Membership wycieczek
- `days` - Dni wycieczek
- `places` - Miejsca
- `day_places` - Przypisanie miejsc do dni

**Travel Management:**
- `reservations` - Rezerwacje (flights, hotels, etc.)
- `costs` - Wydatki
- `packing_items` - Listy pakowania
- `documents` - Załączone pliki

**Collaboration:**
- `collab_messages` - Wiadomości chat
- `collab_notes` - Współdzielone notatki
- `collab_polls` - Ankiety
- `invites` - Linki zaproszeniowe

**Journey (Travel Journal):**
- `journeys` - Wpisy dziennika
- `journey_photos` - Zdjęcia w dzienniku

**Atlas:**
- `atlas_countries` - Odwiedzone kraje
- `atlas_regions` - Odwiedzone regiony

**Vacay:**
- `vacay_years` - Vacation tracking per rok
- `vacay_periods` - Okresy urlopowe

**Admin & System:**
- `categories` - Kategorie miejsc
- `packing_templates` - Szablony list pakowania
- `notifications` - In-app notifications
- `oauth_tokens` - OAuth tokens (MCP)
- `sessions` - WebAuthn sessions
- `totp_secrets` - 2FA secrets

---

### Data Flow

#### 1. User Action (Frontend)
```
User clicks → React component → Zustand action → API client (Axios)
```

#### 2. HTTP Request
```
API client → Express middleware → NestJS controller → Service layer
```

#### 3. Business Logic
```
Service → Database query (better-sqlite3) → Response
```

#### 4. Real-time Updates
```
Service → WebSocket broadcast → All connected clients → Zustand update → UI re-render
```

#### 5. External API Calls
```
Service → External API (Google Places, Open-Meteo, etc.) → Response → Cache → Return
```

---

### Key Design Patterns

#### Frontend
- **Component-based architecture** - Reusable components
- **Container/Presenter pattern** - Smart/dumb components
- **Custom hooks** - Logic reuse (useTrip, useAuth, etc.)
- **State management** - Zustand stores (authStore, tripStore, etc.)
- **API client abstraction** - Centralized w `api/client.ts`

#### Backend
- **Service layer** - Business logic oddzielona od routingu
- **Repository pattern** - Database access abstraction (w przygotowaniu)
- **Dependency injection** - NestJS DI container
- **Middleware pattern** - Authentication, logging, error handling
- **Plugin system** - Rozszerzalność przez pluginy

#### Database
- **Migrations** - Versioned schema changes
- **Seeds** - Initial data i demo data
- **Soft deletes** - Preservation of data

---

## 🚀 Deployment

### Supported Platforms

#### Docker (Recommended)
```bash
docker run -d -p 3000:3000 \
  -e ENCRYPTION_KEY=$(openssl rand -hex 32) \
  -v ./data:/app/data \
  -v ./uploads:/app/uploads \
  mauriceboe/trek
```

#### Docker Compose
- Full production setup z security hardening
- Read-only filesystem
- Dropped capabilities
- Health checks

#### Kubernetes + Helm
```bash
helm repo add trek https://chart.liketrek.com
helm install trek trek/trek
```

#### Unraid
- Dedicated template w Unraid Community Apps
- Pre-configured settings

---

### System Requirements

**Minimum:**
- 1 CPU core
- 512 MB RAM
- 1 GB disk space

**Recommended:**
- 2+ CPU cores
- 2 GB RAM
- 10 GB disk space (dla uploads i backups)

**Network:**
- Outbound: API calls (Google, Open-Meteo, etc.)
- Inbound: HTTPS (443) + WebSocket

---

### Environment Variables

**Kluczowe:**
- `ENCRYPTION_KEY` - At-rest encryption (generate: `openssl rand -hex 32`)
- `PORT` - Server port (default: 3000)
- `NODE_ENV` - Environment (production/development)
- `APP_URL` - Public URL (required dla OIDC)

**OIDC/SSO:**
- `OIDC_ISSUER`
- `OIDC_CLIENT_ID`
- `OIDC_CLIENT_SECRET`
- `OIDC_DISPLAY_NAME`

**Optional APIs:**
- `UNSPLASH_ACCESS_KEY` - Unsplash image search

**Pełna lista:** Zobacz README.md, sekcja "Environment variables"

---

## 📊 Performance & Scalability

### Performance Features

- **SQLite** - Extremely fast dla single-server deployments
- **WebSocket** - Efficient real-time updates
- **Compression** - gzip/brotli response compression
- **Service Worker** - Client-side caching
- **Image optimization** - Jimp dla resize i thumbnails
- **Lazy loading** - React.lazy dla code splitting
- **Virtual scrolling** - react-window dla długich list

### Current Limitations

- **Single-server** - SQLite nie jest distributed
- **Concurrent writes** - SQLite ma locks, ale w praktyce nie problem dla travel planner use case
- **File storage** - Local filesystem (nie S3/cloud storage)

### Scalability Path

Dla bardzo large deployments można rozważyć:
- **PostgreSQL migration** - W roadmap
- **Redis** - Session storage i caching
- **S3-compatible storage** - Dla uploads
- **Horizontal scaling** - Multiple instances z shared DB

---

## 🔒 Security

### Security Features

1. **Authentication**
   - bcrypt password hashing
   - JWT with short expiration
   - TOTP 2FA
   - WebAuthn/Passkeys
   - OAuth 2.1 / OIDC

2. **Encryption**
   - At-rest encryption dla secrets
   - HTTPS enforcement (production)
   - Secure cookies (httpOnly, secure, sameSite)

3. **Authorization**
   - Role-based access control
   - Trip-level permissions
   - OAuth scopes (27 granular scopes)

4. **Infrastructure**
   - Helmet.js security headers
   - CORS protection
   - CSRF protection
   - Rate limiting (MCP endpoints)
   - SSRF protection

5. **Container Security**
   - Read-only filesystem
   - Non-root user
   - Dropped capabilities
   - No new privileges

### Security Best Practices

✅ Always use HTTPS w production  
✅ Set strong `ENCRYPTION_KEY`  
✅ Enable 2FA dla admins  
✅ Regular backups  
✅ Keep up to date (security patches)  
✅ Use reverse proxy (Nginx/Caddy)  
✅ Network isolation (firewall)

---

## 🎯 Use Cases

### 1. Personal Travel Planning
- Plan własnych wycieczek
- Track visited countries
- Journal memories
- Manage budget

### 2. Group Travel
- Plan wycieczki z rodziną/przyjaciółmi
- Real-time collaboration
- Split costs fairly
- Shared packing lists

### 3. Travel Agencies
- Plan client trips
- Professional PDF exports
- Client portals (invite links)
- Multi-trip management

### 4. Digital Nomads
- Track stays across countries
- Visa tracking (via Atlas)
- Budget management (multi-currency)
- Document storage

### 5. AI-Assisted Planning
- Use Claude/Cursor via MCP
- Auto-generate itineraries
- Smart packing suggestions
- Budget optimization

---

## 📈 Roadmap & Future

### Actively Developed Features

Check [public roadmap](https://kanban.pakulat.org/shared/I4wxF6inOOMB0C6hH6kQm3efyNxFjwyI)

### Potential Future Features

- **PostgreSQL support** - Dla larger deployments
- **Mobile apps** (native) - Oprócz PWA
- **Advanced analytics** - Travel statistics
- **Social features** - Public trip sharing, followers
- **Marketplace** - User-contributed templates, itineraries

---

## 🤝 Community & Support

### Links

- **Demo:** https://demo.liketrek.com
- **Docker Hub:** https://hub.docker.com/r/mauriceboe/trek
- **Discord:** https://discord.gg/NhZBDSd4qW
- **GitHub:** https://github.com/liketrek/TREK
- **Roadmap:** https://kanban.pakulat.org/shared/I4wxF6inOOMB0C6hH6kQm3efyNxFjwyI

### Support Options

- 💬 Discord community
- 🐛 GitHub Issues
- 📚 Wiki documentation (in-app Help)
- ☕ Ko-fi / Buy Me a Coffee

---

## 📝 License & Attribution

### License
**AGPL v3** - Self-host freely dla personal lub internal company use. Jeśli modyfikujesz i oferujesz TREK jako network service dla third parties, twoje modyfikacje muszą być open-sourced pod tą samą licencją.

### Data Attributions
- **geoBoundaries** (Atlas boundaries) - CC BY 4.0
- **OpenStreetMap** - ODbL

---

## 🎓 Learning Resources

### For Developers

**Frontend:**
- React 19 docs: https://react.dev
- Zustand: https://zustand.docs.pmnd.rs
- Tailwind CSS: https://tailwindcss.com
- Leaflet: https://leafletjs.com

**Backend:**
- NestJS: https://nestjs.com
- better-sqlite3: https://github.com/WiseLibs/better-sqlite3

**Testing:**
- Vitest: https://vitest.dev
- Playwright: https://playwright.dev

**MCP:**
- Model Context Protocol: https://modelcontextprotocol.io

---

## 📊 Quick Stats

| Metric | Value |
|--------|-------|
| **Version** | 3.4.1 |
| **Lines of Code** | ~150k+ (estimate) |
| **Languages** | TypeScript (primary), JavaScript |
| **Components** | 100+ React components |
| **API Endpoints** | 80+ REST endpoints |
| **MCP Tools** | 150+ tools |
| **Supported Languages** | 20 |
| **Database Tables** | 40+ tables |
| **Test Coverage** | High (unit + integration + e2e) |
| **Docker Pulls** | [Check Docker Hub] |
| **GitHub Stars** | [Check GitHub] |

---

## 🔥 Why TREK?

### Compared to Alternatives

**vs. TripIt / TripCase (Commercial):**
- ✅ Self-hosted (own your data)
- ✅ No subscription fees
- ✅ Open-source (customize freely)
- ✅ Group collaboration built-in
- ✅ AI integration (MCP)

**vs. Google My Maps (Free):**
- ✅ Trip-specific organization
- ✅ Budget & cost tracking
- ✅ Packing lists
- ✅ Travel journal
- ✅ Real-time collaboration
- ✅ Offline support

**vs. Notion Travel Templates:**
- ✅ Purpose-built dla travel
- ✅ Interactive maps
- ✅ Real-time sync
- ✅ Mobile PWA
- ✅ Self-hosted

### Key Differentiators

1. **Self-hosted First** - Pełna kontrola, żadnego cloud lock-in
2. **Real-time Collaboration** - Nie async, instant updates
3. **All-in-one** - Planner + Budget + Packing + Journal w jednej app
4. **Modern Tech Stack** - React 19, NestJS 11, cutting-edge
5. **AI-ready** - Native MCP support
6. **Open Source** - Community-driven, transparent

---

## 💡 Development Philosophy

### Core Principles

1. **Self-hosting First** - Designed dla easy self-hosting
2. **Privacy by Design** - Your data, your server
3. **User Experience** - Modern, intuitive, fast
4. **Developer Experience** - Clean code, good tests, clear docs
5. **Modularity** - Plugin system, addons, extensible
6. **Standards Compliance** - OAuth 2.1, OIDC, WebAuthn, MCP
7. **Progressive Enhancement** - Works without JS (basic), enhanced with JS
8. **Mobile First** - Responsive, touch-optimized, PWA

---

*Last updated: July 26, 2026*  
*Document version: 1.0*  
*TREK version: 3.4.1*
