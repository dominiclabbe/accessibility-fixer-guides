# Web Accessibility Audit Guide
## HTML, React, Vue, Angular, and Web Technologies

---

## Overview

This guide covers accessibility auditing for web applications using:
- HTML5 semantic markup
- ARIA attributes and roles
- CSS (color, focus states, responsive design)
- JavaScript frameworks (React, Vue, Angular, Svelte, etc.)

**Target:** WCAG 2.2 Level AA compliance

> 📖 **For detailed ARIA patterns:** See [ARIA_PATTERNS_REFERENCE.md](ARIA_PATTERNS_REFERENCE.md) for comprehensive guidance on all W3C APG patterns including tabs, accordions, dialogs, comboboxes, menu buttons, and more.

---

## Technology Focus

### Core Web Technologies
- **HTML:** Semantic elements, forms, media elements
- **ARIA:** Roles, states, properties, and live regions
- **CSS:** Color contrast, focus indicators, responsive design
- **JavaScript:** Dynamic updates, keyboard handling, focus management

### Common Frameworks
- React / React with TypeScript
- Vue.js
- Angular
- Svelte
- Next.js / Nuxt.js
- Plain HTML/CSS/JS

---

## ⚠️ CRITICAL: Avoid Redundant Information in Accessible Labels

> 📖 **See comprehensive pattern guide:** [Avoid Redundant Information](patterns/AVOID_ROLE_IN_LABEL.md)

**THE MOST COMMON ACCESSIBILITY MISTAKE:** Adding information to aria-label/alt text that elements already announce automatically.

### Four Types of Redundancy to NEVER Include:

1. **❌ Role Redundancy** - "Submit button", "Home tab", "Search link"
   - Elements announce their role automatically
   - ❌ WRONG: `<button aria-label="Close button">` → Screen reader: "Close button, button"
   - ✅ CORRECT: `<button>Close</button>` → Screen reader: "Close, button"

2. **❌ State Redundancy** - Adding state to aria-label when element announces it
   - Tabs, checkboxes, radio buttons announce state automatically
   - ❌ WRONG: `<button role="tab" aria-selected="true" aria-label="Home selected">` → "Home selected, tab, selected"
   - ✅ CORRECT: `<button role="tab" aria-selected="true">Home</button>` → "Home, tab, selected"
   - ⚠️ EXCEPTION: Custom widgets may need aria-valuetext for value formatting

3. **❌ Position Redundancy** - Adding "tab 1 of 4" or similar to aria-label
   - ARIA tabs announce position automatically with aria-posinset and aria-setsize
   - ❌ WRONG: `<button role="tab" aria-label="Home tab 1 of 4">` → "Home tab 1 of 4, tab, 1 of 4"
   - ✅ CORRECT: Use aria-posinset and aria-setsize attributes, let screen reader announce position

4. **❌ Property/Context Redundancy** - Duplicating visible text or obvious context
   - ❌ WRONG: `<input type="text" aria-label="Email address text field">`
   - ✅ CORRECT: `<label for="email">Email address</label><input type="text" id="email">`
   - ❌ WRONG: `<button aria-label="Close modal">×</button>` when modal context is clear
   - ✅ CORRECT: `<button aria-label="Close">×</button>`

### Quick Rules:
- **Use aria-selected, aria-checked** for state, NOT text in aria-label
- **NEVER add state to aria-label** when element announces it automatically
- **Use aria-posinset/aria-setsize** for position, NOT text in aria-label
- **Don't duplicate visible text** in aria-label
- **Let semantic HTML handle** role announcements (button, link, heading, etc.)
- **Only override with ARIA** when semantic HTML is insufficient

---

## Key Code Patterns to Check

### 1. Semantic HTML

**Issue:** Non-semantic markup for interactive elements

```javascript
// ❌ ISSUE: Non-semantic markup
<div onclick="handleClick()">Click me</div>
<span class="link" onclick="navigate()">Go to page</span>

// ✅ CORRECT: Semantic button
<button onClick={handleClick}>Click me</button>

// ✅ CORRECT: Semantic link
<a href="/page">Go to page</a>
```

**WCAG SC:** 4.1.2 Name, Role, Value (Level A)

**Why:** Screen readers need semantic HTML to communicate the role and purpose of elements.

---

### 2. Alternative Text for Images - IMPORTANT: Check Collection Context First

**⚠️ CRITICAL:** Before flagging missing alt text, determine if the element is part of a collection item (card in a grid, list item, carousel slide).

**Decision Tree:**
1. **Is this image part of a collection item (card/tile in a list/grid)?**
   - YES → Skip to Section 11 (Collection Items) - use aria-label on parent link/button, mark children as aria-hidden
   - NO → Continue to step 2

2. **Is this image clearly decorative?** (gradient, background, border, divider, spacer)
   - YES → Skip reporting, don't include in audit (or ensure it has alt="")
   - UNSURE → Create LOW priority issue, note "fix only if not decorative"
   - NO (conveys information) → Continue to step 3

3. **Is this the ONLY content in a button/link?**
   - YES → CRITICAL issue, MUST have alt text or aria-label
   - NO → Apply standard alt text rules below

4. **Is this image standalone (not in a collection)?**
   - YES → Require alt text as shown below

**Issue:** Missing or inappropriate alt text on **standalone** images

```javascript
// ❌ ISSUE: Standalone image missing alt text
<img src="logo.png" />

// ❌ ISSUE: Redundant alt text
<img src="submit.png" alt="Submit button image" />

// ✅ CORRECT: Descriptive alt text (standalone)
<img src="logo.png" alt="Company name logo" />

// ✅ CORRECT: Decorative image
<img src="decorative.png" alt="" role="presentation" />
<img src="decorative.png" alt="" aria-hidden="true" />

// ✅ CORRECT: Complex image with longer description
<img
  src="chart.png"
  alt="Sales chart"
  aria-describedby="chart-description"
/>
<div id="chart-description">
  Sales increased by 25% in Q2, with the highest growth in June.
</div>
```

**❌ DO NOT flag for missing alt text:**
- Images inside collection items (cards, list items, grid cells)
- Sub-elements of components that will use comprehensive aria-label
- Elements that are part of a larger interactive unit

