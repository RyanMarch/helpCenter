# Help Center SaaS (Option B): Advanced Micro-Chunked Playbook

This playbook breaks down the implementation of tenant authentication, structured storage, customer paywall limits, and Cloudflare rate limit fallback strategies into highly granular, copy-pasteable instructions for your IDE agent.

---

## Step 1: Database Structure & Tenant Metadata
*Define the SQL schema for multi-tenant isolation, user roles, and subscription tier rules.*

### Agent Prompt: Step 1
```markdown
We are designing the data model for our multi-tenant documentation SaaS on Cloudflare D1.

Please create a `migrations/0001_init_schema.sql` file defining:
1. `tenants` table:
   - `id` (TEXT PRIMARY KEY)
   - `project_name` (TEXT)
   - `custom_domain` (TEXT)
   - `subscription_tier` (TEXT) - e.g., 'free', 'pro'
   - `created_at` (TEXT)
2. `articles` table:
   - `id` (TEXT PRIMARY KEY)
   - `tenant_id` (TEXT, FOREIGN KEY references tenants(id))
   - `title` (TEXT)
   - `category` (TEXT)
   - `excerpt` (TEXT)
   - `content_html` (TEXT)
   - `slug` (TEXT)
   - `status` (TEXT) - e.g., 'draft', 'published'
   - `last_updated` (TEXT)
3. `users` table:
   - `id` (TEXT PRIMARY KEY)
   - `tenant_id` (TEXT, FOREIGN KEY references tenants(id))
   - `email` (TEXT UNIQUE)
   - `password_hash` (TEXT)
   - `role` (TEXT) - e.g., 'admin', 'writer'
4. Generate indices for fast lookups on:
   - `articles(tenant_id, slug)`
   - `users(email)`
```

---

## Step 2: Authentication & Token Authorization Middleware
*Secure write APIs using JSON Web Tokens (JWT) inside Cloudflare Workers.*

### Agent Prompt: Step 2
```markdown
We need to implement authentication in our Cloudflare Worker using native Web Crypto APIs (avoiding heavy external NPM packages).

Please write authentication handlers in the worker:
1. `POST /api/auth/login` -> Verifies user credentials in D1, signs a JWT token using a secret key defined in Workers environment variables (`JWT_SECRET`), and returns the token.
2. Implement a helper function `authorizeRequest(request)` that:
   - Extracts the Bearer token from the `Authorization` header.
   - Validates the signature and checks expiration.
   - Attaches `tenant_id` and `role` metadata to the context.
```

---

## Step 3: Customer Limits & Paywall Enforcement
*Validate database writes and active article caps against the tenant's subscription tier before performing database queries.*

### Agent Prompt: Step 3
```markdown
We need to implement paywall logic in our Worker to enforce limits based on the tenant's subscription tier.

Please update the write/create article route (`POST /api/docs`):
1. Prior to inserting/updating an article, query D1 to check the tenant's `subscription_tier`.
2. Enforce limits:
   - If tier is `'free'`, restrict the total number of published articles to 10.
   - If the limit is reached, return a `403 Forbidden` response: `{"error": "Limit reached", "message": "Upgrade to Pro to publish more than 10 articles."}`.
3. Validate user credentials role (only allow roles `'admin'` and `'writer'` to publish).
```

---

## Step 4: Worker API Caching & Rate Limit Handling
*Implement caching on Cloudflare Workers to prevent exceeding free tier database read/write quotas.*

### Agent Prompt: Step 4
```markdown
We want to save D1 database read operations by caching public GET request responses using the Cloudflare Cache API.

Please update the API Worker:
1. In `GET /api/docs` and `GET /api/docs/:slug`, check if the requested resource exists in the Cloudflare Cache.
2. If it is cached, return the cached Response immediately.
3. If not, fetch from D1, add appropriate Cache-Control headers (e.g. public, max-age=60), write to Cache API, and return.
4. When an article is added, updated, or deleted, invalidate the cached endpoints for that tenant.
```

---

## Step 5: Web Components - Offline Search Fallback
*Refactor frontend components to gracefully handle server network errors or Cloudflare Worker limit exhaustion (HTTP 429).*

### Agent Prompt: Step 5
```markdown
We need to make our documentation reader highly resilient to network downtime, rate limits (HTTP 429), or Worker limit exhaustion.

Please update `DocsSearch` and `DocsGrid` in [docs-components.js](file:///Users/ryan/Sites/helpCenter/docs/docs-components.js):
1. When fetching the articles index from `/api/docs?tenantId=...`, save a copy of the JSON payload to `localStorage` under `cached_search_index`.
2. If the API request returns a `429` error, fails, or times out:
   - Read from `localStorage` (`cached_search_index`) to serve cached search and list results.
   - Display a subtle banner UI to the user: "Offline mode: showing locally cached guides."
```

---

## Step 6: Web Components - Static Fallback Navigation
*Implement anchor link fallback navigation if JavaScript components fail to render or block.*

### Agent Prompt: Step 6
```markdown
We want to ensure graceful degradation if JavaScript files fail to load.

Please update the template shell in [templates/index.html](file:///Users/ryan/Sites/helpCenter/scripts/templates/index.html):
1. Wrap the Web Components inside a fallback `<noscript>` container that includes plain static HTML links to key sections.
2. Use standard semantic HTML anchor tags pointing directly to sub-folders (e.g. `/docs/basics/`) so users can navigate pages even with JavaScript disabled.
```
