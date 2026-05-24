# Torongo — Implementation Structure Reference

**Version:** 1.0  
**Date:** 2026-05-24  
**Spec Reference:** torongo-spec.md v1.1  
**Status:** Implementation Reference — Do Not Override Spec Decisions  

---

> This document is the **implementation map** for the Torongo WordPress theme.  
> It translates the architecture decisions in `torongo-spec.md` into a concrete file reference.  
> **It does not replace or override `torongo-spec.md`.**  
> If any entry here conflicts with the spec, the spec takes precedence and this document must be corrected.

---

## Current Project State

| File | Status |
|---|---|
| `torongo-spec.md` | **EXISTS** — Master specification v1.1 |
| `torongo-structure.md` | **EXISTS** — This document |
| All theme source files | **PLANNED** — Approved design, not yet created on disk |

All files documented below reflect the approved design from Phase 1 (architecture) and Phase 2 (folder structure). No theme source files exist on disk yet. When a file is created, its status should be tracked in a separate implementation log.

---

## Table of Contents

1. [Project Structure Overview](#1-project-structure-overview)
2. [Complete Folder Tree](#2-complete-folder-tree)
3. [File Purpose Map](#3-file-purpose-map)
   - [3.1 Root — Configuration and Bootstrap](#31-root--configuration-and-bootstrap)
   - [3.2 PHP — Core Classes](#32-php--core-classes)
   - [3.3 PHP — WooCommerce Classes](#33-php--woocommerce-classes)
   - [3.4 PHP — Block Classes](#34-php--block-classes)
   - [3.5 PHP — SEO and Accessibility Classes](#35-php--seo-and-accessibility-classes)
   - [3.6 PHP — Helper Functions](#36-php--helper-functions)
   - [3.7 FSE Block Templates](#37-fse-block-templates)
   - [3.8 FSE Template Parts](#38-fse-template-parts)
   - [3.9 Block Patterns](#39-block-patterns)
   - [3.10 SCSS Source — Entry Points](#310-scss-source--entry-points)
   - [3.11 SCSS Source — Partials](#311-scss-source--partials)
   - [3.12 JavaScript Source — Frontend Modules](#312-javascript-source--frontend-modules)
   - [3.13 JavaScript Source — Custom Blocks](#313-javascript-source--custom-blocks)
   - [3.14 Compiled Assets](#314-compiled-assets)
   - [3.15 WooCommerce PHP Template Overrides](#315-woocommerce-php-template-overrides)
   - [3.16 Languages](#316-languages)
   - [3.17 Tests](#317-tests)
4. [Template Map](#4-template-map)
   - [4.1 WordPress FSE Template Hierarchy](#41-wordpress-fse-template-hierarchy)
   - [4.2 Template Composition](#42-template-composition)
   - [4.3 Template Part Usage Matrix](#43-template-part-usage-matrix)
   - [4.4 WooCommerce Template Relationships](#44-woocommerce-template-relationships)
5. [Asset Map](#5-asset-map)
   - [5.1 CSS Pipeline](#51-css-pipeline)
   - [5.2 JavaScript Pipeline](#52-javascript-pipeline)
   - [5.3 Image and Icon Organization](#53-image-and-icon-organization)
   - [5.4 Font Strategy](#54-font-strategy)
6. [Pre-Finish Review](#6-pre-finish-review)

---

## 1. Project Structure Overview

### 1.1 Project Purpose

Torongo is a production-ready, premium eCommerce WordPress theme. It is built as a **Full Site Editing (FSE) Block Theme** targeting WooCommerce stores. The theme is designed around four principals: performance by default, accessibility compliance (WCAG 2.2 AA), full Site Editor customization without code, and clean extensibility for developers via a documented hooks/filters API.

### 1.2 Root Organization

The project root contains two levels:

```
D:\AI Content\Torongo\          ← Project root (this workspace)
└── torongo/                    ← Theme directory (deployed to wp-content/themes/torongo/)
    ├── All theme files
    └── ...
```

**Important:** This workspace is the development environment. The `torongo/` subfolder is what gets deployed to WordPress. In practice, the entire workspace IS the `torongo/` folder — the working directory `D:\AI Content\Torongo` maps directly to the WordPress theme folder `wp-content/themes/torongo/`.

### 1.3 Relationship Between Root Files and Theme Behavior

| Root File | WordPress Interaction |
|---|---|
| `style.css` | WordPress reads this for theme registration metadata (`Theme Name`, `Version`, `Text Domain`) |
| `functions.php` | WordPress loads this first on every request — it bootstraps all PHP classes |
| `index.php` | WordPress requires this to exist as a fallback template |
| `theme.json` | WordPress reads this to generate CSS Custom Properties and populate the Site Editor |
| `screenshot.png` | WordPress displays this in Appearance → Themes |
| `templates/` | WordPress FSE reads these as page templates |
| `parts/` | WordPress FSE reads these as template parts in the Site Editor |
| `patterns/` | WordPress auto-registers all `.php` files as block patterns |

---

## 2. Complete Folder Tree

The tree below is the complete approved structure from spec §5 (canonical) expanded with all implied files documented in Phase 2. Every file has an inline annotation.

```
torongo/
│
│  ── ROOT — Configuration & Bootstrap ─────────────────
│
├── .editorconfig                         # Editor config: indent style, charset, line endings
├── .eslintrc.json                        # ESLint: extends @wordpress/eslint-plugin (§23.2)
├── .gitignore                            # Ignores: build/, node_modules/, *.map, vendor/
├── .phpcs.xml                            # PHPCS: WordPress-Core + WordPress-Security + PHPCompatibilityWP (§18.10)
├── .prettierrc                           # Prettier: JS/JSON/SCSS formatting rules (§23.2)
├── .stylelintrc.json                     # Stylelint: stylelint-config-wordpress + BEM enforcement (§23.3)
├── composer.json                         # PSR-4: Torongo\ → inc/classes/. No runtime PHP packages (§21.1)
├── composer.lock                         # Composer dependency lockfile
├── functions.php                         # Bootstrap only. Max 50 lines. Loads autoloader, inits Setup (§6.1)
├── index.php                             # WordPress required fallback: "Silence is golden." (§6.1)
├── package.json                          # npm scripts, dependencies, devDependencies (§21.2)
├── package-lock.json                     # npm lockfile
├── phpstan.neon                          # PHPStan level 5 static analysis config (§23.1)
├── phpunit.xml.dist                      # PHPUnit config for tests/Unit/ (§25.5)
├── readme.txt                            # WordPress theme documentation (production standard)
├── screenshot.png                        # 1200×900px homepage screenshot (§6.1)
├── style.css                             # Theme header comment + minimal CSS reset (§6.1)
├── theme.json                            # Design system: colors, type, spacing, layout (§6.2)
└── webpack.config.js                     # Webpack extending @wordpress/scripts (§2.1)
│
│  ── ASSETS — Production-compiled (git-tracked) ───────
│
├── assets/
│   ├── css/
│   │   ├── admin.css                     # Compiled: src/scss/admin.scss (§M3)
│   │   ├── editor.css                    # Compiled: src/scss/editor.scss
│   │   ├── global.css                    # Compiled: src/scss/global.scss
│   │   ├── rtl.css                       # Compiled: rtlcss from global.css (§20.6)
│   │   ├── woocommerce.css               # Compiled: src/scss/woocommerce.scss
│   │   └── blocks/                       # Per-custom-block CSS, loaded conditionally (§17.2)
│   │       ├── countdown-timer.css
│   │       ├── mega-menu.css
│   │       ├── product-card-enhanced.css
│   │       └── trust-badges.css
│   │
│   ├── fonts/                            # Self-hosted WOFF2 only — no Google Fonts (§17.5)
│   │   ├── body/                         # Body typeface — DS1: font family TBD
│   │   │   ├── [name]-400.woff2
│   │   │   ├── [name]-500.woff2
│   │   │   ├── [name]-600.woff2
│   │   │   └── [name]-700.woff2
│   │   ├── heading/                      # Heading/display typeface — DS1: font family TBD
│   │   │   ├── [name]-400.woff2
│   │   │   ├── [name]-500.woff2
│   │   │   ├── [name]-600.woff2
│   │   │   └── [name]-700.woff2
│   │   └── mono/                         # Monospace typeface — DS1: font family TBD
│   │       └── [name]-400.woff2
│   │
│   ├── images/
│   │   ├── icons/                        # Theme SVG icon set (delivery method: D6 open)
│   │   │   ├── account.svg
│   │   │   ├── cart.svg
│   │   │   ├── check.svg
│   │   │   ├── chevron-down.svg
│   │   │   ├── chevron-left.svg
│   │   │   ├── chevron-right.svg
│   │   │   ├── close.svg
│   │   │   ├── filter.svg
│   │   │   ├── heart.svg
│   │   │   ├── menu.svg
│   │   │   ├── minus.svg
│   │   │   ├── plus.svg
│   │   │   ├── search.svg
│   │   │   ├── share.svg
│   │   │   ├── star-empty.svg
│   │   │   ├── star-half.svg
│   │   │   └── star.svg
│   │   └── placeholders/
│   │       └── product-placeholder.svg   # SVG for products without images (§17.4)
│   │
│   └── js/
│       ├── admin.js                      # Compiled: src/js/admin/index.js
│       ├── frontend.js                   # Compiled: src/js/frontend/index.js
│       └── blocks/
│           ├── countdown-timer/
│           │   ├── block.json            # Copied from src — used by register_block_type()
│           │   ├── index.js              # Compiled block editor JS (edit + save)
│           │   └── view.js               # Compiled frontend interactivity
│           ├── mega-menu/
│           │   ├── block.json
│           │   ├── index.js
│           │   └── view.js
│           ├── product-card-enhanced/
│           │   ├── block.json
│           │   ├── index.js
│           │   └── view.js
│           └── trust-badges/
│               ├── block.json
│               └── index.js              # Static block — no view.js
│
│  ── BUILD — Webpack artifacts (git-ignored) ──────────
│
├── build/                                # Populated by: npm run build. Never committed.
│
│  ── INC — PHP Business Logic ─────────────────────────
│
├── inc/
│   ├── classes/
│   │   ├── Accessibility/
│   │   │   └── Manager.php               # Skip links, ARIA, focus management (§15.2)
│   │   ├── Blocks/
│   │   │   ├── Manager.php               # register_block_type() for 4 custom blocks
│   │   │   ├── PatternManager.php        # register_block_pattern() + pattern categories
│   │   │   └── StylesManager.php         # register_block_style() for native block variants
│   │   ├── Core/
│   │   │   ├── Assets.php                # Enqueue manager: scripts, styles, font preloads, critical CSS
│   │   │   ├── Setup.php                 # Theme init: hooks all Managers. Entry point from functions.php
│   │   │   └── ThemeSupport.php          # add_theme_support() declarations incl. WooCommerce
│   │   ├── SEO/
│   │   │   └── Manager.php               # Schema.org JSON-LD, OG tags, Twitter cards
│   │   └── WooCommerce/
│   │       ├── CartCheckout.php          # Cart/checkout hook integrations
│   │       ├── Manager.php               # WooCommerce coordinator: emails, breadcrumbs, version check
│   │       ├── ProductDisplay.php        # Sale badges, quick-view triggers, product card classes
│   │       └── Schema.php                # Product JSON-LD structured data
│   │
│   └── helpers/
│       ├── template-helpers.php          # torongo_get_breadcrumbs() + TORONGO_BP_* constants
│       └── woocommerce-helpers.php       # WooCommerce utility functions
│
│  ── LANGUAGES — Translation ──────────────────────────
│
├── languages/
│   └── torongo.pot                       # POT template — generated via: wp i18n make-pot
│
│  ── PARTS — FSE Template Parts ───────────────────────
│
├── parts/
│   ├── breadcrumbs.html                  # BreadcrumbList block markup
│   ├── footer.html                       # Default: 4-col + trust strip + bottom bar
│   ├── footer-minimal.html               # Minimal: logo + copyright + legal links
│   ├── footer-shop.html                  # Shop: newsletter + 3-col + trust badges
│   ├── header.html                       # Default: top-bar + logo + nav + actions
│   ├── header-minimal.html               # Minimal: logo + minimal links only
│   ├── header-sticky.html                # Sticky: JS adds is-sticky/is-scrolled classes
│   ├── pagination.html                   # Archive pagination block markup
│   └── post-meta.html                    # Post date, author, categories
│
│  ── PATTERNS — Block Patterns ────────────────────────
│
├── patterns/
│   ├── categories-banner.php             # Category banner-list layout
│   ├── categories-showcase.php           # Category showcase grid tiles
│   ├── content-about.php                 # Brand story / about section
│   ├── content-blog-grid.php             # Recent posts grid section
│   ├── content-team-grid.php             # Team members grid section
│   ├── cta-newsletter.php                # Email newsletter signup
│   ├── cta-promotional.php               # Full-width promotional/sale banner
│   ├── cta-urgency.php                   # Urgency CTA with countdown
│   ├── hero-full.php                     # Full-width hero: background image + headline + dual CTA
│   ├── hero-minimal.php                  # Centered minimal hero: heading + subheading + CTA
│   ├── hero-split.php                    # 50/50 split: content left, image right
│   ├── products-carousel.php             # Horizontal Swiper product carousel
│   ├── products-featured.php             # Large-imagery featured products
│   ├── products-grid-3.php               # 3-column WooCommerce Products block grid
│   ├── products-grid-4.php               # 4-column WooCommerce Products block grid
│   ├── testimonials-carousel.php         # Auto-rotating testimonials carousel
│   ├── testimonials-grid.php             # Multi-testimonial grid layout
│   ├── testimonials-single.php           # Single large testimonial with avatar
│   ├── woo-product-card.php              # Standalone WooCommerce product card
│   ├── woo-rating.php                    # Star rating display pattern
│   └── woo-sale-banner.php               # WooCommerce sale announcement banner
│
│  ── SRC — Source Files (uncompiled) ──────────────────
│
├── src/
│   ├── js/
│   │   ├── admin/
│   │   │   └── index.js                  # Admin scripts entry
│   │   ├── blocks/
│   │   │   ├── countdown-timer/          # torongo/countdown-timer
│   │   │   │   ├── block.json
│   │   │   │   ├── edit.js
│   │   │   │   ├── index.js
│   │   │   │   ├── save.js
│   │   │   │   └── view.js               # @wordpress/interactivity frontend
│   │   │   ├── mega-menu/                # torongo/mega-menu
│   │   │   │   ├── block.json
│   │   │   │   ├── edit.js
│   │   │   │   ├── index.js
│   │   │   │   ├── save.js
│   │   │   │   └── view.js               # Keyboard nav, Escape key, focus trap
│   │   │   ├── product-card-enhanced/    # torongo/product-card-enhanced
│   │   │   │   ├── block.json
│   │   │   │   ├── edit.js
│   │   │   │   ├── index.js
│   │   │   │   ├── save.js
│   │   │   │   └── view.js               # Quick-view trigger, hover state management
│   │   │   └── trust-badges/             # torongo/trust-badges
│   │   │       ├── block.json
│   │   │       ├── edit.js
│   │   │       ├── index.js
│   │   │       └── save.js               # Static block — no view.js needed
│   │   └── frontend/
│   │       ├── index.js                  # Entry: imports and initializes all frontend modules
│   │       ├── infinite-scroll.js        # Optional IntersectionObserver-based infinite scroll
│   │       ├── mobile-menu.js            # Off-canvas mobile nav with focus-trap
│   │       ├── navigation.js             # Keyboard nav, ARIA expanded state management
│   │       ├── product-gallery.js        # Swiper product image gallery (PDP)
│   │       ├── quick-view.js             # a11y-dialog quick view modal controller
│   │       ├── skip-link.js              # Skip link: visible on focus behavior
│   │       └── sticky-header.js          # IntersectionObserver sticky + is-scrolled classes
│   │
│   └── scss/
│       ├── base/
│       │   ├── _reset.scss               # Modern CSS reset
│       │   ├── _root.scss                # --torongo-* computed custom properties
│       │   └── _typography.scss          # Base typographic rules: body, headings, links
│       ├── blocks/
│       │   ├── _button.scss              # core/button overrides
│       │   ├── _columns.scss             # core/columns overrides
│       │   ├── _countdown-timer.scss     # torongo/countdown-timer styles
│       │   ├── _cover.scss               # core/cover overrides
│       │   ├── _gallery.scss             # core/gallery overrides
│       │   ├── _group.scss               # core/group overrides
│       │   ├── _heading.scss             # core/heading overrides
│       │   ├── _image.scss               # core/image overrides
│       │   ├── _list.scss                # core/list overrides
│       │   ├── _media-text.scss          # core/media-text overrides
│       │   ├── _mega-menu.scss           # torongo/mega-menu styles
│       │   ├── _navigation.scss          # core/navigation overrides
│       │   ├── _paragraph.scss           # core/paragraph overrides
│       │   ├── _post-content.scss        # core/post-content overrides
│       │   ├── _post-featured-image.scss # core/post-featured-image overrides
│       │   ├── _post-title.scss          # core/post-title overrides
│       │   ├── _product-card-enhanced.scss # torongo/product-card-enhanced styles
│       │   ├── _pullquote.scss           # core/pullquote overrides
│       │   ├── _query.scss               # core/query overrides
│       │   ├── _quote.scss               # core/quote overrides
│       │   ├── _search.scss              # core/search overrides
│       │   ├── _separator.scss           # core/separator overrides
│       │   ├── _site-logo.scss           # core/site-logo overrides
│       │   ├── _social-links.scss        # core/social-links overrides
│       │   ├── _table.scss               # core/table overrides
│       │   └── _trust-badges.scss        # torongo/trust-badges styles
│       ├── components/
│       │   ├── _breadcrumbs.scss         # Breadcrumb trail component
│       │   ├── _footer.scss              # Footer layout: columns, trust strip, bottom bar
│       │   ├── _header.scss              # Header layout: rows, actions, sticky states
│       │   ├── _mobile-menu.scss         # Off-canvas mobile menu panel
│       │   ├── _pagination.scss          # Pagination: numbers, prev/next, active
│       │   ├── _quick-view.scss          # Quick view modal overlay + panel
│       │   └── _skip-link.scss           # Skip link: visually hidden until focused
│       ├── utilities/
│       │   ├── _functions.scss           # px-to-rem, fluid-type SCSS functions
│       │   └── _mixins.scss              # mobile-only, tablet-up, desktop-up, breakpoint()
│       ├── woocommerce/
│       │   ├── _account.scss             # My Account pages
│       │   ├── _badges.scss              # Sale %, New, Hot overlay badges
│       │   ├── _cart.scss                # Cart block: table, totals, coupon
│       │   ├── _checkout.scss            # Checkout block: form, summary sidebar
│       │   ├── _filters.scss             # Shop filter sidebar
│       │   ├── _global.scss              # WooCommerce global overrides: notices, buttons
│       │   ├── _mini-cart.scss           # Mini Cart block in header
│       │   ├── _order-view.scss          # Order detail view
│       │   ├── _product-card.scss        # Product card loop item
│       │   └── _product-single.scss      # Single product PDP
│       ├── admin.scss                    # Entry → assets/css/admin.css
│       ├── editor.scss                   # Entry → assets/css/editor.css
│       ├── global.scss                   # Entry → assets/css/global.css
│       └── woocommerce.scss              # Entry → assets/css/woocommerce.css
│
│  ── TEMPLATES — FSE Block Templates ──────────────────
│
├── templates/
│   ├── 404.html                          # Not found: search suggestion + popular links
│   ├── archive.html                      # Blog archive: query loop + pagination
│   ├── archive-product.html              # WooCommerce shop: filters + grid + sort
│   ├── front-page.html                   # Homepage: 7-section pattern assembly
│   ├── index.html                        # WordPress fallback template
│   ├── page.html                         # Static page: title + post content
│   ├── page-no-title.html                # Landing page: no title, full-width content
│   ├── search.html                       # Search results: post + product query
│   ├── single.html                       # Blog post: content + meta + comments
│   ├── single-product.html               # WooCommerce PDP: gallery + summary + tabs
│   ├── taxonomy-product_cat.html         # Product category archive
│   └── taxonomy-product_tag.html         # Product tag archive
│
│  ── TESTS — PHPUnit ───────────────────────────────────
│
├── tests/
│   ├── bootstrap.php                     # PHPUnit bootstrap: loads WordPress + autoloader
│   └── Unit/
│       ├── Accessibility/
│       │   └── ManagerTest.php
│       ├── Blocks/
│       │   ├── ManagerTest.php
│       │   ├── PatternManagerTest.php
│       │   └── StylesManagerTest.php
│       ├── Core/
│       │   ├── AssetsTest.php
│       │   ├── SetupTest.php
│       │   └── ThemeSupportTest.php
│       ├── SEO/
│       │   └── ManagerTest.php
│       └── WooCommerce/
│           ├── CartCheckoutTest.php
│           ├── ManagerTest.php
│           ├── ProductDisplayTest.php
│           └── SchemaTest.php
│
│  ── WOOCOMMERCE — PHP Template Overrides ──────────────
│
└── woocommerce/
    ├── emails/
    │   ├── email-footer.php              # Branded email footer: links, unsubscribe
    │   └── email-header.php              # Branded email header: logo + colors
    └── myaccount/
        ├── my-account.php                # Account dashboard overview
        ├── orders.php                    # Order history list
        └── view-order.php                # Single order detail
```

---

## 3. File Purpose Map

### 3.1 Root — Configuration and Bootstrap

| Path | Purpose | Dependencies | Notes |
|---|---|---|---|
| `style.css` | Theme registration header required by WordPress. Contains `Theme Name`, `Version`, `Text Domain: torongo`, `Requires at least`, `Tested up to`. Also holds minimal CSS reset (not layout styles). | None | Must NOT contain layout or component styles. WordPress reads the header comment on every request. |
| `functions.php` | PHP bootstrap entry point. Requires Composer autoloader then instantiates `Torongo\Core\Setup` and calls `::init()`. Nothing else. | `vendor/autoload.php`, `inc/classes/Core/Setup.php` | Hard limit: 50 lines. All logic lives in class files. |
| `index.php` | WordPress-required empty fallback template. Contains only the PHP silence comment. | None | Never displays content. WordPress never loads it in an FSE theme with templates/. |
| `theme.json` | Canonical design token system. Single source of truth for all visual decisions: colors, typography, spacing, borders, shadows, layout dimensions, template part areas, custom templates. | None (WordPress reads directly) | Changing a value here propagates to all CSS Custom Properties automatically. All SCSS must consume these properties. No hardcoded values permitted. |
| `composer.json` | PSR-4 autoloading map: `Torongo\\` → `inc/classes/`. No runtime PHP packages declared. | None | `composer install` creates `vendor/autoload.php`. The vendor/ directory is git-ignored. |
| `webpack.config.js` | Extends `@wordpress/scripts` webpack config. Custom configuration: block source paths in `src/js/blocks/`, externals for all `@wordpress/*` packages, production minification, source maps in dev only. | `package.json`, `@wordpress/scripts` | All `@wordpress/*` packages must be declared as `externals` here to prevent bundling. |
| `package.json` | npm scripts (`build`, `start`, `lint:js`, `lint:css`, `lint:php`) and dependency declarations. `swiper`, `a11y-dialog`, `focus-trap` are `dependencies`. All `@wordpress/*` packages and linting tools are `devDependencies`. | None | See spec §21.2 for the split between `dependencies` and `devDependencies`. |
| `.phpcs.xml` | PHPCS ruleset: WordPress-Core, WordPress-Security, WordPress-Docs, PHPCompatibilityWP (PHP 8.2 target). | None | Run via `npm run lint:php`. Must produce zero errors before any commit. |
| `.eslintrc.json` | ESLint: extends `@wordpress/eslint-plugin`. ES modules, no `var`, arrow functions preferred. | None | Run via `npm run lint:js`. |
| `.stylelintrc.json` | Stylelint: `stylelint-config-wordpress` + `stylelint-selector-bem-pattern` for BEM enforcement. | None | Run via `npm run lint:css`. |
| `.prettierrc` | Prettier: consistent formatting for JS, JSON, SCSS files across editors. | None | Pairs with ESLint and Stylelint via `eslint-config-prettier`. |
| `phpstan.neon` | PHPStan level 5 configuration. Includes WordPress stubs. | None | Run via `vendor/bin/phpstan analyse`. Must pass before release. |
| `phpunit.xml.dist` | PHPUnit configuration: test suite points to `tests/Unit/`, bootstrap file at `tests/bootstrap.php`. | None | Copy to `phpunit.xml` locally if overrides needed. `.xml.dist` is committed. |
| `readme.txt` | Theme documentation in WordPress.org format: description, installation, changelog, FAQ. | None | Not a Markdown file — uses WordPress.org `.txt` format. |
| `screenshot.png` | Theme preview shown in Appearance → Themes. Represents the homepage. | None | Dimensions: 1200×900px. PNG or WebP. Shows assembled homepage from `front-page.html`. |

---

### 3.2 PHP — Core Classes

**Namespace:** `Torongo\Core`  
**Location:** `inc/classes/Core/`  
**All files:** `declare(strict_types=1)`, `defined('ABSPATH') || exit`, typed properties and return types.

| Path | Purpose | Dependencies | Notes |
|---|---|---|---|
| `inc/classes/Core/Setup.php` | Theme initialization hub. `Setup::init()` is called by `functions.php` and registers action/filter hooks for all Manager classes. Controls initialization order. | All other Manager classes, `template-helpers.php` | This is the only class instantiated directly by `functions.php`. All others are delegated from here. |
| `inc/classes/Core/Assets.php` | Script and style enqueue manager. Registers and enqueues all CSS and JS handles. Injects font `<link rel="preload">` tags. Outputs critical CSS via `wp_head` at priority 1. Calls `wp_localize_script` to inject `window.torongoData` including breakpoint constants. | `template-helpers.php` (for BP constants), all compiled `assets/` files | `wp_script_add_data( 'torongo-frontend', 'defer', true )` defers all frontend JS. Conditional WooCommerce script loading via `is_woocommerce()`. |
| `inc/classes/Core/ThemeSupport.php` | Declares all `add_theme_support()` calls. WooCommerce support: `woocommerce`, `wc-product-gallery-zoom`, `wc-product-gallery-lightbox`, `wc-product-gallery-slider`. WordPress support: `title-tag`, `post-thumbnails`, `html5`, `responsive-embeds`, `editor-styles`. | None | All theme support declarations live here and nowhere else. |

---

### 3.3 PHP — WooCommerce Classes

**Namespace:** `Torongo\WooCommerce`  
**Location:** `inc/classes/WooCommerce/`

| Path | Purpose | Dependencies | Notes |
|---|---|---|---|
| `inc/classes/WooCommerce/Manager.php` | WooCommerce integration coordinator. Handles: WooCommerce version check on `admin_notices`, breadcrumb filter, pagination filter, email style filter, HPOS compatibility declaration. | `woocommerce-helpers.php` | If WooCommerce minimum version is not met, outputs a persistent admin notice. HPOS compatibility declared via `FeaturesUtil::declare_compatibility()`. |
| `inc/classes/WooCommerce/ProductDisplay.php` | Product loop and card modifications. Hooks: `woocommerce_before_shop_loop_item_title` (sale badge), `woocommerce_after_shop_loop_item` (quick-view trigger). Filters: `woocommerce_product_loop_item_classes`, `woocommerce_loop_add_to_cart_args`. | None | Sale badge HTML uses `esc_html()` on all output. Quick-view trigger outputs a `<button>` with `.js-quick-view-trigger` class and `aria-label`. |
| `inc/classes/WooCommerce/CartCheckout.php` | Cart and checkout hook integrations. Hooks: `woocommerce_before_cart` (trust strip), `woocommerce_after_cart_table` (cross-sells), `woocommerce_cart_totals_before_order_total` (savings summary). | `woocommerce-helpers.php` | No `cart.php` or `checkout.php` template overrides. Block-based cart/checkout exclusively. |
| `inc/classes/WooCommerce/Schema.php` | Product JSON-LD structured data. Outputs `Product` schema hooked into `woocommerce_single_product_summary`. Also provides `ItemList` schema for shop/archive pages. | `Torongo\SEO\Manager` | Schema data array passes through `torongo_schema_data` filter before `json_encode()`. All output via `wp_json_encode()` inside `<script type="application/ld+json">`. |

---

### 3.4 PHP — Block Classes

**Namespace:** `Torongo\Blocks`  
**Location:** `inc/classes/Blocks/`

| Path | Purpose | Dependencies | Notes |
|---|---|---|---|
| `inc/classes/Blocks/Manager.php` | Registers all 4 custom blocks via `register_block_type()`. Points to `assets/js/blocks/{block-name}/` directory which contains `block.json`. Called on `init` hook. | `assets/js/blocks/*/block.json` | `register_block_type( get_template_directory() . '/assets/js/blocks/countdown-timer' )` — WordPress reads `block.json` from that path. |
| `inc/classes/Blocks/StylesManager.php` | Registers named style variations for native blocks via `register_block_style()`. Example: `core/button` gets `outline`, `ghost`, `underline` style variants. Called on `init` hook. | None | Style names follow `torongo-{name}` convention. Styles appear in the Block Styles panel in the editor. |
| `inc/classes/Blocks/PatternManager.php` | Registers block pattern categories: `torongo-hero`, `torongo-products`, `torongo-woocommerce`, `torongo-cta`, `torongo-testimonials`, `torongo-content`. Individual patterns in `/patterns/` are auto-registered by WordPress (no PHP call needed for PHP pattern files with headers). | `patterns/*.php` (auto-loaded by WP) | The `PatternManager` only registers the categories. WordPress automatically registers all PHP files in the `patterns/` directory as patterns. |

---

### 3.5 PHP — SEO and Accessibility Classes

**Namespace:** `Torongo\SEO`, `Torongo\Accessibility`  
**Location:** `inc/classes/SEO/`, `inc/classes/Accessibility/`

| Path | Purpose | Dependencies | Notes |
|---|---|---|---|
| `inc/classes/SEO/Manager.php` | Structured data and social tags. Outputs: `WebSite` (all pages), `Organization` (homepage/about), `BreadcrumbList` (archives/singles), `Product` (via `WooCommerce\Schema`), `ItemList` (shop). Conditionally outputs OG and Twitter meta when no SEO plugin detected. | `Torongo\WooCommerce\Schema` | Plugin detection: `function_exists('yoast_get_value')`, `function_exists('rank_math')`. If either exists, OG/Twitter output is suppressed. Schema data filtered via `torongo_schema_data`. |
| `inc/classes/Accessibility/Manager.php` | ARIA and accessibility enhancements. Injects skip link HTML via `wp_body_open`. Uses `render_block` filter to inject `torongo_before_header` and `torongo_after_header` action hooks adjacent to the header template part. Manages `aria-live` region for cart count updates. | None | Skip links are output as the very first content inside `<body>` via `wp_body_open` (priority 1). `render_block` filter targets the outermost `core/group` block of each template part. |

---

### 3.6 PHP — Helper Functions

**Location:** `inc/helpers/`  
**Loading:** Required by `Setup.php`, available globally after bootstrap.

| Path | Purpose | Dependencies | Notes |
|---|---|---|---|
| `inc/helpers/template-helpers.php` | Procedural utility functions for templates. Includes: `torongo_get_breadcrumbs( $args )`, `torongo_get_template_class( $classes )`. Also declares PHP breakpoint constants: `TORONGO_BP_MOBILE_SM = 320`, `TORONGO_BP_MOBILE = 480`, `TORONGO_BP_TABLET = 768`, `TORONGO_BP_TABLET_LG = 1024`, `TORONGO_BP_DESKTOP = 1200`, `TORONGO_BP_WIDE = 1536`. These constants are consumed by `Assets.php` for `wp_localize_script`. | None | All functions prefixed `torongo_`. All constants prefixed `TORONGO_BP_`. `defined('ABSPATH') || exit` at top. |
| `inc/helpers/woocommerce-helpers.php` | WooCommerce utility functions. Includes: `torongo_get_product_badge_html( $product )`, `torongo_is_sale_percentage_significant( $product )`, `torongo_format_price_range( $min, $max )`. | None | All functions check `function_exists('WC')` before WooCommerce API calls to avoid fatal errors if WooCommerce is deactivated. |

---

### 3.7 FSE Block Templates

All files in `templates/` are WordPress block templates (HTML containing block markup). Every template follows the pattern: `[header part] → [main content] → [footer part]`.

| Path | WordPress Template Slug | Default Header Part | Default Footer Part | Content Source |
|---|---|---|---|---|
| `templates/index.html` | `index` | `header` | `footer` | WordPress Query Loop (fallback) |
| `templates/front-page.html` | `front-page` | `header` | `footer` | 7 block patterns assembled inline |
| `templates/page.html` | `page` | `header` | `footer` | Post Title block + Post Content block |
| `templates/page-no-title.html` | `page-no-title` | `header-minimal` | `footer-minimal` | Post Content block only (full-width) |
| `templates/single.html` | `single` | `header` | `footer` | Post Content + post-meta part + Comments |
| `templates/single-product.html` | `single-product` | `header` | `footer` | WooCommerce Product blocks (gallery 60% / summary 40%) |
| `templates/archive.html` | `archive` | `header` | `footer` | Archive Title + Query Loop (blog posts) + pagination part |
| `templates/archive-product.html` | `archive-product` | `header` | `footer` | WooCommerce Product query + filter sidebar + sort |
| `templates/taxonomy-product_cat.html` | `taxonomy-product_cat` | `header` | `footer` | Category banner + WooCommerce Product query |
| `templates/taxonomy-product_tag.html` | `taxonomy-product_tag` | `header` | `footer` | WooCommerce Product query (tag-filtered) |
| `templates/search.html` | `search` | `header` | `footer` | Search Results block (posts + products combined) |
| `templates/404.html` | `404` | `header` | `footer` | Static error content + search block + popular links |

---

### 3.8 FSE Template Parts

All files in `parts/` are WordPress block template parts. Registered in `theme.json` under `templateParts` with areas (`header`, `footer`, `general`).

| Path | Area | Used By | Key Blocks |
|---|---|---|---|
| `parts/header.html` | `header` | Most templates | Site Logo, Navigation, WooCommerce Mini Cart Block, Search |
| `parts/header-minimal.html` | `header` | `page-no-title.html` | Site Logo, minimal Navigation only |
| `parts/header-sticky.html` | `header` | Optional user-override | Same as header.html; sticky behavior via JS + `is-sticky`/`is-scrolled` CSS classes |
| `parts/footer.html` | `footer` | Most templates | 4-column Group, Navigation, Social Icons, payment icons |
| `parts/footer-minimal.html` | `footer` | `page-no-title.html` | Site Logo, copyright Paragraph, legal Navigation |
| `parts/footer-shop.html` | `footer` | Optional user-override | Newsletter form, 3-column Group, trust badges block |
| `parts/breadcrumbs.html` | `general` | `single.html`, `archive.html`, WooCommerce templates | Paragraph block (dynamic breadcrumb via Block Bindings or PHP) |
| `parts/post-meta.html` | `general` | `single.html` | Post Date, Post Author, Post Terms blocks |
| `parts/pagination.html` | `general` | `archive.html`, `archive-product.html` | Query Pagination block |

---

### 3.9 Block Patterns

All 21 pattern files in `patterns/` are PHP files. WordPress auto-registers them. Each file declares a pattern header comment (`Title`, `Slug`, `Categories`, `Block Types`).

| Path | Category | Used On | Key Blocks |
|---|---|---|---|
| `patterns/hero-full.php` | `torongo-hero` | `front-page.html` | Cover (full-width), Heading, Paragraph, Buttons |
| `patterns/hero-split.php` | `torongo-hero` | `front-page.html` | Columns (50/50), Image, Heading, Buttons |
| `patterns/hero-minimal.php` | `torongo-hero` | `page-no-title.html` | Group (centered), Heading, Paragraph, Button |
| `patterns/products-grid-3.php` | `torongo-products` | Homepage sections | WooCommerce Products block (3 cols) |
| `patterns/products-grid-4.php` | `torongo-products` | Homepage, best-sellers | WooCommerce Products block (4 cols) |
| `patterns/products-featured.php` | `torongo-products` | `front-page.html` | WooCommerce Products block (featured) |
| `patterns/products-carousel.php` | `torongo-products` | Homepage | WooCommerce Products block + Swiper init |
| `patterns/categories-showcase.php` | `torongo-products` | `front-page.html` | Columns, Cover blocks (one per category) |
| `patterns/categories-banner.php` | `torongo-products` | Category landing | Media & Text blocks stacked |
| `patterns/testimonials-single.php` | `torongo-content` | Any page | Group, Quote, Image |
| `patterns/testimonials-grid.php` | `torongo-content` | `front-page.html` | Columns, Quote blocks |
| `patterns/testimonials-carousel.php` | `torongo-content` | Homepage | Group with Swiper carousel markup |
| `patterns/cta-newsletter.php` | `torongo-cta` | `front-page.html` | Group, Heading, Paragraph, Form block |
| `patterns/cta-promotional.php` | `torongo-cta` | `front-page.html` | Cover (full-width), Heading, Button (sale CTA) |
| `patterns/cta-urgency.php` | `torongo-cta` | Product pages, homepage | Group, Heading, `torongo/countdown-timer` block |
| `patterns/content-about.php` | `torongo-content` | `front-page.html` | Media & Text, Heading, Paragraph, Button |
| `patterns/content-blog-grid.php` | `torongo-content` | Homepage | Query Loop (3 posts), Post Featured Image, Post Title |
| `patterns/content-team-grid.php` | `torongo-content` | About page | Columns, Image, Heading, Paragraph per person |
| `patterns/woo-product-card.php` | `torongo-woocommerce` | Shop patterns | WooCommerce Product block (single) |
| `patterns/woo-rating.php` | `torongo-woocommerce` | Product sections | WooCommerce Product Rating block |
| `patterns/woo-sale-banner.php` | `torongo-woocommerce` | Shop page, homepage | Group, Heading, WooCommerce Products block (on-sale) |

---

### 3.10 SCSS Source — Entry Points

Four SCSS entry points compile to four CSS outputs. Each imports partials via `@use` or `@forward`.

| Source | Compiled Output | Loaded By | Scope |
|---|---|---|---|
| `src/scss/global.scss` | `assets/css/global.css` | `Assets::register()` on `wp_enqueue_styles` | Frontend: all pages |
| `src/scss/editor.scss` | `assets/css/editor.css` | `Assets::register()` on `enqueue_block_editor_assets` | Block editor only |
| `src/scss/woocommerce.scss` | `assets/css/woocommerce.css` | `Assets::register()` on WooCommerce pages only | WooCommerce storefront |
| `src/scss/admin.scss` | `assets/css/admin.css` | `Assets::register()` on `admin_enqueue_scripts` | WordPress admin area |

---

### 3.11 SCSS Source — Partials

Partials are never compiled directly — they are imported by entry points. File names always begin with `_`.

**`src/scss/base/`**

| File | Purpose | Key Rule |
|---|---|---|
| `_reset.scss` | Modern CSS reset targeting semantic HTML elements and layout defaults | No CSS framework resets. Custom minimal reset only. |
| `_root.scss` | Declares `--torongo-*` computed CSS custom properties not auto-generated by `theme.json` | All values reference `var(--wp--preset--*)`. No hardcoded values. |
| `_typography.scss` | Base `body`, `h1`–`h6`, `p`, `a`, `strong`, `em`, `code` styles | Font sizes always via `var(--wp--preset--font-size--*)`. Never `px` or `rem`. |

**`src/scss/blocks/`** — One file per block, overriding WordPress defaults:

| File | Block |
|---|---|
| `_button.scss` | `core/button` |
| `_columns.scss` | `core/columns` |
| `_countdown-timer.scss` | `torongo/countdown-timer` |
| `_cover.scss` | `core/cover` |
| `_gallery.scss` | `core/gallery` |
| `_group.scss` | `core/group` |
| `_heading.scss` | `core/heading` |
| `_image.scss` | `core/image` |
| `_list.scss` | `core/list` |
| `_media-text.scss` | `core/media-text` |
| `_mega-menu.scss` | `torongo/mega-menu` |
| `_navigation.scss` | `core/navigation` |
| `_paragraph.scss` | `core/paragraph` |
| `_post-content.scss` | `core/post-content` |
| `_post-featured-image.scss` | `core/post-featured-image` |
| `_post-title.scss` | `core/post-title` |
| `_product-card-enhanced.scss` | `torongo/product-card-enhanced` |
| `_pullquote.scss` | `core/pullquote` |
| `_query.scss` | `core/query` |
| `_quote.scss` | `core/quote` |
| `_search.scss` | `core/search` |
| `_separator.scss` | `core/separator` |
| `_site-logo.scss` | `core/site-logo` |
| `_social-links.scss` | `core/social-links` |
| `_table.scss` | `core/table` |
| `_trust-badges.scss` | `torongo/trust-badges` |

**`src/scss/components/`** — Structural component styles:

| File | Purpose |
|---|---|
| `_breadcrumbs.scss` | Breadcrumb trail: separator, link, current item styles |
| `_footer.scss` | Footer layout: column grid, trust strip, bottom bar, copyright |
| `_header.scss` | Header layout: top-bar, main row, actions. Sticky states: `.is-sticky`, `.is-scrolled` |
| `_mobile-menu.scss` | Off-canvas panel: transform animation, overlay, open/close states |
| `_pagination.scss` | Numbered pagination, prev/next, active state |
| `_quick-view.scss` | Quick view overlay backdrop, modal panel, close button |
| `_skip-link.scss` | Visually hidden by default, visible on `:focus` with outline |

**`src/scss/woocommerce/`** — WooCommerce-specific styles (imported by `woocommerce.scss`):

| File | WooCommerce Surface |
|---|---|
| `_global.scss` | Notices, form elements, WC button overrides, WC global resets |
| `_product-card.scss` | Product loop item: image ratio, badge overlay, hover states, add-to-cart |
| `_product-single.scss` | PDP: gallery column, summary column, tabs, related products |
| `_cart.scss` | Cart block: table columns, quantity input, totals, coupon form |
| `_checkout.scss` | Checkout block: form fields, summary sidebar, express checkout |
| `_mini-cart.scss` | Mini Cart block: icon, count badge, dropdown panel |
| `_filters.scss` | Filter sidebar: price range slider, attribute checkboxes, rating filter |
| `_badges.scss` | Sale %, New, Hot badge overlays on product cards |
| `_account.scss` | My Account: dashboard tiles, address forms, order table |
| `_order-view.scss` | Single order detail: product table, status timeline, actions |

**`src/scss/utilities/`**

| File | Purpose |
|---|---|
| `_mixins.scss` | Breakpoint mixins: `mobile-only`, `tablet-up`, `desktop-up`, `breakpoint($name)`. Breakpoint values sourced from TORONGO_BP_* constants (documented in comments). |
| `_functions.scss` | SCSS functions: `px-to-rem($px)`, `fluid-size($min, $max, $min-vw, $max-vw)` |

---

### 3.12 JavaScript Source — Frontend Modules

All modules are ES2022+. Entry point is `src/js/frontend/index.js` which imports and initializes each module.

| Path | Purpose | Key API | Dependencies |
|---|---|---|---|
| `src/js/frontend/index.js` | Bundle entry. Imports all frontend modules. Initializes each via `wp.domReady`. | `wp.domReady` | All modules below |
| `src/js/frontend/navigation.js` | Keyboard navigation for dropdowns and mega menus. Manages `aria-expanded` state on toggle elements. | `querySelectorAll`, `addEventListener` | None |
| `src/js/frontend/skip-link.js` | Forces focus to the target element of a skip link when activated. Adds `tabindex="-1"` to `#main-content` if not already focusable. | `addEventListener('click')` | None |
| `src/js/frontend/sticky-header.js` | Adds/removes `is-sticky` and `is-scrolled` CSS classes on the header wrapper. Uses `IntersectionObserver` on a sentinel element above the header — not a `scroll` event listener. | `IntersectionObserver` | None |
| `src/js/frontend/mobile-menu.js` | Controls off-canvas mobile menu. Opens/closes panel, manages `aria-expanded`, traps focus inside open panel. | `focus-trap` (npm package) | `focus-trap` |
| `src/js/frontend/product-gallery.js` | Swiper-based product image gallery on the PDP. Thumbnail strip as navigation. Touch/swipe enabled. | `Swiper` (npm package) | `swiper` |
| `src/js/frontend/quick-view.js` | Quick view modal controller. Triggered by `.js-quick-view-trigger` buttons. Opens `a11y-dialog` modal, fetches product data, renders content. | `a11y-dialog` (npm package), `fetch` | `a11y-dialog` |
| `src/js/frontend/infinite-scroll.js` | Optional infinite scroll for shop/archive pages. Uses `IntersectionObserver` on the pagination element. Fetches next page, appends product grid items. Disabled by default (requires opt-in class on archive template). | `IntersectionObserver`, `fetch` | None |

---

### 3.13 JavaScript Source — Custom Blocks

Four custom blocks under `src/js/blocks/`. Each follows the standard `@wordpress/scripts` block pattern.

| Block | Files | view.js? | Purpose |
|---|---|---|---|
| `countdown-timer/` | `block.json`, `index.js`, `edit.js`, `save.js`, `view.js` | Yes | Countdown to a target date/time. `view.js` handles live countdown via `@wordpress/interactivity`. |
| `mega-menu/` | `block.json`, `index.js`, `edit.js`, `save.js`, `view.js` | Yes | Enhanced navigation with mega menu panels. `view.js` handles open/close, keyboard navigation (arrow keys, Escape), and focus trap. |
| `product-card-enhanced/` | `block.json`, `index.js`, `edit.js`, `save.js`, `view.js` | Yes | Enhanced product card with quick-view integration and hover states. `view.js` wires `.js-quick-view-trigger` to `quick-view.js` modal. |
| `trust-badges/` | `block.json`, `index.js`, `edit.js`, `save.js` | **No** | Static display of payment/trust icons. No frontend JS needed — pure HTML/CSS output. |

**`block.json` fields (all 4 blocks):**

```
name:         torongo/{block-name}
title:        Human-readable name
textdomain:   torongo
editorScript: file:./index.js
viewScript:   file:./view.js  (omitted for trust-badges)
style:        file:./style.css (compiled to assets/css/blocks/{name}.css)
editorStyle:  file:./editor.css
```

---

### 3.14 Compiled Assets

| Path | Source | Build Command | Enqueue Handle | Notes |
|---|---|---|---|---|
| `assets/css/global.css` | `src/scss/global.scss` | `npm run build` | `torongo-global` | Loaded on all frontend pages |
| `assets/css/editor.css` | `src/scss/editor.scss` | `npm run build` | `torongo-editor` | Block editor only |
| `assets/css/woocommerce.css` | `src/scss/woocommerce.scss` | `npm run build` | `torongo-woocommerce` | WooCommerce pages only |
| `assets/css/admin.css` | `src/scss/admin.scss` | `npm run build` | `torongo-admin` | WordPress admin only |
| `assets/css/rtl.css` | `assets/css/global.css` via rtlcss | `npm run build` | Auto-loaded by WordPress on RTL | WordPress auto-enqueues when `is_rtl()` |
| `assets/css/blocks/countdown-timer.css` | `src/js/blocks/countdown-timer/style.scss` | `npm run build` | Auto by `register_block_type()` | Loaded only when block is on page |
| `assets/css/blocks/mega-menu.css` | `src/js/blocks/mega-menu/style.scss` | `npm run build` | Auto by `register_block_type()` | Same as above |
| `assets/css/blocks/product-card-enhanced.css` | `src/js/blocks/product-card-enhanced/style.scss` | `npm run build` | Auto by `register_block_type()` | Same |
| `assets/css/blocks/trust-badges.css` | `src/js/blocks/trust-badges/style.scss` | `npm run build` | Auto by `register_block_type()` | Same |
| `assets/js/frontend.js` | `src/js/frontend/index.js` | `npm run build` | `torongo-frontend` | Deferred. `window.torongoData` injected via `wp_localize_script`. |
| `assets/js/admin.js` | `src/js/admin/index.js` | `npm run build` | `torongo-admin-script` | Admin only. Not deferred. |
| `assets/js/blocks/*/index.js` | `src/js/blocks/*/index.js` | `npm run build` | Auto by `register_block_type()` | Block editor JS (edit + save) |
| `assets/js/blocks/*/view.js` | `src/js/blocks/*/view.js` | `npm run build` | Auto by `register_block_type()` | Frontend interactivity, deferred |
| `assets/js/blocks/*/block.json` | `src/js/blocks/*/block.json` | Copied by webpack | n/a (read by PHP) | PHP `register_block_type()` reads from this path |

---

### 3.15 WooCommerce PHP Template Overrides

Located in `woocommerce/`. Used **only** for non-block surfaces where WooCommerce Blocks do not provide coverage. All other WooCommerce surfaces use block templates or the `woocommerce_*` hook API.

| Path | Overrides WooCommerce Template | Modification |
|---|---|---|
| `woocommerce/emails/email-header.php` | `templates/emails/email-header.php` | Injects brand logo (via `get_template_directory_uri()`), brand primary color via inline CSS |
| `woocommerce/emails/email-footer.php` | `templates/emails/email-footer.php` | Injects store links, unsubscribe notice, brand color footer bar |
| `woocommerce/myaccount/my-account.php` | `templates/myaccount/my-account.php` | Adds Torongo header context, wraps content in branded container |
| `woocommerce/myaccount/orders.php` | `templates/myaccount/orders.php` | Enhances order table with status badges, improved pagination |
| `woocommerce/myaccount/view-order.php` | `templates/myaccount/view-order.php` | Adds order timeline, styled totals, enhanced tracking section |

**Rule from spec §3.5:** No `cart.php`, `checkout.php`, `single-product.php`, or product loop PHP overrides. These are handled by WooCommerce Blocks.

---

### 3.16 Languages

| Path | Purpose | How Generated |
|---|---|---|
| `languages/torongo.pot` | Master POT template containing all translatable strings | `wp i18n make-pot . languages/torongo.pot --domain=torongo` (run at release) |
| `languages/{locale}.po` | Human-readable translation source for a specific locale (e.g., `fr_FR.po`) | Created by translators using Poedit or similar |
| `languages/{locale}.mo` | Compiled binary translation file loaded by WordPress at runtime | Compiled from `.po` via `msgfmt` or Poedit |
| `languages/torongo-{locale}-{hash}.json` | JS translation file for frontend strings | Generated by `npm run build` via `@wordpress/scripts` i18n utilities |

---

### 3.17 Tests

| Path | Purpose | Run Command |
|---|---|---|
| `tests/bootstrap.php` | PHPUnit bootstrap: loads WordPress test suite, Composer autoloader, and initializes theme constants | Called by `phpunit.xml.dist` |
| `tests/Unit/Core/SetupTest.php` | Tests `Setup::init()` registers expected hooks and all Manager classes are instantiated | `./vendor/bin/phpunit` |
| `tests/Unit/Core/AssetsTest.php` | Tests script/style handles are registered with correct versions and dependencies | — |
| `tests/Unit/Core/ThemeSupportTest.php` | Tests all expected `add_theme_support()` calls are made | — |
| `tests/Unit/WooCommerce/ManagerTest.php` | Tests WooCommerce hooks are registered, version check fires | — |
| `tests/Unit/WooCommerce/ProductDisplayTest.php` | Tests sale badge HTML output, escaping, product loop class filter | — |
| `tests/Unit/WooCommerce/CartCheckoutTest.php` | Tests trust strip hook callback output | — |
| `tests/Unit/WooCommerce/SchemaTest.php` | Tests JSON-LD output structure and `torongo_schema_data` filter | — |
| `tests/Unit/SEO/ManagerTest.php` | Tests structured data output, OG tag suppression when SEO plugin active | — |
| `tests/Unit/Accessibility/ManagerTest.php` | Tests skip link output, ARIA hook injection | — |
| `tests/Unit/Blocks/ManagerTest.php` | Tests all 4 blocks are registered via `register_block_type()` | — |
| `tests/Unit/Blocks/PatternManagerTest.php` | Tests pattern categories are registered | — |
| `tests/Unit/Blocks/StylesManagerTest.php` | Tests block style variants are registered | — |

---

## 4. Template Map

### 4.1 WordPress FSE Template Hierarchy

WordPress FSE resolves templates in this precedence order (highest to lowest). The first matching file wins.

```
WordPress FSE Template Resolution

Request Type            Template Checked (in order)
──────────────────────────────────────────────────────────
Homepage (is_front_page)
                        templates/front-page.html       ← Torongo: primary homepage
                        templates/home.html             (not in Torongo)
                        templates/index.html            ← Torongo: fallback

Single Post
                        templates/single-{post-type}-{slug}.html  (not in Torongo)
                        templates/single-{post-type}.html         (not in Torongo)
                        templates/single.html           ← Torongo

Single Product (WooCommerce)
                        templates/single-product-{slug}.html      (not in Torongo)
                        templates/single-product.html   ← Torongo ✦ WooCommerce override

Static Page
                        templates/page-{slug}.html                (not in Torongo)
                        templates/page-{id}.html                  (not in Torongo)
                        templates/page.html             ← Torongo
                        templates/singular.html                   (not in Torongo)
                        templates/index.html            ← Torongo: fallback

Shop Archive (WooCommerce)
                        templates/archive-product.html  ← Torongo ✦ WooCommerce override

Product Category (WooCommerce)
                        templates/taxonomy-product_cat-{slug}.html (not in Torongo)
                        templates/taxonomy-product_cat.html ← Torongo ✦ WooCommerce override
                        templates/archive-product.html  ← Torongo: fallback

Product Tag (WooCommerce)
                        templates/taxonomy-product_tag-{slug}.html (not in Torongo)
                        templates/taxonomy-product_tag.html ← Torongo ✦ WooCommerce override
                        templates/archive-product.html  ← Torongo: fallback

Blog Archive
                        templates/home.html                       (not in Torongo)
                        templates/archive.html          ← Torongo

Category / Tag Archive
                        templates/category-{slug}.html            (not in Torongo)
                        templates/category-{id}.html              (not in Torongo)
                        templates/category.html                   (not in Torongo)
                        templates/archive.html          ← Torongo: catches all generic archives

Search Results
                        templates/search.html           ← Torongo

404
                        templates/404.html              ← Torongo

Any other request
                        templates/index.html            ← Torongo: final fallback
```

Templates marked **✦ WooCommerce override** take priority over WooCommerce's own templates because FSE block templates in the theme's `/templates/` directory have higher precedence than WooCommerce's PHP template hierarchy.

---

### 4.2 Template Composition

Every template composes: `[Header Part] + [Content] + [Footer Part]`. This table shows the default assembly.

```
front-page.html
├── parts/header.html
├── patterns/hero-full.php            (Section 1)
├── patterns/categories-showcase.php  (Section 2)
├── patterns/products-featured.php    (Section 3 — filtered: featured)
├── patterns/cta-promotional.php      (Section 4)
├── patterns/products-grid-4.php      (Section 5 — filtered: best_seller)
├── patterns/testimonials-grid.php    (Section 6)
├── patterns/cta-newsletter.php       (Section 7)
└── parts/footer.html

single-product.html
├── parts/header.html
├── parts/breadcrumbs.html
├── [2-column Group block]
│   ├── [Left 60%] WooCommerce Product Gallery block
│   └── [Right 40%]
│       ├── WooCommerce Product Title block
│       ├── WooCommerce Product Rating block
│       ├── WooCommerce Product Price block
│       ├── WooCommerce Product Short Description block
│       ├── WooCommerce Product Add to Cart block
│       ├── WooCommerce Product SKU block
│       ├── WooCommerce Product Meta block
│       └── torongo/trust-badges block
├── WooCommerce Product Tabs block
├── WooCommerce Related Products block
└── parts/footer.html

archive-product.html
├── parts/header.html
├── parts/breadcrumbs.html
├── [Archive Title + product count + sort-by]
├── [2-column layout]
│   ├── [Filter Sidebar — optional/togglable]
│   │   ├── WooCommerce Price Range Filter block
│   │   ├── WooCommerce Attribute Filter block
│   │   ├── WooCommerce Rating Filter block
│   │   ├── WooCommerce Stock Filter block
│   │   └── WooCommerce Active Filters block
│   └── [Product Grid]
│       └── WooCommerce Products block (current archive query)
├── parts/pagination.html
└── parts/footer.html

single.html
├── parts/header.html
├── parts/breadcrumbs.html
├── Post Featured Image block
├── Post Title block
├── parts/post-meta.html
├── Post Content block
├── Comments block
└── parts/footer.html

page-no-title.html
├── parts/header-minimal.html
├── Post Content block (full-width)
└── parts/footer-minimal.html
```

---

### 4.3 Template Part Usage Matrix

| Template Part | `front-page` | `page` | `page-no-title` | `single` | `single-product` | `archive` | `archive-product` | `search` | `404` |
|---|---|---|---|---|---|---|---|---|---|
| `header.html` | ✓ | ✓ | — | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `header-minimal.html` | — | — | ✓ | — | — | — | — | — | — |
| `header-sticky.html` | User override | User override | — | User override | User override | User override | User override | User override | — |
| `footer.html` | ✓ | ✓ | — | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `footer-minimal.html` | — | — | ✓ | — | — | — | — | — | — |
| `footer-shop.html` | User override | — | — | — | — | — | User override | — | — |
| `breadcrumbs.html` | — | — | — | ✓ | ✓ | ✓ | ✓ | — | — |
| `post-meta.html` | — | — | — | ✓ | — | — | — | — | — |
| `pagination.html` | — | — | — | — | — | ✓ | ✓ | ✓ | — |

---

### 4.4 WooCommerce Template Relationships

```
WooCommerce Template Precedence (highest to lowest)

1. Torongo /templates/*.html           ← FSE block templates (highest)
   └── single-product.html
   └── archive-product.html
   └── taxonomy-product_cat.html
   └── taxonomy-product_tag.html

2. Torongo /woocommerce/*.php          ← PHP overrides (legacy surfaces only)
   └── emails/email-header.php
   └── emails/email-footer.php
   └── myaccount/my-account.php
   └── myaccount/orders.php
   └── myaccount/view-order.php

3. WooCommerce plugin templates        ← Default WooCommerce templates
   └── woocommerce/templates/          (used for ALL surfaces not overridden above)
   └── includes/blocks/                (WooCommerce Cart/Checkout/Mini-Cart blocks)

Surfaces intentionally NOT overridden (use WooCommerce defaults):
   cart.php          → WooCommerce Cart Block handles this via templates/
   checkout.php      → WooCommerce Checkout Block handles this via templates/
   mini-cart.php     → WooCommerce Mini Cart Block (styled via _mini-cart.scss)
   All email body templates → WooCommerce defaults (only header/footer overridden)
```

---

## 5. Asset Map

### 5.1 CSS Pipeline

```
Source (src/scss/)          Build Step              Output (assets/css/)
─────────────────────────────────────────────────────────────────────────
global.scss
 ├── base/_reset.scss
 ├── base/_root.scss          SCSS → PostCSS         global.css
 ├── base/_typography.scss    → Autoprefixer
 ├── blocks/_*.scss           → Minify (prod)
 └── components/_*.scss
                              rtlcss                 rtl.css
                              (post-build step)

editor.scss
 ├── base/_reset.scss
 ├── blocks/_*.scss           SCSS → PostCSS         editor.css
 └── [editor-specific]

woocommerce.scss
 ├── woocommerce/_global.scss
 ├── woocommerce/_product-card.scss
 ├── woocommerce/_product-single.scss
 ├── woocommerce/_cart.scss   SCSS → PostCSS         woocommerce.css
 ├── woocommerce/_checkout.scss
 ├── woocommerce/_account.scss
 ├── woocommerce/_badges.scss
 ├── woocommerce/_filters.scss
 ├── woocommerce/_mini-cart.scss
 └── woocommerce/_order-view.scss

admin.scss                    SCSS → PostCSS         admin.css
 └── [admin-specific]

src/js/blocks/{name}/
  style.scss                  SCSS → PostCSS         assets/css/blocks/{name}.css
```

**CSS Custom Property flow:**

```
theme.json settings.color.palette
  → WordPress generates: --wp--preset--color--primary (etc.)
  → SCSS consumes: var(--wp--preset--color--primary)
  → src/scss/base/_root.scss adds: --torongo-* computed derivatives
  → All component SCSS references either --wp--preset--* or --torongo-*
  → Zero hardcoded color, spacing, or size values permitted
```

**Critical CSS flow:**

```
npm run build
  → critical-css-generator extracts above-fold styles
  → writes: assets/css/critical.css

Runtime (wp_head at priority 1):
  Assets::output_critical_css()
  → file_get_contents( get_template_directory() . '/assets/css/critical.css' )
  → echo '<style id="torongo-critical">' . $css . '</style>'
```

---

### 5.2 JavaScript Pipeline

```
Source (src/js/)              Build Step              Output (assets/js/)
─────────────────────────────────────────────────────────────────────────
src/js/frontend/index.js
 ├── navigation.js
 ├── skip-link.js
 ├── sticky-header.js          Webpack (entry)        frontend.js
 ├── mobile-menu.js            → Babel (ES2022→ES5)
 ├── product-gallery.js        → Minify (prod)
 ├── quick-view.js             → Source map (dev)
 └── infinite-scroll.js
    (Swiper bundled)
    (a11y-dialog bundled)
    (focus-trap bundled)

src/js/admin/index.js          Webpack (entry)        admin.js

src/js/blocks/countdown-timer/
  index.js                     Webpack (block)        blocks/countdown-timer/index.js
  view.js                      Webpack (view)         blocks/countdown-timer/view.js
  block.json                   Copy                   blocks/countdown-timer/block.json

src/js/blocks/mega-menu/       (same pattern)         blocks/mega-menu/*
src/js/blocks/product-card-enhanced/                  blocks/product-card-enhanced/*
src/js/blocks/trust-badges/    (index only)           blocks/trust-badges/*
```

**WordPress externals** (never bundled — provided by WordPress at runtime):

```javascript
// webpack.config.js externals
{
  '@wordpress/blocks':        'wp.blocks',
  '@wordpress/element':       'wp.element',
  '@wordpress/i18n':          'wp.i18n',
  '@wordpress/dom-ready':     'wp.domReady',
  '@wordpress/interactivity': 'wp.interactivity',
}
```

**`window.torongoData` shape** (injected by `wp_localize_script`):

```javascript
window.torongoData = {
  breakpoints: {
    mobileSm:  320,
    mobile:    480,
    tablet:    768,
    tabletLg:  1024,
    desktop:   1200,
    wide:      1536,
  },
  ajaxUrl:  '/wp-admin/admin-ajax.php',
  nonces: {
    quickView: '...',     // generated per-page
  },
}
```

---

### 5.3 Image and Icon Organization

**`assets/images/icons/`** — 17 SVG icons

| Usage Context | Icons |
|---|---|
| Header actions | `cart.svg`, `search.svg`, `account.svg` |
| Product card hover | `heart.svg` (wishlist — D1 open), `search.svg` (quick view) |
| Mobile navigation | `menu.svg`, `close.svg` |
| Navigation/UI | `chevron-down.svg`, `chevron-left.svg`, `chevron-right.svg` |
| Product ratings | `star.svg`, `star-half.svg`, `star-empty.svg` |
| Product quantity | `plus.svg`, `minus.svg` |
| Filter/sort | `filter.svg` |
| Confirmation states | `check.svg` |
| Social sharing | `share.svg` |

**Icon delivery method:** Open decision D6 in spec. All icons reside in this directory regardless of method. Methods under consideration:
1. Inline SVG via `torongo_get_icon( $name )` PHP helper (most accessible, adds DOM weight)
2. `<img src="...">` with SVG MIME type (simplest, requires alt text on all)
3. CSS `mask-image` (smallest DOM, limited older browser support)

**`assets/images/placeholders/`**

| File | Dimensions | Used When |
|---|---|---|
| `product-placeholder.svg` | Scalable (SVG) | WooCommerce product has no featured image |

---

### 5.4 Font Strategy

**Requirement (spec §17.5):** Self-hosted WOFF2 only. No Google Fonts or external CDN.

**Three font families registered in `theme.json`:**

| Token | Role | Weights |
|---|---|---|
| `heading` | Display/heading typeface (DS1: TBD) | 400, 500, 600, 700 |
| `body` | Body text typeface (DS1: TBD) | 400, 500, 600, 700 |
| `mono` | Monospace for `<code>` (DS1: TBD) | 400 only |

**`theme.json` font registration pattern:**

```json
"settings": {
  "typography": {
    "fontFamilies": [
      {
        "fontFamily": "[font-stack], sans-serif",
        "name": "Heading",
        "slug": "heading",
        "fontFace": [
          {
            "src": ["file:./assets/fonts/heading/[name]-400.woff2"],
            "fontWeight": "400",
            "fontStyle": "normal",
            "fontDisplay": "swap"
          }
        ]
      }
    ]
  }
}
```

**Preload strategy (`Assets.php`):**

```php
// Preload body-400.woff2 and heading-700.woff2 only
// (body regular for FCP text, heading bold for LCP headline)
add_filter( 'torongo_preload_fonts', function( $urls ) {
    return [
        get_template_directory_uri() . '/assets/fonts/body/[name]-400.woff2',
        get_template_directory_uri() . '/assets/fonts/heading/[name]-700.woff2',
    ];
} );
```

Preload `<link>` tags are injected by `Assets::output_font_preloads()` hooked on `wp_head` at priority 2 (after critical CSS at priority 1).

---

## 6. Pre-Finish Review

### Missing Files

No files are missing. All files documented above derive from one of:
- `torongo-spec.md` §5 canonical structure
- Phase 2 folder structure design (which expanded §5 with implied files justified by other spec sections)

All 4 patterns present in spec §4 but absent from §5 original list have been included and reconciled in spec v1.1: `testimonials-carousel.php`, `cta-urgency.php`, `content-team-grid.php`, `woo-rating.php`.

### Duplicate Files

No duplicate files exist. Each file has exactly one location. The only apparent near-duplicates are intentional variations:
- `header.html` / `header-minimal.html` / `header-sticky.html` — 3 distinct header layouts
- `footer.html` / `footer-minimal.html` / `footer-shop.html` — 3 distinct footer layouts

### Inconsistent Locations

No location inconsistencies. Verified:
- All PHP classes: `inc/classes/{Namespace}/{ClassName}.php` ✓
- All SCSS partials: `src/scss/{category}/_{name}.scss` ✓
- All compiled CSS: `assets/css/*.css` or `assets/css/blocks/*.css` ✓
- All compiled JS: `assets/js/*.js` or `assets/js/blocks/{name}/*.js` ✓
- All block source: `src/js/blocks/{block-name}/` ✓
- All template parts: `parts/*.html` (flat, per spec §5 canonical rule) ✓
- All block templates: `templates/*.html` ✓
- All block patterns: `patterns/*.php` ✓

### Naming Consistency

Verified against spec §22 (Naming Conventions):

| Category | Rule | Status |
|---|---|---|
| PHP classes | `{ClassName}.php` under `inc/classes/` | ✓ Consistent |
| PHP helpers | `{context}-helpers.php` | ✓ `template-helpers.php`, `woocommerce-helpers.php` |
| SCSS partials | `_{name}.scss` | ✓ All partials use underscore prefix |
| SCSS entries | `{name}.scss` (no underscore) | ✓ `global.scss`, `editor.scss`, `woocommerce.scss`, `admin.scss` |
| JS modules | `{name}.js` (kebab-case) | ✓ All frontend modules kebab-case |
| Block directories | `{block-name}/` (kebab-case) | ✓ `countdown-timer/`, `mega-menu/`, etc. |
| Block patterns | `{pattern-name}.php` (kebab-case) | ✓ All patterns kebab-case |
| Template parts | `{part-name}.html` (kebab-case) | ✓ All parts kebab-case |
| Block templates | `{template-name}.html` | ✓ All templates match WP slug format |
| Custom block names | `torongo/{name}` namespace | ✓ All 4 blocks use `torongo/` prefix |
| CSS BEM classes | `.torongo-{name}`, `.torongo-{name}__{el}`, `.torongo-{name}--{mod}` | Defined as rule — enforced by Stylelint |
| JS-hook classes | `.js-{name}` | Defined as rule — no CSS applied to these |

---

*End of Torongo Implementation Structure Reference v1.0*