**✅ DO flag for missing alt text:**
- Images that are the ONLY content in buttons/links (CRITICAL priority)
- Standalone interactive icons not part of collections
- Hero images conveying information
- Logo images with brand identity importance
- Content images in articles
- Icons in navigation bars
- Informational graphics, charts, diagrams

**🎨 Clearly Decorative - DON'T Report:**
- CSS background images
- Gradient overlays
- Decorative borders or dividers
- Pattern backgrounds
- Spacer/separator images
- Shadow/glow effects
- **Note:** These should have `alt=""` or `role="presentation"`, but don't report if already present

**❓ Unsure if Decorative - LOW Priority:**
- Images that might convey branding but information is elsewhere
- Decorative icons when text label is present
- Background images that might convey mood/context
- **Recommendation text:** "Verify if this image is decorative. If so, use `alt=\"\"` and `aria-hidden=\"true\"`. If it conveys information, add descriptive alt text."

**WCAG SC:** 1.1.1 Non-text Content (Level A), 4.1.2 Name, Role, Value (Level A)

**React Example:**
```jsx
// ❌ ISSUE
<Image src="/logo.png" />

// ✅ CORRECT
<Image src="/logo.png" alt="Company name logo" />

// ✅ CORRECT: Decorative in React
<Image src="/decorative.png" alt="" aria-hidden="true" />
```

---

### 3. Form Labels and Instructions

**Issue:** Inputs without proper labels

```html
<!-- ❌ ISSUE: Placeholder used as only label -->
<input type="text" placeholder="Enter name" />

<!-- ❌ ISSUE: Label not associated -->
<label>Email</label>
<input type="email" />

<!-- ✅ CORRECT: Properly labeled with htmlFor/id -->
<label htmlFor="name">Name</label>
<input type="text" id="name" />

<!-- ✅ CORRECT: Implicit label association -->
<label>
  Email
  <input type="email" />
</label>

<!-- ✅ CORRECT: Using aria-label when visual label isn't possible -->
<input
  type="search"
  aria-label="Search products"
  placeholder="Search..."
/>

<!-- ✅ CORRECT: Required field with clear indication -->
<label htmlFor="email">
  Email <span aria-label="required">*</span>
</label>
<input
  type="email"
  id="email"
  required
  aria-required="true"
/>
```

**WCAG SC:** 3.3.2 Labels or Instructions (Level A), 1.3.1 Info and Relationships (Level A)

**React Example:**
```jsx
// ❌ ISSUE
<input placeholder="Enter email" value={email} onChange={handleChange} />

// ✅ CORRECT
<>
  <label htmlFor="email">Email</label>
  <input
    id="email"
    type="email"
    value={email}
    onChange={handleChange}
  />
</>
```

---

### 4. Keyboard Navigation and Focus Management

**Issue:** Interactive elements not keyboard accessible

```javascript
// ❌ ISSUE: Clickable div without keyboard support
<div onClick={handleClick}>Custom button</div>

// ✅ CORRECT: Keyboard accessible custom button
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      handleClick();
    }
  }}
>
  Custom button
</div>

// ✅ BETTER: Use semantic button
<button onClick={handleClick}>Custom button</button>
```

**WCAG SC:** 2.1.1 Keyboard (Level A), 4.1.2 Name, Role, Value (Level A)

**Modal Focus Management:**
```javascript
// ✅ CORRECT: Trap focus in modal and restore on close
function Modal({ isOpen, onClose, children }) {
  const modalRef = useRef();
  const previousFocus = useRef();

  useEffect(() => {
    if (isOpen) {
      // Save currently focused element
      previousFocus.current = document.activeElement;

      // Focus first focusable element in modal
      const firstFocusable = modalRef.current.querySelector(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      firstFocusable?.focus();

      // Trap focus within modal
      const handleTab = (e) => {
        const focusableElements = modalRef.current.querySelectorAll(
          'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );
        const first = focusableElements[0];
        const last = focusableElements[focusableElements.length - 1];

        if (e.key === 'Tab') {
          if (e.shiftKey && document.activeElement === first) {
            e.preventDefault();
            last.focus();
          } else if (!e.shiftKey && document.activeElement === last) {
            e.preventDefault();
            first.focus();
          }
        }
      };

      document.addEventListener('keydown', handleTab);
      return () => document.removeEventListener('keydown', handleTab);
    } else {
      // Restore focus when modal closes
      previousFocus.current?.focus();
    }
  }, [isOpen]);

  return isOpen ? (
    <div
      role="dialog"
      aria-modal="true"
      ref={modalRef}
    >
      {children}
    </div>
  ) : null;
}
```

**WCAG SC:** 2.4.3 Focus Order (Level A), 2.1.2 No Keyboard Trap (Level A)

---

### 5. Focus Visible Indicators

**Issue:** Missing or insufficient focus indicators

```css
/* ❌ ISSUE: Focus outline removed without replacement */
button:focus {
  outline: none;
}

/* ✅ CORRECT: Custom focus indicator */
button:focus-visible {
  outline: 3px solid #005fcc;
  outline-offset: 2px;
}

/* ✅ CORRECT: Maintaining default with enhancement */
button:focus-visible {
  outline: 2px solid currentColor;
  outline-offset: 2px;
}

/* Modern approach using :focus-visible */
a:focus-visible {
  outline: 2px solid #005fcc;
  border-radius: 3px;
}
```

**WCAG SC:** 2.4.7 Focus Visible (Level AA)

---

### 6. Color Contrast

**Issue:** Insufficient color contrast

```css
/* ❌ ISSUE: Insufficient contrast (2.5:1) */
.text {
  color: #999999;
  background: #ffffff;
}

/* ✅ CORRECT: Minimum 4.5:1 for normal text */
.text {
  color: #767676; /* 4.54:1 ratio */
  background: #ffffff;
}

/* ✅ CORRECT: 3:1 minimum for large text (18pt+) */
.heading {
  color: #949494; /* 3.03:1 ratio */
  background: #ffffff;
  font-size: 24px;
  font-weight: bold;
}

/* ✅ CORRECT: Using CSS variables for consistent contrast */
:root {
  --text-primary: #212121;   /* 16:1 on white */
  --text-secondary: #666666; /* 5.74:1 on white */
  --bg-primary: #ffffff;
}
```

