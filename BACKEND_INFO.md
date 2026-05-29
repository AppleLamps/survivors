# Survivor Sisters (survivorsisters.org) — Backend Intelligence Report

**Collected:** 2026-05-29  
**Source:** Public site inspection, JS bundle analysis, OIDC discovery, Convex HTTP API probing

---

## Executive Summary

[survivorsisters.org](https://survivorsisters.org/) is a **React SPA** built and hosted on the **[Hercules](https://usehercules.com)** platform, with **Convex** as the real-time backend/database, **Hercules Auth** (OIDC) for authentication, **Cloudflare** at the edge, and **Hercules CDN** for static assets/media.

---

## Hosting & Infrastructure

| Layer | Provider / Detail |
|--------|------------------|
| **DNS / CDN / WAF** | Cloudflare (`server: cloudflare`, IPs `104.21.23.132`, `172.67.211.75`) |
| **Site hosting** | Hercules (static SPA + `/_hercules/` reserved API path) |
| **Auth UI subdomain** | `auth.survivorsisters.org` (custom domain on Hercules Auth) |
| **Linked property** | `epsteinfilibuster.com` (Netlify Edge — separate site) |

**HTTP headers (main site):**
- `cf-cache-status`, `cf-placement: local-PDX`
- `referrer-policy: strict-origin-when-cross-origin`
- `x-content-type-options: nosniff`

---

## Platform IDs (Hercules)

| Key | Value |
|-----|--------|
| **Website ID** | `01KRVAP10SX85QWS3RCAMAA82E` |
| **Organization ID** | `org_01KRVANXVDGYEY1BN8QBPJ98W8` |
| **Auth tenant ID** | `01KRVAP10SH9JJGC7Q0P4S7HZY` |
| **Auth tenant name** | `01krvap10sx85qws3rcamaa82e` |
| **Analytics script** | `/_hercules/i.js?websiteId=...&organizationId=...` |

**Site author meta:** `Hercules`  
**Twitter:** `@usehercules`

---

## Frontend Stack

| Technology | Usage |
|------------|--------|
| **React** | UI (`#root` SPA) |
| **Vite** | Build (`/assets/index-*.js`, hashed bundles) |
| **React Router** | Client routing |
| **TanStack Query** | Server state (`QueryClientProvider`) |
| **Convex React** | Backend client (`ConvexProvider`) |
| **next-themes** | Dark/light mode |
| **react-hook-form + Zod** | Forms & validation |
| **oidc-client** (v1.37.0) | OAuth/OIDC via Hercules Auth |
| **Framer Motion** | Animations (`motion.div`) |
| **Sonner** | Toast notifications |
| **shadcn-style UI** | `src/components/ui/*` |

**Source map hints** (from `data-hercules-id` in bundle): full `src/` tree including pages, providers, layout.

---

## Routes (Client-Side)

| Path | Page |
|------|------|
| `/` | Home (`Index.tsx`) |
| `/about` | About |
| `/survivors` | Survivors |
| `/timeline` | Timeline |
| `/media` | Media |
| `/survivor-media` | Survivor media |
| `/gallery` | Gallery (Convex-backed uploads) |
| `/resources` | Resources |
| `/contact` | Contact / press inquiries |
| `/auth/callback` | OAuth callback |
| `*` | 404 |

---

## Backend: Convex

| Setting | Value |
|---------|--------|
| **Deployment URL** | `https://glorious-eel-524.convex.cloud` |
| **Status** | Active (`This Convex deployment is running`) |
| **Storage** | `https://glorious-eel-524.convex.cloud/api/storage/{uuid}` |

**HTTP API endpoints:**
- `POST /api/query` — public queries (confirmed working without auth)
- `POST /api/mutation` — mutations (require valid function + likely auth)
- `POST /api/action` — actions
- `.convex.site` — HTTP actions host (404 on root for this deployment)

### Convex Functions (from client bundle)

| Module | Function | Type | Purpose |
|--------|----------|------|---------|
| `emails` | `sendContactEmail` | mutation | Contact form → email |
| `galleryPhotos` | `listPhotos` | query | List gallery items (optional `category` filter) |
| `galleryPhotos` | `generateUploadUrl` | mutation | Presigned upload URL |
| `galleryPhotos` | `submitPhoto` | mutation | Save photo metadata after upload |
| `survivorNotes` | `listNotes` | query | Public survivor notes/messages |
| `survivorNotes` | `submitNote` | mutation | Submit a note |
| `users` | `updateCurrentUser` | mutation | Sync OIDC user profile to Convex |

### Confirmed Public Data (API probe)

**`galleryPhotos:listPhotos`** — returns records with:
- `_id`, `_creationTime`, `category`, `context`, `imageUrl`, `storageId`, `survivorName`

**`survivorNotes:listNotes`** — returns records with:
- `_id`, `_creationTime`, `authorName`, `message`

---

## Authentication: Hercules Auth (OIDC)

| Setting | Value |
|---------|--------|
| **Issuer** | `https://01krvap10sx85qws3rcamaa82e.hercules-auth.com` |
| **Custom auth domain** | `https://auth.survivorsisters.org` |
| **OAuth client ID** | `TXjuachJylpYWMijzjKAmsOSbjYfMkzb` |
| **Redirect URI** | `{origin}/auth/callback` |
| **Scopes** | `openid profile email offline_access` |
| **Response type** | `code` (PKCE S256) |
| **Prompt** | `select_account` |

### OIDC Endpoints

| Endpoint | URL |
|----------|-----|
| Authorization | `https://auth.survivorsisters.org/api/auth/oauth2/authorize` |
| Token | `https://01krvap10sx85qws3rcamaa82e.hercules-auth.com/api/auth/oauth2/token` |
| UserInfo | `https://01krvap10sx85qws3rcamaa82e.hercules-auth.com/api/auth/oauth2/userinfo` |
| JWKS | `https://01krvap10sx85qws3rcamaa82e.hercules-auth.com/api/auth/jwks` |
| Registration | `https://01krvap10sx85qws3rcamaa82e.hercules-auth.com/api/auth/oauth2/register` |
| Introspection | `https://01krvap10sx85qws3rcamaa82e.hercules-auth.com/api/auth/oauth2/introspect` |
| Revocation | `https://01krvap10sx85qws3rcamaa82e.hercules-auth.com/api/auth/oauth2/revoke` |
| End session | `https://auth.survivorsisters.org/api/auth/oauth2/end-session` |

### Enabled login providers (auth.survivorsisters.org)

- Google ✓
- Apple ✓
- Microsoft ✓
- Email OTP ✓
- Email/password ✗
- Facebook ✗
- LinkedIn ✗
- Phone OTP ✗

**Bot protection:** Cloudflare Turnstile (`turnstileSiteKey`: `0x4AAAAAACkrNnbWxGBVGG6H`)

**Auth environment:** `production`

---

## Analytics

| Service | Endpoint |
|---------|----------|
| **Hercules Analytics** | `https://analytics-ingest.hercules.app` |

Loaded via `/_hercules/i.js` — tracks visitor/session IDs, optional clicks/performance, auto-flush buffer.

---

## CDN & Media

| Asset type | Host |
|------------|------|
| Favicon / OG images | `https://hercules-cdn.com/...` (with Cloudflare image resizing) |
| Gallery images | `https://glorious-eel-524.convex.cloud/api/storage/...` |
| Auth branding CDN | `cdn.hercules.app`, `cdn-dev.hercules.app`, `hercules-cdn.com` |

---

## Contact Form Schema (Zod)

| Field | Validation |
|-------|------------|
| `name` | required |
| `email` | valid email |
| `inquiryType` | required select |
| `message` | min 10 characters |

Submits to Convex mutation `emails:sendContactEmail`.

---

## PWA

**Manifest:** `/site.webmanifest`  
- Name: Survivor Sisters — Truth, Justice & Healing  
- `display: standalone`, theme `#0f172a`  
- Icons: `/icon/icon-192.png`, `/icon/icon-512.png`

**robots.txt:** allows all crawlers (`Disallow:` empty)

---

## App Provider Tree

```
AuthProvider (Hercules OIDC)
  └─ ConvexProvider (glorious-eel-524.convex.cloud)
       └─ QueryClientProvider (TanStack Query)
            └─ TooltipProvider
                 └─ ThemeProvider (next-themes)
                      └─ Toaster + Routes
```

---

## Security Notes

1. **Public Convex queries** — `listPhotos` and `listNotes` are callable without authentication via the HTTP API.
2. **OAuth client ID** is public (expected for SPA).
3. **Turnstile site key** is public (expected).
4. **`/_hercules/`** path is reserved; returns `Not found, /_hercules is reserved for hercules apis` for directory access.

---

## Raw Artifacts

Investigation files saved under `/workspace/.firecrawl/`:
- `index.js` — main app bundle (~1MB)
- `hercules-i.js` — analytics bootstrap
- `home.html`, `robots.txt`, `site.webmanifest`
