# sass-funcs

A collection of Sass utility functions for [lists](#module-list), [strings](#module-string), [numbers](#module-number), and [meta](#module-meta) — inspired by familiar JS APIs. Where behaviour differs from a JS counterpart, the difference is noted. Uses only Sass built-in modules.

---

## Installation

```sh
npm install sass-funcs
```

> Requires [`sass`](https://www.npmjs.com/package/sass) `>= 1.33.0` or [`sass-embedded`](https://www.npmjs.com/package/sass-embedded) `>= 1.33.0` — install one, not both. `sass-embedded` is recommended: it wraps a native Dart binary and is typically 20–30% faster to compile than the pure-JS `sass` package.

---

## Modules

| Module                     | Import path                          | Description                                              |
| -------------------------- | ------------------------------------ | -------------------------------------------------------- |
| [`list`](#module-list)     | [`sass-funcs/list`](./list.scss)     | Functions for creating, transforming, and querying lists |
| [`meta`](#module-meta)     | [`sass-funcs/meta`](./meta.scss)     | Functions for type inspection and emptiness checking     |
| [`number`](#module-number) | [`sass-funcs/number`](./number.scss) | Functions for unit conversion and numeric operations     |
| [`string`](#module-string) | [`sass-funcs/string`](./string.scss) | Functions for string transformation and parsing          |

---

## Module: List

```scss
@use 'sass-funcs/list' as l;
```

### `concat($lists...)`

Concatenates zero or more lists into a single flat list.

```scss
l.concat((a, b), (c, d))  // → a, b, c, d
l.concat()                // → ()
```

### `flat($list, $depth: null)`

Recursively flattens nested lists. Pass `$depth` to limit how many levels deep the flattening goes; omit it (or pass `null`) for unlimited depth. Similar to `Array.prototype.flat(depth)`.

> **Default depth differs:** Sass defaults to `null` (unlimited flattening); JS defaults to `1`.

```scss
l.flat((a, (b, (c, d))))        // → a, b, c, d
l.flat((a, (b, (c, d))), 1)     // → a, b, (c, d)
```

### `includes($list, $value)`

Returns `true` if `$value` is present in `$list`, `false` otherwise. Similar to `Array.prototype.includes(value)`.

```scss
l.includes((a, b, c), b)   // → true
l.includes((a, b, c), z)   // → false
```

### `join($list, $separator: ', ')`

Serializes a list into a string, inserting `$separator` between elements. Similar to `Array.prototype.join(separator)`.

> **Default separator differs:** Sass defaults to `', '` (comma + space); JS defaults to `','` (comma only).

```scss
l.join((a, b, c))          // → 'a, b, c'
l.join((a, b, c), ' | ')   // → 'a | b | c'
```

### `push($list, $val, $separator: comma)`

Appends `$val` to the end of `$list`. `$separator` controls the resulting list's separator: `comma`, `space`, `slash`, or `auto`. Similar to `Array.prototype.push(val)` — unlike JS, this returns a new list rather than mutating in place.

```scss
l.push((a, b), c)   // → a, b, c
```

### `slice($list, $start: null, $end: null)`

Returns the sublist from index `$start` to `$end` (1-based, inclusive). Omitting either bound defaults to the start or end of the list. Similar to `Array.prototype.slice(start, end)`.

> **Index base and bounds differ:** Sass uses 1-based indices with an inclusive end; JS uses 0-based indices with an exclusive end. `slice(list, 2, 3)` returns elements 2 and 3; the JS equivalent would be `array.slice(1, 3)`.

```scss
l.slice((a, b, c, d), 2, 3)   // → b, c
l.slice((a, b, c, d), 3)       // → c, d
```

### `sort($list, $desc: false)`

Sorts a list alphabetically (case-insensitive). Numbers sort by value. Pass `$desc: true` for descending order. Similar to `Array.prototype.sort()`.

> **Intentional differences:** Numbers sort by value (JS sorts lexicographically by default, so `[10, 2, 1].sort()` → `[1, 10, 2]` in JS). Sorting is also case-insensitive; JS is case-sensitive by default.

```scss
l.sort((c, a, b))               // → a, b, c
l.sort((c, a, b), $desc: true)  // → c, b, a
l.sort((3, 1, 2))               // → 1, 2, 3
```

---

## Module: Meta

```scss
@use 'sass-funcs/meta' as m;
```

### `is-empty($value, $depth: 0)`

Returns `true` if `$value` is empty. Covers `null`, blank strings, empty lists, and empty maps. Pass `$depth > 0` or `null` to also check nested structures. Similar to `_.isEmpty(value)` (lodash) — extended with configurable recursion depth.

> **Whitespace strings:** A string containing only spaces (e.g. `'   '`) is considered empty. JS `_.isEmpty('   ')` returns `false`.

```scss
m.is-empty(null)                   // → true
m.is-empty('')                     // → true
m.is-empty('   ')                  // → true  (whitespace-only)
m.is-empty(())                     // → true
m.is-empty((a: ()))                // → false  (shallow)
m.is-empty((a: ()), $depth: 1)     // → true   (checks nested)
```

### `is-type($value, $types)`

Returns `true` if `$value`'s type is in the `$types` list.

```scss
m.is-type(1rem, ('number', 'string'))   // → true
m.is-type(true, ('number', 'string'))   // → false
```

### `type-of($value)`

Returns the Sass type name of any value. A thin wrapper over `meta.type-of` — included for consistent import paths within this package. Similar to `typeof value` — returns Sass-specific type names (`'number'`, `'string'`, `'map'`, `'list'`, `'color'`, `'bool'`, `'null'`).

```scss
m.type-of(1px)         // → 'number'
m.type-of('hello')     // → 'string'
m.type-of((a, b))      // → 'list'
m.type-of((a: 1))      // → 'map'
```

---

## Module: Number

```scss
@use 'sass-funcs/number' as n;
```

### `$SUPPORTED_LENGTH_UNITS`

A list of the CSS length units supported by `add-unit` and `convert-unit`:

```
px, rem, em, cm, mm, Q, in, pc, pt
```

### `add-unit($number, $unit)`

Attaches a CSS length unit to a unitless number.

```scss
n.add-unit(16, 'px')    // → 16px
n.add-unit(1, 'rem')    // → 1rem
```

### `convert-unit($number, $from-unit, $to-unit)`

Converts a number from one CSS length unit to another.

```scss
n.convert-unit(16, 'px', 'rem')   // → 1rem
n.convert-unit(1, 'in', 'px')     // → 96px
```

### `resolve-index($index, $length)`

Resolves a positive or negative integer index to a 1-based index within a collection of `$length`. Returns `null` if out of bounds. Negative indices count from the end, matching Python and modern JS convention.

> **1-based output:** Returns a 1-based index, matching Sass's index convention. Negative indices resolve relative to the end — `-1` on a length-5 list returns `5` (the last position), not `4`.

```scss
n.resolve-index(2, 5)    // → 2
n.resolve-index(-1, 5)   // → 5
n.resolve-index(0, 5)    // → null
n.resolve-index(6, 5)    // → null
```

### `to-fixed($number, $digits: 0)`

Rounds a number to `$digits` decimal places. Similar to `Number.prototype.toFixed(digits)` — returns a number, not a string.

```scss
n.to-fixed(3.14159, 2)   // → 3.14
n.to-fixed(1.5)           // → 2
```

---

## Module: String

```scss
@use 'sass-funcs/string' as s;
```

### `capitalize($string)`

Capitalizes the first character and lowercases the rest. Similar to `_.capitalize(string)` (lodash).

> **Input is trimmed first:** Leading and trailing spaces are stripped before capitalizing. JS lodash preserves them.

```scss
s.capitalize('hello world')   // → 'Hello world'
s.capitalize('HELLO')         // → 'Hello'
```

### `replace-all($string, $pattern, $replacement: '')`

Replaces every occurrence of `$pattern` in `$string` with `$replacement`. Defaults to deletion if `$replacement` is omitted. Similar to `String.prototype.replaceAll(pattern, replacement)`.

```scss
s.replace-all('foo bar foo', 'foo', 'baz')   // → 'baz bar baz'
s.replace-all('a--b--c', '--')               // → 'abc'
```

### `slugify($string)`

Converts a string to a URL-safe slug: lowercased, spaces replaced with hyphens, special characters removed. Similar to `_.kebabCase(string)` (lodash) — with stricter special-character removal.

```scss
s.slugify('Hello World!')     // → 'hello-world'
s.slugify('Font Size (lg)')   // → 'font-size-lg'
```

### `split($string, $separator: null)`

Splits a string into a list using `$separator`. An empty string splits into individual characters. `null` returns the string wrapped in a single-element list. Similar to `String.prototype.split(separator)`.

```scss
s.split('a,b,c', ',')   // → a, b, c
s.split('abc', '')       // → a, b, c
s.split('abc')           // → 'abc'
```

### `trim($string)`

Removes leading and trailing whitespace. Similar to `String.prototype.trim()`.

> **Space characters only:** Strips the space character `' '`. JS strips all whitespace (`\t`, `\n`, `\r`, etc.) — Sass has no native regex equivalent for this.

```scss
s.trim('  hello  ')   // → 'hello'
```

### `upper-snake-case($string)`

Converts a string to `UPPER_SNAKE_CASE` by replacing hyphens and spaces with underscores.

```scss
s.upper-snake-case('font-size')    // → 'FONT_SIZE'
s.upper-snake-case('my variable')  // → 'MY_VARIABLE'
```

[Back to top](#sass-funcs)
