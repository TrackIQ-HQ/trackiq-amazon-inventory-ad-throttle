# Before you send it

## 1. The join

- **Cover was computed per ASIN, never per SKU.** Spot-check a multi-SKU ASIN:
  does its cover match summed stock over summed units?
- Every ad row's ASIN has a cover figure. An ad with no matching inventory row
  means the ASIN is unstocked or the join dropped it — find out which.
- **Every ad for a throttled ASIN is listed**, not just one. One ASIN commonly
  runs in three campaigns. Count the ads per ASIN and sanity-check it against
  the raw `get_product_ads` rows.

## 2. The stock figures

- Cover uses `on_hand`, never `total`.
- The throttle decision uses **`cover_inbound`**, not `on_hand` alone.
- **No ASIN with inbound stock landing inside its cover window is being
  throttled.** This is the most likely false positive; check the top three.
- `reserved` appears in no calculation.

## 3. The ladder

- Every action maps to a rung, and the rung is printed beside it.
- Nothing is paused above 7 days of cover except a zero-stock ASIN.
- No blanket pause of a campaign that also carries healthy ASINs — the pause is
  on the **Product Ad**, using `ad_id`.
- If the client's lead time is long, the ladder was scaled and the report says so.

## 4. Only what can be changed

- **Every ad in the file is `state == 'enabled'`.** Filter, then check.
- No row proposes pausing something already paused.
- Every row has a real `ad_id`, `campaign_id` and `ad_group_id`.

## 5. Both sides of the trade

- **Spend saved and ad-attributed sales given up appear side by side** per ASIN.
- The totals for both are on the summary, not just the saving.
- Zero-stock ASINs are called out as pure saving — nothing is given up there.

## 6. The reallocation

- Destinations have cover **above 45 days** *and* ACOS at or better than the
  account average. Both.
- **The moves sum to zero.** Print the sum.
- No single destination increase exceeds 30% of its current spend.
- Headroom is labelled an estimate.

## 7. The restore list

- **It exists.** Same ads, original bids and states, on its own sheet.
- It is mentioned in the handover note, not only in the workbook.
- The report says when to run it: when stock lands.

## 8. Render check

```js
const action = document.querySelectorAll('table')[0];
({ overflows: document.documentElement.scrollWidth > document.documentElement.clientWidth,
   tables: document.querySelectorAll('table').length,
   rows: [...document.querySelectorAll('table')].map(t => t.querySelectorAll('tbody tr').length),
   logos: [...document.images].map(i => i.naturalWidth > 0),
   // scope to the action table: throttled ads also appear in the full catalogue
   paused: action.querySelectorAll('.state-out').length,
   tokens: (document.body.innerHTML.match(/\{\{[A-Z0-9_]+\}\}/g) || []).length })
```

`overflows` false, `logos` all true, `tokens` zero, and the action table's row
count equal to the number of rows in the bulk file. Then look at it; if it will
not paint, say the check was structural.

## 9. Ship

Save as `<client>-ad-throttle-<YYYY-MM-DD>.html` and
`<client>-ad-throttle-bulk-<YYYY-MM-DD>.xlsx`. Dated by run day — inventory is a
snapshot and a week-old throttle list is dangerous, not merely stale.

Hand over the throttle sheet and the restore sheet together, and say in the
message that the restore is due when stock lands. A throttle nobody undoes is
worse than no throttle.
