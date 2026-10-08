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

### One-time setup

The page sends submissions through [Web3Forms](https://web3forms.com), a free
form-to-email service. It needs no account and no backend, which is why the
form works on a purely static host.

1. Go to **https://web3forms.com**.
2. Enter the work email address where you want intake submissions delivered.
3. Web3Forms emails you an **access key** (a UUID that looks like
   `a1b2c3d4-...`). Copy it.
4. Open `intake.html`, find this line near the top of the `<script>` block, and
   paste your key in place of the placeholder:

   ```js
   const ACCESS_KEY = 'YOUR_ACCESS_KEY_HERE';
   ```

5. Save and re-deploy. That's it.

Until a real key is in place, the page shows an amber "Setup needed" banner and
refuses to submit, so a half-configured form can never silently swallow a
customer's inquiry.

> **Where the destination email lives:** with your Web3Forms account, not in
> this repository. The access key is a public-by-design token that only says
> "deliver to the inbox registered for this key" &mdash; so committing it is safe,
> and your work address is never exposed in the page source. To change the
> destination later, update it in your Web3Forms account; no code change needed.

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

- **Wording and branding** &mdash; edit the `.header` block and the `legend`
  elements directly.
- **Colors** &mdash; the palette is a handful of CSS custom properties in
  `:root` at the top of the file; `--accent` drives the buttons and section
  headings.
- **Adding a field** &mdash; add the input with an `id`, add a matching
  `<p class="field-error" id="<id>-error">`, then add a line to `buildPayload()`.
  The object keys in `buildPayload()` are what appear as labels in the email.

### Free-tier limits

Web3Forms' free tier caps how many submissions you can receive per month; check
the current number on their pricing page when you sign up. If intake volume ever
outgrows it, the swap is a paid Web3Forms plan or moving `buildPayload()` to a
small serverless endpoint &mdash; the form markup itself would not change.

---

## Signing portal (`index.html`)

Embeds a DocuSeal form via `data-src` on the `<docuseal-form>` element. That URL
currently points at an ngrok tunnel to a local DocuSeal container, so it only
resolves while Docker and the tunnel are running, and the tunnel URL changes
each time it restarts. For production use, point `data-src` at a stable hosted
DocuSeal instance.
