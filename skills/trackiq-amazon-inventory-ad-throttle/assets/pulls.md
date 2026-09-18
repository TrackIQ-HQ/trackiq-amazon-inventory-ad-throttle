# The pull sequence

## 0. Account and lead time

`list_marketplaces` first. Never print `account_id`.

Ask for the **lead time** (days from PO to sellable). Default 45, labelled an
assumption. It decides whether a low-cover ASIN is recoverable before it runs
out, which decides whether to throttle or to hold.

## 1. Inventory

```
get_inventory_snapshot(account_id, limit=100, offset=…)
```

**Paginate** — it caps at 100 rows and a real catalogue exceeds it.

Fields: `on_hand`, `inbound`, `reserved`, `unfulfillable`, `researching`,
`total`, `out_of_stock`, `asin`, `sku`, `title`.

- **Available now** = `on_hand`
- **Available with the boat** = `on_hand + inbound`
- **`total` is not available stock.** It is the sum of all five buckets,
  including units already promised to orders and units that are damaged.

## 2. Velocity

```
get_product_performance(account_id, start_date, end_date,
                        group_by='product', limit=200)
```

Trailing **30 days**. Gives `units`, `revenue`, `sessions`, `orders` per SKU.

## 3. Roll both up to ASIN — then divide

Inventory and sales both arrive **per SKU**. Joining at SKU level is wrong in
both directions simultaneously: on the account this family of skills was built
against, ASIN `B0C848NYN6` held 11,032 units on a SKU that sold nothing and
2,178 units on the SKU doing 11,861 a month. Per SKU that is a six-day emergency
*and* an infinite supply on the same product. The ASIN truth is 33 days.

```
cover         = on_hand_asin / (units_asin / 30)
cover_inbound = (on_hand_asin + inbound_asin) / (units_asin / 30)
```

Use `cover_inbound` for the throttle decision. Stock on a boat that lands in
twelve days is not a reason to cut advertising.

## 4. The join — `get_product_ads`

```
get_product_ads(account_id, start_date, end_date, limit=500)
```

This is the tool that makes the skill possible. Rows carry:

```
ad_id, campaign_id, ad_group_id, state, asin, sku, title,
impressions, clicks, spend, sales, orders, units, acos, roas, cpc
```

**It has real campaign and ad group IDs**, unlike `get_search_terms`. That is
why this skill can produce an uploadable bulk file and the harvester cannot.

Paginate to exhaustion.

**One ASIN appears in several rows.** On the account checked, a single ASIN was
advertised across three campaigns and three different ad groups. Throttling that
product means touching all three ads. Group by ASIN, list every ad under it, and
total the spend per ASIN — never assume one ad per product.

Filter to `state == 'enabled'`. A paused ad cannot be throttled and proposing it
wastes the reader's attention.

## 5. Campaign context

```
get_campaigns(account_id, start_date, end_date, limit=200)
```

For campaign names and states, so the report reads in the client's language
rather than in IDs. Note `get_campaigns(granularity="daily")` drops the campaign
dimension entirely — do not use it to build a per-campaign trend.

## 6. Where the money goes

The same two pulls already give it: ASINs with **healthy cover** (well above the
throttle thresholds) whose ads are running at a good ACOS. Those are the
destinations. No extra call is needed.

## 7. The output rows

For each ad to throttle, carry forward:

```
asin, sku, ad_id, campaign_id, ad_group_id, campaign name,
current spend/day, current ad-attributed sales/day,
cover, cover_inbound, action, new bid or state
```

Every one of those is needed for the bulk file or for the reader to audit the
recommendation. An action a client cannot trace back to a number does not get
applied.
