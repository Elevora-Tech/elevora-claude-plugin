# Feature Completeness Reference Checklist

Detailed checklist for the Feature Completeness Analyst agent. Evaluate every applicable item against the target codebase.

---

## 1. Route/Page Implementation Status

For each route/page discovered in the project:

- [ ] Page renders real content (not placeholder/lorem ipsum)
- [ ] Page has a clear purpose and complete UI
- [ ] Navigation to/from the page works correctly
- [ ] Page handles authenticated vs unauthenticated users appropriately
- [ ] Dynamic routes handle valid and invalid parameters

**Grep patterns for placeholders:**
```
lorem ipsum|placeholder|coming soon|under construction   # placeholder content
TODO|FIXME|HACK|XXX|TEMP|REMOVEME                        # incomplete markers
throw new Error\(['"]not implemented|NotImplementedError  # stub implementations
console\.log\(['"]todo|console\.log\(['"]fix              # debug TODOs
```

### Route Status Categories
For each route, classify as:
- **Complete**: Fully functional with real data
- **Partial**: Core functionality works but missing polish (error states, loading, etc.)
- **Stub**: Route exists but content is placeholder or minimal
- **Broken**: Route errors or shows wrong content
- **Missing**: Expected route doesn't exist (e.g., settings page referenced but not created)

---

## 2. CRUD Completeness Per Resource

For each data model/resource in the application:

- [ ] **Create**: Form/input exists, validation works, success feedback shown
- [ ] **Read**: List view exists, detail view exists, data loads correctly
- [ ] **Update**: Edit form exists, pre-populated with current data, saves correctly
- [ ] **Delete**: Delete action exists, confirmation dialog shown, cascade handled

**Grep patterns:**
```
create|insert|POST.*\/api          # create operations
findMany|findAll|GET.*\/api        # read operations (list)
findUnique|findFirst|findById      # read operations (detail)
update|PUT|PATCH.*\/api            # update operations
delete|remove|DELETE.*\/api        # delete operations
```

### Missing CRUD Signals
- API route exists but no corresponding UI
- UI button exists but handler is empty or `console.log` only
- Model has fields that no form can edit
- List view exists but no way to create new items

---

## 3. Error States

Every page and interaction should handle errors gracefully.

### Global Error Handling
- [ ] Root error boundary exists (React `ErrorBoundary`, Next.js `error.tsx`)
- [ ] Error boundary shows user-friendly message (not raw stack trace)
- [ ] Error boundary offers recovery action (retry, go home, contact support)
- [ ] 404 page exists and is styled consistently
- [ ] 500/error page exists for server errors

**Grep patterns:**
```
error\.tsx|error\.jsx|error\.js                # Next.js error boundaries
ErrorBoundary|errorElement                     # React error boundaries
not-found\.tsx|not-found\.jsx|404              # 404 pages
global-error\.tsx|global-error\.jsx            # Next.js global error
componentDidCatch|getDerivedStateFromError     # class error boundaries
```

### Per-Page Error Handling
- [ ] API calls have try-catch or error handling
- [ ] Failed data fetches show error UI (not blank screen)
- [ ] Form submissions show error messages on failure
- [ ] Network errors are handled (offline state, timeout)
- [ ] Permission errors redirect or show appropriate message

**Grep patterns:**
```
try\s*\{|\.catch\(|onError                     # error handling
isError|error\s*&&|error\s*\?                  # error state rendering
toast\.error|showError|setError                # error UI feedback
```

---

## 4. Loading States

Every async operation should show loading feedback.

- [ ] Initial page loads show skeleton or spinner
- [ ] Data fetching shows loading indicator
- [ ] Button submissions show loading state (disabled + spinner)
- [ ] Navigation transitions show progress indicator
- [ ] Large lists show pagination or infinite scroll loading

**Grep patterns:**
```
isLoading|isPending|loading\s*&&|loading\s*\?     # loading state rendering
Skeleton|skeleton|Spinner|spinner                   # loading components
Suspense|fallback=                                  # React Suspense
loading\.tsx|loading\.jsx                           # Next.js loading files
disabled.*loading|loading.*disabled                 # button loading states
useTransition|startTransition                       # React transitions
```

### Missing Loading Signals
- Async function called with no loading state set
- `useQuery`/`useSWR` without loading state rendering
- Form submit handler with no loading indicator
- Page with data fetching but no Suspense or loading UI

---

## 5. Empty States

First-time and no-data experiences should be helpful, not blank.

