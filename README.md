# Open Graph Guide

A practical, copy-paste reference for **Open Graph meta tags**, **Twitter cards** and **link previews**. Use it when you're shipping a new page, fixing a broken preview, or reviewing a pull request that touches `<head>`.

> Want to test a live URL right now? Run it through **[OG Scope](https://www.ogscope.pk/)**, a free Open Graph checker that shows your tags and estimated link previews side by side.

## Contents

- [What Open Graph is](#what-open-graph-is)
- [Essential tags cheat sheet](#essential-tags-cheat-sheet)
- [Twitter card tags](#twitter-card-tags)
- [OG image size and rules](#og-image-size-and-rules)
- [Next.js metadata example](#nextjs-metadata-example)
- [Platform notes](#platform-notes)
- [Debugging checklist](#debugging-checklist)
- [Check your tags with OG Scope](#check-your-tags-with-og-scope)
- [Template](#template)

---

## What Open Graph is

Open Graph (OG) is a metadata vocabulary, first published by Facebook, that tells other apps how to summarise a URL. When a link is shared, the receiving app sends a bot to fetch the page, looks for `og:*` meta tags in the `<head>`, and turns them into a preview card with a title, a short description and an image.

Three facts explain most link preview bugs:

1. **The bot reads raw HTML.** It generally doesn't execute your JavaScript, so tags injected on the client are effectively absent. Render them on the server, at build time, or through a prerender step.
2. **The bot is a stranger.** It has no cookies, no VPN and no staging password. If a logged-out visitor can't load the page or the image, neither can the bot.
3. **The result gets cached.** Each platform keeps its own copy of the card. Fixing your HTML doesn't update a preview that has already been stored.

Open Graph tags shape how a shared link looks. They aren't a search ranking signal, but a clear card does earn more clicks.

---

## Essential tags cheat sheet

Paste this into `<head>` and replace the values:

```html
<meta property="og:type" content="website" />
<meta property="og:site_name" content="Acme Docs" />
<meta property="og:title" content="Deploy a static site in five minutes" />
<meta property="og:description" content="A short walkthrough from empty folder to live URL, no server required." />
<meta property="og:url" content="https://acme.dev/guides/static-deploy" />
<meta property="og:image" content="https://acme.dev/og/static-deploy-v2.png" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
<meta property="og:image:alt" content="Terminal window showing a successful deploy" />
```

| Tag | Required? | What to put in it |
| --- | --- | --- |
| `og:title` | Yes | A headline written for the feed. Aim for 40–60 characters and leave out the "\| Brand" suffix. |
| `og:description` | Recommended | One or two plain sentences, roughly 110–160 characters. |
| `og:image` | Yes (in practice) | An absolute `https://` URL to a public JPG, PNG or WebP. |
| `og:url` | Yes | The canonical address, without tracking parameters. It should match `<link rel="canonical">`. |
| `og:type` | Yes | `article` for blog posts and news, `website` for home, product and landing pages. |
| `og:site_name` | Optional | Your brand name. Some apps show it above the title. |
| `og:image:width` / `og:image:height` | Optional | Lets the app lay out the card before the image has downloaded. |
| `og:image:alt` | Optional | Alt text for the image. Good for accessibility. |
| `og:locale` | Optional | For example `en_US` or `ur_PK`. |

**Syntax gotchas**

- Open Graph tags use the **`property`** attribute. Twitter tags use **`name`**. Mixing them up is a common reason a tag gets ignored.
- Each `og:*` property should appear once. Two plugins that both print `og:title` leave the bot to choose between them.
- HTML-escape quotes and ampersands inside `content` (`&quot;`, `&amp;`).

For articles you can add:

```html
<meta property="og:type" content="article" />
<meta property="article:published_time" content="2026-10-13T09:00:00+05:00" />
<meta property="article:author" content="https://acme.dev/team/abid" />
<meta property="article:tag" content="open graph" />
```

---

## Twitter card tags

X (Twitter) has its own `twitter:*` tags and falls back to the Open Graph values when one of them is missing. Most of the time, you only need `twitter:card` to pick the layout:

```html
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:site" content="@acmedev" />
<meta name="twitter:creator" content="@abidali671" />
<!-- Optional overrides; X will otherwise reuse og:title / og:description / og:image -->
<meta name="twitter:title" content="Deploy a static site in five minutes" />
<meta name="twitter:description" content="From empty folder to live URL, no server required." />
<meta name="twitter:image" content="https://acme.dev/og/static-deploy-v2.png" />
<meta name="twitter:image:alt" content="Terminal window showing a successful deploy" />
```

| `twitter:card` value | Layout |
| --- | --- |
| `summary` | A small square thumbnail next to the text. You get this if you set nothing. |
| `summary_large_image` | A full-width image above the title. This is what most sites want. |
| `player` / `app` | Specialised cards for video and app installs. They need extra tags and approval. |

The fallback only goes one way. X will read `og:image`, but other platforms don't read `twitter:image`. Always set the Open Graph tags first, then add Twitter overrides only where you want different copy.

To check which layout a page asks for, use the [Twitter card checker guide](https://www.ogscope.pk/twitter-card-checker).

---

## OG image size and rules

**Recommended og image size: 1200 × 630 px** (an aspect ratio of about 1.91:1).

| Rule | Why it matters |
| --- | --- |
| 1200 × 630, or at least 600 × 315 | Large cards look blurry below this. Very small images, under about 200 px on a side, may be dropped or shown as a thumbnail. |
| JPG, PNG or WebP | These formats are safe everywhere. Avoid SVG, and don't count on animated GIF. |
| Keep it under ~1 MB (smaller is better) | Heavy files time out on some crawlers. WhatsApp in particular is happier with lighter images. |
| Absolute HTTPS URL | `/og.png` is a relative path. Bots may resolve it wrongly or not at all. |
| Publicly reachable | No auth, no signed URLs that expire, no `robots.txt` block on the folder, no bot challenge page. |
| Correct `Content-Type` | The server should respond with `image/png`, `image/jpeg` or `image/webp`, not `text/html`. |
| Safe zone in the center | Platforms crop edges differently, and some show a square. Keep the logo and text inside the middle ~80%. |
| Large, high-contrast text | Cards are often shown at phone width. Anything below ~40 px type on a 1200 px canvas is hard to read. |
| Version the filename | Cache is keyed by URL. `cover-v3.png` forces a refetch, while overwriting `cover.png` may not. |

Quick check from a terminal:

```bash
curl -sI https://acme.dev/og/static-deploy-v2.png | grep -iE '^HTTP|content-type|content-length'
```

You should see `200`, an `image/*` type and a reasonable size. For a deeper image check (raw tag value vs. absolute URL, load status, real dimensions), see the [OG image checker](https://www.ogscope.pk/og-image-checker).

---

## Next.js metadata example

With the App Router (Next.js 13.4+), `metadata` and `generateMetadata` run on the server, so the tags end up in the first HTML response.

**Site-wide defaults** in `app/layout.tsx`:

```tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  // Makes every relative image path absolute
  metadataBase: new URL("https://acme.dev"),
  title: { default: "Acme Docs", template: "%s | Acme Docs" },
  description: "Guides for shipping fast, static websites.",
  openGraph: {
    type: "website",
    siteName: "Acme Docs",
    locale: "en_US",
    images: [{ url: "/og/default.png", width: 1200, height: 630, alt: "Acme Docs" }],
  },
  twitter: {
    card: "summary_large_image",
    site: "@acmedev",
  },
};
```

**Per-page values** in `app/guides/[slug]/page.tsx`:

```tsx
import type { Metadata } from "next";
import { getGuide } from "@/lib/guides";

type Props = { params: Promise<{ slug: string }> };

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  const guide = await getGuide(slug);
  const path = `/guides/${slug}`;

  return {
    title: guide.title,
    description: guide.excerpt,
    alternates: { canonical: path },
    openGraph: {
      type: "article",
      url: path,
      title: guide.title,
      description: guide.excerpt,
      publishedTime: guide.publishedAt,
      images: [{ url: guide.ogImage, width: 1200, height: 630, alt: guide.title }],
    },
    twitter: {
      card: "summary_large_image",
      title: guide.title,
      description: guide.excerpt,
      images: [guide.ogImage],
    },
  };
}
```

Notes:

- In Next.js 15+, `params` is a Promise, which is why it's awaited above. On 13/14 you can destructure it directly.
- Nested `openGraph` objects **replace** the parent's object rather than merging with it. If a page sets `openGraph`, repeat any fields you still need, such as `siteName` and `images`.
- To generate images instead of hosting them, add `app/guides/[slug]/opengraph-image.tsx` and use `ImageResponse` from `next/og`. Next.js adds the `og:image` tags for you.
- On the Pages Router, put the tags in `next/head` inside the page, and make sure the data comes from `getStaticProps` or `getServerSideProps` rather than a client fetch.

Other stacks: Astro, Nuxt (`useSeoMeta`), SvelteKit (`<svelte:head>` with SSR) and Remix (`meta` export) all render on the server by default. With a plain Vite/React SPA you'll need prerendering or an edge function to inject the tags.

---

## Platform notes

| Platform | What it reads | Things to know |
| --- | --- | --- |
| **Facebook** | `og:title`, `og:description`, `og:image`, `og:url`, `og:type` | Falls back to `<title>` if `og:title` is missing. Shows a wide crop. Caches heavily, so use the Sharing Debugger to refresh. [Facebook link preview guide](https://www.ogscope.pk/facebook-link-preview) |
| **LinkedIn** | Open Graph tags | Ignores `twitter:*` tags. Caches the card when a post is composed, so re-inspect with Post Inspector after changes. [LinkedIn preview guide](https://www.ogscope.pk/linkedin-preview) |
| **X (Twitter)** | `twitter:*`, with OG as a fallback | Layout is decided only by `twitter:card`. Without it you get the small `summary` card. |
| **Slack** | Open Graph, then Twitter tags, then oEmbed | Unfurls show the site name, title, description and image. Slack caches unfurls for a while, so a new query string (`?v=2`) is a quick test, but don't put it in `og:url`. |
| **WhatsApp** | Open Graph | Builds the preview when the message is composed. Small, fast-loading images are most reliable. Previously sent messages keep the old preview. |
| **Discord / Telegram** | Open Graph, then Twitter | Discord can use `theme-color` for the embed accent. Telegram has `@WebpageBot` to refresh its cache. |

The common rule: **get the Open Graph set right first.** It's the one set of tags every platform reads.

---

## Debugging checklist

Work from top to bottom. Most broken previews are found in the first three steps.

**Your HTML**

- [ ] The page is deployed to a public URL (bots can't reach `localhost` or a VPN)
- [ ] `curl -sL <url>` (or *View Source*, not DevTools) shows the `og:*` tags in the raw HTML
- [ ] Each `og:*` tag appears only once
- [ ] `og:title`, `og:description`, `og:image`, `og:url` and `og:type` are all present
- [ ] `og:url` matches the canonical URL and redirects nowhere
- [ ] `twitter:card` is `summary_large_image` when a full-width image card on X is the goal

**Your image**

- [ ] `og:image` is an absolute `https://` URL
- [ ] It returns `200` with an `image/*` content type for a logged-out request
- [ ] It's 1200 × 630 px, saved as JPG, PNG or WebP, and reasonably light
- [ ] It isn't blocked by `robots.txt`, a WAF challenge or a signed URL that expires
- [ ] The filename was versioned if you replaced the image

**Platform caches**

- [ ] **Facebook Sharing Debugger** (developers.facebook.com/tools/debug): click *Scrape Again* to refresh Facebook's copy
- [ ] **LinkedIn Post Inspector** (linkedin.com/post-inspector): inspect the URL to refresh LinkedIn's copy
- [ ] For X: post the link in a draft or private test account. X has no public validator any more.
- [ ] For Slack and WhatsApp: test in a private chat, using a cache-busting query string if needed

If your raw HTML is correct and only one platform looks wrong, the problem is that platform's cache, not your markup.

---

## Check your tags with OG Scope

[**OG Scope**](https://www.ogscope.pk/) is a free Open Graph checker and link preview tool. Paste any public URL and it:

- fetches the page HTML on the server, without running JavaScript, so you see what a crawler sees
- lists the Open Graph tags, Twitter card tags, title, meta description, canonical URL and favicon in one metadata table
- checks the og image: the published value, the resolved URL, whether it loaded and its pixel size
- draws estimated cards for Facebook, LinkedIn, X, WhatsApp, Slack, Discord and Telegram
- doesn't need an account

The cards are estimates built from a fresh fetch, so use OG Scope to confirm your markup and the official debuggers above to clear platform caches.

Focused guides:

| Guide | Use it for |
| --- | --- |
| [OG image checker](https://www.ogscope.pk/og-image-checker) | Image URL, loading, dimensions and og image size issues |
| [Twitter card checker](https://www.ogscope.pk/twitter-card-checker) | `summary` vs `summary_large_image` and which fallbacks apply |
| [LinkedIn preview](https://www.ogscope.pk/linkedin-preview) | The fields LinkedIn uses and its caching behavior |
| [Facebook link preview](https://www.ogscope.pk/facebook-link-preview) | Facebook's tags, crop and Sharing Debugger workflow |
| [Open Graph preview](https://www.ogscope.pk/open-graph-preview) | How to write each OG field and read the preview |

---

## Template

[`templates/basic.html`](templates/basic.html) is a minimal, complete `<head>` with every Open Graph and Twitter card meta tag from this guide. Copy it, replace the `example.com` values, deploy, and then check the URL.

## Contributing

Spotted something outdated? Platforms change their rules often. Open an issue or a PR with a source link.

## License

[MIT](LICENSE) © 2026 Abid Ali
