# First Choice Plumbing — contact form on Formspree

Short handover note: what the production URL is, what was changed, and how to do
this again on another page.

---

## 1. Where it lives

**Current live address (temporary):**
https://anirudhatalmale6-alt.github.io/first-choice-plumbing-preview/

Type that in rather than copying it out of the Freelancer chat — the chat rewrites
links into tracking URLs that expire.

This is a free holding address. The permanent home should be
**yourfirstchoiceplumbing.com**, which is the address already printed on the site
itself. As of 1 October 2026 that domain was unregistered and available to claim.
Register it in your own name at any registrar, then it can be pointed here or the
file can be moved to whatever hosting you prefer.

A `robots.txt` on the temporary address blocks search engines, so this copy cannot
show up in Google and compete with the real site later.

---

## 2. What Formspree does, and what it does not do

Formspree **delivers the email**. It does **not** host your website.

Those are two separate services and you need both:

| Need | Provided by |
|---|---|
| Form submissions turned into email | Formspree |
| The page itself being on the internet | Hosting + a domain |

---

## 3. What was changed in the page

Only two edits. Everything else — layout, styling, validation, the confirmation
panel — was already written and was left alone.

### Edit 1 — the endpoint

One line near the bottom of the page, inside `<script>`:

```js
const FORMSPREE_ID = "xjykdygd";
```

It was previously the placeholder `"YOUR_FORM_ID"`.

That eight-character ID is the whole integration. The form posts to
`https://formspree.io/f/xjykdygd`. Note what is *not* in that line: your email
address. The destination inbox is stored inside the Formspree account, so the
address is never exposed in the page source where spam harvesters scrape it.

### Edit 2 — a bug fix

One line added to the CSS:

```css
[hidden]{display:none!important}
```

**The problem it fixes:** the stylesheet sets `form` and `.done` to
`display:grid`. An explicit `display` rule beats the browser's built-in handling
of the HTML `hidden` attribute. The result was that the green *"Request sent.
Thank you!"* panel was visible to every visitor the moment the page loaded,
before anyone had submitted anything — and after a successful send the form
stayed on screen instead of being replaced by the confirmation.

Worth remembering if you ever add another `hidden` element to this page.

---

## 4. Doing this again on another page

1. Log into Formspree → **New Form**.
2. Give it a name, and set the recipient inbox.
3. Formspree emails that inbox a **confirmation link. Click it.** Until you do,
   submissions are accepted and return a success response, but no email is ever
   sent. This is the single most common way these setups silently fail.
4. Copy the eight-character ID out of the endpoint it shows you
   (`https://formspree.io/f/XXXXXXXX`).
5. Copy the whole `<form>` block and the `<script>` block from this page into the
   new page, and change `FORMSPREE_ID` to the new ID.
6. Submit a real test with distinctive values in every field, then check the inbox.

Use a **separate form ID per page** if you want to tell at a glance which page a
submission came from. One shared ID across pages also works — Formspree records the
submitting URL either way.

---

## 5. Things already built into this form

- **Honeypot.** A hidden field named `_gotcha`. A human never sees it; bots fill
  everything and get silently dropped.
- **Validation before sending.** Missing or malformed fields produce a plain-English
  message and nothing leaves the browser.
- **A readable subject line**, built from the service and the customer's name, so the
  inbox shows `Appointment request: Water heater – Jane Smith` rather than
  `New submission`.
- **Reply-to.** When the customer fills in the optional email field, you can reply
  to their message directly.
- **A text-message copy.** The confirmation panel offers a "Text a copy to
  404-456-1882" button, which opens the customer's SMS app pre-filled.
- **Graceful failure.** If Formspree is unreachable, the customer is shown the phone
  number and email instead of a dead button.

---

## 6. Testing that was done

Every field was filled with a distinct, traceable value and the payload was
inspected on the wire:

- All nine fields arrive exactly as typed — name, phone, email, service, address,
  date, time window, urgency, problem description.
- Apostrophes, ampersands, accented characters and line breaks survive intact.
  (This is where form-to-email setups usually produce garbage like `&amp;` or `Ã©`.)
- The honeypot blocks a filled trap.
- Validation catches missing required fields before any network call.
- Two real submissions returned `200 {"ok":true}` — one from a local server, one
  from the live HTTPS address.
- Receipt in the Yahoo inbox was confirmed by the account owner.

One intentional behaviour: the date is reformatted from `10/15/2026` to
`Thursday, October 15, 2026` in the email, because it reads better.

---

## 7. Two things to keep an eye on

**The free plan has a monthly submission cap.** Your Formspree **Overview** tab
shows the current count and limit. A busy plumbing season could reach it, and once
it's hit, further submissions stop. Worth glancing at occasionally.

**Sending to more than one address.** Multiple recipients is a paid Formspree
feature. The free alternative that works on any plan: turn on auto-forwarding in
the receiving Yahoo inbox (Yahoo Settings → Forwarding) to copy everything to the
second address. Nothing in the website needs to change.
