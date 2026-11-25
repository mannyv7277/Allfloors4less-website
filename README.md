# All Floors 4 Less

Static copy of the live site at [https://www.allfloors4less.com](https://www.allfloors4less.com), a family-owned flooring store in Woodland Hills, CA. Pages are the HTML the site serves. Nothing here is a build step or a redesign.

Cloudflare’s email obfuscation was removed from the saved HTML. `mailto:allfloors4less@gmail.com` links are restored, and the injected `/cdn-cgi/scripts/.../email-decode.min.js` tags are gone. No other edge script (Rocket Loader, Cloudflare Web Analytics) was present.

## Deploy on Cloudflare Pages

Connect this repository. There is no build.

| Setting | Value |
| --- | --- |
| Framework preset | None |
| Build command | *(leave empty)* |
| Build output directory | `/` (repository root) |
| Root directory | `/` |
| Production branch | `main` |

Files are named `page.html` and served **without** `.html` and **without** a trailing slash. That is built-in Pages behavior, verified on the live site: `/about-us` is 200, and both `/about-us.html` and `/about-us/` return **308** to `/about-us`. The same is true for `/blog` (`blog.html`) and each post (`blog/<slug>.html` → `/blog/<slug>`). There is no `_redirects` file because Pages already does this.

`404.html` is the not-found page. A missing URL returns **404** with that HTML. The same file is also a real page at `/404` (200).

`_headers` repeats response headers the live site sends that Pages does not send on its own. See the comments in that file. HTML and `sitemap.xml` are left on the Pages default `Cache-Control: public, max-age=0, must-revalidate`, which matches live.

The apex `https://allfloors4less.com/` **301** to `https://www.allfloors4less.com/` is a Cloudflare zone redirect (or the Pages “primary custom domain” setting). `_redirects` cannot redirect one hostname to another. Keep that redirect in the Cloudflare dashboard when this project is attached to the domain, and set `www` as the primary hostname.

## Folder structure

```
index.html              / 
404.html                /404  and the custom 404 page
about-us.html           /about-us
service.html            /service
contact-us.html         /contact-us
…other top-level pages  /<name>
blog.html               /blog
blog/<slug>.html        /blog/<slug>
site.css                /site.css
favicon.png             /favicon.png  (also used as the apple-touch icon)
og-image.jpg            /og-image.jpg
robots.txt              /robots.txt
sitemap.xml             /sitemap.xml
images/                 shared photos, icons, page heroes
images/blog/            post featured images (and some in-article images)
_headers                Cloudflare Pages response headers (not served as a file)
```

There is no separate CSS or JS bundle per page. Shared styles are `site.css`. Page-specific CSS and scripts are inline in each HTML file. Fonts are loaded from Google Fonts, not stored in this repo.

The homepage, service pages, service-area pages, reviews, contact, privacy, terms, and the instant quote calculator are the top-level HTML files. The blog is 28 posts plus `blog.html`.

## Adding a blog post

The blog already lives at `/blog`. Do not add a second blog directory.

1. Copy a recent post to a new file. Example source: `blog/summer-flooring-trends-for-woodland-hills-homes.html`. Save the copy as `blog/<slug>.html`, with no trailing slash in the filename. The public URL will be `/blog/<slug>`.
2. In the new file, change the title, meta description, canonical URL, Open Graph and Twitter tags, the hero image `src`, the `<h1>`, the `.post-meta` date, and the body inside `<article class="af4l-article">`. Point canonical, `og:url`, and `og:image` at the same host style the other posts already use.
3. Put the card image at `images/blog/<slug>-featured.webp` and reference it as `/images/blog/<slug>-featured.webp`. Recent posts also hotlink extra photos from Firebase Storage inside the article. Those URLs are third-party; leave them as full `https://` links, or replace them with files you add under `images/blog/`.
4. Add a card as the **first** item inside `<div class="bcards" id="af4l-bcards">` in `blog.html`:

```html
<a class="bcard" href="/blog/<slug>"><div class="ph"><img loading="lazy" src="/images/blog/<slug>-featured.webp" alt="Post title"></div><div class="bd"><div class="meta">Flooring Tips · Mon D, YYYY</div><h3>Post title</h3><u>Read more →</u></div></a>
```

The index shows nine cards. Older cards have the `hidden` attribute, and the “Load more posts” button reveals nine at a time. After you insert a new card, add `hidden` to whichever card is now tenth so the first screen stays nine posts.

5. Add a `<url>` entry in `sitemap.xml`, next to the other blog URLs, using the same host and shape those entries already use (no `www`, no trailing slash, no `.html`):

```xml
<url><loc>https://allfloors4less.com/blog/<slug></loc><lastmod>YYYY-MM-DD</lastmod></url>
```

The homepage does not list individual posts. The header and footer are copied into every HTML file, and they already link to `/blog`, so a new post does not require a nav edit. Match the header, footer, and scripts from the post you copied rather than inventing a new layout.

## What is not a static file

These keep working only while the third party stays up. They are not Pages Functions, and this repo has no `functions/` directory.

- **Forms.** The homepage intake form, the contact form, and the instant-quote “send to Ray” form post with `fetch` (`mode: 'no-cors'`) to a Google Apps Script:
  `https://script.google.com/macros/s/AKfycbyjeftlwd52uP75vY8cGTfbacPAwLmLi55ASMinR90F6DDt0jqI28NAht93IUfp-8uY/exec`
  The URL is `window.AF4L_FORM_URL` in each page. `no-cors` means the browser cannot read the script’s response; the page treats a completed request as success. The quote math itself runs in the page. The Apps Script is what stores the lead.
- **Reviews widget.** The homepage and `/reviews` load Elfsight from `https://elfsightcdn.com/platform.js` into `div.elfsight-app-f08b0d66-7785-4cfb-bd7e-b6767add48d6`.
- **Analytics.** Every page loads Google Analytics 4 (`G-0709TS241Q`) and Microsoft Clarity (`z3yjygu84o`) from inline snippets. Those snippets are part of the page, not something Cloudflare injected.
- **Maps.** Showroom and county pages embed Google Maps iframes.
- **Fonts and a few photos.** Barlow Condensed and Inter come from Google Fonts. A handful of hero images are hotlinked from Unsplash. Four blog posts embed article images from Firebase Storage (`sequoia-client-portal.firebasestorage.app`). Those URLs were left as they are.
