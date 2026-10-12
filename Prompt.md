# Prompt IBM Bob — Split Bill

### Layer 1 — ROLE & CONTEXT

```
You are Bob, acting as a senior frontend engineer with a strong product sense.
You are building "Split Bill", a web app that splits restaurant bills fairly
among friends, coworkers, and family, including shared items, tax, service
charge, and discounts.

Who you are building for:
- Indonesian users aged 17-40, mostly on mobile phones, often at a restaurant
  table with an unstable connection and one hand free.
- They want an answer in under 2 minutes, with no login and no installation.

Context:
- This is an MVP judged on (1) calculation correctness, (2) speed and clarity
  of the UX, (3) code quality and maintainability.
- UI language: Bahasa Indonesia. Currency: Rupiah (IDR), whole numbers only.
- Treat the PRD as the source of truth. If something is ambiguous, pick the
  simplest reasonable default, state it in a short "Assumptions" note, and
  keep building. Do not stop to ask questions.
```

### Layer 2 — TASK & REQUIREMENTS

```
Build the Split Bill MVP with exactly these 8 features:

1. PARTICIPANTS
   Add/remove people by name. Names are required, trimmed, max 30 chars,
   case-insensitively unique. Removing a person removes them from all item
   assignments (ask for confirmation if they are assigned to any item).

2. ITEMS
   Add/edit/delete items with: name (required), unit price (integer >= 1),
   quantity (integer >= 1). Show line total = price x quantity.

3. ASSIGN ITEMS
   For each item, the user picks who ate it via checkboxes (one person or
   several), with a "Semua" (select all) shortcut. An item's line total is
   split equally among its assignees. Items with no assignee must be clearly
   flagged and block the final result with a helpful message.

4. TAX & SERVICE
   Two percentage inputs (service charge, PB1/PPN), each 0-100, decimals
   allowed (e.g. 5 or 11). Defaults: service 0, tax 0. Both are distributed
   proportionally to each person's subtotal.

5. DISCOUNT (optional)
   Either a nominal amount (Rp) or a percentage. Cannot exceed the subtotal.
   Distributed proportionally to each person's subtotal.

6. RESULT
   Per-person breakdown: items eaten (with their share), subtotal, discount,
   service, tax, and final amount. Plus a grand total and a visible
   "Total cocok" check showing sum of persons == bill total.

7. SHARE
   "Salin" (copy to clipboard, with a fallback when the Clipboard API is
   unavailable) and "Kirim ke WhatsApp" (https://wa.me/?text=<encoded text>).
   The text must be human-friendly: title, date, one line per person with
   amount, total, and a short breakdown of tax/service/discount.

8. AUTOSAVE
   Persist the whole state to localStorage on every change; restore on load.
   Provide a "Mulai baru" (reset) button with confirmation.

User flow: add people -> add items -> assign who ate what -> set tax/service/
discount -> view result and share. A single scrolling page or a simple
stepper is fine; pick whichever gives the fastest completion on mobile.

CALCULATION RULES (implement as a pure, testable function):
- Work in integer Rupiah. Never display floating point artifacts.
- Person subtotal = sum over items of (item line total / number of assignees).
- discount_total = nominal amount, or percent x subtotal_all.
- service_total = service% x (subtotal_all - discount_total).
- tax_total = tax% x (subtotal_all - discount_total + service_total).
  (Default: tax is applied after service, as is common for PB1 in Indonesia.
  Document this in the UI help text and in the Assumptions note.)
- grand_total = subtotal_all - discount_total + service_total + tax_total,
  rounded to the nearest whole Rupiah.
- Each person's share of discount, service, and tax = proportional to their
  subtotal / subtotal_all.
- ROUNDING: compute each person's exact final amount, floor them, then
  distribute the leftover Rupiah one by one to the people with the largest
  fractional remainders (ties broken by list order). The sum of all people
  MUST equal grand_total exactly, every time.

REFERENCE TEST CASE (must pass):
  People: Ani, Budi, Citra.
  Items: Nasi Goreng 2 x 25.000 (Ani, Budi); Es Teh 3 x 8.000 (all three);
         Pizza 1 x 90.000 (Budi, Citra).
  Discount 10%, service 5%, tax 10%.
  Subtotals: Ani 33.000, Budi 78.000, Citra 53.000 (sum 164.000).
  Grand total: 170.478.
  Expected result: Ani 34.304, Budi 81.081, Citra 55.093.
  Also test: single person, item with 1 assignee, 0% everything,
  nominal discount, 3-way split of 100.000, and a large bill.
```

### Layer 3 — CONSTRAINTS & OUTPUT

```
DELIVERABLE
- One file: index.html, with all CSS in a <style> tag and all JavaScript in a
  <script> tag. Zero external libraries, fonts, images, CDNs, or network calls.
  It must work fully offline when opened from disk.
- Also output a short "Assumptions & How to test" note (max 15 lines).

TECH
- Semantic HTML5, CSS Flexbox/Grid, CSS custom properties for the theme.
- Vanilla JavaScript (ES2020), no frameworks, no build step.
- Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR',
  maximumFractionDigits: 0 }) for all money display.
- localStorage wrapped in try/catch, with a versioned key (e.g.
  "splitbill:v1") and validation of loaded data. Corrupt or missing data must
  fall back to a clean empty state without throwing.
- Separate pure calculation logic from DOM code (a calculateSplit(state)
  function with no side effects). Add a small self-test function that runs
  the reference cases and logs pass/fail only when the URL has ?test=1.

UX & VISUAL
- Mobile-first (design at 360px width first), responsive up to desktop with a
  centered max-width container.
- Touch targets at least 44x44px. Inputs use inputmode="numeric" or
  "decimal" so the right keyboard appears. Font size >= 16px on inputs to
  avoid iOS zoom.
- Clean, friendly, modern look. Light theme by default; leave room for dark
  mode later via CSS variables (do not build dark mode now).
- Show a live running total so users always see progress.
- Empty states with a clear next-step hint (e.g. "Tambah peserta dulu").
- Friendly inline validation messages in Bahasa Indonesia, shown next to the
  field; never use alert() for validation.

ACCESSIBILITY
- WCAG AA contrast (4.5:1 for text). Every input has a visible <label> or an
  associated aria-label. Visible focus styles. Do not rely on color alone for
  errors. Results area uses aria-live="polite".

QUALITY & SAFETY
- Zero console errors or warnings on load and during normal use.
- Never insert user text via innerHTML; use textContent or escape it.
- Reject empty, negative, NaN, and non-integer (where integers are required)
  values. Cap quantities and prices at sane maximums to avoid overflow.
- Clear, concise code comments explaining the why of non-obvious logic,
  especially the rounding algorithm.

OUT OF SCOPE (do not build now)
  Bill history, rounding to 100/500/1000, payer-and-debt tracking,
  print/export, dark mode, accounts, backend.

WORKFLOW
1. First, write a 5-10 line plan (structure, state shape, calc approach).
2. Then build the complete index.html.
3. Finally, self-review against this checklist and report PASS/FAIL for each:
   [ ] Reference test case matches exactly
   [ ] Sum of persons == grand total in every test
   [ ] Works offline, single file, no external requests
   [ ] State survives a page refresh
   [ ] No unassigned item can reach the result screen
   [ ] WhatsApp link opens with correctly encoded text
   [ ] Usable at 360px width, no horizontal scroll
   [ ] No console errors
```
