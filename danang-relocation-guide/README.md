# The Da Nang Relocation Guide

A single-file, offline-capable web app for planning a move to Da Nang, Vietnam —
**and** the seed of a sellable digital product.

It's two things at once:

1. **A tool you use now.** Fill it in as you plan your own move. Everything saves
   to your browser automatically (localStorage). Track tasks, run a live cost-of-living
   budget, count down to your move date.
2. **A product you sell later.** Once you've actually done the move and filled it with
   *verified, real-world specifics* (the visa service that worked, the rent you paid,
   the agent who found your house), it becomes a guide worth $20–50 to the next person.

## How to use it

Just open `index.html` in any browser. No install, no server, no internet required
after first load (fonts load from Google Fonts when online).

- **Type anywhere** — it saves instantly. A "Saved ✓" flash confirms.
- **Set your move date** (top-left) to start the countdown.
- **Check off tasks** — the progress bar and per-section badges update live.
- **Budget page (§5)** — enter your real monthly numbers; totals + VND convert live.
- **Save backup (JSON)** regularly — localStorage is per-device and per-browser, so a
  backup file is your insurance. "Restore backup" loads it on any machine.
- **Export to PDF** — the print button produces a clean, sidebar-free PDF.

## The 12 sections

1. Decision & Big Picture · 2. Visas & Legal Entry · 3. Money, Banking & Getting Paid
· 4. Housing in Da Nang · 5. Cost of Living (live budget) · 6. Healthcare & Insurance
· 7. Stuff: Shipping/Selling/Packing · 8. Connectivity · 9. Income & Remote Work
· 10. Daily Life & Getting Around · 11. Community & Soft Landing · 12. Master Timeline.

## Why this is structured around *your* notes

There is infinite free, generic "move abroad" content. None of it has your real numbers,
your real contacts, or the gotchas you only learn by doing it. The fields and prompts here
are deliberately empty — **the value is what you fill in.** Capture it while it's fresh
(ideally as it happens), not from memory months later.

> ⚠ **Not legal/tax/immigration advice.** Visa rules, fees, and requirements change
> constantly and depend on your nationality. Every section is "confirm against the official
> source before you rely on it."

## Turning it into income — roadmap

The hard part of a digital product is usually "what do I make and how do I know it's good?"
You're solving that by *living it*. Suggested path, lowest-effort first:

1. **Use it for your own move** (next 8–12 weeks). Fill every field with verified specifics.
   Zero extra work — you'd research this anyway.
2. **Polish into a product.** Strip personal/private bits, keep the playbook + your real
   numbers and vetted contacts. Export to PDF as the deliverable. Add screenshots of the
   live budget tool.
3. **Pick a sales channel** (any one to start):
   - **Gumroad / Payhip / Lemon Squeezy** — easiest for a paid PDF or hosted HTML. They
     handle payments and delivery; you keep most of the cut.
   - **Notion template** — duplicate this structure into Notion and sell via Gumroad /
     Notion's marketplace.
   - **Etsy** — surprisingly strong for "relocation planner / printable" digital goods.
4. **Drive traffic for free** with the experience you already have:
   - Answer real questions in Da Nang / Vietnam expat Facebook & Zalo groups, Reddit
     (r/VietNam, r/expats, r/digitalnomad), and link the guide in your profile/replies.
   - A few honest, specific posts ("here's exactly what our first month cost") outperform
     any ad — and you have the receipts.
5. **Expand later** if it sells: a Hoi An / Ho Chi Minh / Hanoi edition reuses 90% of this
   scaffold. One framework, many cities.

**Reality check:** this won't be passive on day one — you put in the lived experience up
front. But the *marginal* effort after launch is near zero: the same file sells again and
again. Price it low, get the first 10 sales for proof, then decide if it's worth scaling.

## Tech notes

- Single file (`index.html`), vanilla JS, no build step, no dependencies.
- All content lives in the `SECTIONS` / `BUDGET_MONTHLY` arrays near the top of the
  `<script>` — edit those to add/remove/rename anything. The UI re-renders from them.
- Data is stored under the localStorage key `danang-guide-v1`. Backup/restore is JSON.