- [ ] List pages show helpful message when no items exist
- [ ] Empty states suggest next action (create first item, invite team, etc.)
- [ ] Dashboard widgets handle zero-data gracefully
- [ ] Search shows "no results" with suggestions
- [ ] Filter results show "no matches" with clear-filter action

**Grep patterns:**
```
\.length\s*===?\s*0|isEmpty|no.*found|no.*results    # empty checks
empty.?state|EmptyState|NoData|NoResults              # empty state components
getStarted|get.?started|create.*first                  # first-use guidance
```

### Missing Empty State Signals
- `.map()` on array with no empty check before it
- Conditional rendering only for data present, no else clause
- Dashboard charts/graphs with no zero-state handling

---

## 6. Form Completeness

Every form should be complete and user-friendly.

- [ ] All form fields have labels (visible or aria-label)
- [ ] Required fields are marked
- [ ] Client-side validation exists with clear error messages
- [ ] Server-side validation exists (not just client-side)
- [ ] Submit button shows loading state during submission
- [ ] Success feedback shown after submission (toast, redirect, inline message)
- [ ] Error feedback shown on submission failure
- [ ] Form preserves input on error (doesn't clear on failed submit)
- [ ] Multi-step forms have progress indicator and back navigation

**Grep patterns:**
```
onSubmit|handleSubmit|action=                  # form submission
required|validate|schema                        # validation
useForm|useFormState|formState                  # form libraries
react-hook-form|formik|@conform-to             # form library usage
zod|yup|joi.*schema                            # validation schemas
```

---

## 7. Navigation & Routing

- [ ] All nav links work and go to correct destination
- [ ] Active nav item is highlighted
- [ ] Breadcrumbs exist on nested pages
- [ ] Deep linking works (direct URL access to any page)
- [ ] Back button behavior is correct (no broken history)
- [ ] 404 page for unknown routes
- [ ] Auth redirects work (login → intended page)
- [ ] Logout redirects to appropriate page

**Grep patterns:**
```
<Link|<NavLink|router\.push|navigate\(          # navigation
href=|to=                                        # link targets
breadcrumb|Breadcrumb                            # breadcrumbs
redirect|useRouter|useNavigate                   # routing
pathname|usePathname|useLocation                 # active route detection
```

---

## 8. Data Consistency

- [ ] Optimistic updates revert on failure
- [ ] Cache invalidation after mutations (React Query, SWR revalidation)
- [ ] Real-time data has sync mechanism (polling, websocket, SSE)
- [ ] Concurrent edit handling (last-write-wins or conflict resolution)
- [ ] Deleted items removed from all views immediately

**Grep patterns:**
```
invalidateQueries|mutate\(|revalidate          # cache invalidation
optimistic|rollback|onMutate                    # optimistic updates
refetchOnWindowFocus|refetchInterval            # auto-refetch
```

---

## 9. TODO/FIXME/HACK Scan

Scan the entire codebase for incomplete work markers:

**Grep patterns:**
```
TODO|FIXME|HACK|XXX|TEMP|REMOVEME|WORKAROUND
@todo|@fixme|@hack
eslint-disable.*next-line                        # suppressed warnings
// @ts-ignore|// @ts-expect-error               # suppressed type errors
```

Classify each finding:
- **CRITICAL**: TODO on security, auth, payment, or data integrity code
- **WARNING**: TODO on user-facing features or error handling
- **INFO**: TODO on nice-to-have features or code cleanup

---

## 10. Feature Flags & Dev Mode

- [ ] No features behind `if (process.env.NODE_ENV === 'development')` that should be in production
- [ ] Feature flags have clear documentation or management system
- [ ] Debug UI elements are hidden in production
- [ ] Console.log statements removed from production code (or using proper logger)
- [ ] Test/seed data not visible in production

**Grep patterns:**
```
NODE_ENV.*development.*&&                      # dev-only features
console\.log\(|console\.debug\(               # debug logging
__DEV__|isDev|isDebug                          # dev mode flags
seed|mock|fake|dummy                           # test data
feature.?flag|Feature.?Flag|FEATURE_           # feature flags
```

---

## Scoring Guide

- **CRITICAL**: Placeholder pages accessible to users, broken routes, missing error boundaries, forms that lose data on error, CRUD operations that silently fail
- **WARNING**: Missing loading states on some pages, no empty states, TODOs in user-facing code, missing 404 page, incomplete form validation
- **GOOD**: All routes functional, error handling present, loading states on most pages, forms validate input, 404 page exists
- **EXCELLENT**: Empty states guide users, all forms have complete validation + feedback, zero TODO/FIXME in production paths, skeleton loaders, optimistic updates
