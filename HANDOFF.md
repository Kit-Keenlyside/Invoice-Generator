# Invoice Generator - handoff

Read this first, then continue from "Next steps". Style rule for everything you write for Kit: never use em dashes, hyphens only. Kit works on a Mac.

## What this is

A single-file HTML tool (`index.html`) that builds an A4 invoice and downloads it as a PDF. No build step, no backend. It runs entirely in the browser.

- PDF: html2pdf.js from cdnjs (renders the sheet to an image inside a PDF, so PDF text is not selectable).
- Font: Inter from Google Fonts.
- Hosted on GitHub Pages: https://kit-keenlyside.github.io/Invoice-Generator/ (repo: Invoice-Generator, user: kit-keenlyside).
- Owner: Kit Keenlyside, sole trader (guitar tuition, moving towards library music composition). Invoices are hourly tuition ("fee notes") plus music advances and flat-fee work.

## Files

- `index.html` - the hosted version and the one to edit. It contains NO personal data. Kit enters their own details once on the page, or the Notion link supplies them, and the browser remembers them (localStorage).
- `invoice-generator.html` - an older private version with Kit's personal details built in as defaults. It may or may not exist on this machine. Never publish it or copy its personal data into `index.html`.

## Layout and behaviour decisions (keep these)

Sheet layout, top to bottom:
- Grey strip at the very top: Date at the far left, Ref at the far right.
- Document title (e.g. INVOICE or FEE NOTE) large and uppercase on the left, Kit's name/company on the right. No logo feature.
- Two columns with small uppercase labels and a thin rule underneath: CONTACT (email, phone, address) and BILL TO (name, address, attn).
- DESCRIPTION section: a description line, then line items.
- Totals, right-aligned, with no "Total" label. The total is just the bold amount. Subtotal and Tax rows only appear when the tax rate is above 0. A due date row appears if set.
- BANK DETAILS on the left, TAX on the right (UTR first, then the tax note and any extra note).
- All section headings use the same style (small uppercase bold label with a rule). Section gaps are 40px. Rows are 11px text with 22px line height.
- The quantity column starts at exactly the same left edge as the BILL TO and TAX columns (grid column `calc(50% + 14px)`). The amount column is 100px, right-aligned, at the far right. Totals line up with it.
- Numbers use Inter with tabular figures. Monospace fonts were tried and rejected as ugly. Do not reintroduce them.

Line items:
- Each line has a description, a quantity and a rate or amount.
- Quantity is free text and can carry the unit, e.g. "4 hrs" or "2 tracks". There is no separate unit field.
- A blank quantity counts as 1, so the rate acts as a flat amount (used for advances).
- Amount = leading number of the quantity x rate. Every line always shows its amount on the far right.
- The "Show rate on every line" checkbox adds a rate column, shown only on lines whose quantity is present and not 1 (so a rate is never repeated next to an identical amount).
- "Add line" adds a blank line. "Duplicate last line" copies description, quantity and rate.

Form:
- Presets block at the top (save, load, delete named setups; stored in the browser only; no built-in presets). A preset stores: title, description line, currency, tax label and rate, rate checkbox, tax heading and note, extra note, line items. It does not store client, date, reference or bank details.
- Below it, an "Edit section" dropdown (Header, Contact, Bill to, Description, Bank details, Tax) that shows one section at a time, then the Download PDF button, then a "Copy prefill link" button.
- The date resets to today on every load.

## URL prefill (the Notion hook)

The page fills itself from the URL fragment, e.g. `index.html#to=Alibi&ref=KK-2026-014&items=Track+one%7C2+tracks%7C250`. The fragment (after `#`) is never sent to the server.

Parameters (any multi-line field takes line breaks as %0A or `\n`):
- Header: `date` (YYYY-MM-DD), `ref`, `title`, `name` (Kit's name/company, top right).
- Contact: `email`, `phone`, `addr`.
- Bill to: `to`, `toaddr`, `attn`.
- Description: `desc`, `cur` (symbol, URL-encoded), `taxlabel`, `taxrate` (number), `due` (YYYY-MM-DD), `rate` (1 shows the rate column, 0 hides it), `items`.
- Bank details: `payhead` (heading), `payname`, `acct`, `sort`, `bank`.
- Tax: `taxhead` (heading), `utr`, `taxnote`, `foot` (additional note).

Items: `description|quantity|rate`, items separated by `;`. In the final URL `|` is `%7C` and `;` is `%3B`. Descriptions must not contain `|` or `;`. Values are URL-encoded (space as `+` or `%20`, `&` as `%26`, `#` as `%23`, `%` as `%25`, `+` as `%2B`).

Rules in the code:
- Only parameters present in the link override the form. Anything else keeps its saved value, so a link should always include every parameter, even empty ones (`attn=`), or stale values from the previous invoice will remain.
- Kit's own details, contact and bank details can be in the link (added 5 Nov 2026 so Notion can supply everything). They are only ever in the link, never in `index.html` or the repo. Whatever a link applies is also saved to localStorage in that browser, like any other field.
- The fragment is cleared from the address bar after applying. A `hashchange` listener handles a new link opened in the same tab.
- The "Copy prefill link" button builds a link from the current form, including contact and bank details. Use it as ground truth when testing a Notion formula. Empty fields are left out of the copied link.

Plan: a Notion formula property builds this link from a database row (one row = one invoice, line items stored in a text property with one `description | quantity | rate` per line). Kit will have Notion AI write the formula using the spec above.

## Storage

localStorage keys: `invoiceGenV1` (form state), `invoiceGenPresets` (saved presets), `invoiceGenSec` (last open form section).

## Verified and not verified

Verified in headless Chromium (Playwright): contact, bank and tax fields from the link (5 Nov 2026), column alignment (quantity, Bill to and Tax left edges identical; amount and total right edges identical), URL prefill, link round trip, percent-encoded links, section dropdown, no JavaScript errors, no personal data left in `index.html`.

Not verified:
- PDF download from the hosted site, including multi-page invoices with many lines.
- Anything on the Notion side (no Notion formula has been written or run).

## Hosting notes

- Publish by committing `index.html` to the root of `main`. GitHub Pages serves it from there.
- A GitHub Pages site is public even if its repository is private (private access only exists on Enterprise Cloud). So never put personal data, bank details or client data into the repo.

## Next steps

1. Confirm Download PDF works on the live page, including an invoice long enough to run onto a second page.
2. Notion: write and test the formula property that builds the prefill link, using the spec above.
3. Optional: return the PDF to Notion automatically. This needs a Notion database automation with a "Send webhook" action (paid plans; it sends page properties only) pointing at a small hosted function that renders the PDF and attaches it back with the Notion API file upload. Not started.
4. Optional: auto-increment the invoice reference. Suggested format `KK-YYYY-NNN`, one running sequence across all invoice types. Not built.
5. Optional: a text-based PDF (selectable text) via server-side rendering.

## Working notes

- Keep it one file with no build step. Make small edits in place.
- After any layout change, re-check that the quantity column's left edge still equals the Bill to and Tax left edges.
- Ask Kit for the local path of the project folder if it is not already connected.
