# Performance Audit Reference Checklist

Detailed checklist for the Performance Analyst agent. Evaluate every applicable item against the target codebase.

---

## 1. Bundle Size & Code Splitting

### Bundle Analysis
- [ ] No barrel file re-exports that pull in entire libraries (`import { x } from '@/lib'`)
- [ ] Tree shaking works (ESM imports, no CommonJS in client bundles)
- [ ] No duplicate dependencies (same library in multiple bundles)
- [ ] Large libraries imported selectively (e.g., `lodash/get` not `lodash`)

**Grep patterns:**
```
import.*from\s+['"]lodash['"]                      # full lodash import (WARNING)
import.*from\s+['"]moment['"]                      # moment.js (suggest dayjs/date-fns)
import.*from\s+['"]@mui/icons-material['"]         # full MUI icons import
import.*from\s+['"]@fortawesome                    # full FontAwesome import
require\(.*\)                                       # CommonJS in client code
export\s*\*\s*from                                  # barrel file re-exports
```

### Dynamic Imports & Code Splitting
- [ ] Route-based code splitting (lazy routes)
- [ ] Heavy components use dynamic imports (`React.lazy`, `next/dynamic`)
- [ ] Third-party widgets load on demand (analytics, chat widgets, maps)
- [ ] Below-the-fold components are deferred

**Grep patterns:**
```
React\.lazy|lazy\(                                  # React lazy loading
next\/dynamic|dynamic\(                             # Next.js dynamic imports
import\(.*\)                                        # dynamic imports
Suspense|fallback=                                  # Suspense boundaries
```

### Bundle Size Targets
- [ ] Initial JS bundle < 200KB gzipped (ideal < 100KB)
- [ ] Per-route chunks are reasonably sized
- [ ] No single dependency > 50KB gzipped in client bundle
- [ ] Bundle analyzer configured (webpack-bundle-analyzer, @next/bundle-analyzer)

**Grep patterns:**
```
@next\/bundle-analyzer|webpack-bundle-analyzer      # bundle analysis tools
analyze|ANALYZE                                     # analyze scripts
```

---

## 2. Image Optimization

### Image Components
- [ ] Framework image component used (Next/Image, Nuxt Image, etc.)
- [ ] Images have `width` and `height` attributes (prevents CLS)
- [ ] Images use responsive `srcset` or `sizes` attribute
- [ ] Hero/LCP images have `priority` prop (Next.js) or `fetchpriority="high"`

**Grep patterns:**
```
next\/image|<Image                                  # Next.js Image component
<img\s(?![^>]*width)                                # img without width (CLS risk)
<img\s(?![^>]*height)                               # img without height (CLS risk)
priority|fetchpriority                              # priority loading
sizes=|srcset=                                      # responsive images
```

### Format & Compression
- [ ] Modern formats used (WebP, AVIF) or auto-conversion enabled
- [ ] Images are appropriately sized (not serving 4000px images for 400px display)
- [ ] Image CDN or optimization service used (Vercel, Cloudinary, imgix)
- [ ] SVGs used for icons and simple graphics

**Grep patterns:**
```
\.png['"]|\.jpg['"]|\.jpeg['"]                      # non-optimized formats in imports
\.webp|\.avif                                       # modern formats
cloudinary|imgix|imagekit                           # image CDN
```

### Lazy Loading
- [ ] Below-fold images use `loading="lazy"`
- [ ] Background images in CSS are optimized
- [ ] Image galleries/carousels load images on demand

**Grep patterns:**
```
loading=["']lazy["']                                # lazy loading attribute
loading=["']eager["']                               # eager loading (check if needed)
IntersectionObserver.*img                           # custom lazy loading
```

---

## 3. Caching Strategy

### HTTP Caching
- [ ] Static assets have long cache headers (Cache-Control: max-age=31536000, immutable)
- [ ] HTML/API responses have appropriate cache headers
- [ ] ETags or Last-Modified headers used for conditional requests
- [ ] CDN caching configured for static assets

**Grep patterns:**
```
Cache-Control|cache-control                         # cache headers
max-age|s-maxage|stale-while-revalidate            # cache directives
ETag|etag|Last-Modified                             # conditional caching
```

### Data Fetching Cache
- [ ] Client-side caching library used (React Query, SWR, Apollo Cache)
- [ ] Stale-while-revalidate pattern for frequently changing data
- [ ] Cache invalidation on mutations
- [ ] Static pages use ISR or static generation where possible

**Grep patterns:**
```
useQuery|useSWR|useApolloQuery                      # data fetching with cache
staleTime|dedupingInterval|revalidate               # cache timing
getStaticProps|generateStaticParams                  # static generation
revalidate|revalidateTag|revalidatePath             # ISR/on-demand revalidation
unstable_cache|cache\(                              # Next.js caching
```

### Service Worker / Offline
- [ ] Service worker configured if PWA (or documented as not needed)
- [ ] Offline fallback page exists if applicable
- [ ] Cache-first strategy for static assets if service worker exists

---

## 4. Database Performance

### Query Efficiency
- [ ] No N+1 queries (queries inside loops, especially in list endpoints)
- [ ] Database queries use `select` to fetch only needed fields
- [ ] Pagination implemented for list endpoints (not fetching all records)
- [ ] Complex queries use appropriate indexes (check schema for index definitions)

