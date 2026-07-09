# Cost Quote

Generate a customer-facing quote using the Profit Engine pricing data, priced at or above the margin floor.

## Inputs

- `Profit-Engine/01-cost-sheet/price-book.json` — SKUs, list prices, floor prices, COGS.
- `Profit-Engine/01-cost-sheet/margin-rules.md` — discount policy and tier rules.
- From the user: customer name, and the SKUs or a description of the services wanted.

## Steps

1. Read `price-book.json` to get available SKUs and their pricing.
2. Read `margin-rules.md` for discount thresholds and floor-price policy.
3. Match the requested services to SKUs. If a request isn't an exact SKU match, suggest the closest one and ask for confirmation.
4. Calculate the proposed price:
   - Start at list price.
   - If the user specified a discount, check it against the approval thresholds in `margin-rules.md`.
   - Ensure the final price is at or above the `floor_price` for every line item.
5. Output a clean quote:

---
**Quote for [Customer Name]**
Date: [today]
Valid through: [today + 30 days]

| SKU | Description | Unit | Qty | Unit Price | Line Total |
|-----|-------------|------|-----|-----------|------------|
| ... | ... | ... | ... | $... | $... |

**Subtotal**: $...
**Discount applied**: X% ([approval level required])
**Total**: $...

*Margin check*: All lines above floor price. ✓ / ⚠ [flag any violations]

---

6. If any line is below floor price, flag it with ⚠ and state the floor price.
7. Note any discount that requires manager or executive approval per `margin-rules.md`.
