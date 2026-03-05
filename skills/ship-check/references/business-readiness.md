# Business & Launch Readiness Reference Checklist

Detailed checklist for the Business & Launch Reviewer agent. Evaluate every applicable item against the target codebase.

---

## 1. Marketing & Landing Pages

### Homepage
- [ ] Clear value proposition above the fold
- [ ] Call-to-action (CTA) button is prominent and clear
- [ ] Hero section communicates what the product does in < 5 seconds
- [ ] Social proof exists (testimonials, logos, user count, case studies)
- [ ] Pricing information is accessible (page or link in nav)
- [ ] Product screenshots or demo show the actual product

**Grep patterns:**
```
hero|Hero|landing|Landing                          # hero/landing sections
cta|CTA|call.?to.?action|get.?started|sign.?up    # CTA elements
pricing|Pricing|plans|Plans                         # pricing page
testimonial|Testimonial|review|Review               # social proof
```

### Marketing Pages
- [ ] Features page exists explaining key capabilities
- [ ] About/team page exists (builds trust)
- [ ] Pricing page clearly shows plans and what's included
- [ ] FAQ section addresses common questions
- [ ] Blog or changelog exists (shows active development)
- [ ] Contact or support page is accessible

**Grep patterns:**
```
features|Features|about|About|pricing|Pricing       # marketing pages
faq|FAQ|frequently                                   # FAQ section
blog|Blog|changelog|Changelog                        # content pages
contact|Contact|support|Support                      # support pages
```

---

## 2. SEO

### Technical SEO
- [ ] Each page has a unique `<title>` tag
- [ ] Each page has a `<meta name="description">` tag
- [ ] Open Graph (OG) tags set for social sharing (`og:title`, `og:description`, `og:image`)
- [ ] Twitter card tags set (`twitter:card`, `twitter:title`, `twitter:image`)
- [ ] Canonical URLs set to prevent duplicate content
- [ ] `sitemap.xml` exists and is accessible
- [ ] `robots.txt` exists with appropriate rules

**Grep patterns:**
```
<title>|title:|metadata.*title                      # page titles
meta.*description|description:                      # meta descriptions
og:title|og:description|og:image|openGraph          # Open Graph tags
twitter:card|twitter:title                          # Twitter cards
canonical|rel=["']canonical                         # canonical URLs
sitemap|sitemap\.xml                                # sitemap
robots\.txt                                         # robots file
```

### Content SEO
- [ ] Pages have semantic HTML (`<h1>`, `<h2>`, `<article>`, `<section>`)
- [ ] URLs are clean and descriptive (not `/page/123`)
- [ ] Image alt text includes relevant keywords
- [ ] Internal linking between related pages
- [ ] Structured data / JSON-LD for rich snippets (if applicable)

**Grep patterns:**
```
application\/ld\+json|structured.?data|jsonld       # structured data
generateMetadata|Head>|next\/head                   # metadata generation
```

### SEO Files
- [ ] `sitemap.xml` or dynamic sitemap generation
- [ ] `robots.txt` allows search engine crawling
- [ ] `manifest.json` / `site.webmanifest` for PWA (if applicable)
- [ ] Favicon set (`favicon.ico`, apple-touch-icon)

**File patterns to check:**
```
public/sitemap.xml|app/sitemap.ts                   # sitemap files
public/robots.txt|app/robots.ts                     # robots files
public/favicon.ico|app/favicon.ico                  # favicon
public/manifest.json|public/site.webmanifest        # PWA manifest
app/opengraph-image|public/og-image                 # OG images
```

---

## 3. Analytics & Tracking

### Analytics Integration
- [ ] Web analytics installed (Google Analytics, Plausible, Fathom, PostHog, Mixpanel)
- [ ] Analytics respects user consent/privacy preferences
- [ ] Page views tracked across all pages
- [ ] Key events tracked (signup, login, core actions)

**Grep patterns:**
```
gtag|GA4|google.*analytics|G-[A-Z0-9]+            # Google Analytics
plausible|Plausible                                 # Plausible
fathom|Fathom                                       # Fathom
posthog|PostHog                                     # PostHog
mixpanel|Mixpanel                                   # Mixpanel
amplitude|Amplitude                                 # Amplitude
@vercel\/analytics                                  # Vercel Analytics
segment|analytics\.track|analytics\.identify         # Segment/generic tracking
```

### Conversion Tracking
- [ ] Signup conversion tracked
- [ ] Payment conversion tracked
- [ ] Key activation events tracked (first meaningful action)
- [ ] Funnel stages defined and measurable

### Error Tracking
- [ ] Client-side errors tracked (Sentry, LogRocket, etc.)
- [ ] Error tracking connected to user sessions
- [ ] Performance monitoring in place (Web Vitals tracking)

**Grep patterns:**
```
@vercel\/speed-insights|web-vitals|reportWebVitals  # Web Vitals tracking
```

---

## 4. Legal & Compliance

### Required Legal Pages
- [ ] Privacy Policy page exists and is linked from footer/signup
- [ ] Terms of Service page exists and is linked from footer/signup
- [ ] Cookie policy exists (if using cookies beyond essential)
- [ ] User agrees to ToS during signup (checkbox or implied)

**Grep patterns:**
```
privacy|Privacy.?Policy                             # privacy policy
terms|Terms.?of.?Service|tos|ToS                   # terms of service
cookie.*policy|Cookie.?Policy                       # cookie policy
agree.*terms|accept.*terms|consent                  # consent mechanisms
```

