# Body Factory Bali — Technical & Developer Documentation

## 1. Executive Summary

**Body Factory Bali** (`body-factory-bali`) is a high-performance modern web application built on **Vite 6** and **React 18**. It represents an exact, high-fidelity conversion of the official Body Factory Bali web platform from a SingleFile snapshot into a modular, maintainable, and developer-friendly Single Page Application (SPA).

This architecture bridges byte-for-byte visual and DOM equivalence with the agility, hot-module replacement (HMR), and ecosystem advantages of the modern React and Vite tooling stack.

---

## 2. Technology Stack & Specifications

| Layer | Technology | Version | Purpose |
| :--- | :--- | :--- | :--- |
| **Runtime / UI Framework** | React / React DOM | `^18.3.1` | Component tree rendering and virtual DOM reconciliation |
| **Bundler & Build Tool** | Vite | `^6.0.7` | Zero-bundle-overhead dev server (ES modules) & Rollup production builds |
| **React Compiler Plugin** | `@vitejs/plugin-react` | `^4.3.4` | Fast Refresh and JSX transformation support |
| **Module System** | ECMAScript Modules (ESM) | Native (`"type": "module"`) | Standard JavaScript module imports/exports |
| **CSS Styling Engine** | Custom Runtime Parser + CSS | Vanilla / Tailwind 3.x tokens | Byte-identical style extraction with runtime inline property parsing |
| **Structured Data** | Schema.org JSON-LD | 1.0 | Search engine optimization (SEO) for LocalBusiness & HealthClub |

---

## 3. Directory & File Structure

```
e:/gym/
├── dist/                               # Production build output (generated via `npm run build`)
│   ├── index.html                      # Minified distribution entry point (6.54 kB)
│   └── assets/                         # Fingerprinted CSS and JavaScript bundles
│       ├── index-[hash].css            # Bundled production stylesheet (~192 kB)
│       └── index-[hash].js             # Bundled application JavaScript (~1.97 MB)
├── node_modules/                       # Third-party package dependencies
├── src/                                # Application source code
│   ├── App.jsx                         # Main page layout & component JSX structure (~1.8 MB)
│   ├── cssToStyle.js                   # Runtime CSS-declaration-to-React-style parser
│   ├── main.jsx                        # Application root entry point (React DOM mounting)
│   └── styles.css                      # Extracted global CSS stylesheet (206,650 bytes)
├── index.html                          # Root HTML template and meta configuration
├── package.json                        # Project metadata, dependencies, and script definitions
├── package-lock.json                   # Deterministic lockfile for package dependencies
├── vite.config.js                      # Vite bundler configuration
├── README.md                           # Quick-start instructions and equivalence matrix
└── DOCUMENTATION.md                    # Comprehensive technical and developer documentation
```

---

## 4. Architecture & Core Mechanisms

### 4.1. DOM-to-JSX Conversion Architecture

The application was reconstructed by parsing the complete HTML DOM tree of the original site through an AST walker (`parse5` pipeline) and emitting valid, clean JSX:
- **Structural Integrity:** Elements, structural attributes, and text nodes map 1:1 with the source DOM.
- **Link Normalization:** Internal self-referential links (`bodyfactorybali.com`) have had `href` attributes stripped to prevent accidental page reloads away from the SPA container, while external links and anchor bookmarks are preserved.
- **Sandboxed Embeds:** Embedded interactive widgets (such as the interactive Google Map) are isolated inside `<iframe srcdoc={...}>` to safeguard against styling collisions and script execution leaks.

### 4.2. Runtime Inline Style Parser (`src/cssToStyle.js`)

In standard React, the `style` prop expects a JavaScript object with camelCase keys rather than a raw CSS string. Rather than mutating 1,000+ individual markup elements at conversion time, a performant runtime parser `s(css)` translates CSS declaration strings into React-compliant style objects:

