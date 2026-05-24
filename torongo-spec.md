# Torongo — Master Theme Specification

**Version:** 1.1  
**Date:** 2026-05-24  
**Author:** Architecture Design Phase  
**Status:** Approved — Source of Truth  

---

> This document is the **sole authoritative specification** for the Torongo WordPress theme.  
> All development decisions, file creation, and integration work must conform to this document.  
> When ambiguity arises during implementation, this document takes precedence.  
> Changes to this document require an explicit version increment and change log entry.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Technology Stack](#2-technology-stack)
3. [Overall Architecture](#3-overall-architecture)
4. [Theme Component Hierarchy](#4-theme-component-hierarchy)
5. [Folder Structure](#5-folder-structure)
6. [File Structure and Purpose](#6-file-structure-and-purpose)
7. [Design System Decisions](#7-design-system-decisions)
8. [Header Builder Architecture](#8-header-builder-architecture)
9. [Footer Builder Architecture](#9-footer-builder-architecture)
10. [Homepage System Architecture](#10-homepage-system-architecture)
11. [Customization Architecture](#11-customization-architecture)
12. [WooCommerce Integration Architecture](#12-woocommerce-integration-architecture)
13. [Responsive Architecture](#13-responsive-architecture)
14. [SEO Architecture](#14-seo-architecture)
15. [Accessibility Architecture](#15-accessibility-architecture)
16. [Database Usage](#16-database-usage)
17. [Performance Strategy](#17-performance-strategy)
18. [Security Strategy](#18-security-strategy)
19. [Hooks and Filters Plan](#19-hooks-and-filters-plan)
20. [Internationalization and Translation](#20-internationalization-and-translation)
21. [Dependencies and Libraries](#21-dependencies-and-libraries)
22. [Naming Conventions](#22-naming-conventions)
23. [Coding Standards](#23-coding-standards)
24. [Child Theme Compatibility](#24-child-theme-compatibility)
25. [Development Roadmap](#25-development-roadmap)
26. [Architecture Review Findings](#26-architecture-review-findings)
27. [Assumptions and Open Decisions](#27-assumptions-and-open-decisions)
28. [Change Log](#28-change-log)

---

## 1. Project Overview

### 1.1 Summary

Torongo is a production-ready, premium eCommerce WordPress theme built exclusively as a Full Site Editing (FSE) Block Theme. It is designed to be highly customizable, performant, accessible, and maintainable. It targets independent online stores, multi-category retail shops, and boutique brands running WooCommerce.

### 1.2 Core Principles

| Principle | Implementation Strategy |
|---|---|
| **Block-first** | All layout and structure via Gutenberg blocks and theme.json; no Classic Editor dependency |
| **Design-token driven** | `theme.json` is the single source of truth for all visual decisions |
| **Hook-extensible** | Every meaningful output point exposes action/filter hooks for child themes and plugins |
| **Zero framework** | No Bootstrap, Tailwind, or Foundation; custom design system only |
| **Performance by default** | Critical CSS inlined, JS deferred, assets split per block |
| **Security by default** | All input sanitized, all output escaped, nonces on all forms |
| **Accessibility compliant** | WCAG 2.2 AA as a minimum target |
| **Translation ready** | 100% of user-facing strings use WordPress i18n functions |

### 1.3 Target Environment

| Component | Version |
|---|---|
| WordPress | 7.x |
| WooCommerce | 10.7.0+ |
| PHP | 8.2+ |
| MySQL / MariaDB | 8.0+ / 10.6+ |
| Node.js (build) | 20.x LTS |
| npm (build) | 10.x |

### 1.4 Browser Support

| Browser | Minimum Version |
|---|---|
| Chrome | Last 2 major |
| Firefox | Last 2 major |
| Safari | Last 2 major |
| Edge | Last 2 major |
| Safari iOS | Last 2 major |
| Chrome Android | Last 2 major |

Internet Explorer is **not supported**.

---

## 2. Technology Stack

### 2.1 Core Technologies

| Layer | Technology | Rationale |
|---|---|---|
| CMS | WordPress 7.x | Target platform |
| eCommerce | WooCommerce 10.7.0 | Primary commerce layer |
| Server Language | PHP 8.2+ | Performance, type safety, attributes |
| Template Engine | WordPress Block Templates (HTML) | FSE standard |
| Design System | `theme.json` + CSS Custom Properties | WordPress native, zero overhead |
| JavaScript | Vanilla ES2022+ modules | No jQuery dependency |
| CSS | SCSS → PostCSS → CSS | Modern, maintainable |
| Build Tool | `@wordpress/scripts` (Webpack 5) | WordPress native toolchain |
| Dependency Manager (PHP) | Composer | PSR-4 autoloading |
| Dependency Manager (JS) | npm | Ecosystem standard |

### 2.2 Architecture Pattern

Torongo follows a **layered block-theme architecture**:

```
┌─────────────────────────────────────────────────────┐
│           Global Styles / Site Editor                │
│         (User customizations via UI)                 │
├─────────────────────────────────────────────────────┤
│              theme.json (Design Tokens)              │
│         (Colors, Typography, Spacing, Borders)       │
├─────────────────────────────────────────────────────┤
│           Block Templates + Template Parts           │
│         (Page structure in HTML block markup)        │
├─────────────────────────────────────────────────────┤
│                  Block Patterns                      │
│       (Reusable pre-designed section layouts)        │
├─────────────────────────────────────────────────────┤
│            PHP Business Logic Layer                  │
│    (WooCommerce hooks, block registration, setup)    │
├─────────────────────────────────────────────────────┤
│             SCSS / CSS Asset Layer                   │
│       (Block styles, component styles, global)       │
└─────────────────────────────────────────────────────┘
```

---

## 3. Overall Architecture

### 3.1 Theme Type

Torongo is a **pure Block Theme** (also called "Full Site Editing theme"). It does NOT use:
- `header.php` / `footer.php` template files
- `wp_head()` / `wp_footer()` in PHP templates
- The WordPress Customizer (Customizer API is deprecated for FSE themes)
- Widget areas / Sidebars
- Classic menus (uses Navigation block only)

It DOES use:
- `theme.json` for all design tokens
- `/templates/*.html` for all page templates
- `/parts/*.html` for all template parts (header, footer, etc.)
- `/patterns/*.php` for all block patterns
- `functions.php` as a minimal PHP bootstrap
- `style.css` only for the theme header comment block + critical resets

### 3.2 PHP Architecture

PHP follows a **class-per-concern** pattern with Composer PSR-4 autoloading:

```
functions.php
    └── requires vendor/autoload.php
    └── instantiates Torongo\Core\Setup
            └── Setup::init() registers all hooks
            └── delegates to specialized Manager classes
```

All PHP classes are namespaced under `Torongo\`. Each class has a single responsibility. No global functions except unavoidable WordPress callback compatibility cases (template tags, etc.).

### 3.3 JavaScript Architecture

JavaScript follows an **ES module pattern with per-block entry points**:

- `src/js/frontend/index.js` — global frontend behaviors (navigation, skip link, etc.)
- `src/js/blocks/{block-name}/index.js` — per-block interactivity (where needed)
- `src/js/admin/index.js` — admin-only JS (theme-specific admin pages)
- No global jQuery; use `wp.domReady` for initialization where needed

### 3.4 CSS Architecture

CSS follows a **design-token-first** approach:

1. `theme.json` generates CSS Custom Properties automatically via WordPress
2. SCSS consumes those custom properties
3. SCSS is compiled per-entry-point (global, editor, WooCommerce, and per-block custom blocks)
4. WordPress Block Editor loads per-block CSS only on pages that use that block

### 3.5 WooCommerce Integration Strategy

- **Block-first**: WooCommerce 10.7.0 is assumed to have mature block support for all storefront surfaces. Block templates override WooCommerce defaults via `/templates/` directory.
- **Hook-based extensions**: Non-block customizations (schema markup, structured data, custom product fields) use WooCommerce action/filter hooks in PHP.
- **No PHP template overrides** for surfaces already handled by WooCommerce blocks (cart, checkout, mini cart). PHP template overrides (`/woocommerce/` directory) are reserved only for legacy or non-block surfaces.

---

## 4. Theme Component Hierarchy

```
Torongo Theme
│
├── Block Templates (page-level structure)
│   ├── index.html              (fallback)
│   ├── front-page.html         (homepage)
│   ├── page.html               (static page)
│   ├── single.html             (blog post)
│   ├── archive.html            (blog archive)
│   ├── search.html             (search results)
│   ├── 404.html                (error page)
│   ├── single-product.html     (WooCommerce PDP)
│   ├── archive-product.html    (WooCommerce shop)
│   ├── taxonomy-product_cat.html
│   ├── taxonomy-product_tag.html
│   └── page-no-title.html      (landing pages)
│
├── Template Parts (structural sections, flat — see §5 canonical rule)
│   ├── header.html              (default)
│   ├── header-minimal.html      (transparent/landing)
│   ├── header-sticky.html       (sticky behavior)
│   ├── footer.html              (default multi-column)
│   ├── footer-minimal.html      (single-row)
│   ├── footer-shop.html         (with newsletter)
│   ├── breadcrumbs.html
│   ├── post-meta.html
│   └── pagination.html
│
├── Block Patterns (reusable content sections)
│   ├── Hero patterns (hero-full, hero-split, hero-minimal)
│   ├── Product patterns (grid-3col, grid-4col, featured, carousel)
│   ├── Category patterns (showcase-grid, banner-list)
│   ├── CTA patterns (newsletter, promotional-banner, urgency)
│   ├── Testimonial patterns (single, grid, carousel)
│   ├── Content patterns (blog-grid, about-split, team-grid)
│   └── WooCommerce patterns (product-card, sale-badge, rating)
│
├── Custom Blocks (registered, not native)
│   ├── torongo/product-card-enhanced
│   ├── torongo/countdown-timer
│   ├── torongo/mega-menu
│   └── torongo/trust-badges
│
├── PHP Business Logic
│   ├── Core Setup
│   ├── Asset Manager
│   ├── WooCommerce Manager
│   ├── Block Manager
│   ├── SEO Manager
│   └── Accessibility Manager
│
└── Design System (theme.json)
    ├── Color Palette
    ├── Typography Scale
    ├── Spacing Scale
    ├── Border Styles
    ├── Shadow Presets
    └── Layout Settings
```

---

## 5. Folder Structure

The following is the **canonical** folder structure. No files may be placed outside this structure without a documented exception in this specification.

```
torongo/
│
├── assets/                         # Compiled/static production assets
│   ├── css/                        # Compiled CSS (build output)
│   │   ├── global.css
│   │   ├── woocommerce.css
│   │   └── editor.css
│   ├── js/                         # Compiled JS (build output)
│   │   ├── frontend.js
│   │   └── blocks/
│   ├── fonts/                      # Self-hosted web fonts (WOFF2 only)
│   │   └── {font-family}/
│   └── images/                     # Theme UI images (icons, placeholders)
│       ├── icons/
│       └── placeholders/
│
├── build/                          # Webpack build artifacts (gitignored)
│
├── inc/                            # PHP includes (all autoloaded via Composer)
│   ├── classes/                    # Core PHP classes
│   │   ├── Core/
│   │   │   ├── Setup.php           # Theme bootstrap and init
│   │   │   ├── Assets.php          # Script/style enqueue manager
│   │   │   └── ThemeSupport.php    # add_theme_support() declarations
│   │   ├── WooCommerce/
│   │   │   ├── Manager.php         # WooCommerce integration coordinator
│   │   │   ├── ProductDisplay.php  # Product loop and card modifications
│   │   │   ├── CartCheckout.php    # Cart/checkout block modifications
│   │   │   └── Schema.php          # WooCommerce structured data
│   │   ├── Blocks/
│   │   │   ├── Manager.php         # Block registration coordinator
│   │   │   ├── StylesManager.php   # Block style registration
│   │   │   └── PatternManager.php  # Block pattern registration
│   │   ├── SEO/
│   │   │   └── Manager.php         # SEO meta, schema, open graph
│   │   └── Accessibility/
│   │       └── Manager.php         # ARIA, skip links, focus management
│   │
│   └── helpers/                    # Procedural helper functions
│       ├── template-helpers.php    # Template utility functions
│       └── woocommerce-helpers.php # WooCommerce utility functions
│
├── languages/                      # Translation source files
│   ├── torongo.pot                 # POT template (generated)
│   └── {locale}.po / .mo           # Compiled translation files
│
├── parts/                          # Block template parts (FSE)
│   ├── header.html
│   ├── header-minimal.html
│   ├── header-sticky.html
│   ├── footer.html
│   ├── footer-minimal.html
│   ├── footer-shop.html
│   ├── breadcrumbs.html
│   ├── post-meta.html
│   └── pagination.html
│
├── patterns/                       # Block patterns (.php for i18n, .html for static)
│   ├── hero-full.php
│   ├── hero-split.php
│   ├── hero-minimal.php
│   ├── products-grid-3.php
│   ├── products-grid-4.php
│   ├── products-featured.php
│   ├── products-carousel.php
│   ├── categories-showcase.php
│   ├── categories-banner.php
│   ├── testimonials-single.php
│   ├── testimonials-grid.php
│   ├── testimonials-carousel.php
│   ├── cta-newsletter.php
│   ├── cta-promotional.php
│   ├── cta-urgency.php
│   ├── content-about.php
│   ├── content-blog-grid.php
│   ├── content-team-grid.php
│   ├── woo-product-card.php
│   ├── woo-rating.php
│   └── woo-sale-banner.php
│
├── src/                            # Source files (not deployed directly)
│   ├── scss/
│   │   ├── base/
│   │   │   ├── _reset.scss
│   │   │   ├── _root.scss          # CSS custom property declarations
│   │   │   └── _typography.scss    # Base typographic rules
│   │   ├── blocks/
│   │   │   ├── _navigation.scss
│   │   │   ├── _button.scss
│   │   │   ├── _image.scss
│   │   │   └── ... (one file per block)
│   │   ├── woocommerce/
│   │   │   ├── _global.scss
│   │   │   ├── _product-card.scss
│   │   │   ├── _product-single.scss
│   │   │   ├── _cart.scss
│   │   │   ├── _checkout.scss
│   │   │   └── _account.scss
│   │   ├── components/
│   │   │   ├── _header.scss
│   │   │   ├── _footer.scss
│   │   │   ├── _breadcrumbs.scss
│   │   │   └── _pagination.scss
│   │   ├── utilities/
│   │   │   ├── _mixins.scss
│   │   │   └── _functions.scss
│   │   ├── admin.scss              # Admin-area styles entry (compiles → assets/css/admin.css)
│   │   ├── editor.scss             # Block editor styles entry
│   │   ├── global.scss             # Frontend global styles entry
│   │   └── woocommerce.scss        # WooCommerce styles entry
│   │
│   └── js/
│       ├── frontend/
│       │   ├── index.js            # Main frontend bundle entry
│       │   ├── navigation.js       # Navigation accessibility
│       │   ├── skip-link.js        # Skip link behavior
│       │   └── sticky-header.js    # Sticky header logic
│       ├── blocks/
│       │   ├── countdown-timer/
│       │   │   ├── index.js        # Block registration (edit+save)
│       │   │   ├── edit.js
│       │   │   ├── save.js
│       │   │   └── view.js         # Frontend interactivity
│       │   ├── mega-menu/
│       │   ├── trust-badges/
│       │   └── product-card-enhanced/
│       └── admin/
│           └── index.js            # Admin-only scripts
│
├── templates/                      # Block templates (FSE)
│   ├── index.html
│   ├── front-page.html
│   ├── page.html
│   ├── page-no-title.html
│   ├── single.html
│   ├── single-product.html
│   ├── archive.html
│   ├── archive-product.html
│   ├── taxonomy-product_cat.html
│   ├── taxonomy-product_tag.html
│   ├── search.html
│   └── 404.html
│
├── woocommerce/                    # WooCommerce PHP template overrides
│   │                               # (only for non-block legacy surfaces)
│   ├── myaccount/
│   │   ├── my-account.php
│   │   ├── orders.php
│   │   └── view-order.php
│   └── emails/
│       ├── email-header.php
│       └── email-footer.php
│
├── .gitignore
├── .phpcs.xml                      # PHPCS configuration
├── .stylelintrc.json               # Stylelint configuration
├── .eslintrc.json                  # ESLint configuration
├── .prettierrc                     # Prettier configuration
├── composer.json
├── composer.lock
├── functions.php                   # Bootstrap only (< 50 lines)
├── index.php                       # Required by WordPress (empty/redirect)
├── package.json
├── package-lock.json
├── phpstan.neon                    # PHPStan configuration
├── screenshot.png                  # 1200×900px theme screenshot
├── style.css                       # Theme header + minimal global CSS
├── theme.json                      # Design system (canonical design tokens)
└── webpack.config.js               # Webpack configuration extending @wordpress/scripts
```

**RULE:** The `build/` and `node_modules/` directories are git-ignored. The `assets/` directory for production-compiled assets IS committed to git to allow deployment without a build step on servers.

---

## 6. File Structure and Purpose

### 6.1 Root-Level Files

| File | Purpose | Rules |
|---|---|---|
| `style.css` | Theme registration header + minimal CSS resets only | Must contain the `Theme Name: Torongo` header comment. Must NOT contain layout styles. |
| `functions.php` | PHP bootstrap only | Maximum 50 lines. Only requires autoloader and instantiates `Setup`. No logic. |
| `index.php` | WordPress-required fallback | Contains only `<?php // Silence is golden.` |
| `theme.json` | All design tokens | Single source of truth for colors, typography, spacing, borders, shadows, layout |
| `screenshot.png` | Theme screenshot | 1200×900px, WebP or PNG, shows homepage layout |

### 6.2 `theme.json` Scope

`theme.json` defines and controls:
- **`settings.color.palette`** — all named colors (primary, secondary, accent, neutrals, semantic)
- **`settings.color.gradients`** — all named gradient presets
- **`settings.typography.fontFamilies`** — registered font stacks
- **`settings.typography.fontSizes`** — fluid font size scale (xs through 5xl)
- **`settings.spacing.spacingSizes`** — spacing scale (10 through 100, using 8px base unit)
- **`settings.spacing.padding`** — enable padding controls on blocks
- **`settings.spacing.margin`** — enable margin controls on blocks
- **`settings.layout.contentSize`** — default content width
- **`settings.layout.wideSize`** — wide content width
- **`settings.border`** — border presets and controls
- **`settings.shadow.presets`** — named shadow styles
- **`settings.blocks.*`** — per-block settings overrides
- **`styles.*`** — global style definitions applied to `body`, headings, links, buttons
- **`customTemplates`** — register named custom templates
- **`templateParts`** — register template parts with areas

### 6.3 `functions.php` Responsibility Delegation

```
functions.php
 ↓
Torongo\Core\Setup::init()
 ↓ delegates to:
 ├── Torongo\Core\ThemeSupport::register()     — add_theme_support() calls
 ├── Torongo\Core\Assets::register()           — wp_enqueue_scripts() hooks
 ├── Torongo\Blocks\Manager::register()        — register_block_type() calls
 ├── Torongo\Blocks\PatternManager::register() — register_block_pattern() calls
 ├── Torongo\Blocks\StylesManager::register()  — register_block_style() calls
 ├── Torongo\WooCommerce\Manager::init()       — WooCommerce hook registrations
 ├── Torongo\SEO\Manager::init()               — SEO hook registrations
 └── Torongo\Accessibility\Manager::init()     — A11y hook registrations
```

### 6.4 Template Files

Each template file in `/templates/` is a WordPress block template. All templates share the same structural pattern:

```
[Template Part: header]
[Main Content Area — block-driven]
[Template Part: footer]
```

The "main content area" differs per template:
- `front-page.html` — composed entirely of block patterns (hero, products, etc.)
- `single-product.html` — uses WooCommerce product blocks
- `archive-product.html` — uses WooCommerce product query blocks
- `single.html` — uses Post Content block + Post Meta blocks

### 6.5 PHP Class Files

Each PHP class file follows this structure:
1. `<?php` opening tag
2. `declare(strict_types=1);` — always present
3. Namespace declaration
4. `defined('ABSPATH') || exit;` — always present, prevents direct access
5. Class definition with typed properties and return types

---

## 7. Design System Decisions

### 7.1 Color System

The color system uses **semantic naming** — not descriptive naming (not "blue-500" but "primary").

**Tier 1 — Brand Colors:**
- `primary` — main brand color (action color for CTAs, links)
- `primary-hover` — darkened primary for interactive states
- `secondary` — supporting brand color
- `accent` — high-emphasis highlight (sale badges, notifications)

**Tier 2 — Neutral Scale (10 steps):**
- `neutral-0` through `neutral-900` — white to near-black
- Used for backgrounds, borders, text

**Tier 3 — Semantic Colors:**
- `success` — positive states, in-stock, order confirmed
- `warning` — low stock, pending
- `error` — validation errors, out-of-stock
- `info` — informational notices

**Tier 4 — Surface Colors:**
- `background` — page background
- `surface` — card/panel background
- `surface-hover` — card hover state
- `border` — default border color
- `text` — body text
- `text-muted` — secondary text
- `text-inverse` — text on dark backgrounds

**Rule:** No hardcoded hex values anywhere in SCSS. All colors must reference a `theme.json` CSS custom property using the pattern `var(--wp--preset--color--{name})`.

### 7.2 Typography System

**Font Families:**
- `heading` — display/heading font (self-hosted WOFF2, loaded via `@font-face` in theme.json)
- `body` — body text font
- `mono` — monospace for code blocks

**Font Size Scale (fluid, using CSS clamp()):**

| Token | Min | Max | Usage |
|---|---|---|---|
| `xs` | 11px | 12px | Legal, captions |
| `sm` | 13px | 14px | Meta text, labels |
| `base` | 15px | 16px | Body text |
| `md` | 17px | 18px | Large body |
| `lg` | 19px | 22px | Lead text |
| `xl` | 22px | 28px | Section headings |
| `2xl` | 26px | 36px | Page headings |
| `3xl` | 32px | 48px | Hero subheadings |
| `4xl` | 40px | 60px | Hero headings |
| `5xl` | 48px | 80px | Display type |

All font sizes defined in `theme.json` with `fluid: true` and explicit `min` / `max` values. WordPress generates the `clamp()` values automatically.

**Line Height Presets:**
- `tight` — 1.2 (headings)
- `normal` — 1.5 (body)
- `relaxed` — 1.75 (long-form content)

**Font Weight Presets:**
- `regular` — 400
- `medium` — 500
- `semibold` — 600
- `bold` — 700

### 7.3 Spacing System

Base unit: **8px**

| Token | Value | Usage |
|---|---|---|
| `10` | 4px | Micro (icon padding) |
| `20` | 8px | XS (tight inline spacing) |
| `30` | 12px | SM (compact elements) |
| `40` | 16px | Base (standard gap) |
| `50` | 24px | MD (section element spacing) |
| `60` | 32px | LG (component spacing) |
| `70` | 48px | XL (section padding) |
| `80` | 64px | 2XL (major section gaps) |
| `90` | 96px | 3XL (hero padding) |
| `100` | 128px | 4XL (page-level vertical rhythm) |

All spacing uses `var(--wp--preset--spacing--{token})`. Custom spacing values are not permitted without explicit specification update.

### 7.4 Border System

- `radius-sm` — 4px (inputs, small cards)
- `radius-md` — 8px (buttons, standard cards)
- `radius-lg` — 16px (large cards, modals)
- `radius-xl` — 24px (hero sections)
- `radius-full` — 9999px (pills, circular elements)

Default border width: 1px  
Default border color: `var(--wp--preset--color--border)`

### 7.5 Shadow Presets

- `shadow-sm` — subtle card shadow
- `shadow-md` — standard elevation
- `shadow-lg` — modal/dropdown elevation
- `shadow-focus` — focus ring alternative (use with care, see A11y rules)

### 7.6 Layout Dimensions

| Setting | Value |
|---|---|
| `contentSize` | 800px (blog content max-width) |
| `wideSize` | 1200px (standard layout max-width) |
| Outer padding | `var(--wp--preset--spacing--50)` (24px) on mobile |

---

## 8. Header Builder Architecture

### 8.1 Approach

The header is managed entirely through the **Site Editor** (FSE) using block template parts. There is no PHP-based header builder. The "builder" experience is achieved by:

1. **Multiple header template part variations** — users can switch between pre-designed layouts via Site Editor
2. **Block patterns for header sections** — registered patterns for common header arrangements
3. **Navigation block** — all menus managed through the WordPress Navigation block
4. **Synced template parts** — header is a synced template part, edited once, reflected everywhere

### 8.2 Template Part Variations

| File | Use Case | Features |
|---|---|---|
| `parts/header.html` | Default | Logo, primary nav, search, mini-cart, account icon |
| `parts/header-minimal.html` | Landing pages, checkout | Logo + minimal links only, no distractions |
| `parts/header-sticky.html` | Content-heavy sites | Sticky on scroll, condenses on scroll, same content as default |

**Assumption documented:** A "mega menu" capability is planned as a custom block (`torongo/mega-menu`) but its behavior is controlled via the block's settings panel, not a separate header part.

### 8.3 Header Structural Regions

The default header is composed of these logical regions (each a separate Group block within the template part):

1. **Top Bar** (optional, togglable via block visibility)  
   - USP strip: free shipping notice, contact info, language/currency switcher
2. **Main Header Row**  
   - Logo (Site Logo block)
   - Primary Navigation (Navigation block)
   - Header Actions (search, wishlist, account, mini-cart)
3. **Secondary Nav Row** (optional, togglable)  
   - Category mega-menu or secondary navigation

### 8.4 Mini-Cart

The mini-cart in the header uses the **WooCommerce Mini Cart Block**. No custom PHP mini-cart implementation. Styling is applied via SCSS targeting WooCommerce block CSS classes.

### 8.5 Sticky Header Behavior

Sticky behavior is implemented via:
- CSS `position: sticky` as the baseline
- A small vanilla-JS module (`src/js/frontend/sticky-header.js`) that:
  - Adds `is-sticky` class to the header wrapper on scroll threshold
  - Adds `is-scrolled` class for condensed styling
  - Uses `IntersectionObserver`, not `scroll` event listener
  - Is initialized via `wp.domReady`

### 8.6 Header Hooks

The PHP layer exposes hooks for child theme modifications:

| Hook | Type | Location |
|---|---|---|
| `torongo_before_header` | Action | Before the `<header>` wrapper |
| `torongo_after_header` | Action | After the `</header>` wrapper |
| `torongo_header_classes` | Filter | CSS classes on the `<header>` element |

**Note:** Because FSE header template parts are HTML files, PHP hooks cannot be placed inside them directly. These hooks are implemented via a PHP-registered Block Binding or a `render_block` filter on the header template part's outer Group block wrapper.

---

## 9. Footer Builder Architecture

### 9.1 Approach

Identical to the header: footer is managed via FSE template parts. Multiple pre-designed footer variations are provided.

### 9.2 Template Part Variations

| File | Use Case | Layout |
|---|---|---|
| `parts/footer.html` | Default | 4-column widget area + bottom bar |
| `parts/footer-minimal.html` | Landing pages, thank-you | Logo + copyright + minimal links |
| `parts/footer-shop.html` | Shops with newsletter | Newsletter signup + 3 columns + trust badges |

### 9.3 Footer Structural Regions

**Default Footer:**
1. **Footer Widget Area** (4-column grid of Group blocks)
   - Column 1: Brand — logo, tagline, social icons
   - Column 2: Navigation — shop links, category links
   - Column 3: Navigation — info links (about, contact, blog)
   - Column 4: Contact details / newsletter widget
2. **Trust Strip** (optional)
   - Payment method icons
   - Security badges
3. **Bottom Bar**
   - Copyright text (uses dynamic Post Meta block or custom text)
   - Footer navigation (legal, privacy, terms)

### 9.4 Footer Hooks

| Hook | Type | Location |
|---|---|---|
| `torongo_before_footer` | Action | Before `<footer>` wrapper |
| `torongo_after_footer` | Action | After `</footer>` wrapper |
| `torongo_footer_classes` | Filter | CSS classes on `<footer>` element |

---

## 10. Homepage System Architecture

### 10.1 Template

The homepage uses `templates/front-page.html`. It is the most pattern-dense template in the theme.

### 10.2 Section Architecture

The homepage is assembled from **registered block patterns**, each mapped to a logical section:

| Section | Pattern(s) Available | Description |
|---|---|---|
| Hero | `hero-full`, `hero-split`, `hero-minimal` | Primary above-the-fold section |
| Featured Categories | `categories-showcase`, `categories-banner` | Visual product category navigation |
| Featured Products | `products-grid-3`, `products-grid-4`, `products-featured` | Top or sale products |
| Promotional Banner | `cta-promotional` | Sale/discount full-width banner |
| Best Sellers | `products-grid-4` | Secondary product grid |
| Brand Story | `content-about` | Brand trust section |
| Testimonials | `testimonials-grid`, `testimonials-single` | Social proof |
| Newsletter CTA | `cta-newsletter` | Email capture |
| Blog Preview | `content-blog-grid` | Recent posts section |

### 10.3 Default Homepage Assembly

The shipped default `front-page.html` assembles:
1. `hero-full` — full-width hero with headline, subheadline, CTA
2. `categories-showcase` — category grid
3. `products-featured` — featured products (uses WooCommerce Products block filtered by "featured")
4. `cta-promotional` — promotional banner
5. `products-grid-4` — best sellers (uses WooCommerce Products block filtered by "best_seller")
6. `testimonials-grid` — testimonials
7. `cta-newsletter` — newsletter signup

### 10.4 Homepage Customization Rules

- All section visibility is controlled by the **Block Visibility** toggle in the Site Editor
- The order of sections is controlled by dragging blocks in the Site Editor
- Content (text, images, colors) is controlled via block controls in the Site Editor
- Product queries (which products appear) are controlled by WooCommerce Products block query parameters

---

## 11. Customization Architecture

### 11.1 Customization Tiers

Torongo supports four tiers of customization, in ascending order of complexity:

| Tier | Method | Who | Requires Code |
|---|---|---|---|
| 1 | Site Editor / Global Styles | Store owner | No |
| 2 | Block patterns in Site Editor | Store owner | No |
| 3 | Child theme — `theme.json` | Developer | JSON only |
| 4 | Child theme — PHP hooks + SCSS | Developer | Yes |

### 11.2 Global Styles (Tier 1)

Users can change all `theme.json`-exposed settings via **Appearance → Editor → Styles**:
- Color palette selections
- Font family and size adjustments
- Spacing adjustments
- Per-block style overrides

**Rule:** All design tokens that should be user-adjustable MUST be declared in `theme.json` under `settings`. Hardcoded SCSS values that bypass design tokens are **prohibited**.

### 11.3 Child Theme Customization (Tier 3 & 4)

Child theme architecture:

```
torongo-child/
├── style.css              (registers child theme, declares Template: torongo)
├── functions.php          (minimal — loads child CSS, registers child hooks)
├── theme.json             (overrides parent theme.json values only)
├── templates/             (override specific parent templates)
├── parts/                 (override specific parent template parts)
└── patterns/              (add new or override parent patterns)
```

**Rule:** Child `theme.json` uses WordPress theme.json inheritance. Only include settings that differ from parent. The parent's full token set remains available.

**Rule:** Child `functions.php` must call `wp_enqueue_scripts` with `parent-style` as a dependency when enqueuing child stylesheet.

### 11.4 Customization via Hooks

All major output points in PHP expose hooks. Child themes hook into these to:
- Add content before/after structural sections
- Modify CSS class lists
- Replace or remove default outputs
- Add custom structured data

See Section 19 for the full hooks and filters catalog.

---

## 12. WooCommerce Integration Architecture

### 12.1 Compatibility Declaration

WooCommerce compatibility is declared in `functions.php` using:
- `add_theme_support('woocommerce')` — core compatibility
- `add_theme_support('wc-product-gallery-zoom')` — product image zoom
- `add_theme_support('wc-product-gallery-lightbox')` — product image lightbox
- `add_theme_support('wc-product-gallery-slider')` — product image slider
- `add_theme_support('woocommerce', [...])` — with product grid dimensions

These are registered in `Torongo\Core\ThemeSupport::register()`.

### 12.2 WooCommerce Block Templates

WooCommerce 10.7.0 block templates are overridden in `/templates/`:

| Template File | Overrides | Purpose |
|---|---|---|
| `single-product.html` | WooCommerce single product | Full PDP layout with blocks |
| `archive-product.html` | WooCommerce shop page | Product listing with filters |
| `taxonomy-product_cat.html` | Category archive | Category-branded shop listing |
| `taxonomy-product_tag.html` | Tag archive | Tag-filtered product listing |

### 12.3 Single Product Page (PDP) Architecture

`single-product.html` uses WooCommerce product blocks assembled as:

**Left Column (60% width on desktop):**
- Product Gallery block (images + zoom)
- Sale badge overlay

**Right Column (40% width on desktop):**
- Product Title block
- Product Rating block
- Product Price block (with sale price handling)
- Product Short Description block
- Product Add to Cart block (includes quantity selector)
- Product SKU block
- Product Meta block (categories, tags)
- Trust badges pattern (custom block)

**Below the fold:**
- Product Tabs block (Description, Additional Info, Reviews)
  - Reviews tab uses WooCommerce Reviews block
- Related Products block
- Recently Viewed Products block (if supported by WooCommerce 10.7.0)

**Product Schema:** Injected via `Torongo\WooCommerce\Schema::output()` hooked into `woocommerce_single_product_summary`.

### 12.4 Shop / Archive Page Architecture

`archive-product.html` uses:
- **Top Row:** Page title + product count + sort-by dropdown
- **Filter Sidebar** (optional, togglable):
  - WooCommerce Filters blocks: Price Range, Attribute, Rating, Stock Status
  - Filter Reset button
- **Product Grid:**
  - WooCommerce Products block with query set to "current archive"
  - Default: 4 columns desktop, 2 columns mobile
  - Pagination at bottom (infinite scroll as an option via JS module)

### 12.5 Product Card Architecture

The product card used within the Products block grid is styled entirely via SCSS targeting WooCommerce block CSS classes. The card anatomy:

1. Image wrapper (aspect ratio enforced via CSS)
2. Badges overlay (Sale %, New, Hot)
3. Quick-view trigger (appears on hover)
4. Wishlist button (appears on hover)
5. Product title
6. Rating stars
7. Price (regular + sale)
8. "Add to Cart" button

**Rule:** Product card hover effects use CSS-only transitions where possible. JS is only used for quick-view modal triggering.

### 12.6 Cart Architecture

Cart uses the **WooCommerce Cart Block** exclusively. No `cart.php` override. Styling via SCSS.

Custom additions via hooks:
- `woocommerce_before_cart` — trust badges strip
- `woocommerce_after_cart_table` — cross-sell products
- `woocommerce_cart_totals_before_order_total` — savings summary line

### 12.7 Checkout Architecture

Checkout uses the **WooCommerce Checkout Block** exclusively. No `checkout.php` override.

Custom additions:
- Order summary sidebar enhanced via block extensibility API
- Express checkout (Shop Pay, Apple Pay, etc.) via WooCommerce Payments block
- Coupon field uses WooCommerce Coupon Form block

**Note on Checkout:** WooCommerce 10.7.0 block checkout is assumed to expose all necessary extensibility APIs. If a feature cannot be achieved via WooCommerce's block extensibility APIs, document it as a known limitation and use the `woocommerce_checkout_*` hooks as a fallback.

### 12.8 My Account Architecture

My Account uses a mix:
- WooCommerce My Account Block (if available in 10.7.0) for the account dashboard overview
- PHP template overrides in `/woocommerce/myaccount/` for specific account pages (orders, view-order) where block coverage is incomplete

### 12.9 WooCommerce Email Templates

Email templates in `/woocommerce/emails/` override only:
- `email-header.php` — injects brand logo and colors
- `email-footer.php` — injects brand links and unsubscribe

All other email templates use WooCommerce defaults. Email styling uses inline CSS (WooCommerce email standard). The `Torongo\WooCommerce\Manager` class filters `woocommerce_email_styles` to inject brand color tokens.

### 12.10 WooCommerce Hooks Used

**Actions attached (theme → WooCommerce):**

| Hook | Handler | Purpose |
|---|---|---|
| `woocommerce_before_shop_loop_item_title` | `ProductDisplay::render_sale_badge` | Sale badge above product title |
| `woocommerce_after_shop_loop_item` | `ProductDisplay::render_quick_view_trigger` | Quick view button |
| `woocommerce_single_product_summary` | `Schema::output` | Product JSON-LD schema |
| `woocommerce_before_cart` | `CartCheckout::render_trust_strip` | Trust badges above cart |
| `woocommerce_email_styles` | `Manager::filter_email_styles` | Brand color in emails |

**Filters attached (theme modifying WooCommerce):**

| Filter | Handler | Purpose |
|---|---|---|
| `woocommerce_product_loop_item_classes` | `ProductDisplay::filter_card_classes` | Add theme CSS classes to product loop items |
| `woocommerce_loop_add_to_cart_args` | `ProductDisplay::filter_add_to_cart_args` | Modify add to cart button text/classes |
| `woocommerce_get_breadcrumb` | `Manager::filter_breadcrumb` | Integrate WooCommerce breadcrumbs with theme style |
| `woocommerce_pagination_args` | `Manager::filter_pagination` | Unify pagination style with theme |

---

## 13. Responsive Architecture

### 13.1 Breakpoint System

Breakpoints are defined once in `src/scss/utilities/_mixins.scss` as SCSS variables and mixins. The same values are exposed to JavaScript via `wp_localize_script()` in `Torongo\Core\Assets`, which injects them as `window.torongoData.breakpoints`. PHP constants declared at the top of `inc/helpers/template-helpers.php` (e.g. `TORONGO_BP_TABLET = 768`) serve as the single authoritative source, consumed by both the Assets class (for JS serialization) and documented in SCSS comments for alignment. `theme.json` does not have a breakpoints key and cannot expose breakpoints to JavaScript.

| Name | Min Width | Max Width | Device Target |
|---|---|---|---|
| `mobile-sm` | 320px | 479px | Small phones |
| `mobile` | 480px | 767px | Standard phones |
| `tablet` | 768px | 1023px | Tablets portrait |
| `tablet-lg` | 1024px | 1199px | Tablets landscape, small laptops |
| `desktop` | 1200px | 1535px | Standard desktop |
| `wide` | 1536px | — | Large monitors |

The `desktop` breakpoint (1200px) aligns with `theme.json` `wideSize` as the primary layout width.

### 13.2 SCSS Breakpoint Mixins

Four mixing patterns are used:

1. `@include mobile-only` — styles applied only on mobile-sm and mobile
2. `@include tablet-up` — styles applied from tablet breakpoint and up
3. `@include desktop-up` — styles applied from desktop and up  
4. `@include breakpoint($name)` — utility mixin for specific named breakpoints

**Rule:** Mobile-first approach. Base SCSS is written for mobile. `@include tablet-up` and `@include desktop-up` progressively enhance.

### 13.3 Fluid Typography

All font sizes use CSS `clamp()` generated by `theme.json`'s `fluid: true` setting. No fixed `px` or `rem` font sizes anywhere in SCSS. This means text scales smoothly without breakpoint jumps.

### 13.4 Fluid Spacing

Key section paddings use `clamp()` for fluid spacing:
```
padding-block: clamp(var(--wp--preset--spacing--70), 8vw, var(--wp--preset--spacing--100))
```
This is defined as a named spacing preset where appropriate, otherwise applied directly in pattern HTML.

### 13.5 Responsive Images

- All theme images use `srcset` and `sizes` attributes via `wp_get_attachment_image()` or the Image block
- Hero images use the `cover` image block with a defined aspect-ratio constraint
- Product images use a fixed aspect ratio (1:1 default, configurable via theme.json per-block settings)
- No images are hardcoded at fixed pixel dimensions; all use relative units

### 13.6 WooCommerce Responsive Grid

| Layout | Mobile | Tablet | Desktop |
|---|---|---|---|
| Product Grid | 2 columns | 3 columns | 4 columns |
| Category Grid | 2 columns | 3 columns | 4–5 columns |
| Cart Table | Stacked layout | Standard table | Standard table |
| Checkout | Single column | Single column | 2-column (form + summary) |

Grid column configuration is done in `theme.json` under `settings.blocks.woocommerce/product-template` overrides.

---

## 14. SEO Architecture

### 14.1 Philosophy

Torongo provides SEO **foundations**, not a full SEO plugin. Full SEO management (meta tags, sitemaps, canonical URLs) is delegated to a dedicated SEO plugin (Yoast SEO or RankMath). The theme does not duplicate plugin functionality.

### 14.2 Theme SEO Responsibilities

`Torongo\SEO\Manager` handles:

1. **Structured Data / Schema.org**
   - `WebSite` schema on all pages (includes `SearchAction` for sitelinks search box)
   - `Organization` / `LocalBusiness` schema on homepage and about pages
   - `BreadcrumbList` schema on all archive/single pages
   - `Product` schema on WooCommerce PDP (via `Torongo\WooCommerce\Schema`)
   - `ItemList` schema on shop/archive pages

2. **Open Graph Tags**
   - Applied only when no SEO plugin is active (detected via `function_exists('yoast_get_value')` etc.)
   - `og:title`, `og:description`, `og:image`, `og:url`, `og:type`

3. **Twitter Card Tags**
   - Applied under same condition as Open Graph
   - `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`

4. **HTML Semantics**
   - Correct heading hierarchy enforced via documentation and pattern defaults
   - `<article>`, `<section>`, `<aside>`, `<nav>`, `<main>` used correctly in template parts
   - `<h1>` appears exactly once per page
   - `lang` attribute on `<html>` (WordPress handles this natively)

5. **Canonical URLs**
   - Delegated to WordPress core (`rel=canonical` added automatically by WP 5.0+)
   - Theme does NOT add duplicate canonical tags

6. **Performance as SEO**
   - Core Web Vitals targets influence all performance decisions (see Section 17)

---

## 15. Accessibility Architecture

### 15.1 WCAG Target

**WCAG 2.2 Level AA** is the minimum compliance target. Level AAA success criteria are pursued where practical.

### 15.2 Accessibility Implementation Areas

`Torongo\Accessibility\Manager` handles:

**Skip Links:**
- A "Skip to main content" link is the first focusable element in every page
- A "Skip to navigation" link is provided
- Skip links become visible on focus (not permanently visible)

**Focus Management:**
- All interactive elements have visible focus indicators (minimum 3:1 contrast ratio for focus ring)
- Focus ring styles use `outline` not `box-shadow` (for Windows High Contrast compatibility)
- Focus trap is implemented in all modal/overlay interactions (quick view, mobile menu)

**Keyboard Navigation:**
- All interactive elements are keyboard-operable
- Custom dropdown menus support arrow key navigation
- Mega menu dismissible via Escape key
- Mobile navigation menu has ARIA `expanded` state management

**ARIA Implementation:**
- `aria-label` on icon-only buttons (cart, search, account)
- `aria-expanded` on toggle elements (mobile menu, accordion, dropdowns)
- `aria-live` regions for dynamic content (cart count updates, filter results)
- `role="navigation"` with unique `aria-label` on every `<nav>` element
- Product loading states use `aria-busy`

**Color Contrast:**
- All text/background combinations meet minimum 4.5:1 ratio
- Large text (18px+ normal, 14px+ bold) meets minimum 3:1 ratio
- UI components and state indicators meet 3:1 ratio
- Color is never the sole means of conveying information

**Images:**
- All `<img>` elements have `alt` attributes
- Decorative images use `alt=""`
- Complex images (charts, infographics) have extended descriptions

**Forms:**
- All form inputs have visible, associated `<label>` elements
- Error messages are associated with inputs via `aria-describedby`
- Required fields are indicated via `aria-required="true"` and visible marker
- WooCommerce checkout/account forms must pass this standard

**Animation:**
- All animations respect `prefers-reduced-motion` media query
- No content flashes or rapid blinking
- Carousel auto-play is off by default; if enabled, has pause control

---

## 16. Database Usage

### 16.1 Philosophy

Torongo minimizes database footprint. All customization data is stored in:
1. WordPress core tables (posts, options, usermeta) via standard WordPress APIs
2. WooCommerce tables for product/order data

No custom database tables are created by Torongo v1.0.

### 16.2 Options Storage

Theme-specific options are stored in `wp_options` using the `torongo_` prefix:

| Option Key | Type | Content |
|---|---|---|
| `torongo_version` | string | Current theme version (for upgrade routines) |
| `torongo_installed` | string | Installation date |

No custom options panel exists in v1.0. All design customization is in `theme.json` / Global Styles. All behavior customization is in `wp_options` only via WordPress core mechanisms (menus, widgets, etc. are obsolete in FSE).

### 16.3 Transient Caching

WooCommerce queries that cannot be handled natively by WooCommerce's own cache are cached using transients:

| Transient Key | Expiry | Content |
|---|---|---|
| `torongo_featured_products` | 1 hour | Featured product IDs for homepage |
| `torongo_best_sellers` | 3 hours | Best-selling product IDs |

**Rule:** All transients must be deleted on `woocommerce_product_set_stock`, `save_post_product`, and related hooks to prevent stale data.

### 16.4 User Meta

No custom user meta is stored by Torongo v1.0.

### 16.5 Post Meta

No custom post meta is stored by Torongo v1.0. Any product-level customization uses WooCommerce's existing post meta system.

---

## 17. Performance Strategy

### 17.1 Core Web Vitals Targets

| Metric | Target | Strategy |
|---|---|---|
| LCP (Largest Contentful Paint) | < 2.5s | Preload hero image, critical CSS inline |
| FID / INP (Interaction to Next Paint) | < 200ms | Minimal JS, no blocking scripts |
| CLS (Cumulative Layout Shift) | < 0.1 | Defined image dimensions, font-display swap |
| FCP (First Contentful Paint) | < 1.8s | Critical CSS inline, deferred non-critical |
| TTFB (Time to First Byte) | < 800ms | Server-side caching, not theme concern |

### 17.2 CSS Performance

- **Block-level CSS splitting:** Each custom block has its own CSS file, loaded only when the block is present on the page (WordPress handles this natively for blocks registered with `style` handles)
- **Critical CSS:** Above-the-fold styles for the header, hero, and product grid are output directly to `<head>` via `add_action( 'wp_head', [ $this, 'output_critical_css' ], 1 )` in `Torongo\Core\Assets`. Using `wp_head` at priority 1 ensures the `<style>` tag fires before any enqueued stylesheets, achieving true render-critical inlining. The CSS string is generated as a separate file during the build step and read from disk at runtime. `wp_add_inline_style()` is **not** used for this purpose — it appends styles after an existing enqueued handle and cannot guarantee head placement.
- **CSS Custom Properties:** Eliminates duplicate value declarations — one property, referenced everywhere
- **No CSS framework:** No unused CSS from Bootstrap/Tailwind bloat

### 17.3 JavaScript Performance

- **No jQuery:** All theme JS is vanilla ES2022+. jQuery is not enqueued unless a plugin requires it.
- **Deferred loading:** All frontend JS uses `defer` attribute (set via `wp_script_add_data('handle', 'defer', true)`)
- **Module pattern:** JS split into small, focused modules. No monolithic bundle.
- **Intersection Observer:** Used for lazy initialization of off-screen components (carousels, animation triggers, infinite scroll)
- **No third-party analytics in theme:** No Google Analytics, Facebook Pixel, etc. baked into theme. These are plugin/tag-manager concerns.

### 17.4 Image Performance

- **WebP support:** All theme-generated `srcset` includes WebP variants when WordPress generates them
- **Native lazy loading:** All non-hero images use `loading="lazy"` 
- **Hero image preload:** LCP hero image is preloaded via `<link rel="preload">` added to `wp_head` by `Torongo\Core\Assets`
- **Image dimensions always defined:** Prevents CLS from undimensioned images
- **Placeholder images:** Products without images use a theme-provided SVG placeholder (not an external URL)

### 17.5 Font Performance

- **Self-hosted fonts only:** No Google Fonts or external font CDN (privacy + performance)
- **WOFF2 format only:** Smallest file size, 95%+ browser support
- **`font-display: swap`:** Prevents invisible text during font load
- **Subset fonts:** Only required Unicode ranges included in font files
- **Preconnect:** Not needed since fonts are self-hosted
- **Font preload:** `<link rel="preload" as="font">` for the primary body and heading fonts, added via `wp_head`

### 17.6 WooCommerce Performance

- **Transient caching:** See Section 16.3 for cached WooCommerce queries
- **Selective WooCommerce script loading:** WooCommerce scripts only enqueued on WooCommerce pages (not blog posts, not static pages) via `is_woocommerce()` and `is_cart()` / `is_checkout()` checks
- **Cart fragments optimization:** WooCommerce cart fragment AJAX is limited to WooCommerce pages only (not every page load). Configurable via `woocommerce_cart_fragment_requests` filter.

### 17.7 Caching Compatibility

Torongo is compatible with and tested against:
- **WP Rocket** — no theme-specific WP Rocket config; theme output is static-friendly
- **LiteSpeed Cache** — compatible; theme uses no persistent PHP sessions
- **Object Cache (Redis/Memcached)** — all `get_option()` / `get_transient()` calls benefit automatically
- **Full Page Cache** — theme output is deterministic and cacheable (no user-specific output in cached templates; cart/checkout excluded from caching by WooCommerce)

---

## 18. Security Strategy

### 18.1 Input Sanitization

**Rule:** Every piece of data entering the system (from `$_GET`, `$_POST`, `$_REQUEST`, database reads of user-supplied data) must be sanitized before use.

Sanitization functions used by context:

| Data Type | Function |
|---|---|
| Plain text | `sanitize_text_field()` |
| Rich text / HTML | `wp_kses_post()` or `wp_kses()` with explicit allowed tags |
| Email address | `sanitize_email()` |
| URL | `esc_url_raw()` |
| Integer | `absint()` or `(int)` cast |
| Float | `(float)` cast |
| Slug / key | `sanitize_key()` |
| File name | `sanitize_file_name()` |
| Array of text | `array_map('sanitize_text_field', ...)` |

### 18.2 Output Escaping

**Rule:** Every piece of dynamic data output to HTML must be escaped at the point of output.

Escaping functions used by context:

| Output Context | Function |
|---|---|
| HTML element content | `esc_html()` |
| HTML attribute value | `esc_attr()` |
| URL (href, src) | `esc_url()` |
| JavaScript context | `esc_js()` |
| CSS value | `esc_attr()` (safe subset only) |
| Rich HTML content | `wp_kses_post()` |
| Translated strings with dynamic content | `esc_html( sprintf( __( '...%s', 'torongo' ), $var ) )` |

**Rule:** `echo $variable` without escaping is **prohibited**. No exceptions. PHPCS enforces this.

### 18.3 Nonces

All forms and AJAX requests that perform state-changing operations use WordPress nonces:
- `wp_nonce_field()` in HTML forms
- `check_admin_referer()` or `check_ajax_referer()` on the handler
- Nonce names follow the pattern `torongo_{action}_nonce`

### 18.4 Capability Checks

**Rule:** Every admin action, AJAX handler, and REST endpoint registered by Torongo must check the current user's capability before executing.

```php
if ( ! current_user_can( 'manage_options' ) ) {
    wp_die( esc_html__( 'Unauthorized', 'torongo' ) );
}
```

### 18.5 Direct File Access Prevention

**Rule:** Every PHP file (except `functions.php`, `index.php`, and template files that WordPress loads directly) must begin with:

```php
defined( 'ABSPATH' ) || exit;
```

### 18.6 SQL Injection Prevention

**Rule:** Never use raw SQL string concatenation. Always use `$wpdb->prepare()` for any custom queries (though Torongo v1.0 aims to use zero custom queries — all data access through WordPress/WooCommerce APIs).

### 18.7 XSS Prevention

- Output escaping (Section 18.2) is the primary defense
- Content Security Policy headers: Torongo adds recommended CSP headers via `send_headers` hook for environments that support it. Default policy: `default-src 'self'`, with appropriate exceptions for trusted WooCommerce payment gateways
- `wp_enqueue_script` is used exclusively for all JavaScript (no inline `<script>` tags except for JSON-LD structured data, which is sanitized)

### 18.8 CSRF Prevention

- WordPress nonces on all forms (Section 18.3)
- WooCommerce's own nonce system used for cart/checkout AJAX operations — no duplication

### 18.9 Sensitive Data Handling

- No API keys, credentials, or sensitive configuration in theme files
- License keys (for premium distribution) handled via a separate plugin/updater mechanism, not stored in `wp_options` without encryption
- Payment credentials handled entirely by WooCommerce payment gateways — theme has no access to payment data

### 18.10 PHPCS Security Rules

The `.phpcs.xml` configuration enforces:
- `WordPress-Core` standard
- `WordPress-Security` standard
- `WordPress-Docs` standard (for docblocks)
- `PHPCompatibilityWP` for PHP 8.2 compatibility

---

## 19. Hooks and Filters Plan

### 19.1 Philosophy

Every meaningful output point exposes an action hook. Every filterable value exposes a filter hook. Hook names follow the pattern: `torongo_{context}_{timing}` for actions and `torongo_{context}_{value}` for filters.

### 19.2 Action Hooks

| Hook | Arguments | Description |
|---|---|---|
| `torongo_before_header` | none | Before `<header>` wrapper opens |
| `torongo_after_header` | none | After `</header>` wrapper closes |
| `torongo_header_top_bar_start` | none | Inside top bar, before content |
| `torongo_header_top_bar_end` | none | Inside top bar, after content |
| `torongo_before_footer` | none | Before `<footer>` wrapper opens |
| `torongo_after_footer` | none | After `</footer>` wrapper closes |
| `torongo_before_main_content` | none | Before `<main>` content area |
| `torongo_after_main_content` | none | After `</main>` content area |
| `torongo_before_breadcrumbs` | none | Before breadcrumb list |
| `torongo_after_breadcrumbs` | none | After breadcrumb list |
| `torongo_product_card_before` | `int $product_id` | Before product card markup |
| `torongo_product_card_after` | `int $product_id` | After product card markup |
| `torongo_before_shop_sidebar` | none | Before shop filter sidebar |
| `torongo_after_shop_sidebar` | none | After shop filter sidebar |
| `torongo_checkout_trust_badges` | none | Inside checkout, trust badge placement |
| `torongo_after_page_hero` | none | After page hero section |

### 19.3 Filter Hooks

| Hook | Arguments | Return | Description |
|---|---|---|---|
| `torongo_header_classes` | `string[] $classes` | `string[]` | Classes on `<header>` element |
| `torongo_footer_classes` | `string[] $classes` | `string[]` | Classes on `<footer>` element |
| `torongo_main_classes` | `string[] $classes` | `string[]` | Classes on `<main>` element |
| `torongo_body_classes` | `string[] $classes` | `string[]` | Additional body classes (merged with wp_body_class) |
| `torongo_product_card_classes` | `string[] $classes, int $product_id` | `string[]` | Classes on product card wrapper |
| `torongo_breadcrumb_args` | `array $args` | `array` | Arguments for breadcrumb output function |
| `torongo_schema_data` | `array $schema, string $type` | `array` | JSON-LD schema array before encoding |
| `torongo_preload_fonts` | `string[] $font_urls` | `string[]` | Font URLs to add `<link rel="preload">` for |
| `torongo_critical_css` | `string $css` | `string` | Critical CSS string for inline injection |
| `torongo_block_patterns` | `array $patterns` | `array` | Registered block patterns array (for conditional loading) |
| `torongo_woocommerce_card_image_size` | `string $size` | `string` | Image size used in product card |
| `torongo_404_suggestions` | `WP_Query $query` | `WP_Query` | Query for suggested posts on 404 page |

### 19.4 Hooks Implementation Method

Because Torongo is an FSE theme with HTML template parts, PHP hooks cannot be placed directly inside `.html` template part files. Implementation methods:

1. **`render_block` filter on the outermost block:** The `Torongo\Core\Setup` class uses `add_filter('render_block', ...)` to inject hook output before/after specific named blocks (e.g., the first Group block in `header.html`)
2. **Block Bindings API (WordPress 7.x):** Where WordPress 7.x supports Block Bindings, use this to bind dynamic PHP output to block attributes
3. **Template hooks via `wp_body_open` and `wp_footer`:** These native WordPress hooks serve as `torongo_before_header` equivalents where needed

---

## 20. Internationalization and Translation

### 20.1 Text Domain

The text domain for all theme strings is **`torongo`**. This matches the theme slug and directory name.

### 20.2 Translation Functions

| Context | Function |
|---|---|
| Simple string | `__( 'string', 'torongo' )` |
| Echoed string | `_e( 'string', 'torongo' )` |
| Plural string | `_n( 'singular', 'plural', $count, 'torongo' )` |
| String with context | `_x( 'string', 'context', 'torongo' )` |
| No-op string (collect for translation) | `_nx_noop()` / `_n_noop()` |

**Rule:** Every user-facing string (including screen-reader-only strings and ARIA labels) must use a translation function. No hardcoded English strings in output.

### 20.3 POT File Generation

The `torongo.pot` file is generated using WP-CLI:
```
wp i18n make-pot . languages/torongo.pot --domain=torongo
```

This command is run as part of the release process.

### 20.4 Block Pattern Translation

Block patterns using `.php` files (not `.html`) support translation via `_e()` calls in the pattern registration `content` argument. PHP block pattern files are preferred over HTML for any pattern containing user-facing text.

### 20.5 JavaScript Translations

JavaScript translation is handled via `wp_set_script_translations('torongo-frontend', 'torongo', get_template_directory() . '/languages')`. The `.json` translation files for JS are generated during the build process using `@wordpress/scripts`.

### 20.6 RTL Support

- A `rtl.css` file is generated by the build process via `rtlcss`
- WordPress automatically enqueues `rtl.css` when a RTL language is active
- All SCSS uses logical CSS properties (`margin-inline-start` vs `margin-left`, `padding-block` vs `padding-top/bottom`) where browser support allows, falling back to physical properties with RTL overrides in `rtl.css`

---

## 21. Dependencies and Libraries

### 21.1 PHP Dependencies (Composer)

| Package | Version | Purpose |
|---|---|---|
| *(none required in v1.0)* | — | WordPress core provides all needed PHP APIs |

**Decision:** No Composer PHP packages in v1.0. WordPress's built-in APIs (WP_Query, wpdb, WP_HTTP, etc.) satisfy all requirements. This decision reduces attack surface, simplifies deployment, and eliminates composer dependency management overhead. If a DI container or additional library becomes necessary in a future version, document the justification here.

The `composer.json` file exists and defines PSR-4 autoloading for the `Torongo\` namespace pointing to `inc/classes/`.

### 21.2 JavaScript Dependencies (npm)

**`dependencies` (bundled into theme JS):**

| Package | Version | Purpose |
|---|---|---|
| `swiper` | ^11.x | Touch slider/carousel (product galleries, carousels) |
| `a11y-dialog` | ^8.x | Accessible modal dialogs (quick view) |
| `focus-trap` | ^7.x | Focus trap for modals/mobile menu |

**`devDependencies` (build tools and type checking only — never bundled):**

| Package | Version | Purpose |
|---|---|---|
| `@wordpress/scripts` | ^30.x | Build toolchain (Webpack, Babel, ESLint, Stylelint) |
| `@wordpress/blocks` | ^13.x | Type checking only — runtime is WordPress global `wp.blocks` |
| `@wordpress/element` | ^6.x | Type checking only — runtime is WordPress global `wp.element` |
| `@wordpress/i18n` | ^5.x | Type checking only — runtime is WordPress global `wp.i18n` |
| `@wordpress/dom-ready` | ^4.x | Type checking only — runtime is WordPress global `wp.domReady` |
| `@wordpress/interactivity` | ^6.x | Type checking only — runtime is WordPress global `wp.interactivity` |
| `postcss` | ^8.x | CSS processing pipeline |
| `rtlcss` | ^4.x | RTL CSS generation |
| `stylelint` | ^16.x | CSS linting |

**Rule:** All `@wordpress/*` packages except `@wordpress/scripts` must appear in `devDependencies` only and must be explicitly listed as `externals` in `webpack.config.js`. They are provided as globals by WordPress at runtime (`wp.blocks`, `wp.element`, etc.) and must never be bundled into theme output. Bundling them causes version conflicts with the copy WordPress ships and doubles the JS payload.

**Note on Swiper:** Swiper is used for product image galleries and carousels. It is loaded only on pages that include the relevant blocks (conditional enqueuing via `wp_enqueue_script` called conditionally).

### 21.3 WordPress Plugin Dependencies

**Required:**
- WooCommerce 10.7.0+

**Recommended (not required):**
- WooCommerce Payments — for block-based payment methods
- Yoast SEO or RankMath — for advanced SEO (theme provides foundations only)

**Compatible with:**
- WP Rocket
- LiteSpeed Cache
- WPML / Polylang (theme is translation-ready)
- ACF (Advanced Custom Fields) — no dependency, but compatible for blocks
- Elementor — **not supported within theme templates** (theme is FSE; Elementor integration is a separate concern)

---

## 22. Naming Conventions

### 22.1 PHP Naming

| Element | Convention | Example |
|---|---|---|
| Class | PascalCase, under namespace | `Torongo\WooCommerce\Manager` |
| Method | camelCase | `renderProductCard()` |
| Property | camelCase | `$productId` |
| Local variable | camelCase | `$cartItems` |
| Global function (helpers only) | `torongo_{snake_case}` | `torongo_get_breadcrumbs()` |
| Constants | `TORONGO_UPPER_SNAKE` | `TORONGO_VERSION` |
| Hook names | `torongo_{context}_{name}` | `torongo_product_card_before` |
| Option keys | `torongo_{snake_case}` | `torongo_version` |
| Transient keys | `torongo_{snake_case}` | `torongo_featured_products` |

### 22.2 CSS / SCSS Naming

CSS classes use **BEM (Block__Element--Modifier)** methodology.

| Element | Convention | Example |
|---|---|---|
| Block | `.torongo-{name}` | `.torongo-product-card` |
| Element | `.torongo-{name}__{element}` | `.torongo-product-card__image` |
| Modifier | `.torongo-{name}--{modifier}` | `.torongo-product-card--on-sale` |
| State class | `.is-{state}` | `.is-sticky`, `.is-open`, `.is-loading` |
| JS hook class | `.js-{name}` | `.js-quick-view-trigger` |
| SCSS variable | `$torongo-{name}` | `$torongo-breakpoint-desktop` |
| SCSS mixin | `torongo-{name}` | `torongo-flex-center` |

**Rule:** CSS classes that start with `.js-` are for JavaScript targeting only. They must not have CSS styles applied to them. This separates JS hooks from styling hooks.

**Rule:** WooCommerce block CSS classes (prefixed with `.wc-block-`) are never renamed. SCSS styles target these classes directly for WooCommerce surface styling.

### 22.3 JavaScript Naming

| Element | Convention | Example |
|---|---|---|
| Variable | camelCase | `cartItemCount` |
| Function | camelCase | `initStickyHeader()` |
| Class | PascalCase | `QuickViewModal` |
| Constant | UPPER_SNAKE_CASE | `STICKY_THRESHOLD` |
| File name | kebab-case | `sticky-header.js` |
| Event name | `torongo:{camelCase}` | `torongo:cartUpdated` |

### 22.4 File Naming

| Type | Convention | Example |
|---|---|---|
| PHP class file (PSR-4, all classes) | `{ClassName}.php` under `inc/classes/` | `Manager.php`, `Assets.php` |
| PHP helper file (non-namespaced) | `{context}-helpers.php` under `inc/helpers/` | `woocommerce-helpers.php` |
| SCSS partial | `_{name}.scss` | `_product-card.scss` |
| JS module | `{name}.js` | `sticky-header.js` |
| Block template | `{template-name}.html` | `single-product.html` |
| Template part | `{part-name}.html` | `header-minimal.html` |
| Block pattern | `{pattern-name}.php` | `hero-full.php` |
| Block directory | `{block-name}/` | `countdown-timer/` |

### 22.5 WordPress Block Naming

All custom blocks use the `torongo/` namespace:

| Block Name | Block Slug |
|---|---|
| Product Card Enhanced | `torongo/product-card-enhanced` |
| Countdown Timer | `torongo/countdown-timer` |
| Mega Menu | `torongo/mega-menu` |
| Trust Badges | `torongo/trust-badges` |

---

## 23. Coding Standards

### 23.1 PHP Standards

- **Standard:** WordPress Coding Standards (WPCS) enforced by PHPCS
- **Configuration:** `.phpcs.xml` at root
- **Additional:** PHPStan at level 5 for static analysis
- **PHP Version:** All code must be compatible with PHP 8.2+
- **Strict types:** `declare(strict_types=1)` at the top of every PHP class file
- **Type hints:** All method parameters and return types must be type-hinted
- **Nullable types:** Use `?Type` for nullable parameters; avoid union types unless necessary for clarity
- **Docblocks:** Required for all public methods; optional for private/protected methods if the type hints are self-documenting

### 23.2 JavaScript Standards

- **Standard:** WordPress ESLint configuration (`@wordpress/eslint-plugin`)
- **Configuration:** `.eslintrc.json` at root
- **Module system:** ES modules (`import`/`export`) exclusively
- **No `var`:** Use `const` by default, `let` when mutation is required
- **Arrow functions:** Preferred for callbacks; avoid if `this` context is needed
- **Async/await:** Preferred over Promise chains

### 23.3 CSS Standards

- **Standard:** Stylelint with `stylelint-config-wordpress` configuration
- **Configuration:** `.stylelintrc.json` at root
- **BEM:** Enforced by `stylelint-selector-bem-pattern` plugin
- **Custom properties:** Only `var(--wp--*)` for design tokens; custom `--torongo-*` properties for computed values only
- **No magic numbers:** All spacing/size values must reference a design token or be explicitly documented

### 23.4 Git Standards

- **Branching:** `main` (production), `develop` (integration), `feature/{name}`, `fix/{name}`, `chore/{name}`
- **Commit messages:** Conventional Commits format — `feat:`, `fix:`, `docs:`, `style:`, `refactor:`, `test:`, `chore:`
- **PR process:** All features require a PR against `develop`; `develop` merges to `main` for releases
- **Version tags:** Semantic versioning — `v1.0.0`, `v1.0.1`, `v1.1.0`

### 23.5 Build Standards

- **Source maps:** Generated in `development` mode; NOT included in production builds
- **Minification:** All CSS and JS minified in production
- **Bundle analysis:** `webpack-bundle-analyzer` run before each major release to check for bloat
- **Asset versioning:** WordPress `wp_enqueue_style/script` version parameter set to `TORONGO_VERSION` constant (busts cache on theme update)

---

## 24. Child Theme Compatibility

### 24.1 Child Theme Contract

Torongo guarantees the following for child themes:

1. All hooks documented in Section 19 will remain stable across minor versions
2. `theme.json` structure follows WordPress FSE specification; child themes can override any key
3. Template files in `/templates/` and `/parts/` can be overridden by placing files at the same relative path in the child theme
4. CSS custom properties generated by `theme.json` are stable and follow the `--wp--preset--*` naming convention (WordPress managed)

### 24.2 Breaking Change Policy

Hooks, filters, and CSS custom property names are considered part of the public API. Changes to these in a minor version are considered breaking changes and require a major version bump.

Removal of a hook, filter, or class requires a minimum 1 full major version deprecation period with a documented alternative.

### 24.3 Child Theme Template Override Rules

- A child theme file at `child-theme/templates/single-product.html` fully replaces `torongo/templates/single-product.html`
- There is no partial template inheritance for `.html` template files (WordPress FSE limitation)
- PHP helper functions from `torongo/inc/helpers/` are available in child themes since they are loaded by the parent's `functions.php`
- Child theme `functions.php` must call `wp_enqueue_style( 'torongo-style', get_template_directory_uri() . '/style.css' )` to ensure parent styles load

---

## 25. Development Roadmap

### 25.1 Phase 1 — Foundation (v1.0)

**Goal:** A complete, production-ready theme skeleton with full architecture implemented.

| Deliverable | Description |
|---|---|
| `theme.json` | Complete design token system |
| PHP bootstrap | `Setup`, `Assets`, `ThemeSupport` classes |
| Base templates | All 12 block templates |
| Base template parts | All 9 template parts (header x3, footer x3, content x3) |
| Core block styles | Styling for all native WordPress blocks |
| WooCommerce base | `Torongo\WooCommerce\Manager` + basic WooCommerce hook integrations |
| Global SCSS | Base styles, reset, typography |
| Build pipeline | Webpack configuration, SCSS compilation |
| Accessibility | Skip links, focus management, ARIA |
| SEO | Schema.org structured data |

### 25.2 Phase 2 — eCommerce (v1.1)

| Deliverable | Description |
|---|---|
| PDP full implementation | Complete single product page with all WooCommerce blocks |
| Shop/archive implementation | Complete shop page with filters, sorting, grid |
| Product card complete | All states: hover, sale, out-of-stock, new |
| Cart and checkout styling | Complete WooCommerce Cart/Checkout block styling |
| My Account pages | PHP template overrides for account pages |
| Quick view integration | `torongo/product-card-enhanced` block + `a11y-dialog` modal (see §12.5) |
| Trust badges block | `torongo/trust-badges` custom block |
| WooCommerce email templates | Branded email header/footer |

### 25.3 Phase 3 — Homepage and Patterns (v1.2)

| Deliverable | Description |
|---|---|
| All hero patterns | 3 hero variations |
| All product patterns | 4 product section variations |
| Category patterns | 2 category showcase variations |
| Testimonial patterns | 3 testimonial variations (single, grid, carousel) |
| CTA patterns | Newsletter + promotional banner + urgency countdown |
| Content patterns | About, blog grid, team grid |
| Homepage template | Complete `front-page.html` assembly |
| Landing page template | `page-no-title.html` |

### 25.4 Phase 4 — Custom Blocks and Enhancement (v1.3)

| Deliverable | Description |
|---|---|
| Countdown Timer block | `torongo/countdown-timer` with view.js |
| Mega Menu block | `torongo/mega-menu` with accessibility |
| Product Card Enhanced | `torongo/product-card-enhanced` |
| Sticky header JS | Complete with IntersectionObserver |
| Mobile navigation | Off-canvas menu with focus trap |
| Product image gallery | Swiper-based gallery for PDP |
| Quick view modal | a11y-dialog based product quick view |
| Infinite scroll | Optional for shop/archive pages |

### 25.5 Phase 5 — Polish and Production (v1.4)

| Deliverable | Description |
|---|---|
| Critical CSS generation | Build step to extract and inline critical CSS |
| Performance audit | Core Web Vitals optimization pass |
| Accessibility audit | WCAG 2.2 AA compliance audit and fixes |
| Browser testing | Cross-browser QA on supported browsers |
| WooCommerce audit | Full WooCommerce compatibility checklist |
| RTL support | `rtl.css` generation and testing |
| Translation testing | POT file, `.po` / `.mo` for test locale |
| Documentation | Inline code documentation (docblocks) |
| Unit tests (PHP) | PHPUnit tests for helper functions and hooks |
| Security review | PHPCS security audit, penetration test checklist |

---

## 26. Architecture Review Findings

This section documents the findings of the pre-specification review conducted after initial architecture design.

### 26.1 Missing Architecture Decisions (Resolved)

**Finding:** How are WooCommerce hooks injected into FSE HTML template parts?  
**Resolution:** Documented in Section 19.4. Uses `render_block` filter to inject PHP hook output adjacent to named blocks. This is the established pattern for FSE + PHP hook compatibility.

**Finding:** What happens if WooCommerce 10.7.0 does not support full block checkout for all gateways?  
**Resolution:** Documented in Section 12.7 as a known risk. The architecture includes a fallback path to PHP template overrides (`/woocommerce/`) for any surface not covered by WooCommerce blocks. The theme does not assume 100% block coverage for all WooCommerce surfaces.

**Finding:** Font loading strategy undefined.  
**Resolution:** Documented in Sections 7.2 and 17.5. Self-hosted WOFF2 only, registered via `theme.json` `fontFace` declarations, with preload links added via `wp_head`.

**Finding:** How does the theme handle WooCommerce HPOS (High Performance Order Storage)?  
**Resolution:** Documented in Section 12 implicitly — the theme uses zero direct order table queries. All order data access is via WooCommerce's order APIs (`wc_get_order()`, etc.), which are HPOS-compatible. No direct `wp_posts` or `postmeta` queries for orders.

### 26.2 WooCommerce Compatibility Review

**Result:** Satisfactory with two noted risks:

1. **WooCommerce 10.7.0 is a future version** as of this specification's creation. Architecture assumes this version has feature-complete block templates for all storefront surfaces. If launched with an earlier WooCommerce version, the PHP template override path in `/woocommerce/` serves as fallback. The `Torongo\WooCommerce\Manager::check_version()` method should validate minimum version on `admin_notices`.

2. **WooCommerce Blocks extensibility API:** The checkout block customization in Section 12.7 depends on WooCommerce's slot-fill extensibility system. This API must be validated against WooCommerce 10.7.0's actual API surface during Phase 2 development.

### 26.3 Scalability Review

**Result:** Good. Key scalability points confirmed:

- PSR-4 autoloading means adding new PHP features (new Manager class) requires only a class file — no `functions.php` changes
- Block pattern registration via `PatternManager` — adding patterns requires only a new `.php` file in `/patterns/`
- No centralized CSS file that grows unboundedly — SCSS is split per-concern
- `theme.json` design tokens allow global design changes without hunting through SCSS files
- Child theme architecture (Section 24) provides a clean extension path without forking

**Risk flagged:** If WooCommerce requires more than ~20 PHP hook integrations, the single `Torongo\WooCommerce\Manager` class should be split further (e.g., `ProductManager`, `CartManager`, `OrderManager`). This split is defined in the folder structure (`inc/classes/WooCommerce/`) and should be implemented if the class exceeds 300 lines.

### 26.4 Maintainability Review

**Result:** Strong. Key maintainability decisions confirmed:

- `theme.json` as single source of truth eliminates "where is this color defined?" questions
- BEM CSS naming makes component boundaries clear
- One class per file, one concern per class, clear namespace hierarchy
- Hooks/filters documented in Section 19 serve as an API contract — team members know exactly what's hookable
- Coding standards enforced by automated tools (PHPCS, ESLint, Stylelint) — not dependent on code review discipline

**Concern:** Template parts are HTML files; changes to them require Site Editor access, not file editing. Developers may be unfamiliar with the flow of "edit template part HTML → commit to git → deploy → Site Editor shows update." This must be covered in developer onboarding documentation (outside scope of this specification).

### 26.5 Performance Concerns Review

**Result:** Architecture is performance-positive. Concerns and mitigations:

| Concern | Mitigation |
|---|---|
| WooCommerce adds many scripts globally | Section 17.6 — conditional enqueuing |
| Large hero images cause poor LCP | Section 17.4 — preload link + `fetchpriority="high"` |
| Swiper library is ~30KB | Loaded conditionally via block registration |
| WooCommerce cart fragments AJAX on every page | Section 17.6 — `woocommerce_cart_fragment_requests` filter |
| Many block patterns = large DOM on homepage | Progressive enhancement: patterns below fold use `loading="lazy"` on images; JS modules use IntersectionObserver |
| Google Fonts (if used) = render blocking | Prohibited by design — self-hosted fonts only (Section 17.5) |

---

## 27. Assumptions and Open Decisions

This section documents decisions that were made under uncertainty. If any assumption proves incorrect during implementation, this section must be updated and affected architecture sections revised.

### 27.1 Technology Assumptions

| # | Assumption | Impact if Wrong | Confidence |
|---|---|---|---|
| A1 | WordPress 7.x retains and matures the FSE block template system without breaking changes to `theme.json` v3 | Significant — template and design token system may need changes | High |
| A2 | WooCommerce 10.7.0 supports full block checkout for all payment gateways | Medium — `/woocommerce/` PHP override path is the fallback | Medium |
| A3 | WooCommerce 10.7.0 maintains the `woocommerce_*` action/filter hook API | High — entire PHP WooCommerce integration layer depends on this | High |
| A4 | WordPress 7.x Block Bindings API is stable and allows PHP-sourced content in template parts | Medium — alternative `render_block` filter approach is documented | Medium |
| A5 | `@wordpress/interactivity` API is stable in WordPress 7.x | Low — affects view.js for custom blocks; alternative vanilla JS approach exists | High |

### 27.2 Open Decisions

| # | Decision Needed | Options | Current Stance |
|---|---|---|---|
| D1 | Wishlist functionality | (a) Native WooCommerce wishlists, (b) third-party plugin, (c) custom implementation | Defer to Phase 2 — assume WooCommerce 10.7.0 includes native wishlist blocks |
| D2 | Recently Viewed Products | (a) WooCommerce native block, (b) custom JS + localStorage | Defer to Phase 2 — prefer WooCommerce native if available |
| D3 | Product Comparison | (a) Plugin-based, (b) custom block | Out of scope for v1.0; plugin-compatible by design |
| D4 | Advanced product filtering (AJAX) | (a) WooCommerce Blocks filters, (b) custom AJAX filter | Default to WooCommerce Blocks filters; custom only if blocks insufficient |
| D5 | Carousel/slider library | Swiper selected (Section 21.2) | Confirmed — Swiper 11.x |
| D6 | Inline SVG vs `<img>` for icons | (a) Inline SVG via PHP helper, (b) `<img>` with SVG src, (c) CSS mask-image | **Undecided** — Inline SVG is accessible but increases DOM size. Decision deferred to Phase 1 implementation |
| D7 | Product quickview implementation | Modal-based using `a11y-dialog` | Confirmed |

### 27.3 Design System Assumptions

| # | Assumption |
|---|---|
| DS1 | The primary font family is undecided — a placeholder font stack is used in `theme.json`. Font selection is a brand decision outside this specification's scope. |
| DS2 | Specific hex values for the color palette are undecided — semantic names and roles are defined (Section 7.1) but actual brand colors are a brand decision. |
| DS3 | The WooCommerce "accent" color (sale badges, etc.) defaults to `accent` token from the palette, which defaults to a warm red. This should be confirmed with brand guidelines. |

---

## 28. Change Log

| Version | Date | Author | Summary |
|---|---|---|---|
| 1.1 | 2026-05-24 | Consistency Review | C1: Fixed §25.2 Quick View block (countdown-timer → product-card-enhanced). C2: Fixed §17.2 critical CSS mechanism (wp_add_inline_style → wp_head priority 1). S1: Fixed §4 template parts structure (nested → flat, matches §5 canonical). S2: Fixed §22.4 PHP naming (removed conflicting class- prefix row, all classes use PSR-4 ClassName.php). S3: Fixed §13.1 breakpoints (theme.json has no breakpoints key → wp_localize_script via PHP constants). S4: Fixed §21.2 WordPress packages (moved @wordpress/* to devDependencies, added externals rule). M1: Aligned pattern counts across §4/§5/§25.3 (added testimonials-carousel, cta-urgency, content-team-grid, woo-rating to §5; fixed §25.3 counts). M2: Fixed §3.4 CSS entry point list (added editor). M3: Added admin.scss entry point to §5. |
| 1.0 | 2025-05-24 | Architecture Design Phase | Initial specification — complete architecture for Torongo v1.0 development |

---

*End of Torongo Master Specification v1.1*

---

> **How to use this document:**  
> - Before writing any code for this theme, re-read the relevant section of this document.  
> - When a decision is not covered here, make an explicit decision and add it to Section 27 or the relevant section.  
> - When a decision changes, update this document and increment the version.  
> - This document is version-controlled in the project's git repository alongside the code.
