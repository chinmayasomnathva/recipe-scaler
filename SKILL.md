# Recipe Scaler Tool — Skill

## Overview
A single-file HTML recipe scaler tool for GitHub hosting. Supports multiple recipes via tabs, Excel paste, smart unit conversion, and scaling by servings or oz-based portion sizes.

---

## Inputs
- **Recipe Name** — text field (also renames the tab)
- **Base Recipe Makes** — number + unit dropdown (servings, pieces, 16oz, 12oz, 8oz, 4oz)
- **Target Amount Needed** — number + unit dropdown (same options)
- **Ingredients** — table with Name, Qty, Unit columns; supports manual entry and Excel paste

---

## Scaling Logic
- Scaling factor = (target count × oz value) / (base count × oz value)
- Each ingredient qty is multiplied by the scaling factor
- Results show: original qty, scaled qty (same unit), and kitchen measurement (converted unit)

---

## Unit Conversion Rules
- **Liquid ingredients** (detected by keyword match): tsp → tbsp → fl oz → cup → qt → gal
- **Dry ingredients**: tsp/tbsp → oz → lbs, using per-ingredient gram weights
- **Cups → lbs**: ingredient-aware lookup table (e.g. flour=0.275 lbs/cup, sugar=0.44, beans=0.42, oats=0.18)
- **oz vs fl oz**: differentiated by liquid keyword detection on the ingredient name
- **tsp/tbsp gram weights**: per-ingredient table (salt, baking soda, flour, sugar, etc.)
- **Pass-through units** (no conversion, just scale qty): `bunch`, `clove`, `sprig`, `stalk`, `slice`, `pinch`, `dash`, `handful`, `count`

---

## Excel Paste Handling

### Key fix: column order is auto-detected
Do NOT assume a fixed column order. Detect dynamically:
1. Find the **quantity column** — the cell matching a number or fraction pattern
2. Find the **unit column** — the cell whose value matches a known unit keyword (check multi-word units BEFORE stripping spaces)
3. The **name column** is whatever remains

### `normalizeUnit` function rules
- Check multi-word units FIRST before stripping spaces:
  - `"fl oz"`, `"fluid oz"`, `"fl. oz"` → `floz`
  - `"oz (weight)"` → `oz`
  - `"piece / each"` → `count`
- Then strip non-alpha characters for single-word lookup
- Return `'count'` as fallback only

### Unrecognized units — NEVER silently drop
- If a unit can't be mapped, **preserve the raw unit text as-is**
- Render the ingredient row with a **yellow background** (`#fff8e1`) and an **orange ⚠️** icon
- The unit cell becomes a **free-text input** (not a dropdown) pre-filled with the raw string, styled with an orange border
- The user can type any correction directly — on scale, `normalizeUnit()` is called again so known units convert properly; truly unknown units pass through as labels
- `addRow(rid, name, qty, unit, unknown=true)` — the 5th `unknown` argument triggers highlighted custom input mode
- `parseRecipeLine` returns `_rawUnit` on the result object when a unit was found in text but not recognized

### Qualifier word stripping
Free-text lines often have descriptors after the unit: `"20 palm size pieces"`, `"large sized"`.
Strip these before unit matching using a `QUALIFIER_WORDS` regex:
`large, small, medium, big, sized?, palm, size, fresh, dried, chopped, minced, sliced, diced, whole, ripe, raw, cooked, boneless, skinless, approx, approximately, about`

### Concatenated lines
Some pastes combine two entries on one line: `"curry leaves - 1 bunch coriander leaves - 3 bunches"`.
Fix: after newline injection, apply a regex to split on `digit + word(s) + " - " + digit` patterns.

### Supported paste formats
- `Name | Qty | Unit`
- `Qty | Unit | Name`
- `Name | Qty` (no unit column)
- Free text: `Yogurt - 1/2 cup`, `Ginger: 20 palm size pieces`, `Garlic: 80 cloves`

---

## Paste Hint UI
Keep the hint short — two example lines only:
```
Flour   2   cups
Yogurt  0.5  cup
```
No verbose explanations of column orders or supported unit lists.

---

## Multiple Recipes
- Tab bar with add (+) and close (✕) buttons
- Double-click tab to rename
- Each recipe is fully independent (own ingredients, scaling config, results)

---

## Export
- **Copy for Google Sheets** — tab-separated `Ingredient \t Kitchen Measurement`
- **Copy as Text** — plain text with scaling summary and ingredient list

---

## Recognized Unit Keywords
`tsp, tbsp, cup, fl oz, fluid oz, qt, quart, gal, gallon, oz, lbs, lb, g, gm, gms, gram, kg, ml, l, liter, litre, bunch, clove, sprig, stalk, slice, piece, pieces, pc, pcs, each, pack, packet, pinch, dash, handful, whole`
