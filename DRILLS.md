## Practice drills — the page, and how to work through it

This page exists so that every drill below happens on something real, not in a video. It is a
single file with no dependencies, deliberately containing the failure modes that show up in
almost every real container: a duplicate listener, a missing attribute, a fetch-based form the
native trigger can't see, a router whose page changes nothing listens for, a purchase that leaks
stale items, and an untracked button.

**Read the notation section once, then never again.** It is the shorthand the whole file uses.

### How to read the steps

- **Bold text** is the exact label on a button, tab, menu or field. Click or type into the things
  in the order they are written. An arrow means a nested path: **Admin → Data streams** means
  click *Admin*, then *Data streams* inside it.
- `Code text` is something you type or paste exactly as shown. Don't retype it by hand — copy it.
- `{{Something}}` is a GTM variable. You type two curly braces, then the variable's name. GTM
  shows these in blue when it recognises the name, and in red or plain text when it doesn't.
- **Check:** tells you what you should see on screen before you carry on. If you don't see it,
  stop and fix it — every later step assumes the earlier ones worked.
- **If it fails:** the usual cause, in the order worth checking.
- Do the steps in the order written. GTM is genuinely order-sensitive: variables must exist
  before a tag can reference them, and triggers must exist before a tag can use them.

### What you need before you start

| Thing | Why | Where |
|---|---|---|
| A Google account | One account can hold a GA4 property and a GTM container | Any personal account is fine |
| Chrome, signed in | GTM Preview and the analytics debugger work best in Chrome | — |
| Two browser tabs | One for GTM, one for GA4. You will switch between them constantly | — |
| A plain text editor | To paste your container snippet into `index.html` | Any editor; VS Code, TextEdit, Notepad |
| Python 3 (optional) | Only to serve the page locally on your machine | Pre-installed on macOS and most Linux; `python.org` on Windows |

Total setup time: about 25 minutes. You only do it once.

---

## Part 1 — Set up (once, ~25 min)

Work top to bottom. Each step ends with a **Check**.

### Step 1 — Create a GA4 property (~8 min)

This is where events will land. "Property" is Google's word for one place that collects data for
one website.

