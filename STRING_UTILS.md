# String Utilities

Utilities for normalized string comparison, implemented in `backend/src/utils/string-utils.ts`.

## Module Surface

The module exports exactly three functions:

- `normalizeString(str)`
- `compareStrings(str1, str2)`
- `findInArray(arr, target)`

Functions such as `containsString`, `startsWithString`, `endsWithString`, and `matchesAnyPattern` do **not** exist in this module. Use plain `includes()` / `startsWith()` / `endsWith()` with `normalizeString()` when you need those semantics.

## Normalization Rule

`normalizeString`:

1. lowercases the input
2. applies Unicode NFD decomposition
3. removes combining diacritical marks (`U+0300..U+036F`)
4. removes every character that is not `a-z` or `0-9`

```ts
normalizeString('USDC.e');      // 'usdce'
normalizeString('Aave-V3');     // 'aavev3'
normalizeString('Aave V3');     // 'aavev3'
normalizeString('aave_v3');     // 'aavev3'
normalizeString('USD₮0');       // 'usd0'  (the ₮ symbol is removed)
normalizeString('');            // ''
```

### Important limitation

Because all non-alphanumeric characters are dropped, the Tugrik sign is not treated as a `T`:

```ts
compareStrings('USD₮0', 'USDT0'); // false  ('usd0' !== 'usdt0')
```

For token/asset identity across venues, use `backend/src/utils/symbolNormalizer.ts` (`normalizeCanonicalAssetSymbol`, `normalizeSpotSymbol`, `normalizePerpSymbol`, `smartMatch`, etc.) rather than `string-utils.ts`.

## API

### `normalizeString(str: string): string`

Returns the normalized key. Returns `''` for empty/falsy input.

```ts
normalizeString('Hyperliquid'); // 'hyperliquid'
normalizeString('Hyper-Liquid'); // 'hyperliquid'
```

### `compareStrings(str1: string, str2: string): boolean`

Compares two strings after normalization.

```ts
compareStrings('USDC', 'usdc');          // true
compareStrings('USDC.e', 'USDCe');       // true
compareStrings('Aave-V3', 'aave_v3');    // true
compareStrings('Aave V3', 'AAVEV3');     // true
compareStrings('USDC', 'USDT');          // false
```

### `findInArray(arr: string[], target: string): boolean`

Checks normalized membership in an array.

```ts
findInArray(['USDC', 'USDT', 'DAI'], 'usdc');   // true
findInArray(['USDC', 'USDT', 'DAI'], 'dai');    // true
findInArray(['USDC', 'USDT', 'DAI'], 'USDC.e'); // false
```

## Usage

### Lending filters

`backend/src/config/filters.ts` uses these helpers for case/format-insensitive protocol, chain and symbol matching:

```ts
import { compareStrings, findInArray } from '../utils/string-utils.js';

if (compareStrings(project, 'kiloex')) {
  if (findInArray(FilterConfig.lending.symbolGroups.stablecoins, symbol)) {
    return FilterConfig.lending.tvl.kiloex;
  }
}

if (compareStrings(chain, 'Hyperliquid')) {
  return FilterConfig.lending.tvl.hyperliquid;
}
```

### Membership checks

```ts
if (findInArray(allowedTokens, token)) {
  // Token is allowed after normalization
}
```

### When to use something else

Use `symbolNormalizer.ts` when the task involves:

- spot vs perp symbol formats
- wrapped/bridged assets (`UBTC`, `UETH`)
- 1000-prefix contracts (`1000PEPE`)
- contract multipliers (`10000SATS`)
- cross-venue canonical asset identity

`string-utils.ts` is intentionally limited to forgiving human-style name matching.

## Testing

The helpers are covered indirectly by the lending/filters and config tests. Quick assertions:

```ts
console.assert(normalizeString('USDC.e') === 'usdce');
console.assert(normalizeString('Aave-V3') === 'aavev3');
console.assert(compareStrings('Aave V3', 'aave_v3') === true);
console.assert(compareStrings('USD₮0', 'USDT0') === false);
console.assert(findInArray(['USDC', 'USDT'], 'usdc') === true);
```

Run backend tests:

```bash
cd backend
pnpm test
```
