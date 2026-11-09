## Practice target: how to use `index.html`

This page exists so that every drill below happens on something real, not in a video. It is a
single file with no dependencies, deliberately containing the four failure modes that show up in
almost every real container: a duplicate listener, a missing attribute, a fetch-based form the
native trigger can't see, and a SPA router that never announces a page change.

### Setup (once, ~15 minutes)

1. **Create a GA4 property.** [analytics.google.com](https://analytics.google.com) → **Admin → Create → Property**.
   Name it `Practice — Northwind`. Industry: Retail. Create a **Web** data stream pointing at
   `https://northwindstore.netlify.app` and copy the Measurement ID (`G-XXXXXXX`).
2. **Create a GTM container.** [tagmanager.google.com](https://tagmanager.google.com) → **Create account → Web**.
   Copy the two snippet blocks it gives you.
3. **Paste them into `index.html`** at the two marked comments: the head snippet in `<head>`, the body
   snippet after `<body>`. Save.
4. **Deploy to Netlify.** The page is hosted at `https://northwindstore.netlify.app`. After each edit to
   `index.html`, redeploy (drag the `practice` folder onto the site's **Deploys** page in Netlify, or
   use `netlify deploy --prod`). Do not open the file by double-clicking: `file://` URLs break some
   browser APIs and make Preview mode awkward.
5. **Turn on debug mode for testing.** Install the **Google Analytics Debugger** extension, or add
   `?debug_mode=true` to the URL and read events in **Admin → DebugView**.

> **SAY in the room:** "I deployed it to a real https URL rather than opening the file directly —
> dataLayer and Preview both behave properly, and it's the same as production."

That sentence alone signals you've done this before.

---

## The bugs planted in the page

Find them by reading the `<script>` block, then prove each one in Preview mode. Do not fix them
yet — diagnosing before fixing is the skill being tested.

| # | Bug | Where | Why it matters |
|---|---|---|---|
| 1 | Cards 1 and 2 have **two click listeners**, so `add_to_cart` fires twice | `querySelectorAll('[data-add]').forEach(...)` — the `if (i === 0 \|\| i === 1)` line | Classic double-fire. Inflates conversion counts. The most common real-world tracking bug there is |
| 2 | The lantern card has **no `data-item-category`**, so it pushes `undefined` | the `SKU-LAMP-04` article | GA4 receives the string `"undefined"`, which pollutes the dimension |
| 3 | The contact form uses `fetch`, so GTM's **Form trigger never fires** | `#enquiryForm` submit handler | The single best interview talking point in this whole file |
| 4 | The router changes views but **never pushes a pageview** | `function route()` | SPA measurement — appears in most agency audits |
| 5 | `purchase` **doesn't reset the ecommerce object** first | checkout handler | Stale items leak between transactions |
| 6 | Newsletter click is **completely untracked** | `#newsletterBtn` | Your clean-slate build task |

---

## Drill sequence — seven sessions, ~3 hours each

Do them in order. Each ends with something you can demonstrate, not something you've read.

### Session 1 — Preview mode fluency (the highest-value 3 hours in this list)

Goal: you can explain the GTM Preview interface out loud without hunting for anything.

1. Open `https://northwindstore.netlify.app`, click **Preview** in GTM, connect to the tab.
2. Click **Add to cart** on the tent. In the Preview event stream, find the `add_to_cart` event.
3. Open each of its four tabs and say out loud what you see:
   **Tags** (which fired, which didn't, and the reason), **Variables** (what GTM knew at that moment),
   **Data Layer** (the pushed object), **Summary/Errors**.
4. Repeat for the lantern — watch `item_category` come through as `undefined`.
5. Click the tent again and count how many `add_to_cart` events appear. Find the duplicate.

**Pass bar:** you can talk for 60 seconds about what's on screen without clicking anything new.

### Session 2 — Build the missing tracking

Build these in GTM, in this order. Each one: create → Preview → verify → publish.

1. **Newsletter click.** Custom Event trigger won't help — there's no dataLayer push. Use a
   **Click - All Elements** trigger with a CSS selector on `#newsletterBtn`, then a GA4 Event
   tag sending `newsletter_click`. *Then* fix it properly by adding the push to the page and
   switching to a Custom Event trigger — and be ready to say why the second way is better.
2. **Contact form.** Because of bug 3, you cannot use the Form trigger. Use a **Custom Event**
   trigger on `generate_lead` and map `form_topic` through a **Data Layer Variable**.
   This is the drill that will come up in the interview.
3. **Virtual pageview.** Add the push inside `route()`, then a Custom Event trigger on it.
4. **Scroll depth** on the shop page.

### Session 3 — Variables, properly

This targets the thing you flagged as wobbly. Build one of each, and be able to say the
difference between all six:

| Type | Build it to capture |
|---|---|
| Data Layer Variable | `ecommerce.items.0.item_name` from the cart push |
| Constant | your GA4 Measurement ID |
| URL | the query string parameter `utm_source` |
| DOM Element | the text of the first `.price` on the page |
| Custom JavaScript | a variable returning the page path *without* query string |
| Auto-event (built-in) | enable `Click Classes`, `Form ID`, `Page Path` — then explain what each holds |

**Pass bar:** for each, name it, name its type, and say what it returns right now on the page.

### Session 4 — GA4 side

With at least some events flowing:

1. Mark `purchase` and `generate_lead` as **key events**; check them in **Admin → Key events**.
2. Register custom dimensions for `form_topic`, `item_category`, `form_name` — **event** scope.
   Write down why user scope would be wrong for these.
3. Build a **funnel exploration**: `view_item` → `add_to_cart` → `begin_checkout` → `purchase`.
   Note that this page emits no `view_item` or `begin_checkout` — decide how you'd add them.
4. Read **Realtime** while clicking, then compare with **Reports → Engagement → Events** and
   notice the delay. Be able to explain that delay.

### Session 5 — JavaScript, from the page up

Open devtools on the page and work in the console:

1. `window.dataLayer` — expand it, find your `add_to_cart` object.
2. `window.dataLayer.length` — then push a test event manually and watch it grow.
3. `document.querySelectorAll('[data-add]').length` — four.
4. Write, in the console, a `closest()`-based delegated listener that logs the item name of whatever
   card was clicked. Then convert it into a **Custom HTML tag**.
5. Patch `history.pushState` and confirm your virtual pageview fires on route change.

### Session 6 — Wire up a dashboard

Connect Looker Studio to your GA4 property and build, in under 30 minutes: four scorecards
(sessions, engaged sessions, key events, key event rate), a time series, a bar chart by event name,
a table, and a date filter. Then critique it out loud as if it were someone else's.

### Session 7 — Rehearse under observation

Screen-record yourself doing Session 2 from scratch, talking the whole time. Watch it back.
You will hear the silences. That recording is the single most useful artefact of the week.

---

## What to deliberately skip

Be honest with yourself about the maths of one week:

- **Do not learn the gcloud CLI.** The job ad never mentions a terminal. Skip it entirely, guilt-free.
- **Do not memorise BigQuery.** Do the one UNNEST pattern in `04-bigquery-gcp-playbook.md` until you
  can write it from memory; treat everything else as "I'd look that up".
- **Do not attempt server-side tagging.** Know the one-sentence definition and move on.
- **Do not build a portfolio of dashboards.** One good one you can explain beats five you can't.
