# Base44 Dev Environment

This is a Next.js 10 + Shopify Storefront API + Builder.io CMS storefront (`nextjs-commerce`).

## Running the app

```sh
docker compose -f docker-compose.base44.yml up -d
```

- Web entry point is on host port **3000** (`next dev`).
- The container bind-mounts the repo at `/app` and runs `npm ci && npx next dev -H 0.0.0.0 -p 3000`, so source edits hot-reload.
- `node_modules` lives in an anonymous container volume (not on the host).

## Credentials / env vars

The app requires three env vars to boot (`config/builder.ts` and `config/shopify.ts` throw without them):

- `SHOPIFY_STOREFRONT_API_TOKEN`
- `SHOPIFY_STORE_DOMAIN`
- `BUILDER_PUBLIC_KEY`

These are **not** user secrets — the repo ships committed demo values in
`.env.development` (the public `builder-io-demo.myshopify.com` store + a public
Builder.io space). Next.js auto-loads `.env.development` in dev mode, so the app
boots with no external credentials. To use your own store/space, replace those
values (or provide real secrets via the platform secrets flow).

## Quirks

- **Next.js 10 on Node 16**: the compose pins `node:16` to avoid the OpenSSL 3
  breakage that affects Next 10's webpack on Node 17+. Do not bump to node:18+
  without testing.
- **CSP `frame-ancestors`**: `next.config.js` restricts framing to builder.io.
  This blocks the Base44 preview iframe, so the headers() function now allows
  `frame-ancestors *` in development and keeps the strict builder.io policy in
  production. Do not remove the production branch.
- Pages use `getStaticProps` with `fallback: true`; in dev the first request to
  a path shows a brief "Loading..." state while Builder.io content is fetched.
- `next.config.js` uses `target: 'serverless'` (original repo setting).

## Verifying it works

```sh
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:3000/   # expect 200
docker compose -f docker-compose.base44.yml ps                    # web should be (healthy)
```
