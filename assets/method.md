# Method

## The throttle ladder

The instinct is to pause ads on anything running low. That is wrong, and
expensively so: pausing surrenders organic rank that took months and a lot of ad
spend to build, and the product will need that rank the day stock lands.

Throttle in proportion to how close the product is to zero, using
`cover_inbound` — stock on a boat that lands next week is not a reason to cut
anything.

| `cover_inbound` | Action | Why |
|---|---|---|
| **> 45 days** | nothing | healthy; it may be a destination for freed budget |
| **30–45 days** | watch | on the report, no action |
| **21–30 days** | **bid down 25%** | slow the burn, keep the rank |
| **14–21 days** | **bid down 50%** | protect the remaining stock for organic demand |
| **7–14 days** | **bid down 75%**, or cap the campaign budget | the last of the stock should go to buyers who were coming anyway |
| **0–7 days** | **pause the ad** | every click now is paid for and cannot be fulfilled |
| **0 on hand** | **pause the ad** | the only unambiguous case |

The thresholds are defaults. A product with a 90-day lead time needs a wider
ladder than one restocked weekly — if the client's lead time is long, scale the
ladder with it and say so.

**Never throttle on `on_hand` alone.** An ASIN with 5 days of stock and a
container landing in 3 is fine. Check `on_hand + inbound` first; if inbound
lands inside the cover window, leave the ads alone and note why.

## What the throttle costs

Cutting ads on a product that is still selling loses sales today to protect rank
tomorrow. That is usually the right trade, and the client is entitled to see
both sides:

```
spend_saved_per_day = current ad spend / days in window
sales_given_up      = current ad-attributed sales / days in window x throttle_pct
net_per_day         = spend_saved_per_day x throttle_pct - sales_given_up x margin
```

Show **spend saved** and **ad-attributed sales given up** side by side per ASIN.
A report that only shows the saving is selling, not advising.

Where the throttled product is at zero stock, nothing is given up — those clicks
were being paid for and could not convert. Say so; it is the strongest line in
the report.

## Where the money goes

The freed budget goes to ASINs with **cover above 45 days** and an ACOS at or
better than the account average. Both conditions, not one: adding budget to a
well-stocked product that loses money is not an improvement.

```
for each destination ASIN:
    headroom = how much more it could absorb before ACOS degrades
             — judge from current spend and impression volume, and say
               plainly that headroom is an estimate
```

**The moves balance to zero.** Print the sum. A reallocation that does not
balance is a budget cut or a budget increase wearing a disguise, and the client
will find out at month end.

Cap any single destination increase at 30% of its current spend. Campaigns do
not absorb a doubling gracefully and the money will simply not spend.

## The bulk file

Sponsored Products bulk operations. Two entity types, depending on the rung:

| Rung | Entity | Operation | What changes |
|---|---|---|---|
| Bid down | `Keyword` / `Product Targeting` | `update` | Bid |
| Budget cap | `Campaign` | `update` | Daily Budget |
| Pause | `Product Ad` | `update` | State → `paused` |

Pausing the **Product Ad** rather than the campaign is the precise move: the
campaign keeps running for its other ASINs and only the at-risk product stops.
This is why `ad_id` matters and why it is carried through from
`get_product_ads`.

Every row gets a **TrackIQ reason** column: the ASIN, its cover, and the rung.
Amazon ignores unrecognised columns, so it rides along harmlessly and the file
stays auditable. A bulk file whose rows cannot be explained does not get
uploaded.

## Restoring afterwards

A throttle is temporary and somebody has to undo it. The report ends with a
**restore list** — the same ads with their original bids and states — so the
client can put everything back when stock lands.

This is the part that gets forgotten, and a product left at a 75% bid cut three
months after restocking is a worse outcome than never throttling it. Ship the
restore list in the same workbook, on its own sheet.

## What this skill does not do

- **It does not change anything.** A human reviews and uploads.
- **No forecast.** Velocity is a trailing 30-day average.
- **No MOQ, case pack or PO logic.** That is `trackiq-restock-priority`.
- **It cannot see campaign budgets.** There is no budget field in the API, so a
  budget-cap recommendation is expressed as a percentage change, not an amount.
