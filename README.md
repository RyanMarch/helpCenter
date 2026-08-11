# Portable Help Center

A zero-dependency, self-contained documentation system built with native Web Components and static indexing. You can drop this directly into any website project to get a fully searchable help center running in minutes.

---

## How to Integrate into Another Project

Follow these steps to copy and set up the documentation center in your target repository.

### Step 1: Copy the Directories
Copy the `docs/` and `scripts/` directories directly into the root of your target project:
```bash
cp -r /path/to/helpCenter/docs /path/to/targetProject/
cp -r /path/to/helpCenter/scripts /path/to/targetProject/
```

### Step 2: Configure Project Metadata
Open the newly copied `docs/docs-config.json` inside the target project and update the configuration parameters:
```json
{
  "projectName": "Target Project",
  "componentPrefix": "docs",
  "baseUrl": "https://targetproject.app",
  "faviconDir": "./assets/favicon"
}
```
* **`projectName`:** The display name of your site (e.g., used in header/footer text and document titles).
* **`faviconDir`:** The relative or absolute path to your favicon assets directory. If using local favicons inside the `docs/` folder, use `./assets/favicon`. If sharing favicons with your host application, use the path to the host application's asset directory (e.g., `../assets/favicon`).
* **`sidebarCta`** *(optional)*: promo card pinned to the bottom of every article's sidebar. Omit the key entirely to leave the sidebar without one.
  ```json
  "sidebarCta": {
    "title": "Your App",
    "description": "One line pitching the app itself.",
    "buttonText": "Go to Your App",
    "buttonUrl": "/app"
  }
  ```

> **Favicon paths are one level deeper on article pages.** `faviconDir` is resolved relative to `docs/` for `index.html`/`list.html`, but article pages live one level below that at `docs/<slug>/`, so `scripts/add-new-doc.js` automatically prepends an extra `../` for those. If you ever hand-write or script your own article pages instead of going through `npm run docs:new`, remember that extra level, it fails silently (a 404'd favicon, not a visible error).

### Step 3: Register Commands in package.json
Open your target project's `package.json` and append the following scripts:
```json
"scripts": {
  "docs:init": "node scripts/init-docs.js",
  "docs:new": "node scripts/add-new-doc.js",
  "docs:build": "node scripts/generate-docs-index.js"
}
```

### Step 4: Run Initial Setup
Initialize the templates and build the static index:
```bash
# Compile index.html and list.html shells
npm run docs:init

# Generate the initial search index
npm run docs:build
```

---

## Daily Workflow Commands

* **Create a New Article:**
  ```bash
  npm run docs:new
  ```
  Follow the command-line prompts to specify title, category, and meta descriptions. The script will scaffold the new HTML folder structure automatically.
  
* **Regenerate Search Index:**
  ```bash
  npm run docs:build
  ```
  Run this command whenever you add, update, or remove help articles to keep the client-side fuzzy search accurate.

---

## Embedding Screenshots

Drop a `<figure class="doc-screenshot-figure">` anywhere inside an `<article class="doc-section">` and it's automatically clickable, opening in a full-screen lightbox (the modal markup is already in the page template, `docs-components.js` wires it up on load):

```html
<figure class="doc-screenshot-figure">
    <img src="../../assets/screenshots/example.png" alt="Description for screen readers" loading="lazy">
    <figcaption>Caption shown under the image, and in the lightbox.</figcaption>
</figure>
```

`figcaption` is optional. Keyboard users can tab to the image and press Enter/Space to open it; Escape, clicking the backdrop, or the close button all dismiss it.
