# Assumptions & How to Test — Split Bill MVP

## Assumptions

1. **Tax order**: PB1/PPN is calculated on `(subtotal − discount + service)`, i.e. tax is applied after service charge is added. This matches common Indonesian restaurant receipt practice (tax on service-inclusive amount).
2. **Unassigned items**: Items with no assignees are excluded from all calculations and block the Results tab until resolved.
3. **Discount validation**: Nominal discount is capped at total subtotal (all items × qty). Percent discount is capped at 100%.
4. **Single file**: Zero external dependencies — works fully offline when opened from `file://` or any HTTP server.
5. **Rounding method**: Largest-remainder (Hamilton method). Each person's share is floored; leftover Rupiah (1 at a time) goes to persons with the largest fractional part, ties broken by list order. Guarantees `sum(persons) === grand_total` always.
6. **Integer prices**: Price and quantity inputs only accept whole positive integers. Service/tax/discount percentages accept decimals.

## How to Test

**Manual (reference test case)**:
Open `index.html` → add Ani, Budi, Citra → add Nasi Goreng (2×25.000, assign Ani+Budi), Es Teh (3×8.000, assign all), Pizza (1×90.000, assign Budi+Citra) → set Diskon 10%, Service 5%, Pajak 10% → Hasil tab should show: Ani Rp34.304, Budi Rp81.081, Citra Rp55.093, Total Rp170.478.

**Automated self-tests**:
Open `index.html?test=1` in a browser → open DevTools console → tests print PASS/FAIL for 8 scenarios including the reference case, edge cases, and large bills.

**Persistence**: Fill in any data → refresh the page → data should be fully restored.
