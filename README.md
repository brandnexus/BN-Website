# brandnexus.com

Static one-page site with a single serverless contact handler. No build step.

```
index.html        the page, all CSS and JS inline
api/contact.js    POST handler, sends the enquiry via Resend
img/              hero, six band photographs, fourteen client logos, mark, OG image
vercel.json       security headers, immutable caching on /img
robots.txt        allows everything, points at the sitemap
sitemap.xml       one URL
```

## Deploy

Vercel auto-detects this as a static site with one Node function. Nothing to configure.

```bash
npm i -g vercel
cd <this folder>
vercel --prod --yes          # first run links or creates the project
```

Or drag the folder onto https://vercel.com/new.

## Required environment variable

Set in Vercel under Project → Settings → Environment Variables, for
Production and Preview:

| Name | Value |
|---|---|
| `RESEND_API_KEY` | from https://resend.com/api-keys |

Until this is set the form returns a clear error rather than failing silently.

## Resend sending domain

`api/contact.js` sends from `website@send.brandnexus.com`.

Verify **send.brandnexus.com** in Resend, not the apex. The apex already carries
the Google Workspace SPF record, and a domain may only have one SPF record.
Using a subdomain gives Resend its own SPF and DKIM and leaves Workspace mail
untouched.

Resend will supply three records to add at the DNS host, all on the
`send` subdomain.

## Domain cutover

Point the domain only after the site is confirmed working on its Vercel URL.

In Vercel: add `brandnexus.com` and `www.brandnexus.com` to the project, and
set www to redirect to the apex.

At DreamHost, in this order:

1. Remove the web redirect to pedrolaboy.com first, or it keeps intercepting.
2. Change the `@` A record from `69.163.187.230` to `76.76.21.21`.
3. Delete the `www` A record.
4. Add `www` CNAME → `cname.vercel-dns.com`.

Leave every MX and TXT record alone. SPF, DKIM and DMARC are correct and mail
must not be disturbed.

DreamHost queues zone edits, so the panel showing a record is not the same as
the record being live. Verify against the authoritative nameservers, not the panel.

Move nameservers to Vercel and transfer the registrar afterwards, never during.

## Payload

| | |
|---|---|
| index.html | 85 KB |
| Desktop first paint | 164 KB |
| Mobile first paint | 126 KB |
| Everything, fully scrolled | 668 KB |

Every photograph below the fold is lazy loaded. The hero and the six band
photographs each ship a half-resolution variant that phones pick up via
`srcset`. `/img` is cached immutably for a year, so a repeat visit downloads
only the HTML.