1. Go to [analytics.google.com](https://analytics.google.com) and sign in.
2. Click **Admin**. It's the gear icon at the bottom of the left-hand menu, not the top.
3. In the **Account** column, click **Create**, then **Property**.
4. Type a name: `Practice — Northwind`. Set your time zone and currency. Click **Next**.
5. Answer the business questions however you like — they don't affect anything you're practising.
   Click **Create**.
6. When asked to choose a platform, click **Web**.
7. **Website URL:** type `https://northwindstore.netlify.app/`.
   **Stream name:** type `Northwind practice`.
   Click **Create stream**.
8. A panel opens. Find the **Measurement ID** — it looks like `G-` followed by about ten
   characters. Copy it somewhere you can paste from.

**Check:** you have a string starting `G-` copied. If you only see `G-XXXXXXX` with Xs in it, you're
looking at the example in help text, not your real ID — scroll up to **Stream details**.

**If it fails:** no **Create** button in the Account column means you're not an Administrator on
any account. Sign out and back in with a personal Google account rather than a work one, or create
a fresh account.

### Step 2 — Create a GTM container (~5 min)

GTM is where the tags live. "Container" is Google's word for the box of configuration for one
site.

1. Go to [tagmanager.google.com](https://tagmanager.google.com).
2. Click **Create Account**. Account name: `Practice`. Country: yours. Click **Continue**.
3. Container name: `Northwind practice`. Target platform: **Web**. Click **Create**.
4. Accept the terms. Two code blocks appear, each in a box with a **Copy** button.

**Check:** you can see two snippets — one labelled for `<head>`, one for `<body>`. You do not need
to copy them if you take the shortcut in Step 3, but you should know what they are: the head one
loads GTM, the body one is a fallback for browsers with JavaScript off.

### Step 3 — Point `index.html` at your container (~2 min)

The page already contains a GTM loader line, so you only need to change one identifier in it.
Skipping this step is the single most common reason Preview connects and then shows nothing.

1. Open `practice/index.html` in your text editor.
2. Near the top, find this line:

   ```html
   <script async src="https://www.googletagmanager.com/gtm.js?id=GTM-P65RK7H6"></script>
   ```

3. Your own container ID is the `GTM-XXXXXXX` shown in the GTM header, top right of the screen,
   next to the container name. Replace `GTM-P65RK7H6` in that line with your own ID. Keep the
   quotes and the rest of the line exactly as it is.
4. Save the file.

**Check:** the line now reads `gtm.js?id=` followed by *your* container ID, and the two comment
markers `<!-- GTM HEAD SNIPPET GOES HERE -->` and `<!-- GTM BODY SNIPPET GOES HERE -->` are still
in the file. You can leave them; they mark where the snippets would go on a real site.

**If it fails:** if you paste the full snippet from GTM instead, you'll end up with two copies of
the script and **every event will fire twice** — which you'd then spend an hour debugging as a
bug in the page. Change the ID only.

### Step 4 — Open the page, and learn the redeploy loop (~3 min)

The page is already deployed and live at **https://northwindstore.netlify.app/**. You don't need a
local server at all — open that URL in Chrome and you're working.

The one thing to understand before you start is that **the deployed site is not the file on your
disk**. Editing `index.html` locally changes nothing on the live site until you redeploy. Several
drills ask you to edit the page, so learn the loop now:

1. Open `https://northwindstore.netlify.app/` in Chrome and confirm you see the shop.
2. When a drill says "edit `index.html`", you make the change in your editor, **save**, then
   redeploy:
   - **If you deploy by drag-and-drop:** open your site on [app.netlify.com](https://app.netlify.com),
     go to the **Deploys** tab, and drag the `practice` folder onto the drop zone. It takes about
     twenty seconds.
   - **If the site is connected to a Git repo:** commit and push, and Netlify builds automatically.
3. Wait for the deploy to show **Published**, then reload the page in Chrome.

**Check:** you see the Northwind Outdoor shop with four product cards and a newsletter button.

**If it fails:** after a redeploy, an old version can be cached. Force a refresh with
**Ctrl+Shift+R** (Windows/Linux) or **Cmd+Shift+R** (Mac). If the page shows a "Page not found"
after deploying, check you dragged the folder that actually *contains* `index.html`, not its parent.

> **Say this in the room, and mean it:** "I deployed it to a real https URL rather than opening the
> file directly — dataLayer and Preview both behave properly, and it's the same as production."
>
> That sentence alone signals you've done this before. It's also why this page is deployed rather
> than served locally: a `file://` URL breaks some browser APIs and makes Preview awkward.

**One consequence to keep in mind.** Because the site is deployed, every edit in a drill is a
two-part action: change the file, then redeploy. Any step below that says "save and reload" means
"save, redeploy, reload". If a change you made doesn't appear in Preview, redeploy is the first
thing to check — before you go looking for a bug in GTM.

### Step 5 — Connect Preview (~3 min)

Preview is a panel that shows you, live, which tags fired on which event. It is the tool the whole
assessment is built around. Learn it now.

1. In GTM, click **Preview** in the top right. (Older versions open a dialog asking for the URL;
   enter `https://northwindstore.netlify.app/` and click **Connect**.)
2. A new tab opens — Tag Assistant, or the debug panel attached to your page. If it opens
   `tagassistant.google.com`, it will ask you to **Add domain** or choose a connected target. Pick
   your page.
3. Switch back to the `https://northwindstore.netlify.app/` tab. A debug panel appears along the
   bottom or in a side pane.

**Check:** the panel's left column lists events, with `Container Loaded`, `DOM Ready` and `Window
Loaded` at the top. You can click one and see tabs for **Tags**, **Variables**, **Data Layer** and
**Summary**.

**If it fails:** "Your container is not installed on the page" means Step 3's ID doesn't match the
container you're previewing — compare the two IDs character by character. If the panel never
appears, an ad blocker or a privacy extension is the usual cause; disable it for
`northwindstore.netlify.app`, then reload.

### Step 6 — Turn on a second opinion (~2 min)

Preview shows what GTM did. DebugView shows what **GA4** received. They are different, and telling
them apart is the core skill here.

1. In GA4, click **Admin**, then under **Data display** click **DebugView**.
2. Leave that tab open. Preview mode sets `debug_mode` automatically, so events you trigger with
   Preview connected should appear here within a few seconds.

**Check:** you can name both windows without hesitating — Preview (left) and DebugView (right).

**If it fails:** DebugView says "No devices found" until it receives an event in debug mode. Make
a click on the page with Preview connected, then check again.

### Ready-check

You're ready for the drills when all five are true. Don't start Session 1 until they are.

- [ ] `https://northwindstore.netlify.app/` shows the shop.
- [ ] GTM Preview connects and lists `Container Loaded`.
- [ ] The page's GTM ID matches the container you're previewing.
- [ ] GA4 → Admin → DebugView opens without error.
- [ ] You can find your GA4 **Measurement ID** (`G-…`) without hunting.
- [ ] You know how to redeploy, and a redeploy you did just now showed **Published**.

---

## Part 2 — The six bugs planted in the page

Find them by reading the `<script>` block at the bottom of `index.html`, then prove each one in
Preview. **Do not fix anything yet.** Diagnosing before fixing is the skill being tested — and in
the real session, the diagnosis is what they're watching for.

Open `index.html` in your editor alongside this table. All six live in the single `<script>` block
near the end of the file.

| # | What's wrong | Where to look in the file | How to see it happen | Why it matters |
|---|---|---|---|---|
| 1 | Cards 1 and 2 attach **two click listeners** — the second is a *different function object*, so the browser doesn't deduplicate it — and `add_to_cart` pushes twice | `document.querySelectorAll('[data-add]').forEach(...)` — the line with `if (i === 0 \|\| i === 1)` | Click *Add to cart* on the tent, and count the `add_to_cart` events in Preview. There are two | Classic double-fire. Inflates conversion counts. The most common real-world tracking bug there is |
| 2 | The lantern card has **no `data-item-category`**, so the push contains `undefined` | the `<article class="card">` for `SKU-LAMP-04` | Add the lantern, open the `add_to_cart` event's **Data Layer** tab, look at `item_category` | GA4 receives the string `undefined`, which pollutes the dimension and can never be filtered out cleanly |
| 3 | The contact form submits over `fetch`, so GTM's **Form trigger never fires** | the `#enquiryForm` submit handler | With a Form trigger attached, submit the form in Preview. No event appears | The single best interview talking point in this whole file |
| 4 | *(Already fixed for you — see below.)* The router pushes `virtual_pageview` but nothing in GTM listens for it | `function route()` already contains the push | Click Shop → Cart with Preview open. A `virtual_pageview` event appears, and no tag fires on it | SPA measurement — appears in most agency audits |
| 5 | `purchase` **doesn't clear the `ecommerce` object** first, so stale items can leak between transactions | the `#checkoutBtn` click handler | Add two items, check out, then check out again with one item — read both `ecommerce` objects in the Data Layer tab | GA4 can carry the previous transaction's items into the next one |
| 6 | Newsletter click is **completely untracked** | the `#newsletterBtn` click handler — it only calls `alert()` | Click it. Nothing appears in Preview at all | Your clean-slate build task in Session 2 |

### A note on bug 4

The page **already contains** the fix — `route()` pushes a `virtual_pageview` event with
`virtualPagePath` and `virtualPageTitle`. What's missing is the GTM side that listens for it.

That's deliberate, and it's the more realistic drill: on a real site the developer owns the push
and you own the container. If you want the harder version, delete the `window.dataLayer.push({...})`
block inside `route()` yourself, reload, and write it back from the example in Session 2.3.

---

## Part 3 — The drills

Seven sessions, ~3 hours each. Do them in order — each depends on the one before. Every session
ends with something you can demonstrate, not something you've read.

### Session 1 — Preview mode fluency

**Time:** ~3 hours. **This is the highest-value 3 hours in the list.** **Needs:** Steps 1–6 done.

**Goal:** you can explain the Preview interface out loud without hunting for anything.

1. Open `https://northwindstore.netlify.app/`. In GTM click **Preview**, and confirm the panel is
   connected (setup Step 5).
2. On the page, click **Add to cart** on the tent.
3. In the Preview panel's left column, a new event appears. Click it.
4. Work through its four tabs, saying out loud what you see in each:
   - **Tags** — which tags fired, which didn't, and the reason given for each. Greyed out with a
     reason means "considered and skipped"; red means an error.
   - **Variables** — what GTM knew at that moment. Blank means `undefined`.
   - **Data Layer** — the pushed object, as it existed at that event.
   - **Summary / Errors** — anything that went wrong.
5. Repeat with the lantern. Watch `item_category` arrive as `undefined`.
6. Click the tent again, and count how many `add_to_cart` events appear. Find the duplicate.

**Check:** for any event you click, you can say which tags fired, which variable was empty, and
what the push contained, without clicking anything new.

**Pass bar:** you can talk for 60 seconds about what's on screen without touching the mouse.

### Session 2 — Build the missing tracking

**Time:** ~3 hours. **Needs:** Session 1. **Do these in order, one at a time.** Each one follows the
same rhythm: **create → Preview → verify → publish.** Resist publishing more than one change at a
time; if something breaks you'll want to know which change did it.

The GTM order is always the same, for the same reason every time:

> **variables first** (the values) → **triggers** in the middle (the when) → **tags** last (the what).

Going the other way round means your tag form shows `{{Unknown variable}}` and you have to go back.

#### 2.1 — Track the newsletter click (~30 min)

The page gives you nothing to listen to: `#newsletterBtn` has no dataLayer push, so a Custom Event
trigger has nothing to match. So the first version listens to the click itself.

1. **Trigger.** In GTM, click **Triggers** in the left menu, then **New**.
2. Click the **Trigger Configuration** box. Choose **Click - All Elements**.
3. Leave **This trigger fires on** as **Some Clicks** — *not* **All Elements**, which would fire
   on every click on the page.
4. Set the condition row to: **Click Element** · **matches CSS selector** · `#newsletterBtn`.
5. Name the trigger `Click - Newsletter` (top left of the form), then **Save**.
6. **Tag.** Click **Tags → New**. Click **Tag Configuration**. Choose **Google Analytics**, then
   **Google Analytics: GA4 Event**.
7. In **Measurement ID**, paste your `G-…` ID. (If you've already built a Google tag, you can
   select it from the dropdown instead, labelled **Configuration Settings**.)
8. **Event Name:** type `newsletter_click`.
9. **Triggering:** click the box, choose `Click - Newsletter`, then **Save**.
10. Click **Submit** (top right), give the version a name like `Newsletter click`, then **Publish**,
    then **Continue**.

**Check:** with Preview connected and the page reloaded, click the newsletter button. A **Click**
event appears in the stream, and `GA4 - newsletter_click` appears under **Tags Fired**. Within a
minute it should also appear in GA4 → **Admin → DebugView**.

**If it fails:** no **Click** event at all means the container isn't on the page — go back to
setup Step 3. A **Click** event with the tag greyed out means the condition didn't match; click the
event, open **Variables**, and check what **Click Element** actually resolves to.

**Then do it properly (~15 min).** A CSS selector is a fragile contract — someone renames the ID
and tracking dies silently. The maintainable version has the page announce the click:

1. In `index.html`, inside the `#newsletterBtn` click handler, add this push **above** the `alert`:

   ```js
   window.dataLayer.push({ event: 'newsletter_signup', signup_source: 'shop_page' });
   ```

2. Save, redeploy, then reload the page (setup Step 4 if you need the redeploy steps).
3. In GTM, create a new **Trigger**: type **Custom Event**, **Event name** `newsletter_signup`,
   fires on **All Custom Events**. Name it `CE - newsletter_signup`.
4. Either repoint the existing tag at the new trigger, or build a new tag with event name
   `newsletter_signup` and the new trigger. Publish.
5. Be ready to say why the second way is better: the page owns the truth about what happened, the
   selector breaks on someone else's redesign, and you no longer depend on the DOM's shape.

#### 2.2 — Track the contact form (~45 min) — *the one that comes up in the interview*

Because of bug 3, you cannot use the built-in Form trigger: the page calls `preventDefault()` and
submits with `fetch`, so the browser's native submit event never completes the way the trigger
expects. The page **already pushes** `generate_lead` (with `form_name` and `form_topic`) when the
form is submitted. Your job is to receive it.

Build in this order:

1. **Variable 1.** **Variables → New** → click **Variable Configuration** → choose **Data Layer
   Variable**. Name it `DLV - form_topic`. In **Data Layer Variable Name**, type `form_topic`
   (exactly — this is the key in the push). **Save**.
2. **Variable 2.** Same again: name `DLV - form_name`, Data Layer Variable Name `form_name`.
3. **Trigger.** **Triggers → New** → **Trigger Configuration** → **Custom Event**. **Event name:**
   `generate_lead`. Leave **This trigger fires on** as **All Custom Events**. Name it
   `CE - generate_lead`. **Save**.
4. **Tag.** **Tags → New** → **Tag Configuration → Google Analytics → Google Analytics: GA4
   Event**. Measurement ID as before. **Event Name:** `generate_lead`.
5. In **Event Parameters**, click **Add parameter** twice:
   - `form_topic` = `{{DLV - form_topic}}`
   - `form_name` = `{{DLV - form_name}}`
6. **Triggering:** attach `CE - generate_lead`. **Save**, **Submit**, **Publish**.

**Check:** reload with Preview connected, go to **Contact**, fill in the form, submit it. In the
stream:

- a `generate_lead` event appears,
- its **Data Layer** tab shows `form_name` and `form_topic`,
- its **Variables** tab shows both DLV variables with real values rather than `undefined`,
- its **Tags** tab shows your tag fired.

**If it fails:** the event never appears → the push isn't running, so check you're on the Contact
view and that the form actually submitted (the "Thanks" note should replace it). Tags fired but
the parameters are `undefined` → your **Data Layer Variable Name** doesn't match the key exactly,
including case.

#### 2.3 — Virtual pageview (~40 min)

In a single-page app the browser never reloads between views, so GA4 gets no `page_view` for them
and GTM can't see the change. The page's code has to push an event, and you build the GTM side
that listens for it.

**How to tell this is needed on a real site:** click through it in Preview. If the content changes
but no new Page View appears in the event stream, it's a single-page app. Other clues: `#/` in the
URL, `<div id="root">` or `<div id="app">` in the source, or only XHR/fetch calls in the Network
tab when you navigate. Dev teams won't always tell you, so check it yourself and ask: *"Do you push
a dataLayer event on route change?"*

The push is already in `index.html`. Read it in `route()` before you build anything:

```js
window.dataLayer.push({
  event: 'virtual_pageview',
  virtualPagePath: '/' + h,
  virtualPageTitle: document.title + ' – ' + h
});
```

That's the shape of every push: an `event` key GTM triggers on, plus whatever data the tags need.

1. **Prove the push exists.** Reload with Preview connected, click **Cart** in the nav. A
   `virtual_pageview` event should appear in the stream, with the two `virtual…` keys in its
   **Data Layer** tab.
2. **Variable 1.** **Variables → New → Data Layer Variable.** Name `DLV - virtualPagePath`, key
   `virtualPagePath`. **Save**.
3. **Variable 2.** Same: name `DLV - virtualPageTitle`, key `virtualPageTitle`. **Save**.

   > The name field is the label GTM shows in your tag. The **Data Layer Variable Name** field is
   > the key inside the push. They are different fields, and mixing them up is the bug you'll hit.

4. **Trigger.** **Triggers → New → Custom Event**, event name `virtual_pageview` (exact match, no
   capitals). Name it `CE - virtual_pageview`. **Save**.
5. **Tag.** **Tags → New → Google Analytics: GA4 Event**. Name `GA4 - Virtual Page View`. **Event
   Name:** `page_view`.
6. Add parameters:
   - `page_location` = `https://northwindstore.netlify.app{{DLV - virtualPagePath}}`
   - `page_title` = `{{DLV - virtualPageTitle}}`
7. Attach `CE - virtual_pageview`. **Save**, **Submit**, **Publish**.

**Check:** in Preview, click between Shop, Contact and Cart. Each click adds a
`virtual_pageview` event, and `GA4 - Virtual Page View` appears under **Tags Fired** with a
`page_location` that ends in the view name.

**If it fails:** if `page_location` reads `https://northwindstore.netlify.app/undefined`, the variable name and
the dataLayer key don't match — check both are `virtualPagePath`, capital P, capital P.

> **Watch for double-counting.** GA4's enhanced measurement also sends `page_view` on history
> changes on some setups. If you see two, either turn that setting off for history changes or send
> a distinctly-named event instead and report on that. Being able to say this is worth more than
> the tag itself.

#### 2.4 — Scroll depth on the shop page (~20 min)

1. **Trigger.** **Triggers → New → Trigger Configuration → Scroll Depth**.
2. Tick **Vertical Scroll Depths**, and set the percentages to `25, 50, 75, 90`.
3. Tick **Percentages** (not Pixels). Leave **Horizontal Scroll Depths** off.
4. Add a condition so it only fires on the shop view: **This trigger fires on → Some Scroll
   Depths**, then `{{Page Path}}` **equals** `/`.
5. Name it `Scroll - Shop`, **Save**.
6. **Tag.** **Tags → New → GA4 Event**. Event name `scroll`. Add parameter `percent_scrolled` =
   `{{Scroll Depth Threshold}}`. Attach the trigger. **Save**, **Submit**, **Publish**.

**Check:** in Preview, scroll down the shop page slowly and watch four events appear as you cross
each threshold.

**If it fails:** nothing at all → the page may be too short to scroll. In the console, run
`document.body.scrollHeight` and compare with your window height; a page that fits on one screen
has no scroll distance to measure.

### Session 3 — Variables, properly

**Time:** ~2 hours. **Needs:** Session 2.

This targets the thing most people find wobbly. Build one of each type, and be able to say the
difference between all six out loud.

All of them start the same way: **Variables → New → Variable Configuration**, then pick the type.

| Type | What to type into the form | What it should return |
|---|---|---|
| **Data Layer Variable** | Name `DLV - item name`; **Data Layer Variable Name** `ecommerce.items.0.item_name` | The tent's name, once a cart push has happened |
| **Constant** | Name `Constant - GA4 Measurement ID`; **Value** your `G-…` | Your measurement ID, everywhere you need it |
| **URL** | Choose **URL**; **Component type** `Query`; **Query key** `utm_source` | A UTM value — will be empty until you visit with `?utm_source=test` on the URL |
| **DOM Element** | Choose **DOM Element**; **Selection Method** `CSS Selector`; **Element Selector** `.price` | The first price on the page, as text |
| **Custom JavaScript** | Name `CJS - clean path`; paste the function below | The page path with no query string |
| **Auto-event (built-in)** | **Variables → Configure** (top of the Variables list) → tick under *Clicks*: `Click Classes`, `Click Text`, `Click Element`; under *Pages*: `Page Path` | Whatever the last click carried |

For the Custom JavaScript one, paste this into the code box:

```js
function () {
  return window.location.pathname;
}
```

A Custom JavaScript variable must be a function that **returns** something. It doesn't run until
a tag that uses it fires, which is why you test it through a tag rather than on its own.

**How to test any variable without building a tag:** create a throwaway **Custom HTML** tag with
this inside, attach it to **All Pages**, then read the console:

```html
<script>console.log('VAR:', {{CJS - clean path}});</script>
```

**Delete it before publishing.**

**Check — the pass bar:** for each of the six, you can name it, name its type, and say what it
returns *right now* on the page. If a value is empty, explain why rather than guessing.

**If it fails:** `{{Unknown variable}}` in a tag means the variable was renamed after the tag
referenced it, or it lives in a different workspace. `undefined` at runtime means the variable
resolved but the data wasn't there yet.

### Session 4 — The GA4 side

**Time:** ~2 hours. **Needs:** at least some events flowing from Session 2.

1. **Mark key events.** GA4 → **Admin → Data display → Key events → New key event**. Type
   `purchase`, **Save**. Repeat for `generate_lead`. (Older properties label this **Conversions**.)
2. **Register custom dimensions.** **Admin → Data display → Custom definitions → Create custom
   dimension**. Do this once for each of `form_topic`, `item_category`, `form_name`.
   **Scope: Event** for all three. The **Event parameter** field must match the parameter name
   exactly, character for character.
3. **Write down why user scope would be wrong here.** One sentence is enough: these are facts
   about *an event that happened*, not facts about the person. A user-scoped dimension can only
   hold one value per user, so it would show whichever value was seen last.
4. **Build a funnel.** **Explore → Funnel exploration → Create**. Set the steps to `view_item` →
   `add_to_cart` → `begin_checkout` → `purchase`. Note that this page emits neither `view_item`
   nor `begin_checkout` — write down how you'd add them, don't build them yet.
5. **Compare timings.** Leave **Reports → Realtime** open while clicking on the page, then compare
   with **Reports → Engagement → Events**. Notice the delay, and be able to explain it: Realtime is
   near-instant, standard reports are batched and refreshed on a delay of up to a day or so.

**Check:** you can point at `purchase` in the Key events list, name three dimensions you created
with their scope, and explain the Realtime-vs-Reports gap in one sentence.

### Session 5 — JavaScript, from the page up

**Time:** ~2–3 hours. **Needs:** Sessions 1–2.

Open devtools on the page (**F12**, or right-click → **Inspect**) and work in the **Console** tab.

1. Type `window.dataLayer` and press Enter. Expand the result and find your `add_to_cart` object.
2. Type `window.dataLayer.length`. Then push a test event by hand:
   `window.dataLayer.push({event:'test'})`. Check the length went up by one and that the event
   appears in Preview.
3. Type `document.querySelectorAll('[data-add]').length`. **Check:** it returns `4`.
4. Write a `closest()`-based delegated listener in the console that logs the item name of whatever
   card was clicked:

   ```js
   document.addEventListener('click', function (e) {
     var card = e.target.closest('.card');
     if (card) console.log('clicked', card.dataset.itemName);
   });
   ```

   Click each card and confirm the name logs. Then move it into a **Custom HTML** tag in GTM, with
   a trigger of your choice, and confirm it still works.
5. Patch `history.pushState` so you can see the mechanism the SPA router uses:

   ```js
   var original = history.pushState;
   history.pushState = function () {
     original.apply(this, arguments);
     console.log('route changed to', location.hash);
   };
   ```

   Click the nav links. **Check:** each click logs a line.

**Check:** all five steps produce something visible in the console or Preview, not just a plausible
result.

### Session 6 — Wire up a dashboard

**Time:** ~1 hour. **Needs:** events flowing.

Connect Looker Studio to your GA4 property and build this, in under 30 minutes:

1. **Create → Report**, add the GA4 data source.
2. Add a **date range control** first.
3. Four **scorecards**: sessions, engaged sessions, key events, key event rate.
4. One **time series**: date on the x-axis, sessions as the metric.
5. One **bar chart**: event name on the category axis, event count as the metric.
6. One **table**: channel group, sessions, key events.
7. One **filter control** that drives every chart.

Then critique it out loud as if it were someone else's: what decision does it support, what's
missing for context, is any rate averaged instead of recalculated.

**Check:** every chart moves when you change the date range or the filter.

### Session 7 — Rehearse under observation

**Time:** ~2 hours. **Needs:** everything above.

1. Screen-record yourself doing Session 2.2 from scratch, talking the whole time.
2. Watch it back without pausing.
3. Note every moment you dislike, with a timestamp.

You will hear the silences. That recording is the single most useful artefact of the week.

---

## Troubleshooting — the eight things that usually go wrong

Work down this list before assuming anything is broken.

| Symptom | Most likely cause | Fix |
|---|---|---|
| Preview says "container not installed" | The `GTM-` ID in `index.html` isn't the container you're previewing | Compare the two IDs character by character; fix Step 3 |
| Preview connects but the event stream stays empty | Ad blocker, or a privacy extension blocking `googletagmanager.com` | Disable it for `northwindstore.netlify.app` and reload |
| Tags are greyed out in Preview | The trigger condition didn't match — this is normal and informative, not an error | Open the event's **Variables** tab and compare the real value with your condition |
| `{{Unknown variable}}` in a tag | Variable renamed, or it doesn't exist yet | Re-match the name; create the variable before the tag |
| Parameter arrives as `undefined` | The Data Layer Variable's **key** doesn't match the push, or the push happens after the tag fires | Check the key's exact spelling; check the **Data Layer** tab ordering |
| DebugView is empty but Preview shows the tag fired | Consent, an internal traffic filter, or the tag is pointed at a different measurement ID | Check the Measurement ID in the tag against **Admin → Data streams** |
| Everything fires twice | Two GTM snippets on the page, or the listener is attached twice — bug 1 is real | Count the tags in the page source; check `querySelectorAll` in the console |
| `Address already in use` from Python | Only applies if you chose to run a local server after all | Use port 8001, or just use the deployed URL |

---

## What to deliberately skip

Be honest with yourself about the maths of one week:

- **Do not learn the gcloud CLI.** The job ad never mentions a terminal. Skip it entirely, guilt-free.
- **Do not memorise BigQuery.** Do the one UNNEST pattern in `04-bigquery-gcp-playbook.md` until you
  can write it from memory; treat everything else as "I'd look that up".
- **Do not attempt server-side tagging.** Know the one-sentence definition and move on.
- **Do not build a portfolio of dashboards.** One good one you can explain beats five you can't.
