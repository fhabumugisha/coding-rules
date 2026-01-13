---
description: Best practices for Thymeleaf, HTML, CSS, Tailwind 4, Alpine.js (updated 2025)
---

# Frontend Rules (Thymeleaf + Tailwind CSS 4 + Alpine.js) — 2025 Update

These are recommendations and conventions for building accessible, secure, performant, and maintainable server-rendered frontends with Thymeleaf, Tailwind CSS 4, and Alpine.js. They assume a build pipeline (npm / pnpm / yarn + Tailwind CLI / PostCSS) that generates versioned assets.

---

## High-level principles
- Prefer progressive enhancement and server-side rendering (Thymeleaf) with small, focused client interactivity added by Alpine.js.
- Accessibility first: follow WCAG 2.2/3.0 guidance where applicable and test with automated and manual tools.
- Security: escape user data, avoid unsafe inline JS/CSS, adopt CSP with nonces or strict policy, validate inputs server-side.
- Performance: produce optimized, versioned assets and minimize runtime JavaScript. Use preconnect/preload for critical resources.
- Maintainability: keep templates modular, avoid duplicated markup, keep styling consistent with Tailwind utility approach and controlled custom classes.

---

## General guidelines
- Act as an experienced frontend engineer: write clear, semantic HTML and small focused Alpine components.
- Keep UI text localized via message bundles (messages.properties, messages_en.properties, messages_fr.properties, etc.). Avoid hardcoded text.
- Use a consistent component / fragment structure under `/templates/fragments/` and `/templates/components/`.
- Prefer server-provided state (model attributes) injected safely into data attributes or JSON inside a script tag with proper escaping and CSP handling.
- Document component API (expected model attributes, message keys, ARIA expectations) next to fragments.

---

## HTML & Thymeleaf best practices
- Use Thymeleaf attributes appropriately: `th:text`, `th:utext` (only with sanitized content), `th:if`, `th:each`, `th:attr`, `th:href="@{...}"`, `th:src="@{...}"`.
- Never concatenate message properties or fragments of text in templates. Use parameterized messages: `#{label.with.param(${count})}`.
- Use `th:replace` / `th:insert` for layout composition. Use a layout dialect where helpful (e.g., Thymeleaf Layout Dialect) but keep it simple.
- Keep semantic HTML (header, nav, main, footer, form, fieldset, legend, label, button, etc.).
- Accessibility:
  - Always associate `<label for="...">` with inputs or use `aria-labelledby`.
  - Use landmark roles and skip links (e.g., "Skip to main content").
  - Use `aria-live` regions for dynamic messages (e.g., form submission responses).
  - Manage focus when showing/hiding UI (modals, dialogs, dropdowns).
- Escaping and XSS:
  - Use `th:text` for untrusted content (it auto-escapes).
  - Use `th:utext` only for trusted, sanitized HTML (and document why content is safe).
  - Prefer `th:href="@{...}"` and `th:src="@{...}"` to avoid accidental malformation.
- Forms:
  - Use server-side validation errors rendered in templates (bind errors to form fields).
  - Use proper `autocomplete`, `autocomplete="off"` only when appropriate, and `inputmode` when helpful.
  - Use the Tailwind Forms plugin for consistent form styling.

---

## Internationalization (i18n)
- Always use message bundles; avoid hardcoding or concatenating fragments.
- Provide locale negotiation on the server (Accept-Language, user preference).
- Use parameterized messages and pluralization support when needed.
- Keep message keys organized (e.g., entity.action.field).
- Include translations in CI checks to detect missing keys.

---

## Tailwind CSS 4 guidance (practical)
- Build Tailwind from source in CI: install via package manager and compile a single `main.css` with Tailwind + plugins.
- Tailwind config:
  - Set `content` to include Thymeleaf templates and other source files, e.g.:
    - content: ["src/main/resources/templates/**/*.{html,thymeleaf}", "src/main/resources/fragments/**/*.{html}"]
  - If using Thymeleaf expressions inside class attributes, consider `safelist` or use the JIT-safe extraction patterns.
  - Use plugins: `@tailwindcss/forms`, `@tailwindcss/typography`, and any internal plugins.
- Prefer utility classes. For repeated patterns, use `@apply` in a well-organized `styles.css` (component classes) or create "design tokens" in config.
- Dark mode: choose a strategy (class-based `'class'` or `'media'`) and be consistent site-wide. Provide a theme toggle that persists user choice (localStorage + `class="dark"`).
- Avoid inline styles — prefer Tailwind classes or `@apply`.
- Accessibility & contrast: ensure color choices meet contrast ratios; use `ring` utilities for focus styles.
- Purge/CSS size: include Thymeleaf paths in Tailwind content for proper tree-shaking. Use a safelist for dynamic classes generated at runtime if necessary.

---

## Alpine.js guidance
- Use the latest stable Alpine.js (v3.x or later current stable). Prefer the npm-built asset included via the build pipeline.
- Prefer declarative `x-data` state and `x-on` / `@` handlers for small interactive components.
- Server-to-client state: inject only non-sensitive state via `data-*` attributes or safely-encoded JSON in a script tag (escaped and counted against CSP).
  - Example: <div th:attr="data-user-id=${user.id}"> or <div x-data="{ userId: [[${user.id}]] }"> but avoid raw inline script json unless escaped and CSP-safe.
- Avoid embedding large JSON blobs inline; fetch them via authenticated endpoints when possible.
- Accessibility behaviors:
  - Use focus-trap behavior when showing modals (either Alpine plugin or custom focus management).
  - Ensure keyboard operability (Escape to close, proper tab order).
  - Manage aria-expanded, aria-controls for disclosure components.