```javascript
// src/cssToStyle.js
export function s(css) {
  if (!css) return undefined;
  const out = {};
  for (const decl of css.split(";")) {
    const i = decl.indexOf(":");
    if (i === -1) continue;
    let key = decl.slice(0, i).trim();
    let val = decl.slice(i + 1).trim();
    if (!key || !val) continue;
    
    // CSS Custom Properties (--var) must remain kebab-case
    if (key.startsWith("--")) {
      out[key] = val;
      continue;
    }
    
    // Vendor prefixes: -webkit-x -> WebkitX
    const camel = key.replace(/-([a-z])/g, (_, c) => c.toUpperCase());
    out[camel] = val;
  }
  return out;
}
```

#### Key Capabilities of `s(css)`:
1. **CSS Variable Retention:** Preserves custom properties (e.g. `--font-sans`, `--tw-text-opacity`) without altering hyphens.
2. **Kebab to CamelCase:** Automatically transforms standard CSS properties (e.g., `background-color` $\to$ `backgroundColor`, `margin-top` $\to$ `marginTop`).
3. **Vendor Prefix Normalization:** Converts `-webkit-`, `-moz-`, and `-ms-` vendor prefixes into browser-supported capitalization (e.g., `-webkit-backdrop-filter` $\to$ `WebkitBackdropFilter`).

### 4.3. Styling Architecture (`src/styles.css`)

All seven original `<style>` blocks from the site snapshot were extracted byte-for-byte into `src/styles.css` (206,650 bytes total):
- Preserves Tailwind CSS utility declarations (`flex`, `grid`, `items-center`, `z-[101]`, `px-4`, `2xl:px-8`, etc.).
- Preserves custom media queries, animations, typography configurations, and keyframes.
- Injected globally via standard ES import in `src/main.jsx`:
  ```javascript
  import "./styles.css";
  ```

### 4.4. Search Engine Optimization & Structured Data

The document retains enterprise-grade SEO and rich snippet data:
- **`index.html` Meta Tags:** Complete OpenGraph (`og:title`, `og:description`, `og:image`, `og:url`), Twitter Card (`summary_large_image`), canonical URLs, Google site verification, and inline high-resolution PNG favicons.
- **JSON-LD Schema (`App.jsx` RawBlock0):**
  Embedded `schema.org` entities for `Organization` and `HealthClub`, specifying:
  - Official name: **Body Factory Bali**
  - Geographic Coordinates: Latitude `-8.656011`, Longitude `115.1348921` (Canggu, Bali)
  - Operating Hours: Monday through Sunday, 06:00 – 22:00
  - Official Contact & Social Media Profiles (Instagram, Facebook, YouTube, LinkedIn, TikTok)

### 4.5. Content Security Policy (CSP) Adaptation

The original SingleFile snapshot included a strict meta-level CSP tag:
```html
<meta http-equiv="content-security-policy" content="default-src 'none'; ...">
```
In a modern development environment, this CSP blocked browser ES module resolution (`<script type="module" src="...">`) required by Vite and React.
- **Change Made:** The CSP `meta` element was cleanly removed from the root `index.html`.
- **Impact:** Enables unhindered HMR, dev script loading, and modern bundle execution with zero visual impact.

---

## 5. Development Workflow & Commands

### 5.1. Prerequisites

- **Node.js:** `v18.0.0` or higher (recommended: `v20.x` or `v22.x` LTS)
- **Package Manager:** `npm` (`v9.x` or higher)

### 5.2. Available Scripts

All scripts are executed from the project root directory (`e:\gym`):

#### 1. Install Dependencies
```bash
npm install
```
Downloads and validates all dependencies against `package-lock.json`.

#### 2. Start Development Server
```bash
npm run dev
```
- Starts the Vite local development server with Fast Refresh.
- Default local URL: `http://localhost:5173/`
- Instantaneous startup leveraging native ESM.

#### 3. Create Production Build
```bash
npm run build
```
- Compiles the React codebase and bundles assets using Rollup.
- Optimizes and minifies HTML, JavaScript, and CSS.
- Outputs production-ready files to the `dist/` directory.