**WCAG SC:** 1.4.3 Contrast (Minimum) (Level AA), 1.4.11 Non-text Contrast (Level AA)

**Requirements:**
- Normal text: 4.5:1 minimum
- Large text (18pt+ or 14pt+ bold): 3:1 minimum
- UI components and icons: 3:1 minimum

---

### 7. Dynamic Content and ARIA Live Regions

**Issue:** Screen readers not notified of updates

```javascript
// ❌ ISSUE: Content update without announcement
function StatusMessage({ message }) {
  return <div>{message}</div>;
}

// ✅ CORRECT: ARIA live region for polite announcements
function StatusMessage({ message }) {
  return (
    <div role="status" aria-live="polite" aria-atomic="true">
      {message}
    </div>
  );
}

// ✅ CORRECT: Assertive for urgent messages
function ErrorMessage({ error }) {
  return (
    <div role="alert" aria-live="assertive" aria-atomic="true">
      {error}
    </div>
  );
}

// ✅ CORRECT: Loading state
function LoadingSpinner({ isLoading }) {
  return isLoading ? (
    <div role="status" aria-live="polite">
      <span className="visually-hidden">Loading, please wait...</span>
      <Spinner aria-hidden="true" />
    </div>
  ) : null;
}
```

**WCAG SC:** 4.1.3 Status Messages (Level AA)

**CSS for visually-hidden content:**
```css
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  margin: -1px;
  padding: 0;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
```

---

### 8. ARIA Roles and Landmarks

**Issue:** Poor semantic structure or missing landmarks

```html
<!-- ❌ ISSUE: No semantic structure -->
<div class="header">
  <div class="nav">...</div>
</div>
<div class="content">...</div>
<div class="footer">...</div>

<!-- ✅ CORRECT: Semantic HTML5 -->
<header>
  <nav aria-label="Main navigation">...</nav>
</header>
<main>
  <h1>Page Title</h1>
  ...
</main>
<footer>...</footer>

<!-- ✅ CORRECT: Multiple navigation with labels -->
<nav aria-label="Main navigation">...</nav>
<nav aria-label="Breadcrumb" aria-label="Breadcrumb">...</nav>
<nav aria-label="Footer navigation">...</nav>

<!-- ✅ CORRECT: Regions with labels -->
<section aria-labelledby="section-heading">
  <h2 id="section-heading">Related Articles</h2>
  ...
</section>
```

**WCAG SC:** 1.3.1 Info and Relationships (Level A), 2.4.1 Bypass Blocks (Level A)

---

### 9. Headings Hierarchy

**Issue:** Improper heading structure

```html
<!-- ❌ ISSUE: Skipped heading levels -->
<h1>Main Title</h1>
<h3>Section Title</h3> <!-- Skipped h2 -->

<!-- ❌ ISSUE: Visual styling used instead of headings -->
<div class="big-text">Section Title</div>

<!-- ✅ CORRECT: Proper hierarchy -->
<h1>Page Title</h1>
<h2>Section 1</h2>
<h3>Subsection 1.1</h3>
<h3>Subsection 1.2</h3>
<h2>Section 2</h2>

<!-- ✅ CORRECT: Style headings with CSS, not by changing level -->
<h2 class="h3-style">Section Title</h2> <!-- Semantically h2, visually h3 -->
```

**WCAG SC:** 1.3.1 Info and Relationships (Level A), 2.4.6 Headings and Labels (Level AA)

---

### 10. Skip Links

**Issue:** Missing skip navigation links

```html
<!-- ✅ CORRECT: Skip link at top of page -->
<body>
  <a href="#main-content" class="skip-link">
    Skip to main content
  </a>
  <header>
    <nav>...</nav>
  </header>
  <main id="main-content" tabindex="-1">
    ...
  </main>
</body>
```

```css
/* Skip link visible only when focused */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
  text-decoration: none;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
```

**WCAG SC:** 2.4.1 Bypass Blocks (Level A)

---

### 11. Collection/Grid Items

**Issue:** Collection items with multiple sub-elements require multiple tab stops to navigate

**⚠️ CRITICAL:** Items in grids, carousels, or lists should be treated as **a whole unit**. Users should NOT have to tab through multiple sub-elements to understand a single item.

**Example:** TV show card, movie card, product card - all information should be accessible in one interaction.

**🚫 IMPORTANT - Avoid Contradictory Guidance:**
When auditing collection items, **DO NOT** flag child elements (images, text, links within the card) for missing individual `alt` text or `aria-label`. Instead:
1. ✅ Flag the entire collection item for lacking comprehensive accessible name
2. ✅ Recommend setting `aria-label` on the parent `<a>` or `<button>`
3. ✅ Recommend marking ALL child elements with `alt=""` and `aria-hidden="true"`
4. ❌ Do NOT separately flag child elements for missing alt/aria-label

**Why:** If you recommend using aria-label on the parent (which you should), individual child alt texts become useless and create contradictory guidance.