- Transitions: use `x-show` with `x-transition` for smooth, performant transitions. Keep heavy animation to CSS.
- Security & CSP:
  - If you run a strict CSP disallowing inline scripts, use a nonce-based approach or include Alpine as a non-inline module build. Test CSP early.
- Testing: test Alpine behaviors in E2E tests (Playwright) and snapshot important interactions.

---

## Security checklist
- Use CSP with a conservative policy; avoid `'unsafe-inline'` where possible.
- Use SRI when loading third-party CDNs, and prefer local, versioned builds.
- Escape everything by default; whitelist only trusted HTML.
- Avoid generating script blocks from untrusted input.
- Validate all inputs server-side and sanitize outgoing HTML as needed.

---

## Performance checklist
- Build and version assets (main.[hash].css, main.[hash].js). Serve with long cache headers and immutable caching for hashed assets.
- Preconnect/preload critical origins (fonts) and preload the main css or critical styles.
- Use font-display: swap for web fonts and preload critical fonts only.
- Use Lighthouse and compare scores; optimize largest contentful paint (LCP) and Cumulative Layout Shift (CLS).
- Bundle minimal Alpine.js usage; keep client JS size small.

---

## Testing & QA
- Run automated accessibility scans (axe-core) in CI, and include a11y checks in E2E runs (Playwright).
- Use Lighthouse in CI (or GitHub Action) for performance budgets.
- Include unit tests for server logic and snapshot/E2E tests for critical flows (login, forms, navigation).
- Add a regression test for localized messages to ensure no missing keys.

---

## Project conventions
- Templates:
  - /templates/base.html — global layout and head tags
  - /templates/fragments/header.html, footer.html, nav.html — fragments
  - /templates/components/* — small reusable components (table, button, input)
- Styles:
  - src/styles/styles.css — imports Tailwind base/components/utilities and applies `@apply` for repeated styles
  - tailwind.config.js — keep tokens and colors here
- Scripts:
  - src/js/main.js — minimal client glue, imports Alpine from node_modules and initializes theme toggles and small behaviors

---

## Examples (recommended patterns)

### Example base layout (recommended)
- Use built assets (local) and proper meta tags, preconnect/preload fonts, CSP friendly inclusion.

```html
<!doctype html>
<html lang="en" th:lang="${#locale}">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title th:text="${title}">App</title>

  <!-- Preconnect & preload -->
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link rel="preload" as="style" href="/assets/main.css">
  <link rel="stylesheet" href="/assets/main.css">

  <!-- Built Alpine included via the build pipeline (no inline scripts) -->
  <script type="module" src="/assets/main.js" defer></script>

  <!-- CSP: use nonce or strict policy; avoid inline handlers -->
</head>
<body class="bg-gray-50 dark:bg-gray-900 text-gray-900 dark:text-slate-100">
  <a class="sr-only focus:not-sr-only p-2" href="#main-content" th:text="#\{link.skip\}">Skip to main content</a>

  <header th:replace="fragments/header :: header"></header>

  <main id="main-content" class="container mx-auto p-4">
    <section th:replace="${content}"></section>
  </main>

  <footer th:replace="fragments/footer :: footer"></footer>
</body>
</html>
```

Notes:
- Include built assets (`/assets/main.css`, `/assets/main.js`) produced by your bundler.
- `main.js` should import Alpine and initialize any global behaviors (theme toggle, etc.).

### Example small component: accessible button (Tailwind + Alpine)
- Use proper `type`, ARIA when stateful, and store state in `x-data`.

```html
<button
  type="button"
  class="inline-flex items-center gap-2 px-4 py-2 rounded-md bg-blue-600 text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-400"
  x-data="{ active: false }"
  @click="active = !active"
  :aria-pressed="active.toString()"
  x-bind:class="active ? 'bg-green-600' : 'bg-blue-600'">
  <span x-text="active ? 'Active' : 'Click me'">Click me</span>
</button>
```

Prefer using message properties:
```html
<span th:text="#\{button.toggle\}">Toggle</span>
```

---

## Tailwind config hints (example)
- Ensure `content` covers your template paths. Example snippet for tailwind.config.js:

```js
module.exports = {
  content: [
    './src/main/resources/templates/**/*.html',
    './src/main/resources/templates/**/*.thymeleaf',
    // add any directories where class names might be present
  ],
  darkMode: 'class', // or 'media'
  theme: {
    extend: {
      // design tokens
    }
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
  safelist: [
    // add dynamic classes here if you cannot make them static in templates
  ],
};
```

---

## Migration notes (if coming from CDN / inline approach)
- Move to npm-installed Tailwind and Alpine; compile assets as part of CI.
- Add content paths for Thymeleaf templates to Tailwind config.
- Replace remote asset references with built, versioned files; add preconnect/preload as necessary.
- Audit templates for unsafe `th:utext` and fix or sanitize.

---

## Checklist before merging UI work
- [ ] All user-facing text is localized with message keys.
- [ ] No unsanitized HTML rendered with `th:utext`.
- [ ] CSP is considered; inline scripts minimized or handled with nonces.
- [ ] Accessibility basics covered: skip link, focus states, ARIA where needed.
- [ ] Forms use Tailwind Forms plugin and show inline server-side validation.
- [ ] Assets built and referenced with content-hash filenames (immutable caching).
- [ ] Automated a11y and Lighthouse checks added to CI.