### GDPR / Privacy Signals
- [ ] Cookie consent banner exists (if tracking cookies used)
- [ ] Users can request data export
- [ ] Users can request account deletion
- [ ] Data processing purposes are disclosed
- [ ] No unnecessary data collection

**Grep patterns:**
```
cookie.*consent|CookieConsent|cookie.*banner        # cookie consent
delete.*account|account.*deletion                   # account deletion
export.*data|data.*export|download.*data            # data export
gdpr|GDPR                                          # GDPR references
```

### Compliance Signals
- [ ] CCPA compliance signals (if serving California users)
- [ ] Accessibility statement (recommended)
- [ ] DMCA policy (if user-generated content)
- [ ] Age restrictions (if applicable)

---

## 5. Payment & Billing

### Payment Integration
- [ ] Payment provider integrated (Stripe, Paddle, LemonSqueezy, etc.)
- [ ] Payment form is secure (uses provider's hosted elements, not raw card input)
- [ ] Subscription management exists (upgrade, downgrade, cancel)
- [ ] Payment receipts/invoices generated
- [ ] Free trial flow works (if offered)
- [ ] Pricing page matches actual plan configuration

**Grep patterns:**
```
stripe|Stripe|@stripe\/                             # Stripe
paddle|Paddle                                       # Paddle
lemon.?squeezy|LemonSqueezy                        # LemonSqueezy
CardElement|PaymentElement|Elements                  # Stripe Elements
checkout.*session|createCheckoutSession             # Checkout sessions
subscription|Subscription                            # Subscription management
customer.*portal|billingPortal                       # Billing portal
```

### Webhook Handling
- [ ] Payment webhooks configured and verified
- [ ] Webhook signature verification implemented
- [ ] Webhook idempotency handled (duplicate events)
- [ ] Failed payment handling (dunning, grace period)
- [ ] Subscription lifecycle events handled (created, updated, cancelled, past_due)

**Grep patterns:**
```
webhook|Webhook                                     # webhook endpoints
constructEvent|verifyHeader|svix                     # signature verification
checkout\.session\.completed|invoice\.paid           # Stripe events
customer\.subscription\.(created|updated|deleted)    # subscription events
```

### Billing Edge Cases
- [ ] Plan change proration handled
- [ ] Cancelled subscription continues until period end
- [ ] Failed payment retry logic exists
- [ ] Tax handling (if applicable)
- [ ] Refund capability exists

---

## 6. Transactional Email

### Email Templates
- [ ] Welcome email sent on signup
- [ ] Email verification sent (if required)
- [ ] Password reset email works
- [ ] Payment receipt email sent
- [ ] Subscription change confirmation email
- [ ] Emails have unsubscribe link (if non-transactional)

**Grep patterns:**
```
resend|Resend|@sendgrid|sendgrid                    # email providers
nodemailer|postmark|ses|mailgun                     # email providers
welcome.*email|send.*welcome                        # welcome email
verify.*email|email.*verify|confirmation            # verification email
reset.*password|password.*reset                     # password reset
receipt|invoice.*email                              # receipt emails
unsubscribe                                         # unsubscribe handling
```

### Email Quality
- [ ] Emails have both HTML and plain text versions
- [ ] Sender address uses custom domain (not @gmail.com)
- [ ] Reply-to address is monitored
- [ ] Email templates are responsive (mobile-friendly)
- [ ] SPF/DKIM/DMARC configured (check DNS setup documentation)

---

## 7. Social Proof & Trust

- [ ] Testimonials from real users (not placeholder)
- [ ] Company logos of customers (if B2B)
- [ ] User count or usage statistics
- [ ] Trust badges (security, compliance, awards)
- [ ] Case studies or success stories
- [ ] Public changelog showing active development

**Grep patterns:**
```
testimonial|Testimonial                             # testimonials
trust.*badge|security.*badge|ssl.*badge             # trust badges
customer.*logo|client.*logo                         # customer logos
case.?study|CaseStudy                               # case studies
```

---

## 8. Support & Help

### Support Channels
- [ ] Contact form or email prominently displayed
- [ ] Support documentation / help center exists
- [ ] FAQ addresses common questions
- [ ] Error pages include guidance (not just "Something went wrong")
- [ ] In-app help or tooltips for complex features

**Grep patterns:**
```
help|Help|support|Support                           # support pages
documentation|docs|Docs                             # documentation
intercom|crisp|drift|zendesk|freshdesk              # support tools
feedback|Feedback                                   # feedback mechanism
```

### Feedback Mechanisms
- [ ] Bug report or feedback form exists
- [ ] Feature request mechanism exists
- [ ] NPS or satisfaction survey planned
- [ ] Public roadmap (optional but valuable)

---

## Scoring Guide

- **CRITICAL**: No privacy policy or ToS, payment flow broken, no webhook signature verification, no email verification on accounts that handle money, missing unsubscribe links on marketing emails
- **WARNING**: No analytics tracking, missing OG tags, no cookie consent (when tracking), no support channel, incomplete SEO, no social proof, missing transactional emails
- **GOOD**: Legal pages present, SEO basics covered, analytics tracking key events, payment flow works end-to-end, transactional emails sent, support contact available
- **EXCELLENT**: Comprehensive SEO with structured data, conversion funnel tracked, A/B testing capability, complete transactional email suite, subscription lifecycle fully handled, public documentation, social proof throughout, cookie consent with granular controls
