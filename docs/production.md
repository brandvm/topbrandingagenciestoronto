# Production deployment

Canonical domain: `https://www.topbrandingagenciestoronto.ca`. Production Worker: `topbrandingagenciestoronto`. Protected review Worker: `topbrandingagenciestoronto-staging`.

## Release

Run `npm ci`, `npm run verify`, and `npm run audit`. The audit checks every public content page on mobile and desktop, three times, requiring performance at least 95 and accessibility, best practices and SEO 100. Run `npm run deploy:production` to rebuild, verify and publish to the public Worker. The **Publish production** manual GitHub workflow includes the same checks and the full audit.

Production uses `wrangler.production.jsonc` and contains only static assets. There is no authentication Worker, review API, review JavaScript, D1 binding or password/session secret. The protected staging configuration and comment data remain separate. `scripts/verify-production.mjs` rejects a staging build or leaked review features.

Production pages permit crawling, contain a unique title and description, and point canonicals and the sitemap at the domain above. Old article URLs retain permanent redirects. `robots.txt` includes the canonical sitemap. The production Worker disables version preview URLs.

## Domain connection

The registrar can remain external. Cloudflare Workers Custom Domains require an active Cloudflare DNS zone. Add this domain to the Cloudflare account, inspect the imported DNS records against the registrar's full record list, and preserve any mail, verification or other service records. Then replace the registrar nameservers with the exact pair Cloudflare assigns. If DNSSEC is enabled, follow Cloudflare's migration procedure before changing nameservers.

The production Wrangler configuration declares both `www.topbrandingagenciestoronto.ca` and `topbrandingagenciestoronto.ca` as Custom Domains on `topbrandingagenciestoronto`. Wrangler creates the routing and certificates and replaces conflicting parking records during the authorized cutover. Both hosts must point to this production Worker, never the `-staging` Worker. An active Cloudflare zone is needed for the domains to be publicly reachable.

Use `www.topbrandingagenciestoronto.ca` as canonical. Configure a permanent (301) Cloudflare Redirect Rule matching the apex host or an HTTP request on either public host, with the dynamic target `concat("https://www.topbrandingagenciestoronto.ca", http.request.uri.path)` and query-string preservation enabled. Domain-level redirects are not supported by the Worker static `_redirects` file. Verify apex/www, HTTPS, old article redirects, 404 status, robots.txt and sitemap.xml after propagation.

## Search Console

Add a Domain property for `topbrandingagenciestoronto.ca`, copy Google's exact TXT verification record into the active DNS provider, then verify and submit `https://www.topbrandingagenciestoronto.ca/sitemap.xml`. Domain-property verification does not require an HTML meta tag or a rebuild. Do not invent a verification value.

Run PageSpeed Insights on the actual domain after cutover. Local Lighthouse reports are prelaunch measurements, not public PageSpeed or real-user field data; scores can vary between runs.

## Rollback

Record the production Worker deployment/version before replacing it. `npx wrangler deployments list --name topbrandingagenciestoronto` lists prior deployments; `npx wrangler rollback <version-id> --name topbrandingagenciestoronto` restores one. Production rollback does not touch staging review databases.