```javascript
// ❌ ISSUE: User must tab through 5+ elements per card
// Tab stops: Image → Title link → Season text → Episode text → Channel → Progress
function ShowCard({ show }) {
  return (
    <div className="show-card">
      <img src={show.posterUrl} alt={show.title} />
      <a href={`/shows/${show.id}`}>{show.title}</a>
      <p>Season {show.season}</p>
      <p>Episode {show.episode}</p>
      <img src={show.channelLogo} alt={show.channelName} />
      <progress value={show.watchProgress} max="1" />
    </div>
  );
}

// ✅ CORRECT: Single tab stop with complete information
function ShowCard({ show }) {
  const watchedPercent = Math.round(show.watchProgress * 100);

  return (
    <a
      href={`/shows/${show.id}`}
      className="show-card"
      aria-label={`${show.title}, ${show.episodeName}, Season ${show.season}, Episode ${show.episode}, on ${show.channelName}, ${watchedPercent} percent watched`}
    >
      <img src={show.posterUrl} alt="" aria-hidden="true" />
      <div aria-hidden="true">
        <h3>{show.title}</h3>
        <p>S{show.season} E{show.episode}</p>
        <img src={show.channelLogo} alt="" />
        <progress value={show.watchProgress} max="1">
          {watchedPercent}% watched
        </progress>
      </div>
    </a>
  );
}

// ✅ CORRECT: Alternative with button and comprehensive visible text
function ShowCard({ show, onClick }) {
  const watchedPercent = Math.round(show.watchProgress * 100);

  return (
    <article className="show-card">
      <button
        onClick={onClick}
        aria-label={`${show.title}, ${show.episodeName}, Season ${show.season}, Episode ${show.episode}, on ${show.channelName}, ${watchedPercent} percent watched. Press to open.`}
      >
        <img src={show.posterUrl} alt="" />
        <div className="show-info">
          <h3>{show.title}</h3>
          <span className="visually-hidden">{show.episodeName}, </span>
          <p>Season {show.season}, Episode {show.episode}</p>
          <span className="visually-hidden">on </span>
          <img src={show.channelLogo} alt={show.channelName} />
          <div className="progress-container">
            <span className="visually-hidden">{watchedPercent} percent watched</span>
            <progress value={show.watchProgress} max="1" aria-hidden="true">
              {watchedPercent}%
            </progress>
          </div>
        </div>
      </button>
    </article>
  );
}

// ✅ CORRECT: Using proper ARIA with list structure
function ShowGrid({ shows }) {
  return (
    <ul role="list" className="show-grid">
      {shows.map((show) => (
        <li key={show.id}>
          <a
            href={`/shows/${show.id}`}
            aria-label={getShowAccessibilityLabel(show)}
          >
            <img src={show.posterUrl} alt="" aria-hidden="true" />
            <div aria-hidden="true">
              <h3>{show.title}</h3>
              <p>S{show.season} E{show.episode}</p>
              <p>{show.channelName}</p>
            </div>
          </a>
        </li>
      ))}
    </ul>
  );
}

function getShowAccessibilityLabel(show) {
  const watchedPercent = Math.round(show.watchProgress * 100);
  return `${show.title}, ${show.episodeName}, Season ${show.season}, Episode ${show.episode}, on ${show.channelName}, ${watchedPercent} percent watched`;
}
```

**Action Buttons on Collection Items:**

If items have action buttons (delete, favorite, share), keep them focusable but ensure they provide context:

```javascript
// ✅ CORRECT: Actions with context
function ShowCard({ show, onDelete, onFavorite }) {
  return (
    <article className="show-card">
      <a href={`/shows/${show.id}`} aria-label={getShowLabel(show)}>
        <img src={show.posterUrl} alt="" />
        <h3>{show.title}</h3>
        <p>S{show.season} E{show.episode}</p>
      </a>
      <div className="card-actions">
        <button
          onClick={onFavorite}
          aria-label={`Add ${show.title} to favorites`}
        >
          <HeartIcon aria-hidden="true" />
        </button>
        <button
          onClick={onDelete}
          aria-label={`Remove ${show.title} from watchlist`}
        >
          <TrashIcon aria-hidden="true" />
        </button>
      </div>
    </article>
  );
}

// ✅ CORRECT: Alternative with action menu
function ShowCard({ show }) {
  const [menuOpen, setMenuOpen] = useState(false);

  return (
    <article className="show-card">
      <a href={`/shows/${show.id}`} aria-label={getShowLabel(show)}>
        {/* Card content */}
      </a>
      <button
        aria-label={`Actions for ${show.title}`}
        aria-expanded={menuOpen}
        aria-haspopup="menu"
        onClick={() => setMenuOpen(!menuOpen)}
      >
        <MoreIcon aria-hidden="true" />
      </button>
      {menuOpen && (
        <ul role="menu">
          <li role="none">
            <button role="menuitem" onClick={onFavorite}>
              Add to favorites
            </button>
          </li>
          <li role="none">
            <button role="menuitem" onClick={onRemove}>
              Remove from watchlist
            </button>
          </li>
        </ul>
      )}
    </article>
  );
}
```

**Key Considerations for Web Collections:**
- Use semantic HTML: `<a>` for navigation, `<button>` for actions
- Provide comprehensive `aria-label` on the interactive container
- Mark decorative child images with `alt=""` and `aria-hidden="true"`
- Use `visually-hidden` class to include information for screen readers only
- Ensure keyboard navigation flows logically through the collection
- Consider using `<article>` for semantic structure
- Action buttons should include item context in their label

**WCAG SC:** 1.3.1 Info and Relationships (Level A), 2.4.4 Link Purpose (In Context) (Level A), 2.1.1 Keyboard (Level A)

**Key Points:**
- **One tab stop = one complete item** (or item + actions)
- Include ALL relevant information in aria-label or visible text
- Mark decorative elements with `aria-hidden="true"`
- Action buttons must include item context ("Add [Title] to favorites")
- Test with keyboard navigation and screen readers

---

## 12. Common ARIA Patterns (W3C APG)

This section covers the most common interactive ARIA patterns based on the W3C ARIA Authoring Practices Guide (APG). Each pattern includes required roles, attributes, keyboard interactions, and implementation examples.

### 12.1 Tabs Pattern

**Issue:** Improperly implemented tabs missing required ARIA attributes

The tabs pattern presents layered sections of content (tab panels), displaying one panel at a time. Each tab panel connects to an associated tab element that activates its display.

**Required ARIA Roles & Attributes:**