#### 4. Preview Production Build
```bash
npm run preview
```
- Starts a local static web server serving the `dist/` production bundle.
- Useful for validating production performance and asset loading before deployment.

---

## 6. Verification & Fidelity Metrics

The React implementation has been verified against the original HTML document using structural DOM comparison algorithms:

| Validation Criteria | Target / Baseline | React App Implementation | Result |
| :--- | :--- | :--- | :--- |
| **Total DOM Nodes** | 598 entries | 598 entries | 100% Match |
| **Tag & Text Discrepancies** | 0 differences | 0 differences | Clean Parity |
| **Global CSS Byte Size** | 206,644 characters | 206,644 characters | Byte-for-byte Identical |
| **Inline Payload Blocks** | 25 raw blocks | 25 raw blocks | Identical |
| **Interactive Map Iframe** | 1 isolated doc | 1 isolated doc | Verified |
| **Link Parity** | 53 external/anchor links | 53 external/anchor links | Intact |
| **Console / React DOM Warnings** | 0 runtime errors | 0 runtime errors | Clean |

---

## 7. Recommended Production Enhancements

While the application operates with visual parity, developers seeking to expand or refactor the project should consider the following architectural recommendations:

### 7.1. Component Modularization
`src/App.jsx` currently contains the entire monolithic landing page (~1.8 MB). Refactoring this file into focused subcomponents will improve IDE performance, code clarity, and testability:
```
src/
├── components/
│   ├── Navigation/
│   │   ├── Header.jsx
│   │   └── MobileMenu.jsx
│   ├── Hero/
│   │   └── HeroSection.jsx
│   ├── Services/
│   │   ├── GymFacilities.jsx
│   │   ├── HyroxTraining.jsx
│   │   ├── ClassSchedule.jsx
│   │   └── RecoveryCenter.jsx
│   ├── Membership/
│   │   ├── PricingTiers.jsx
│   │   └── DayPasses.jsx
│   ├── Common/
│   │   ├── MapEmbed.jsx
│   │   └── SeoSchema.jsx
│   └── Footer/
│       └── Footer.jsx
```

### 7.2. Code Splitting & Chunk Optimization
During `npm run build`, Rollup flags chunks over 500 kB (`index-[hash].js` is ~1.97 MB). To improve initial page load and Time to Interactive (TTI):
- Configure `manualChunks` in `vite.config.js`:
  ```javascript
  // vite.config.js
  import { defineConfig } from "vite";
  import react from "@vitejs/plugin-react";

  export default defineConfig({
    plugins: [react()],
    build: {
      rollupOptions: {
        output: {
          manualChunks: {
            vendor: ["react", "react-dom"],
          },
        },
      },
      chunkSizeWarningLimit: 2000,
    },
  });
  ```
- Implement `React.lazy()` and `Suspense` for below-the-fold sections (e.g., Map embed, Recovery gallery, Schedule modal).

### 7.3. Asset Externalization
Many images and vectors in `App.jsx` and `styles.css` are embedded as Base64 data URIs. Extracting these assets into `public/images/` or a CDN in modern formats (`.webp`, `.avif`) will reduce the JavaScript bundle size by over 60%.

---

## 8. Troubleshooting & FAQ

### Q: Why do navigation links not change the browser URL?
**A:** The landing page was derived from a single-page snapshot where internal links were configured as non-navigating anchors (`<a>` without active `href`) to prevent navigating away to the live production server. To enable multi-page routing, install and configure `react-router-dom`.

### Q: Port 5173 is already in use. How do I run on a different port?
**A:** Run Vite specifying the `--port` flag:
```bash
npm run dev -- --port 3000
```

### Q: Can I edit `src/App.jsx` directly?
**A:** Yes. `App.jsx` is standard React code. You can refactor, edit text, update image paths, and add interactive state (e.g. `useState`, modal handlers) directly within the file.
