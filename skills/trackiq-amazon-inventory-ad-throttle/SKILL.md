---
name: trackiq-amazon-inventory-ad-throttle
description: Finds the Amazon campaigns still paying to advertise products that are about to run out of stock, and produces an upload-ready bulk file that throttles or pauses those ads with the real campaign, ad group and ad IDs attached — plus where the freed budget should go instead. Use when the user asks about advertising products that are out of stock, throttling ads on low inventory, wasted spend on stockouts, protecting rank before a stockout, or which campaigns to pause because stock is low.
---

# Inventory Risk → Ad Spend Throttle

A stockout destroys organic rank **and** wastes ad spend at the same time.
Nobody does this join by hand because it spans two systems.

**Which campaigns are paying to advertise something about to go out of stock,
and where should that money go instead?**

Output is a branded HTML report plus a Sponsored Products bulk file.

## Requires

- The TrackIQ MCP, for `list_marketplaces`, `get_inventory_snapshot`,
  `get_product_performance`, `get_product_ads` and `get_campaigns`.
- **A lead time** — days from purchase order to sellable. Ask for it; it is in
  no tool. Default 45 days, labelled an assumption.
- Nothing else. No filesystem, no shell, no internet.
- **Without the MCP:** works from an FBA inventory export plus an advertised-ASIN
  report with campaign and ad group IDs.

## First run

Fill in a copy of `assets/account.example.md` saved as account.md beside the
skill. Every TrackIQ skill reads the same file, so an account already set up
for another TrackIQ report needs nothing added here.

If the runtime has no filesystem, print the same block and ask the user to
paste it into their project instructions once.

## Read first

- `assets/pulls.md` — the calls, the join, and the ASIN-vs-SKU trap
- `assets/method.md` — the throttle ladder and where the money goes
- `assets/checks.md` — what to verify before anything is uploaded

Copy `assets/report-template.html` and replace every `{{TOKEN}}`.

## Non-negotiables

1. **Cover is computed per ASIN, never per SKU.** Inventory and sales both
   arrive per SKU and a per-SKU join is wrong in both directions at once — see
   `trackiq-restock-priority`, where one ASIN held 11,032 units on a dormant SKU
   and 2,178 on the one selling 11,861 a month. Roll both sides up to ASIN, then
   divide.
2. **Cover uses `on_hand`, never `total`.** `total` includes inbound, reserved,
   unfulfillable and researching units.
3. **`get_product_ads` is the join and it carries real IDs** — `ad_id`,
   `campaign_id`, `ad_group_id`, `asin`, `sku`, `state`. Unlike search terms,
   this data supports a bulk file that can actually be uploaded. Use it.
4. **One ASIN is usually advertised in several campaigns.** Throttling a product
   means touching every ad for it, not one. Group the output by ASIN and list
   every ad beneath it.
5. **Throttle, do not blanket-pause.** Pausing ads on a product with 20 days of
   cover surrenders rank it will need when stock lands. The ladder is in
   `assets/method.md`: bid down, then budget down, then pause, and only pause at
   zero stock.
6. **Never throttle an ASIN with inbound stock landing inside the cover window.**
   It is not going to run out. Check `on_hand + inbound`, not `on_hand`, before
   recommending a cut.
7. **Only `state == 'enabled'` ads can be throttled.** Filter first. Proposing to
   pause something already paused is how a client stops reading these.
8. **Say what the throttle costs.** Cutting ads on a selling product loses sales
   today to protect rank tomorrow. Show the daily ad-attributed revenue being
   given up beside the spend being saved.
9. **Nothing is uploaded or changed.** This produces a file a human reviews.
10. **Never print `account_id`.**

## The other direction

The freed budget has somewhere to go: ASINs with **healthy cover** that are
currently under-advertised. The report proposes moves from the throttle list to
those, and the moves balance to zero. A throttle report that only takes money
away gets read as a cut.

## What it pairs with

`trackiq-restock-priority` produces the cover figures this skill consumes — run
it first and the two reports agree. `trackiq-budget-pacing` decides how big the
budget is; this decides which products deserve it this week.

## Delivery

The output is produced in the chat first. Delivery is the last step and the
method comes from the Delivery block in account.md — never ask per run.

| Method | What to do | Needs |
|---|---|---|
| `in-chat` | Return the report. The default, and the fallback for every other method. | nothing |
| `file` | Write it beside the skill, dated. | a filesystem |
| `slack` | Post the headline findings as text, then upload the file. | a connected Slack tool |
| `n8n` | POST it to the configured webhook. | network access |
| `email` | Hand it to the connected mail tool. | a connected mail tool |

Confirm before the first outward send of a session, fall back to in-chat
loudly when a method is unavailable, and never substitute a different
outward channel.

## Version

`trackiq-amazon-inventory-ad-throttle` v1.0.0 (2026-09-18).

If the user asks whether this skill is current, fetch
`https://trackiq.com/skills/registry.json`, compare the `version` field for
`trackiq-amazon-inventory-ad-throttle`, and if it is newer, give them the download link
and the one-line changelog. Do not fetch at any other time.
