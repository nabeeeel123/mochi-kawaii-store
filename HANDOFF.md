# MOCHI — Kawaii Store · Developer Handoff

A complete, working front-end for a kawaii products store. **Static site — no build step, no framework, no dependencies to install.** Open `index.html` and it runs.

---

## 1. What you're getting

| File / folder | What it is |
|---|---|
| `index.html` | **The entire website.** All HTML, CSS and JavaScript in one file. |
| `img/` | 34 product photos (560×560 JPG). These are live on the site. |
| `_spare-photos/` | 56 extra product photos, unused. Reference only — **do not upload.** |
| `_reference/themes.html` | Colour-theme comparison page used during design. **Not part of the site — do not upload.** |
| `HANDOFF.md` | This document. **Do not upload.** |

**Deploy `index.html` + `img/` only.** Total payload ≈ **0.67 MB**.

---

## 2. Deploying it

It is a plain static site, so any host works. Pick one:

**Netlify (easiest)** — go to [app.netlify.com/drop](https://app.netlify.com/drop) and drag in a folder containing `index.html` and `img/`. Live in about 30 seconds. Add the custom domain under *Site settings → Domain management*.

**Cloudflare Pages / GitHub Pages / Vercel** — same idea; connect a repo or upload the folder. No build command, no output directory, no environment variables.

**Traditional hosting (cPanel/FTP)** — upload `index.html` and `img/` into `public_html/`. Done.

Requirements: none. No Node, no PHP, no database, no server-side anything.

### Before going live
1. In `index.html`, set the real domain in `<link rel="canonical">` (currently `REPLACE-WITH-YOUR-DOMAIN.com`).
2. Serve over **HTTPS** (all the hosts above do this automatically).
3. Replace the emoji favicon with a real one if the client has a logo.

---

## 3. How to edit content

### Products
Everything is driven by **one array** near the top of the `<script>` block — search for `const P=[`:

```js
{id:1, img:"p001", n:"Bunny Hoodie Plush", c:"Plush", price:24.90, tag:"Best seller"},
```

| Field | Meaning |
|---|---|
| `id` | Unique number. Must not repeat. |
| `img` | Filename in `img/` **without** the `.jpg` extension. |
| `n` | Product name shown on the card. |
| `c` | Category. **The filter buttons are generated automatically** from these values — add a new category here and its button appears by itself. |
| `price` | Number, no currency symbol. |
| `tag` | Corner badge text, or `null` for none. |

The product grid, the category filters, the 3D ring (first 8 products) and the bag all read from this one array. Change it in one place and everything updates.

### Hero background wall
Search for `const WALL=[` — five arrays of image filenames, one per scrolling column.

### Branding
- Store name: search for `MOCHI` (appears in the `<title>`, preloader, nav and footer).
- Colours: all defined as CSS custom properties in `:root` at the top of the `<style>` block. The theme is **"Y2K Bubblegum"** — pink `#ff4fa3` → violet `#8b5cf6`, on a `#fff0f7` background. Change `--pink`, `--lilac` and `--g-main` and the whole site follows.
- Font: **Baloo 2** from Google Fonts. Note it has **no italic cut** — do not add `font-style:italic` anywhere or the browser will fake-slant it and it will look broken.

---

## 4. IMPORTANT — the cart is a demo

**The bag/cart is front-end only. It does not take payment, and nothing is stored or sent anywhere.** Adding items, quantities, subtotal and the free-shipping threshold all work visually, but "Checkout" only shows a toast message. There is no backend, no order storage, no email.

The client's intended model is a **catalogue that funnels traffic to existing marketplace listings** (Temu / Etsy / TikTok Shop). So the most likely task is:

**Option A — link out to marketplace listings (what the client asked for).**
Add a `url` field to each product in `P`, then change the button in the product grid template from:
```js
<button class="cta line addbtn" onclick="add(${p.id})">Add to bag</button>
```
to:
```js
<a class="cta line addbtn" href="${p.url}" target="_blank" rel="noopener noreferrer">View product</a>
```
Then delete the bag drawer, its markup and the `add/bump/draw/openBag/closeBag` functions, plus the nav bag button.

**Option B — real checkout.** Wire the existing `bag` object to Stripe Checkout, Snipcart, or Shopify Buy Buttons. The bag state is a simple `{productId: quantity}` object, so it maps cleanly onto any of these.

Confirm which one with the client before building.

---

## 5. Verified working

Tested in-browser over HTTP before handoff:

- No JavaScript console errors
- All 34 images return 200 — no broken images
- Baloo 2 webfont loads
- 28 products, 7 category filters, 8 ring cards, 7 bento tiles all render
- Category filtering works (All → 28, Plush → 6)
- Bag: add, quantity +/−, subtotal, free-shipping threshold at $80, open/close all correct
- Mobile menu opens and closes
- Products begin at 1.05 screens down — immediately below the hero

### Bugs already found and fixed (do not reintroduce)
- `overflow-x:hidden` on `html`/`body` **silently breaks `position:sticky`**. The file uses `overflow-x:clip` deliberately.
- 3D ring cards showed mirrored on the far side → fixed with `backface-visibility:hidden`.
- Ring radius was hardcoded and threw cards off-screen on mobile → now responsive, rebuilt on resize (debounced).
- Magnetic hover on product buttons made them dodge the cursor → magnetic effect now only on hero/CTA buttons.
- The ring's animation loop ran forever off-screen and in background tabs → now pauses via `document.hidden` and an IntersectionObserver.
- Hover/tilt effects fired on touchscreens leaving stuck states → gated behind `matchMedia("(pointer:fine)")`.
- A pinned horizontal-scroll section reserved 330vh but travelled 0px on wide screens, causing ~3 screens of dead scrolling → section removed entirely.

---

## 6. Known limitations / suggested next steps

**Should be done before launch**
1. **The photos are AI-generated placeholders.** They are not photographs of real inventory. Selling against them may breach marketplace/advertising rules — replace with real product photography, or confirm with the client that the actual goods match.
2. **The copy is placeholder marketing.** "12,400 objects shipped", "48 countries", "4.9 average from 12,400 happy people", the "300 per batch" tile and the Yuki N. testimonial are all invented. **Publishing invented review counts and ratings is illegal in many jurisdictions (FTC in the US, CMA in the UK).** Replace with real figures or delete these elements.
3. **Missing legal pages** — no Privacy Policy, Terms, Returns or Contact page exists. The footer links are `href="#"` placeholders. Required by most payment processors and by GDPR/CCPA.
4. **Cookie/consent** — none needed as-is (no analytics, no cookies), but that changes the moment analytics is added.

**Nice to have**
5. Convert images to **WebP/AVIF** with `<picture>` fallback — roughly 30–50% smaller.
6. Add `width`/`height` attributes to `<img>` tags to eliminate layout shift (improves Core Web Vitals).
7. **Self-host the font** to remove the Google Fonts request (faster, and avoids the GDPR question about Google receiving visitor IPs).
8. Add `sitemap.xml` and `robots.txt`.
9. Analytics (Plausible/Fathom are cookieless; GA4 needs a consent banner).
10. Individual product pages — currently single-page only, which limits SEO to one URL.
11. The three floating hero cards use a CSS float animation that is overridden by the JS mouse-parallax on desktop. Harmless, but the animation effectively stops after first mouse move. Move the animation to an inner wrapper element if you want both.

---

## 7. Browser support

Modern evergreen browsers (Chrome, Edge, Firefox, Safari). Uses CSS custom properties, `aspect-ratio`, `backdrop-filter`, `overflow: clip`, CSS 3D transforms, IntersectionObserver and Pointer Events. **No IE11 support** and none is practical. `prefers-reduced-motion` is fully honoured — all animation is disabled for users who request it.

---

## 8. Questions for the client

1. Marketplace links or real checkout? (Section 4)
2. Real store name, logo and brand colours?
3. Real product names, prices and photos?
4. Domain name, and who owns it?
5. Real shipping rates and returns policy — the site currently claims free shipping over $80 and 30-day returns.
