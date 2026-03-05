# UX & Accessibility Reference Checklist

Detailed checklist for the UX & Accessibility Inspector agent. Evaluate every applicable item against the target codebase.

---

## 1. WCAG 2.1 AA Compliance

### Images & Media
- [ ] All `<img>` elements have meaningful `alt` text (not empty string unless decorative)
- [ ] Decorative images use `alt=""` or `role="presentation"`
- [ ] Icon-only buttons have `aria-label` or visually hidden text
- [ ] Videos have captions/transcripts (if applicable)
- [ ] SVG icons have `aria-hidden="true"` when decorative, or `role="img"` with title when meaningful

**Grep patterns:**
```
<img(?![^>]*alt=)                              # img without alt attribute
alt=["']["']                                    # empty alt (check if decorative)
<svg(?![^>]*aria)                               # SVG without ARIA
<button[^>]*>[^<]*<svg|<button[^>]*>[^<]*<Icon  # icon-only buttons
aria-label|aria-labelledby                       # ARIA labels
```

### Headings & Structure
- [ ] Pages have exactly one `<h1>`
- [ ] Heading hierarchy is sequential (no skipping h2 → h4)
- [ ] Sections have appropriate landmark roles (`<main>`, `<nav>`, `<aside>`, `<footer>`)
- [ ] Page has a `<title>` that describes the current page

**Grep patterns:**
```
<h1|<h2|<h3|<h4|<h5|<h6                        # heading usage
<main|<nav|<aside|<footer|<header|<section      # landmark elements
role=["']main|role=["']navigation                # ARIA landmark roles
<title>|Head>.*<title                            # page titles
```

### Color & Contrast
- [ ] Text meets 4.5:1 contrast ratio (normal text) or 3:1 (large text)
- [ ] Information is not conveyed by color alone (errors, status, etc.)
- [ ] Focus indicators are visible (not hidden with `outline: none` without replacement)
- [ ] Links are distinguishable from surrounding text (not just color)

**Grep patterns:**
```
outline:\s*none|outline:\s*0                     # removed focus outline (WARNING)
:focus\s*\{|focus-visible                        # focus styling
text-red|text-green|text-yellow                  # color-only indicators (check for icon/text backup)
```

### Forms & Labels
- [ ] All form inputs have associated `<label>` elements (via `htmlFor`/`for` or wrapping)
- [ ] Required fields are indicated (not just by color)
- [ ] Error messages are associated with inputs via `aria-describedby`
- [ ] Form groups use `<fieldset>` and `<legend>` where appropriate
- [ ] Autocomplete attributes used on common fields (name, email, address, etc.)

**Grep patterns:**
```
<input(?![^>]*(id=.*<label|aria-label))          # input without label association
htmlFor=|for=                                     # label associations
aria-describedby|aria-errormessage                # error message association
<fieldset|<legend                                 # form grouping
autoComplete=|autocomplete=                       # autocomplete attributes
aria-required|required                            # required indication
```

---

## 2. Keyboard Navigation

