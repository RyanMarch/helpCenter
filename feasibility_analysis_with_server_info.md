# Standalone Help Center: Hosting & Styling Roadmap

We have refined the product direction based on two goals:
1. **Isolated Styling:** The documentation folder must have its own self-contained CSS, removing the dependency on the host application's root stylesheet.
2. **$0 Cloudflare Hosting:** Leveraging Cloudflare's free tiers to distribute or host this product without incurring infrastructure costs.

---

## Cloudflare Free-Tier Hosting Strategies

Since this Help Center compiles to static assets (HTML, vanilla JS, CSS, and a static JSON search index), we can host and run it on Cloudflare for **$0**. Here are three ways to architecture this:

### Option A: The "Bring-Your-Own-Host" Starter (Decentralized SaaS)
You distribute the CLI and components as an open-source project or package. Other developers host it themselves.
* **Infrastructure Cost for You:** $0 (no hosting server required).
* **Cost for Users:** $0 (they deploy to their own free Cloudflare Pages project).
* **Cloudflare Service:** **Cloudflare Pages** (Free Tier: Unlimited projects, custom domains, SSL, and bandwidth; 500 builds/month).
* **Pros:** Zero maintenance, zero liability, completely private/client-side.

### Option B: The "Serverless Dynamic" SaaS (Hosted Multi-Tenant)
You host the dashboard/admin console where users sign up and write articles, and we serve the rendered docs.
* **Infrastructure Cost for You:** $0 (within free tiers).
* **Cloudflare Service:** 
  * **Cloudflare Pages:** Serves the authoring dashboard and the client docs reader.
  * **Cloudflare Workers:** Serverless API for handling auth and article CRUD operations (Free Tier: 100k requests/day).
  * **Cloudflare D1 (SQL Database) or KV:** Stores article content and configurations (Free Tier: 5M reads/month, 100k writes/month).
* **Pros:** A true SaaS product experience. Can be monetized later.

### Option C: The CDN-Config Reader (Hybrid Multi-Tenant)
You host a single static Reader app on Cloudflare Pages. Users host their own `nav.json`, `search-index.json`, and HTML articles (e.g., on GitHub or their own server). The Reader loads their configuration dynamically via a URL parameter (e.g., `help.yoursite.com/?config=https://raw.githubusercontent.com/...`).
* **Infrastructure Cost for You:** $0 (just static Page traffic).
* **Cloudflare Service:** **Cloudflare Pages** (Free Tier).
* **Pros:** Fast, lightweight, keeps you from storing user data, yet offers a unified reader interface.

---

## Next Action Plan: Decoupling the CSS

To isolate the styles, we will implement the following changes in [docs/style.css](file:///Users/ryan/Sites/iconStudio/docs/style.css):

1. **Remove the Import/Dependency:** Delete any import of or reference to `../style.css`.
2. **Define Base Typography & Normalizations:** Add standard styling for base elements (`body`, `a`, `h1-h6`, `code`) directly inside the local stylesheet.
3. **Decouple App Components:** Remove references to site-specific elements like `<icon-studio-footer>` and design a native `<docs-footer>` Custom Element.