```javascript
// ❌ ISSUE: Missing ARIA roles and attributes
<div className="tabs">
  <div className="tab-buttons">
    <button onClick={() => setTab(0)}>Tab 1</button>
    <button onClick={() => setTab(1)}>Tab 2</button>
  </div>
  <div>{activeTab === 0 && <div>Panel 1</div>}</div>
  <div>{activeTab === 1 && <div>Panel 2</div>}</div>
</div>

// ✅ CORRECT: Complete tabs implementation
function Tabs({ tabs }) {
  const [activeTab, setActiveTab] = useState(0);
  const tablistRef = useRef();

  const handleKeyDown = (e, index) => {
    const tabCount = tabs.length;
    let newIndex = index;

    switch (e.key) {
      case 'ArrowRight':
        e.preventDefault();
        newIndex = (index + 1) % tabCount; // Wrap to first
        break;
      case 'ArrowLeft':
        e.preventDefault();
        newIndex = (index - 1 + tabCount) % tabCount; // Wrap to last
        break;
      case 'Home':
        e.preventDefault();
        newIndex = 0;
        break;
      case 'End':
        e.preventDefault();
        newIndex = tabCount - 1;
        break;
      default:
        return;
    }

    setActiveTab(newIndex);
    // Focus the newly activated tab
    const newTab = tablistRef.current.children[newIndex];
    newTab?.focus();
  };

  return (
    <div>
      <div
        role="tablist"
        aria-label="Content sections"
        ref={tablistRef}
      >
        {tabs.map((tab, index) => (
          <button
            key={index}
            role="tab"
            id={`tab-${index}`}
            aria-selected={activeTab === index}
            aria-controls={`panel-${index}`}
            tabIndex={activeTab === index ? 0 : -1}
            onClick={() => setActiveTab(index)}
            onKeyDown={(e) => handleKeyDown(e, index)}
          >
            {tab.label}
          </button>
        ))}
      </div>

      {tabs.map((tab, index) => (
        <div
          key={index}
          role="tabpanel"
          id={`panel-${index}`}
          aria-labelledby={`tab-${index}`}
          hidden={activeTab !== index}
          tabIndex={0}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

**Keyboard Interactions:**
- **Tab**: Moves focus into tablist (activates current tab), then exits to next page element
- **Arrow Right/Down**: Moves focus to next tab (wraps to first)
- **Arrow Left/Up**: Moves focus to previous tab (wraps to last)
- **Home**: Moves focus to first tab
- **End**: Moves focus to last tab

**Key Requirements:**
- Only the active tab should have `tabIndex={0}`
- Inactive tabs should have `tabIndex={-1}`
- Arrow keys should navigate between tabs and wrap around
- Each tab must have unique `id` and `aria-controls` pointing to its panel
- Each panel must have `aria-labelledby` pointing to its tab

**WCAG SC:** 1.3.1 Info and Relationships (Level A), 4.1.2 Name, Role, Value (Level A), 2.1.1 Keyboard (Level A)

---

### 12.2 Accordion Pattern

**Issue:** Accordion missing proper ARIA structure and keyboard support

An accordion is a vertically stacked set of interactive headings that each contain content sections.

**Required ARIA Roles & Attributes:**

```javascript
// ❌ ISSUE: Missing proper ARIA structure
<div className="accordion">
  <div onClick={() => toggle(0)}>Section 1</div>
  {isOpen[0] && <div>Content 1</div>}
</div>

