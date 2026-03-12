# String Utilities

Comprehensive string normalization and comparison utilities for consistent string handling across the entire project.

## Overview

The `string-utils.ts` module provides functions for:
- **Normalizing strings**: Converting to lowercase, removing special characters
- **Comparing strings**: Case-insensitive, special-character-insensitive comparison
- **Searching arrays**: Finding strings with normalized matching
- **Pattern matching**: Checking if strings contain or match patterns

## Why Use These Utilities?

### Problem
Different data sources use different formats for the same tokens:
- `USDC` vs `usdc` vs `USDC.e` vs `USDCe`
- `Aave V3` vs `aave-v3` vs `aave_v3`
- `USD₮0` vs `USDT0` vs `usdt-0`

### Solution
Normalize all strings before comparison to handle:
- ✅ Case differences (`USDC` = `usdc`)
- ✅ Special characters (`USDC.e` = `USDCe`)
- ✅ Hyphens/underscores (`aave-v3` = `aave_v3`)
- ✅ Unicode characters (`USD₮0` = `USDT0`)

## Core Functions

### `normalizeString(str: string): string`

Converts a string to normalized form (lowercase, alphanumeric only).

```typescript
import { normalizeString } from '../utils/string-utils.js';

normalizeString('USDC.e')    // 'usdce'
normalizeString('Aave-V3')   // 'aavev3'
normalizeString('USD₮0')     // 'usd0'
```

### `compareStrings(str1: string, str2: string): boolean`

Compare two strings for equality after normalization.

```typescript
import { compareStrings } from '../utils/string-utils.js';

compareStrings('USDC', 'usdc')           // true
compareStrings('USDC.e', 'USDCe')        // true
compareStrings('Aave-V3', 'aave_v3')     // true
compareStrings('USD₮0', 'USDT0')         // true
compareStrings('USDC', 'USDT')           // false
```

### `findInArray(arr: string[], target: string): boolean`

Check if a normalized string exists in an array.

```typescript
import { findInArray } from '../utils/string-utils.js';

const symbols = ['USDC', 'USDT', 'DAI'];

findInArray(symbols, 'usdc')     // true
findInArray(symbols, 'USDC.e')   // false
findInArray(symbols, 'dai')      // true
```

## Additional Functions

### `containsString(haystack, needle)`
Check if one string contains another (normalized).

```typescript
containsString('Aave V3', 'aave')        // true
containsString('USDC.e', 'usdc')         // true
containsString('Hyperliquid', 'liquid')  // true
```

### `startsWithString(str, prefix)`
Check if string starts with prefix (normalized).

```typescript
startsWithString('PT-KHYPE', 'pt')       // true
startsWithString('USDC.e', 'usdc')       // true
```

### `endsWithString(str, suffix)`
Check if string ends with suffix (normalized).

```typescript
endsWithString('USDC.e', 'e')            // true
endsWithString('WBTC', 'btc')            // true
```

### `matchesAnyPattern(patterns, target)`
Check if target contains any pattern from array.

```typescript
matchesAnyPattern(['aave', 'compound'], 'Aave V3')  // true
matchesAnyPattern(['perp', 'dex'], 'Hyperliquid')   // false
```

## Usage Examples

### Example 1: Filtering Symbols

```typescript
import { findInArray } from '../utils/string-utils.js';

const stablecoins = ['USDC', 'USDT', 'DAI', 'USDC.E', 'USDT.E'];

// Check if a symbol is a stablecoin (handles variants)
function isStablecoin(symbol: string): boolean {
  return findInArray(stablecoins, symbol);
}

isStablecoin('usdc')     // true
isStablecoin('USDC.e')   // true (matches 'USDC.E')
isStablecoin('usdt0')    // false
```

### Example 2: Protocol Matching

```typescript
import { compareStrings, containsString } from '../utils/string-utils.js';

function isKiloEx(projectName: string): boolean {
  return compareStrings(projectName, 'kiloex') || 
         compareStrings(projectName, 'kilo-ex');
}

isKiloEx('KiloEx')       // true
isKiloEx('kilo-ex')      // true
isKiloEx('KILOEX')       // true
```

### Example 3: Chain Comparison

```typescript
import { compareStrings } from '../utils/string-utils.js';

function isHyperliquid(chain: string): boolean {
  return compareStrings(chain, 'Hyperliquid') || 
         compareStrings(chain, 'HyperEVM');
}

isHyperliquid('hyperliquid')   // true
isHyperliquid('HYPEREVM')      // true
isHyperliquid('hyper-evm')     // true
```

## Best Practices

### ✅ DO

```typescript
// Use normalized comparison for user input
if (compareStrings(userInput, 'USDC')) {
  // Handle USDC
}

// Use findInArray for checking membership
if (findInArray(allowedTokens, token)) {
  // Token is allowed
}

// Use containsString for partial matching
if (containsString(projectName, 'aave')) {
  // Handle Aave projects
}
```

### ❌ DON'T

```typescript
// Don't use direct string comparison
if (symbol === 'USDC') {  // Won't match 'usdc' or 'USDC.e'
  // ...
}

// Don't use .includes() without normalization
if (symbol.includes('USDC')) {  // Won't match 'usdc'
  // ...
}

// Don't use .toLowerCase() alone
if (symbol.toLowerCase() === 'usdc.e') {  // Won't match 'USDCe'
  // ...
}
```

## Integration with FilterConfig

The `filters.ts` configuration uses these utilities:

```typescript
import { compareStrings, findInArray } from '../utils/string-utils.js';

export function getLendingTvlThreshold(symbol: string, chain: string, project: string): number {
  // Normalized comparison for project
  if (compareStrings(project, 'kiloex')) {
    // Normalized array search for symbol
    if (findInArray(FilterConfig.lending.symbolGroups.stablecoins, symbol)) {
      return FilterConfig.lending.tvl.kiloex;
    }
  }
  
  // Normalized chain comparison
  if (compareStrings(chain, 'Hyperliquid')) {
    return FilterConfig.lending.tvl.hyperliquid;
  }
  
  // ... more comparisons
}
```

## Performance

All functions are optimized for performance:
- ✅ Single-pass normalization
- ✅ No regex unless necessary
- ✅ Early returns when possible
- ✅ Minimal string allocations

## Testing

```typescript
// Test normalization
console.assert(normalizeString('USDC.e') === 'usdce');
console.assert(normalizeString('Aave-V3') === 'aavev3');

// Test comparison
console.assert(compareStrings('USDC', 'usdc') === true);
console.assert(compareStrings('USDC.e', 'USDCe') === true);

// Test array search
const arr = ['USDC', 'USDT'];
console.assert(findInArray(arr, 'usdc') === true);
console.assert(findInArray(arr, 'dai') === false);
```

## Migration Guide

### Before (Manual Normalization)

```typescript
const symbolUpper = symbol.toUpperCase();
const projectLower = project.toLowerCase();

if (projectLower === 'kiloex') {
  if (stablecoins.includes(symbolUpper)) {
    // ...
  }
}
```

### After (Using Utilities)

```typescript
if (compareStrings(project, 'kiloex')) {
  if (findInArray(stablecoins, symbol)) {
    // ...
  }
}
```

## Benefits

✅ **Consistency**: All string comparisons use the same normalization  
✅ **Maintainability**: Change normalization logic in one place  
✅ **Reliability**: Handles edge cases (unicode, special chars)  
✅ **Type Safety**: TypeScript ensures correct usage  
✅ **Performance**: Optimized for common use cases