### Focus Management
- [ ] All interactive elements are keyboard-accessible (focusable via Tab)
- [ ] Custom components (dropdowns, modals, tabs) support keyboard interaction
- [ ] Focus order follows visual layout (no confusing tab jumps)
- [ ] Focus is trapped in modals/dialogs (can't tab to background)
- [ ] Focus returns to trigger element when modal/dialog closes
- [ ] Skip-to-content link exists for page header bypass

**Grep patterns:**
```
tabIndex|tabindex                                 # custom tab order
onKeyDown|onKeyUp|onKeyPress                      # keyboard event handling
focus\(\)|\.focus\(|autoFocus|autofocus           # focus management
FocusTrap|focus-trap|createFocusTrap              # focus trapping
role=["']dialog|<dialog|Modal|modal               # dialog/modal components
skip.*content|skip.*nav|skip.*main                # skip links
```

### Keyboard Interactions
- [ ] Buttons activate on Enter and Space
- [ ] Links activate on Enter
- [ ] Dropdown menus support arrow keys
- [ ] Tab components support arrow key navigation
- [ ] Escape closes modals, dropdowns, and popups
- [ ] Custom controls announce state changes

**Grep patterns:**
```
Escape|Esc|key.*escape                            # Escape key handling
ArrowDown|ArrowUp|ArrowLeft|ArrowRight            # Arrow key handling
role=["']tab|role=["']tabpanel                    # tab components
role=["']menu|role=["']menuitem                   # menu components
role=["']listbox|role=["']option                  # listbox components
```

---

## 3. Screen Reader Support

### ARIA Usage
- [ ] Dynamic content updates use `aria-live` regions
- [ ] Loading states announced to screen readers
- [ ] Error messages announced to screen readers (`aria-live="assertive"` or `role="alert"`)
- [ ] Toggle buttons use `aria-pressed` or `aria-expanded`
- [ ] Custom widgets have appropriate ARIA roles
- [ ] `aria-hidden="true"` used on decorative/duplicate content

**Grep patterns:**
```
aria-live|aria-atomic                             # live regions
role=["']alert|role=["']status                    # status announcements
aria-pressed|aria-expanded|aria-selected          # state attributes
aria-hidden                                       # hidden from screen readers
role=["']button|role=["']link                     # semantic role overrides
sr-only|visually-hidden|VisuallyHidden            # screen-reader-only text
```

### Announcements
- [ ] Route changes announced (SPA navigation)
- [ ] Toast/notification messages announced
- [ ] Form submission results announced
- [ ] Data table changes announced

---

## 4. Responsive Design

### Viewport & Layout
- [ ] Viewport meta tag set correctly: `<meta name="viewport" content="width=device-width, initial-scale=1">`
- [ ] Content is readable at 320px width without horizontal scroll
- [ ] No fixed-width containers that overflow on small screens
- [ ] Text can be resized to 200% without loss of content
- [ ] Layout adapts to portrait and landscape orientations

**Grep patterns:**
```
viewport|initial-scale                            # viewport meta
max-width|min-width                               # responsive constraints
@media|useMediaQuery|useBreakpoint                # responsive breakpoints
overflow-x:\s*hidden|overflow:\s*hidden           # hidden overflow (check for content loss)
w-full|w-screen|max-w-                            # Tailwind responsive
```

### Breakpoint Coverage
- [ ] Mobile layout (< 640px)
- [ ] Tablet layout (640px - 1024px)
- [ ] Desktop layout (> 1024px)
- [ ] Large screen handling (> 1440px) — content doesn't stretch excessively

**Grep patterns:**
```
sm:|md:|lg:|xl:|2xl:                              # Tailwind breakpoints
@media.*640|@media.*768|@media.*1024              # CSS breakpoints
container|max-w-screen                            # container constraints
```

---

## 5. Mobile Readiness

### Touch Interactions
- [ ] Touch targets are at least 44x44px (WCAG) or 48x48px (Material Design)
- [ ] Adequate spacing between touch targets (no accidental taps)
- [ ] No hover-only interactions (tooltips, dropdown menus)
- [ ] Swipe gestures have alternative actions
- [ ] Long-press actions have alternative access

**Grep patterns:**
```
onMouseEnter|onMouseOver|:hover                   # hover-only interactions (check for touch alternative)
min-h-\[44px\]|min-w-\[44px\]|h-11|w-11         # touch target sizing
p-2|p-3|gap-2|gap-3|space-                        # spacing between targets
```

### Mobile Navigation
- [ ] Mobile menu/hamburger exists for small screens
- [ ] Navigation is reachable with one hand (bottom or collapsible)
- [ ] Fixed/sticky headers don't consume too much screen space
- [ ] Forms are usable on mobile (appropriate input types, no tiny inputs)

**Grep patterns:**
```
hamburger|mobile.*menu|menu.*mobile               # mobile menu
type=["']email|type=["']tel|type=["']number       # mobile input types
inputMode|inputmode                                # mobile keyboard hints
sticky|fixed.*top                                  # fixed headers
```

### Touch-Specific
- [ ] No 300ms tap delay (modern browsers handle this, but check for `touch-action: manipulation`)
- [ ] Pinch-to-zoom not disabled (`user-scalable=no` is an accessibility violation)
- [ ] Pull-to-refresh handled if applicable

**Grep patterns:**
```
user-scalable\s*=\s*no|maximum-scale\s*=\s*1      # zoom disabled (CRITICAL a11y issue)
touch-action                                        # touch behavior control
```

---

## 6. Form UX

### Validation & Feedback
- [ ] Inline validation on blur (not just on submit)
- [ ] Error messages are specific (not just "Invalid input")
- [ ] Error messages appear near the relevant field
- [ ] Success states are clearly communicated
- [ ] Form doesn't clear on validation error

### Input Enhancement
- [ ] Appropriate HTML5 input types (`email`, `tel`, `url`, `number`, `date`)
- [ ] Autocomplete attributes on common fields
- [ ] Placeholder text is supplementary (not replacing labels)
- [ ] Password fields have show/hide toggle
- [ ] File inputs show selected file name and allow removal
- [ ] Date pickers are accessible and mobile-friendly

**Grep patterns:**
```
type=["']text["'].*email|type=["']text["'].*phone  # wrong input types
placeholder=(?!.*<label|.*aria-label)               # placeholder without label
onBlur.*valid|onBlur.*error                         # blur validation
showPassword|togglePassword|eye.*icon               # password visibility toggle
```

### Multi-Step Forms
- [ ] Progress indicator shows current step
- [ ] Back button preserves entered data
- [ ] Step validation before proceeding
- [ ] Summary/review step before final submission

---

## 7. Content & Typography

- [ ] Body text is at least 16px (1rem) on mobile
- [ ] Line height is at least 1.5 for body text
- [ ] Paragraph max-width is ~65-75 characters for readability
- [ ] Text content has sufficient whitespace/padding
- [ ] No text truncation that hides important information without expansion

**Grep patterns:**
```
text-sm|text-xs|font-size:\s*(0\.\d|1[0-3]px)    # potentially small text
leading-|line-height                               # line height
max-w-prose|max-w-\[.*ch\]                         # reading width constraints
truncate|text-ellipsis|overflow.*hidden.*text       # text truncation
```

---

## 8. Animation & Motion

- [ ] Animations respect `prefers-reduced-motion` media query
- [ ] No content that flashes more than 3 times per second
- [ ] Loading animations don't block content access
- [ ] Transitions are smooth (no janky/stuttering animations)

**Grep patterns:**
```
prefers-reduced-motion                             # motion preference
motion-reduce|motion-safe                          # Tailwind motion utilities
@keyframes|animation:|transition:                  # animations
animate-|transition-                               # Tailwind animations
```

---

## Scoring Guide

- **CRITICAL**: Missing alt text on functional images, zoom disabled (`user-scalable=no`), no keyboard access to critical functions, focus outline removed without replacement, form inputs without labels
- **WARNING**: Missing skip link, incomplete heading hierarchy, hover-only interactions, no `prefers-reduced-motion` support, small touch targets, missing ARIA on custom widgets
- **GOOD**: Alt text present, keyboard navigation works, responsive layout, form labels exist, basic ARIA usage, focus management in modals
- **EXCELLENT**: Comprehensive ARIA, skip links, focus trapping, `prefers-reduced-motion`, all touch targets sized correctly, inline form validation, live region announcements
