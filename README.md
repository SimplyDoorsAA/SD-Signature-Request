# SD-Signature-Request

Two standalone static pages for SimplyDoors. There is no build step and no
server to run &mdash; both are plain HTML files that can be opened directly or
hosted anywhere static files are served (GitHub Pages, Netlify, S3, etc.).

| File | Purpose |
| --- | --- |
| `index.html` | DocuSeal document signing portal. |
| `intake.html` | Client / builder / contractor project intake form. |

---

## Project intake form (`intake.html`)

Collects company, contact name, billing address, job address, phone, email, and
a description of the work, then emails the submission to your work inbox.

### Setup &mdash; already done

The page sends submissions through [Web3Forms](https://web3forms.com), a
form-to-email service that needs no backend, which is why the form works on a
purely static host. The access key is configured in `intake.html`:

```js
const ACCESS_KEY = '496b94f6-00a0-43cc-b485-f71dff873dad';
```

Web3Forms designates this a **public** key, intended for client-side code, so
committing it is expected and safe.

> **Where the destination email lives:** in the Web3Forms dashboard, not in this
> repository. The key only says "deliver to the inbox registered for this form"
> &mdash; the address itself never appears in the page source. To change where
> submissions land, update the recipient in the Web3Forms dashboard; no code
> change and no redeploy needed.
>
> Routing is two steps there, and the first alone does nothing: add the address
> under **Account Settings &rarr; Linked Emails** and verify it, then select it
> as the recipient under the form's own **Settings**. A linked-but-unselected
> address still leaves submissions going to the account's signup address.

If the key is ever cleared or replaced with the `YOUR_ACCESS_KEY_HERE`
placeholder, the page shows an amber "Setup needed" banner and refuses to
submit, so a misconfigured form can never silently swallow a customer's inquiry.

### What the email looks like

Each submission arrives with the subject `New Project Inquiry - <Company>` and a
table of these rows:

```
Company                    Northside Builders LLC
Contact Name               Dana Reyes
Phone                      (216) 555-0184
Email                      dana@northsidebuilders.com
Billing Address            4120 Industrial Pkwy, Suite 7, Cleveland, OH 44109
Job Address                88 Larchmere Blvd, Shaker Heights, OH 44120
What They Are Looking For  Need 14 solid-core interior doors plus hardware...
Submitted                  10/8/2026, 2:35 PM
```

**Reply-to** is set to the customer's own email address, so hitting Reply in
your mail client writes straight back to them.

### Sending it to customers

Once the site is published, the form lives at a single public URL:

```
https://simplydoorsaa.github.io/SD-Signature-Request/intake.html
```

That link is all a customer needs &mdash; paste it into a text message or an
email and they can fill the form out on their own phone. Nothing is required of
them beyond a browser: no app, no login, no account.

If that URL 404s, GitHub Pages is not switched on yet. Turn it on under the
repository's **Settings &rarr; Pages**, with the source set to deploy from the
`main` branch. The address will then be live within a minute or two.

### Saving it to your phone or iPad

The page carries web-app metadata, so it can be pinned to a home screen and
opened like a native app &mdash; full screen, no browser chrome, with the door
icon from `apple-touch-icon.png`.

**On iPhone or iPad (Safari):** open the link, tap the Share button, choose
**Add to Home Screen**, then **Add**. It appears as "Doors Intake".

**On Android (Chrome):** open the link, tap the three-dot menu, choose
**Add to Home screen** or **Install app**.

Pinning must be done from Safari on iOS; Chrome on iPhone cannot add a
home-screen app that launches full screen.

This is the hand-it-over flow: tap the icon, pass the device to the customer,
let them fill it in and submit. The success screen then offers **Start Another
Request**, which clears every field and returns to a blank form, so the same
device can be handed to the next customer without reloading or backing out.

> **No offline support.** The form needs a live connection to submit, since the
> email is sent by Web3Forms rather than the device. On a job site with no
> signal, submitting will show a connection error rather than losing the entry
> silently &mdash; but the entered text is still on screen, so it can be sent
> once there is signal again.

### Behavior worth knowing

- **"Same as billing"** checkbox copies the billing address into the job
  address and hides those fields, so repeat customers fill the form once.
- **Validation** runs before anything is sent: every required field, a real
  email shape, and a phone number with at least 10 digits. Errors appear under
  the offending field and clear as soon as the person starts correcting.
- **Spam** is filtered by a hidden honeypot checkbox that only bots fill in.
- **Failures are visible.** A rejected or unreachable submission shows an error
  and re-enables the button; the success screen only appears when Web3Forms
  confirms the email was sent.

### Customizing

- **Brand colors** &mdash; every colour in the page resolves to a token in the
  `:root` block at the top of `intake.html`. Swapping the real SimplyDoors
  palette in is an edit to that block alone:
  `--brand` (buttons, section numbers, focus rings), `--brand-dark` (header
  bar), `--brand-tint` (soft fills), `--accent` (required markers, eyebrow),
  and `--canvas` / `--paper` / `--ink` for the ground and text.
- **Logo** &mdash; the header currently uses a type wordmark beside an inline
  SVG door mark. To use a real logo, replace the `<svg>` inside
  `.wordmark` with an `<img>` and drop the `.wordmark-text` div.
- **Wording** &mdash; the `.page-intro` block holds the eyebrow, headline and
  standfirst; section titles are the `legend` elements (they number themselves
  through a CSS counter, so reordering sections renumbers them automatically).
- **Typefaces** &mdash; `--serif` (Georgia) for headings and `--sans` (system
  stack) for everything else. Both are installed on effectively every device,
  so the page loads no webfonts and makes no third-party requests.
- **Adding a field** &mdash; add the input with an `id`, add a matching
  `<p class="field-error" id="<id>-error">`, then add a line to `buildPayload()`.
  The object keys in `buildPayload()` are what appear as labels in the email.

### Free-tier limits

The account is on the free tier, which allows **250 submissions per month**
(the usage meter in the Web3Forms dashboard sidebar shows the current count).
If intake volume ever outgrows that, the swap is a paid Web3Forms plan or moving
`buildPayload()` to a small serverless endpoint &mdash; the form markup itself
would not change.

---

## Signing portal (`index.html`)

Embeds a DocuSeal form via `data-src` on the `<docuseal-form>` element. That URL
currently points at an ngrok tunnel to a local DocuSeal container, so it only
resolves while Docker and the tunnel are running, and the tunnel URL changes
each time it restarts. For production use, point `data-src` at a stable hosted
DocuSeal instance.