**Grep patterns:**
```
\.map\(.*await|forEach.*await.*find|for.*await.*query  # N+1 pattern
findMany|find\(\)|\.all\(\)                              # bulk queries (check for pagination)
select:\s*\{|\.select\(                                  # field selection
take:|limit:|\.limit\(|LIMIT                             # pagination
skip:|offset:|\.offset\(|OFFSET                          # pagination offset
@@index|createIndex|add_index|\.index\(                  # database indexes
include:|populate\(|\.join\(                              # eager loading (check necessity)
```

### Connection Management
- [ ] Database client is singleton (not creating new connection per request)
- [ ] Connection pooling configured
- [ ] Connection timeouts set
- [ ] Prepared statements used for repeated queries

**Grep patterns:**
```
new PrismaClient|createPool|createClient            # client instantiation (should be singleton)
global.*prisma|globalThis.*prisma                    # singleton pattern
pool.*max|connectionLimit                            # pool configuration
```

---

## 5. Core Web Vitals Indicators

### Largest Contentful Paint (LCP)
- [ ] Hero images/content load quickly (no render-blocking resources before LCP)
- [ ] Fonts don't block rendering (`font-display: swap` or `optional`)
- [ ] Critical CSS is inlined or loaded first
- [ ] Server-side rendering for LCP content
- [ ] No large synchronous scripts before main content

**Grep patterns:**
```
font-display:\s*swap|font-display:\s*optional       # font loading strategy
@font-face|next\/font|google.*fonts                 # font usage
<script(?![^>]*async|[^>]*defer)                    # render-blocking scripts
preload|prefetch|preconnect                         # resource hints
```

### Cumulative Layout Shift (CLS)
- [ ] Images/videos have explicit dimensions
- [ ] No content injected above visible content after load
- [ ] Web fonts don't cause layout shift (FOUT/FOIT handled)
- [ ] Ads/embeds have reserved space
- [ ] Dynamic content has reserved space (skeleton loaders)

**Grep patterns:**
```
<img(?![^>]*width)(?![^>]*height)                   # images without dimensions
aspect-ratio|aspect-\[                              # aspect ratio CSS
min-h-|min-height                                   # reserved space
Skeleton|skeleton                                    # skeleton loaders
```

### Interaction to Next Paint (INP)
- [ ] No long-running synchronous operations on interaction
- [ ] Heavy computations use Web Workers or are debounced
- [ ] Event handlers don't block the main thread
- [ ] Virtualized lists for large datasets (react-virtuoso, react-window, tanstack-virtual)

**Grep patterns:**
```
react-virtuoso|react-window|react-virtual|@tanstack\/virtual  # list virtualization
useTransition|startTransition                                   # React concurrent features
requestIdleCallback|requestAnimationFrame                       # scheduling
Worker|new Worker|web-worker                                    # Web Workers
useDeferredValue                                                # deferred values
debounce|throttle|useDebounce                                   # input debouncing
```

---

## 6. Font Loading

- [ ] Font files are self-hosted or loaded from CDN with `preconnect`
- [ ] `next/font` used (Next.js projects) for automatic optimization
- [ ] `font-display: swap` or `optional` set (not `block`)
- [ ] Font subset used if only Latin characters needed
- [ ] Number of font families limited (< 3 families)
- [ ] Font files use WOFF2 format

**Grep patterns:**
```
next\/font|@next\/font                              # Next.js font optimization
font-display                                        # font display strategy
woff2|\.woff2                                       # modern font format
preconnect.*fonts|dns-prefetch.*fonts               # font preconnection
@font-face                                          # custom font declarations
```

---

## 7. Compression & Transfer

- [ ] Gzip or Brotli compression enabled on server
- [ ] API responses are compressed
- [ ] Large JSON payloads are paginated or streamed
- [ ] No unnecessary data sent to client (over-fetching)

**Grep patterns:**
```
compression|gzip|brotli                             # compression middleware
Content-Encoding|accept-encoding                    # encoding headers
stream|ReadableStream|createReadStream              # streaming
```

---

## 8. Third-Party Scripts

- [ ] Third-party scripts loaded with `async` or `defer`
- [ ] Analytics/tracking scripts don't block rendering
- [ ] Third-party scripts loaded from CDN (not bundled)
- [ ] Number of third-party domains minimized (fewer DNS lookups)
- [ ] `next/script` used with appropriate strategy (Next.js)

**Grep patterns:**
```
<script.*src=.*http                                 # external scripts
next\/script|Script.*strategy                       # Next.js Script component
gtag|analytics|mixpanel|segment                     # analytics scripts
intercom|drift|crisp|hubspot                        # chat/support widgets
```

---

## Scoring Guide

- **CRITICAL**: N+1 queries in list endpoints, no pagination on large datasets, render-blocking scripts before main content, full library imports dramatically inflating bundle, images without dimensions causing major CLS
- **WARNING**: No code splitting, no image optimization component, missing lazy loading, no caching strategy, full lodash/moment imports, no font optimization, no compression
- **GOOD**: Route-based code splitting, image component used, caching configured, database queries paginated, fonts load efficiently, gzip enabled
- **EXCELLENT**: Dynamic imports for heavy components, WebP/AVIF images, stale-while-revalidate caching, N+1 detection clean, virtualized lists, bundle < 100KB, all Core Web Vitals indicators addressed, third-party scripts deferred