// ✅ CORRECT: Proper accordion implementation
function Accordion({ items }) {
  const [expandedItems, setExpandedItems] = useState([0]); // First item expanded

  const toggleItem = (index) => {
    setExpandedItems((prev) =>
      prev.includes(index)
        ? prev.filter((i) => i !== index)
        : [...prev, index]
    );
  };

  const handleKeyDown = (e, index) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        const nextButton = document.getElementById(`accordion-header-${index + 1}`);
        nextButton?.focus();
        break;
      case 'ArrowUp':
        e.preventDefault();
        const prevButton = document.getElementById(`accordion-header-${index - 1}`);
        prevButton?.focus();
        break;
      case 'Home':
        e.preventDefault();
        document.getElementById('accordion-header-0')?.focus();
        break;
      case 'End':
        e.preventDefault();
        document.getElementById(`accordion-header-${items.length - 1}`)?.focus();
        break;
    }
  };

  return (
    <div className="accordion">
      {items.map((item, index) => {
        const isExpanded = expandedItems.includes(index);
        const headerId = `accordion-header-${index}`;
        const panelId = `accordion-panel-${index}`;

        return (
          <div key={index}>
            <h3>
              <button
                id={headerId}
                aria-expanded={isExpanded}
                aria-controls={panelId}
                onClick={() => toggleItem(index)}
                onKeyDown={(e) => handleKeyDown(e, index)}
              >
                {item.title}
              </button>
            </h3>
            <div
              id={panelId}
              role="region"
              aria-labelledby={headerId}
              hidden={!isExpanded}
            >
              {item.content}
            </div>
          </div>
        );
      })}
    </div>
  );
}
```

**Keyboard Interactions:**
- **Enter/Space**: Toggle panel visibility
- **Tab**: Move to next focusable element
- **Shift + Tab**: Move to previous focusable element
- **Arrow Down** (optional): Move focus to next accordion header
- **Arrow Up** (optional): Move focus to previous accordion header
- **Home** (optional): Move focus to first header
- **End** (optional): Move focus to last header

**Key Requirements:**
- Button must be wrapped in a heading element (h2-h6) with appropriate level
- Use `aria-expanded` to indicate panel state (not in label text)
- Use `aria-controls` to reference the panel ID
- Panel should use `role="region"` and `aria-labelledby`
- Avoid `region` role if accordion has 6+ simultaneously expandable panels

**WCAG SC:** 1.3.1 Info and Relationships (Level A), 4.1.2 Name, Role, Value (Level A), 2.1.1 Keyboard (Level A)

---

### 12.3 Modal Dialog Pattern (Enhanced)

**Issue:** Modal missing focus trap, proper ARIA attributes, or return focus

The guide already includes a basic modal example (lines 284-339), but here are the critical W3C APG requirements:

**Required ARIA Roles & Attributes:**

```javascript
// ✅ CORRECT: W3C APG compliant modal
function Modal({ isOpen, onClose, title, children }) {
  const modalRef = useRef();
  const previousFocus = useRef();

  useEffect(() => {
    if (isOpen) {
      // Save currently focused element
      previousFocus.current = document.activeElement;

      // Focus the modal container or first focusable element
      setTimeout(() => {
        const firstFocusable = modalRef.current.querySelector(
          'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );
        firstFocusable?.focus();
      }, 10);

      // Trap focus within modal
      const handleKeyDown = (e) => {
        if (e.key === 'Escape') {
          onClose();
          return;
        }

        if (e.key === 'Tab') {
          const focusableElements = modalRef.current.querySelectorAll(
            'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
          );
          const first = focusableElements[0];
          const last = focusableElements[focusableElements.length - 1];

          if (e.shiftKey && document.activeElement === first) {
            e.preventDefault();
            last.focus();
          } else if (!e.shiftKey && document.activeElement === last) {
            e.preventDefault();
            first.focus();
          }
        }
      };

      document.addEventListener('keydown', handleKeyDown);
      return () => document.removeEventListener('keydown', handleKeyDown);
    } else {
      // Restore focus when modal closes
      previousFocus.current?.focus();
    }
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div className="modal-backdrop" onClick={onClose}>
      <div
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        ref={modalRef}
        onClick={(e) => e.stopPropagation()}
      >
        <h2 id="modal-title">{title}</h2>
        <button
          onClick={onClose}
          aria-label="Close"
        >
          ×
        </button>
        <div>{children}</div>
      </div>
    </div>
  );
}
```

**Critical Requirements:**
- Must have `role="dialog"` and `aria-modal="true"`
- Must have accessible name via `aria-labelledby` or `aria-label`
- Must trap focus (Tab/Shift+Tab wrap within dialog)
- Must handle Escape key to close
- Must return focus to invoking element when closed
- Must include a visible close button

**WCAG SC:** 2.1.2 No Keyboard Trap (Level A), 4.1.2 Name, Role, Value (Level A), 2.4.3 Focus Order (Level A)

---

### 12.4 Combobox / Autocomplete Pattern

**Issue:** Custom select or autocomplete missing required ARIA attributes

A combobox is an input widget with an associated popup that enables users to select values from a collection.

**Required ARIA Roles & Attributes:**

```javascript
// ✅ CORRECT: Accessible combobox with autocomplete
function Combobox({ label, options, value, onChange }) {
  const [isOpen, setIsOpen] = useState(false);
  const [inputValue, setInputValue] = useState(value || '');
  const [activeIndex, setActiveIndex] = useState(-1);
  const [filteredOptions, setFilteredOptions] = useState(options);
  const comboboxRef = useRef();
  const listboxRef = useRef();

  useEffect(() => {
    const filtered = options.filter((opt) =>
      opt.toLowerCase().includes(inputValue.toLowerCase())
    );
    setFilteredOptions(filtered);
  }, [inputValue, options]);

  const handleKeyDown = (e) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        if (!isOpen) {
          setIsOpen(true);
          setActiveIndex(0);
        } else {
          setActiveIndex((prev) =>
            prev < filteredOptions.length - 1 ? prev + 1 : prev
          );
        }
        break;
      case 'ArrowUp':
        e.preventDefault();
        if (isOpen) {
          setActiveIndex((prev) => (prev > 0 ? prev - 1 : prev));
        }
        break;
      case 'Enter':
        e.preventDefault();
        if (isOpen && activeIndex >= 0) {
          selectOption(filteredOptions[activeIndex]);
        }
        break;
      case 'Escape':
        e.preventDefault();
        setIsOpen(false);
        setActiveIndex(-1);
        break;
      case 'Home':
        e.preventDefault();
        if (isOpen) setActiveIndex(0);
        break;
      case 'End':
        e.preventDefault();
        if (isOpen) setActiveIndex(filteredOptions.length - 1);
        break;
    }
  };

  const selectOption = (option) => {
    setInputValue(option);
    onChange(option);
    setIsOpen(false);
    setActiveIndex(-1);
  };

  return (
    <div className="combobox-container">
      <label id="combobox-label" htmlFor="combobox-input">
        {label}
      </label>
      <input
        id="combobox-input"
        type="text"
        role="combobox"
        aria-expanded={isOpen}
        aria-controls="listbox"
        aria-activedescendant={
          activeIndex >= 0 ? `option-${activeIndex}` : undefined
        }
        aria-autocomplete="list"
        aria-labelledby="combobox-label"
        value={inputValue}
        onChange={(e) => {
          setInputValue(e.target.value);
          setIsOpen(true);
        }}
        onKeyDown={handleKeyDown}
        onFocus={() => setIsOpen(true)}
        ref={comboboxRef}
      />
      {isOpen && filteredOptions.length > 0 && (
        <ul
          id="listbox"
          role="listbox"
          aria-labelledby="combobox-label"
          ref={listboxRef}
        >
          {filteredOptions.map((option, index) => (
            <li
              key={option}
              id={`option-${index}`}
              role="option"
              aria-selected={activeIndex === index}
              onClick={() => selectOption(option)}
            >
              {option}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

**Keyboard Interactions:**
- **Down Arrow**: Opens popup, moves to next option
- **Up Arrow**: Moves to previous option
- **Enter**: Accepts selected option and closes popup
- **Escape**: Closes popup, optionally clears input
- **Home/End**: Move to first/last option
- **Alt + Down Arrow**: Opens popup without moving focus (optional)

**Key Requirements:**
- Input must have `role="combobox"`
- Must include `aria-expanded`, `aria-controls`, and `aria-autocomplete`
- Use `aria-activedescendant` to maintain focus on combobox while indicating active option
- Popup must have `role="listbox"` (or grid/tree/dialog)
- Options must have `role="option"` and `aria-selected`
- DOM focus stays on input, not the popup

**WCAG SC:** 4.1.2 Name, Role, Value (Level A), 2.1.1 Keyboard (Level A), 1.3.1 Info and Relationships (Level A)

---

### 12.5 Menu Button Pattern

**Issue:** Action menu or dropdown menu missing proper ARIA attributes

A menu button opens a menu of commands or actions (not navigation links).

**Required ARIA Roles & Attributes:**

```javascript
// ✅ CORRECT: Menu button with actions
function MenuButton({ label, actions }) {
  const [isOpen, setIsOpen] = useState(false);
  const [focusedIndex, setFocusedIndex] = useState(0);
  const buttonRef = useRef();
  const menuRef = useRef();

  const handleButtonKeyDown = (e) => {
    switch (e.key) {
      case 'Enter':
      case ' ':
        e.preventDefault();
        setIsOpen(true);
        setFocusedIndex(0);
        break;
      case 'ArrowDown':
        e.preventDefault();
        setIsOpen(true);
        setFocusedIndex(0);
        break;
      case 'ArrowUp':
        e.preventDefault();
        setIsOpen(true);
        setFocusedIndex(actions.length - 1);
        break;
    }
  };

  const handleMenuKeyDown = (e) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setFocusedIndex((prev) => (prev + 1) % actions.length);
        break;
      case 'ArrowUp':
        e.preventDefault();
        setFocusedIndex((prev) => (prev - 1 + actions.length) % actions.length);
        break;
      case 'Home':
        e.preventDefault();
        setFocusedIndex(0);
        break;
      case 'End':
        e.preventDefault();
        setFocusedIndex(actions.length - 1);
        break;
      case 'Enter':
      case ' ':
        e.preventDefault();
        actions[focusedIndex].onClick();
        setIsOpen(false);
        buttonRef.current?.focus();
        break;
      case 'Escape':
        e.preventDefault();
        setIsOpen(false);
        buttonRef.current?.focus();
        break;
    }
  };

  useEffect(() => {
    if (isOpen && menuRef.current) {
      const items = menuRef.current.querySelectorAll('[role="menuitem"]');
      items[focusedIndex]?.focus();
    }
  }, [isOpen, focusedIndex]);

  return (
    <div className="menu-button-container">
      <button
        ref={buttonRef}
        aria-haspopup="menu"
        aria-expanded={isOpen}
        aria-controls="action-menu"
        onClick={() => setIsOpen(!isOpen)}
        onKeyDown={handleButtonKeyDown}
      >
        {label}
      </button>
      {isOpen && (
        <ul
          id="action-menu"
          role="menu"
          ref={menuRef}
          onKeyDown={handleMenuKeyDown}
        >
          {actions.map((action, index) => (
            <li key={index} role="none">
              <button
                role="menuitem"
                tabIndex={-1}
                onClick={() => {
                  action.onClick();
                  setIsOpen(false);
                  buttonRef.current?.focus();
                }}
              >
                {action.label}
              </button>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

// Usage example
<MenuButton
  label="Actions"
  actions={[
    { label: 'Edit', onClick: handleEdit },
    { label: 'Delete', onClick: handleDelete },
    { label: 'Share', onClick: handleShare },
  ]}
/>
```

**Keyboard Interactions:**
- **Enter/Space** (on button): Opens menu, focuses first item
- **Arrow Down** (on button): Opens menu, focuses first item
- **Arrow Up** (on button): Opens menu, focuses last item
- **Arrow Down/Up** (in menu): Navigate between items
- **Home/End** (in menu): Move to first/last item
- **Enter** (in menu): Activate menu item
- **Escape** (in menu): Close menu, return focus to button

**Key Requirements:**
- Button must have `aria-haspopup="menu"` and `aria-expanded`
- Menu must have `role="menu"`
- Menu items must have `role="menuitem"` and `tabIndex={-1}`
- Only use for action menus (commands), not navigation links
- Focus must return to button when menu closes

**WCAG SC:** 4.1.2 Name, Role, Value (Level A), 2.1.1 Keyboard (Level A), 2.4.3 Focus Order (Level A)

---

### 12.6 Disclosure (Show/Hide) Pattern

**Issue:** Expandable content missing aria-expanded state

A disclosure widget manages visibility of content through a button control.

**Required ARIA Attributes:**

```javascript
// ❌ ISSUE: Missing aria-expanded
<div>
  <button onClick={() => setIsOpen(!isOpen)}>
    Show details
  </button>
  {isOpen && <div>Details content</div>}
</div>

// ✅ CORRECT: Proper disclosure pattern
function Disclosure({ summary, children }) {
  const [isExpanded, setIsExpanded] = useState(false);
  const contentId = useId();

  return (
    <div>
      <button
        aria-expanded={isExpanded}
        aria-controls={contentId}
        onClick={() => setIsExpanded(!isExpanded)}
      >
        <span aria-hidden="true">{isExpanded ? '▼' : '▶'}</span>
        {summary}
      </button>
      <div id={contentId} hidden={!isExpanded}>
        {children}
      </div>
    </div>
  );
}

// ✅ CORRECT: Alternative using HTML details/summary
function NativeDisclosure({ summary, children }) {
  return (
    <details>
      <summary>{summary}</summary>
      {children}
    </details>
  );
}
```

**Keyboard Interactions:**
- **Enter/Space**: Toggle content visibility

**Key Requirements:**
- Button must have `aria-expanded` attribute
- Optionally use `aria-controls` to reference content element
- Visual indicator (arrow/chevron) should be marked `aria-hidden="true"`
- Consider using native `<details>` and `<summary>` elements when appropriate

**When to Use:**
- FAQ sections
- Expandable descriptions
- "Show more" / "Read more" links
- Collapsible card content

**WCAG SC:** 4.1.2 Name, Role, Value (Level A), 2.1.1 Keyboard (Level A)

---

## 13. ARIA Pattern Quick Reference

| Pattern | Key Roles | Key Attributes | Keyboard |
|---------|-----------|----------------|----------|
| **Tabs** | `tablist`, `tab`, `tabpanel` | `aria-selected`, `aria-controls`, `aria-labelledby` | Arrow keys, Home, End |
| **Accordion** | `button`, `heading`, `region` | `aria-expanded`, `aria-controls`, `aria-labelledby` | Enter/Space, optional arrows |
| **Modal Dialog** | `dialog` | `aria-modal`, `aria-labelledby` | Tab (trapped), Escape |
| **Combobox** | `combobox`, `listbox`, `option` | `aria-expanded`, `aria-activedescendant`, `aria-autocomplete` | Arrows, Enter, Escape |
| **Menu Button** | `button`, `menu`, `menuitem` | `aria-haspopup`, `aria-expanded` | Arrows, Enter, Escape |
| **Disclosure** | `button` | `aria-expanded`, `aria-controls` | Enter/Space |

---

## 14. Additional W3C APG Resources

For more patterns and detailed examples, refer to:
- **W3C ARIA Authoring Practices Guide:** https://www.w3.org/WAI/ARIA/apg/
- **Pattern Examples Index:** https://www.w3.org/WAI/ARIA/apg/example-index/
- **All Patterns:** https://www.w3.org/WAI/ARIA/apg/patterns/

Other common patterns not covered in detail above:
- **Alert/Alert Dialog**: For urgent messages requiring immediate attention
- **Breadcrumb**: Navigation showing current page location
- **Carousel**: Rotating content with prev/next controls
- **Grid/Data Grid**: Interactive tables with keyboard navigation
- **Radio Group**: Set of mutually exclusive options
- **Slider**: Input for selecting value from range
- **Spinbutton**: Input with increment/decrement buttons
- **Switch**: Binary on/off control
- **Toolbar**: Container for grouped controls
- **Tooltip**: Popup description on hover/focus
- **Tree View**: Hierarchical list structure

---

## Files to Analyze

When auditing a web project, review these files:

### Component Files
- `**/*.jsx`, `**/*.tsx` (React)
- `**/*.vue` (Vue)
- `**/*.component.ts`, `**/*.component.html` (Angular)
- `**/*.svelte` (Svelte)
- `**/*.html` (HTML templates)

### Style Files
- `**/*.css`, `**/*.scss`, `**/*.less`
- `**/*.styled.js`, `**/*.styles.ts` (CSS-in-JS)
- `**/*.module.css` (CSS Modules)

### Configuration & Routing
- Router configuration files
- Page/layout components
- Navigation components

### Interactive Components
- Modal/Dialog components
- Dropdown/Select components
- Tab components
- Carousel/Slider components
- Form components
- Tooltip/Popover components

---

## Testing Tools

### Automated Testing
- **axe DevTools** - Browser extension for automated scanning
- **WAVE** - Browser extension with visual feedback
- **Lighthouse** - Built into Chrome DevTools
- **Pa11y** - Command-line tool for CI/CD
- **axe-core** - JavaScript library for automated testing
- **jest-axe** - Jest matcher for accessibility testing

### Screen Readers
- **NVDA** - Free, Windows
- **JAWS** - Commercial, Windows
- **VoiceOver** - Built-in, macOS/iOS
- **ChromeVox** - Chrome extension
- **Orca** - Linux

### Contrast Checkers
- **WebAIM Contrast Checker** - Online tool
- **TPGI Colour Contrast Analyser** - Desktop app
- **Chrome DevTools** - Built-in contrast checker

### Manual Testing
- **Keyboard-only navigation** - Unplug mouse, use Tab, Enter, Space, Arrow keys
- **Zoom to 200%** - Test text scaling
- **Browser extensions** - Landmark navigation, heading structure

---

## Common Web-Specific Issues

### 1. Single Page Applications (SPAs)

**Issues:**
- Focus not managed on route changes
- Page title not updated on route changes
- Screen reader not notified of content changes

**Solutions:**
```javascript
// ✅ Announce route changes
useEffect(() => {
  // Update document title
  document.title = `${pageTitle} - Site Name`;

  // Announce page change to screen readers
  const announcement = document.createElement('div');
  announcement.setAttribute('role', 'status');
  announcement.setAttribute('aria-live', 'polite');
  announcement.textContent = `Navigated to ${pageTitle}`;
  document.body.appendChild(announcement);

  setTimeout(() => announcement.remove(), 1000);

  // Focus management
  const mainHeading = document.querySelector('h1');
  if (mainHeading) {
    mainHeading.setAttribute('tabindex', '-1');
    mainHeading.focus();
  }
}, [location]);
```

### 2. Custom Dropdown/Select

```javascript
// ✅ CORRECT: Accessible custom select
function CustomSelect({ options, value, onChange, label }) {
  const [isOpen, setIsOpen] = useState(false);
  const [focusedIndex, setFocusedIndex] = useState(-1);

  return (
    <div className="custom-select">
      <label id="select-label">{label}</label>
      <button
        aria-haspopup="listbox"
        aria-expanded={isOpen}
        aria-labelledby="select-label"
        onClick={() => setIsOpen(!isOpen)}
      >
        {value || 'Select an option'}
      </button>
      {isOpen && (
        <ul role="listbox" aria-labelledby="select-label">
          {options.map((option, index) => (
            <li
              key={option.value}
              role="option"
              aria-selected={value === option.value}
              onClick={() => {
                onChange(option.value);
                setIsOpen(false);
              }}
            >
              {option.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

### 3. Infinite Scroll / Lazy Loading

```javascript
// ✅ CORRECT: Announce new content loading
function InfiniteScroll({ items }) {
  const [isLoading, setIsLoading] = useState(false);

  return (
    <>
      <div role="feed" aria-busy={isLoading}>
        {items.map((item, index) => (
          <article key={item.id} aria-posinset={index + 1} aria-setsize={items.length}>
            {item.content}
          </article>
        ))}
      </div>
      {isLoading && (
        <div role="status" aria-live="polite">
          Loading more items...
        </div>
      )}
    </>
  );
}
```

---

## Resources

### Documentation
- **MDN Accessibility:** https://developer.mozilla.org/en-US/docs/Web/Accessibility
- **WAI-ARIA Practices:** https://www.w3.org/WAI/ARIA/apg/
- **WebAIM:** https://webaim.org/
- **A11y Project:** https://www.a11yproject.com/

### React-Specific
- **React Accessibility:** https://react.dev/learn/accessibility
- **React ARIA:** https://react-spectrum.adobe.com/react-aria/
- **Radix UI:** https://www.radix-ui.com/ (accessible primitives)

### Vue-Specific
- **Vue Accessibility:** https://vuejs.org/guide/best-practices/accessibility.html
- **Vue A11y:** https://vue-a11y.com/

### Angular-Specific
- **Angular CDK A11y:** https://material.angular.io/cdk/a11y/overview
- **Angular Material:** https://material.angular.io/ (accessible components)

---

## Quick Checklist

When auditing web code, verify:

- [ ] All `<img>` elements have appropriate `alt` text
- [ ] All form inputs have associated `<label>` elements
- [ ] All interactive elements are keyboard accessible
- [ ] Focus indicators are visible and have sufficient contrast
- [ ] Color contrast meets WCAG AA standards (4.5:1 for text, 3:1 for UI)
- [ ] Heading hierarchy is logical (no skipped levels)
- [ ] Landmarks are used (`<header>`, `<nav>`, `<main>`, `<footer>`)
- [ ] Skip link is present and functional
- [ ] Dynamic content updates are announced to screen readers
- [ ] Custom widgets have appropriate ARIA roles and properties
- [ ] Modal dialogs trap focus and restore focus on close
- [ ] Page titles are unique and descriptive
- [ ] Language is declared (`<html lang="en">`)

---

**Related Guides:**
- ARIA_PATTERNS_REFERENCE.md - Complete W3C APG patterns reference (tabs, accordion, dialog, combobox, menu, disclosure, etc.)
- GUIDE_WCAG_REFERENCE.md - WCAG principles and methodology
- COMMON_ISSUES.md - Cross-platform issue patterns
- AUDIT_REPORT_TEMPLATE.md - Report format
