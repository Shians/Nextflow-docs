# Operators

## Transformation Operators

These operators apply functions to each item in a channel, producing a new channel with transformed data. Use them to modify, aggregate, or restructure data as it flows through your pipeline.

### map
Use `map` to transform each item in a channel, such as converting formats, scaling values, or extracting fields.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting the result of applying a function to each input item.
- **Arguments:**
    - `closure` (required): The transformation function to apply to each item.

```groovy
Channel.of(1, 2, 3)
    .map { it * 2 }
    .view() // Output: 2, 4, 6
```
**See also:** `flatMap`. While `map` transforms each item individually, `flatMap` can emit multiple items for each input by flattening collections.
**Example (difference from `flatMap`):**
```groovy
// map emits a list as a single item
Channel.of([1, 2], [3, 4])
    .map { it }
    .view() // Output: [1, 2], [3, 4]
// flatMap emits each element of the list as a separate item
Channel.of([1, 2], [3, 4])
    .flatMap { it }
    .view() // Output: 1, 2, 3, 4
```

### flatMap
Use `flatMap` to expand nested lists or split items into multiple outputs for further processing.

- **Input:** A channel emitting collections or items that can be expanded.
- **Output:** A channel emitting each element from the collections, flattened into a single stream.
- **Arguments:**
    - `closure` (required): The transformation function returning a collection for each item.

```groovy
Channel.of([1, 2], [3, 4])
    .flatMap { it }
    .view() // Output: 1, 2, 3, 4
```
**See also:** `map`. Use `flatMap` when your transformation returns collections and you want to emit their elements individually, unlike `map` which emits the collection as a single item.
**Example (difference from `map`):**
```groovy
// flatMap emits each element of the list as a separate item
Channel.of([1, 2], [3, 4])
    .flatMap { it }
    .view() // Output: 1, 2, 3, 4
// map emits a list as a single item
Channel.of([1, 2], [3, 4])
    .map { it }
    .view() // Output: [1, 2], [3, 4]
```

### collect
Use `collect` to gather all items into a single collection for summary operations or final output.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting a single list containing all input items.
- **Arguments:**
    - `closure` (optional): Mapping function applied to each item before it is collected. If omitted, items are collected as-is.
    - `flat` (optional): Flatten nested list structures into individual items (default: `true`).
    - `sort` (optional): Sort collected items, by natural ordering or a custom closure/comparator (default: `false`).

```groovy
Channel.of(1, 2, 3)
    .collect()
    .view() // Output: [1, 2, 3]
Channel.of('hello', 'ciao', 'bonjour')
    .collect { v -> v.length() }
    .view() // Output: [5, 4, 7]
```
**See also:** `toList`. Both aggregate all items into a list, but `collect` accepts an optional mapping/sorting closure. On an empty channel, `collect` emits nothing while `toList` emits `[]`.
**Example (difference from `toList`):**
```groovy
// on an empty channel, collect emits nothing...
Channel.empty().collect().view() // Output: (nothing emitted)
// ...but toList always emits a list, even if empty
Channel.empty().toList().view() // Output: []
```

### flatten
Use `flatten` to remove one level of nesting from channel items, useful after aggregation or grouping.

- **Input:** A channel emitting nested collections.
- **Output:** A channel emitting all elements from the nested collections as a flat stream.
- **Arguments:** None.

```groovy
Channel.of([1, 2], [3, 4])
    .flatten()
    .view() // Output: 1, 2, 3, 4
```
**See also:** `flatMap`. Both flatten nested collections, but `flatMap` applies a transformation before flattening, while `flatten` only removes one level of nesting.
**Example (difference from `flatMap`):**
```groovy
// flatten removes one level of nesting, but does not apply a transformation
Channel.of([1, 2], [3, 4])
    .flatten()
    .view() // Output: 1, 2, 3, 4
// flatMap applies a transformation and flattens
Channel.of([1, 2], [3, 4])
    .flatMap { it }
    .view() // Output: 1, 2, 3, 4
```

### toList
Use `toList` to collect all items for batch processing or output as a single list.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting a single list of all items, similar to `collect`.
- **Arguments:** None.

```groovy
Channel.of('a', 'b', 'c')
    .toList()
    .view() // Output: ['a', 'b', 'c']
```
**See also:** `collect`. Both gather all items into a list, but `toList` is a convenience method for this specific case.
**Example (difference from `collect`):**
```groovy
// toList always collects all items into a list
Channel.of(1, 2, 3)
    .toList()
    .view() // Output: [1, 2, 3]
// collect can be customized for other aggregations
Channel.of(1, 2, 3)
    .collect { acc, val -> acc + val * 2 }
    .view() // Output: [2, 4, 6]
```

### toSortedList
Use `toSortedList` to collect and sort all items, such as for reporting or ordered output.

- **Input:** A channel emitting comparable items.
- **Output:** A channel emitting a single sorted list of all items.
- **Arguments:**
    - `closure` (optional): Custom comparator for sorting.

```groovy
Channel.of(3, 1, 2)
    .toSortedList()
    .view() // Output: [1, 2, 3]
```
**See also:** `toList`. `toSortedList` sorts the items, while `toList` preserves their original order.
**Example (difference from `toList`):**
```groovy
// toSortedList sorts the items
Channel.of(3, 1, 2)
    .toSortedList()
    .view() // Output: [1, 2, 3]
// toList preserves order
Channel.of(3, 1, 2)
    .toList()
    .view() // Output: [3, 1, 2]
```

### transpose
Use `transpose` to flatten nested lists in tuples, emitting a new tuple for each element in the nested lists.

- **Input:** A channel emitting tuples or lists, where at least one element is a list.
- **Output:** A channel emitting tuples with the nested lists expanded into separate items.
- **Arguments:**
    - `by` (optional): Index or list of indices to transpose (default: all list elements).
    - `remainder` (optional): If true, emits incomplete tuples with `null` for missing elements.

```groovy
Channel.of(
    [1, ['A', 'B', 'C']],
    [2, ['C', 'A']],
    [3, ['B', 'D']]
)
.transpose()
.view()
// Output:
// [1, A]
// [1, B]
// [1, C]
// [2, C]
// [2, A]
// [3, B]
// [3, D]
```
**See also:** `groupTuple`. While `groupTuple` groups by key, `transpose` expands nested lists into separate tuples.
**Example (with remainder):**
```groovy
Channel.of(
    [1, [1], ['A']],
    [2, [1, 2], ['B', 'C']],
    [3, [1, 2, 3], ['D', 'E']]
)
.transpose(remainder: true)
.view()
// Output:
// [1, 1, A]
// [2, 1, B]
// [2, 2, C]
// [3, 1, D]
// [3, 2, E]
// [3, 3, null]
```

---

## Filtering Operators

These operators filter items in a channel based on specified conditions. Use them to select, limit, or exclude data as it flows through your pipeline.

### filter
Use `filter` to select items that meet specific criteria, such as filtering by value, pattern, or type.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting only items that match the given predicate (closure, regex, or type).
- **Arguments:**
    - `closure` (optional): Predicate function.
    - `regex` (optional): Regular expression for string matching.
    - `type` (optional): Class type for filtering by type.

```groovy
// By closure
Channel.of(1, 2, 3, 4)
    .filter { it % 2 == 0 }
    .view() // Output: 2, 4
// By regex
Channel.of('apple', 'banana', 'apricot')
    .filter(/^a.*/)
    .view() // Output: apple, apricot
// By type
Channel.of(1, 'a', 2, 'b')
    .filter(Number)
    .view() // Output: 1, 2
```
**See also:** `take`, `skip`. While `filter` selects items based on a condition, `take` and `skip` select items based on their position in the stream.
**Example (difference from `take` and `skip`):**
```groovy
// filter selects items by condition
Channel.of(1, 2, 3, 4)
    .filter { it > 2 }
    .view() // Output: 3, 4
// take selects by position
Channel.of(1, 2, 3, 4)
    .take(2)
    .view() // Output: 1, 2
// skip ignores by position
Channel.of(1, 2, 3, 4)
    .skip(2)
    .view() // Output: 3, 4
```

### take
Use `take` to limit the number of items processed downstream, such as for sampling or testing.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting only the first N items.
- **Arguments:**
    - `n` (required): Number of items to emit.

```groovy
Channel.of('a', 'b', 'c', 'd')
    .take(2)
    .view() // Output: 'a', 'b'
```
**See also:** `skip`, `filter`. `take` emits the first N items, while `skip` ignores the first N. Use `filter` for condition-based selection.
**Example (difference from `filter` and `skip`):**
```groovy
// take emits the first N items
Channel.of('a', 'b', 'c', 'd')
    .take(2)
    .view() // Output: 'a', 'b'
// filter emits items matching a condition
Channel.of('a', 'b', 'c', 'd')
    .filter { it > 'b' }
    .view() // Output: 'c', 'd'
// skip ignores the first N items
Channel.of('a', 'b', 'c', 'd')
    .skip(2)
    .view() // Output: 'c', 'd'
```

### skip
Use `skip` to ignore a fixed number of initial items, such as skipping headers or warm-up data.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting all items except the first N.
- **Arguments:**
    - `n` (required): Number of items to skip.

```groovy
Channel.of(10, 20, 30, 40)
    .skip(2)
    .view() // Output: 30, 40
```
**See also:** `take`. `skip` ignores the first N items, while `take` emits only the first N.
**Example (difference from `take`):**
```groovy
// skip ignores the first N items
Channel.of(10, 20, 30, 40)
    .skip(2)
    .view() // Output: 30, 40
// take emits only the first N items
Channel.of(10, 20, 30, 40)
    .take(2)
    .view() // Output: 10, 20
```

### distinct
Use `distinct` to remove consecutively repeated items, such as collapsing runs of identical sorted values.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting items with consecutive duplicates removed.
- **Arguments:**
    - `closure` (optional): Transform each item before comparing it to its predecessor.

```groovy
Channel.of(1, 1, 2, 2, 2, 3, 1, 1)
    .distinct()
    .view() // Output: 1, 2, 3, 1
```
**See also:** `unique`. `distinct` only collapses duplicates that are adjacent in the stream and can emit immediately, while `unique` removes duplicates across the whole channel and must buffer everything first.
**Example (difference from `unique`):**
```groovy
// distinct only collapses adjacent duplicates
Channel.of(1, 1, 2, 1)
    .distinct()
    .view() // Output: 1, 2, 1
// unique removes duplicates across the entire channel
Channel.of(1, 1, 2, 1)
    .unique()
    .view() // Output: 1, 2
```

### unique
Use `unique` to remove all duplicate items from a channel, such as deduplicating sample IDs regardless of order.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting only the first occurrence of each distinct item.
- **Arguments:**
    - `closure` (optional): Transform each item before evaluating uniqueness.

```groovy
Channel.of(1, 1, 2, 2, 2, 3, 1)
    .unique()
    .view() // Output: 1, 2, 3
```

### first
Use `first` to grab a single item from a channel, such as picking a reference sample or the first match.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting the first item, or the first item matching a condition.
- **Arguments:**
    - `condition` (optional): Value, regex, type, or predicate closure used to select the first matching item. If omitted, emits the very first item.

```groovy
Channel.of(1, 2, 3)
    .first()
    .view() // Output: 1
Channel.of(1, 2, 3, 4)
    .first { it > 2 }
    .view() // Output: 3
```

### ifEmpty
Use `ifEmpty` to supply a fallback value when a channel might not emit anything, such as defaulting an optional input.

- **Input:** A channel emitting any type of item.
- **Output:** The source channel unchanged, or a channel emitting a single default value if the source was empty.
- **Arguments:**
    - `value` (required): Default value (or closure) to emit if the source channel is empty.

```groovy
Channel.empty()
    .ifEmpty('none found')
    .view() // Output: none found
```

---

## Combining Operators

These operators combine data from multiple channels. Use them to merge, join, or relate data streams for complex workflows. `join`, `combine`, and `cross` typically operate on key-based [tuples](Inputs.md#tuple), the same shape used by [`tuple` inputs](Inputs.md#tuple) and [`tuple` outputs](Outputs.md#tuple).

### mix
Use `mix` to merge multiple data sources into a single stream for unified processing.

- **Input:** Two or more channels emitting any type of item.
- **Output:** A single channel emitting all items from the input channels, in the order they become available.
- **Arguments:**
    - `channels` (required): Two or more channels to merge.

```groovy
Channel.of(1, 2)
    .mix(Channel.of(3, 4))
    .view() // Output: 1, 2, 3, 4 (order may vary)
```
**See also:** `concat`. Both merge multiple channels, but `mix` emits items as soon as they are available from any input, while `concat` emits all items from one channel before moving to the next.
**Example (difference from `concat`):**
```groovy
// mix interleaves items as they arrive
Channel.of(1, 2)
    .mix(Channel.of(10, 20))
    .view() // Output: 1, 10, 2, 20 (order may vary)
// concat preserves channel order
Channel.of(1, 2)
    .concat(Channel.of(10, 20))
    .view() // Output: 1, 2, 10, 20
```

### concat
Use `concat` to chain channels together in a specific order, such as appending results or logs.

- **Input:** Two or more channels emitting any type of item.
- **Output:** A single channel emitting all items from the first channel, then all items from the next, and so on.
- **Arguments:**
    - `channels` (required): Two or more channels to concatenate.

```groovy
Channel.of('a', 'b')
    .concat(Channel.of('c', 'd'))
    .view() // Output: 'a', 'b', 'c', 'd'
```
**See also:** `mix`. Use `concat` when you need to preserve the order of channels, unlike `mix` which interleaves items as they arrive.
**Example (difference from `mix`):**
```groovy
// concat emits all items from the first channel, then the next
Channel.of('a', 'b')
    .concat(Channel.of('x', 'y'))
    .view() // Output: 'a', 'b', 'x', 'y'
// mix interleaves items as they become available
Channel.of('a', 'b')
    .mix(Channel.of('x', 'y'))
    .view() // Output: 'a', 'x', 'b', 'y' (order may vary)
```

### combine
Use `combine` to produce all combinations (Cartesian product) of two channels, optionally filtering by a matching key.

- **Input:** Two channels emitting any type of item, or tuples/lists when using the `by` option.
- **Output:** A channel emitting all combinations of items from both channels. When `by` is used, only combinations with matching keys are emitted, merged and flattened into single tuples.
- **Arguments:**
    - `other` (required): The other channel to combine with.
    - `by` (optional): Index or list of indices to use as the matching key. When specified, only items with matching keys are combined, and the result is flattened.

```groovy
// Without by: produces full Cartesian product
Channel.of(1, 2, 3)
    .combine(Channel.of('hello', 'ciao'))
    .view()
// Output: [1, hello], [2, hello], [3, hello], [1, ciao], [2, ciao], [3, ciao]

// With by: combines items with matching keys, flattened
Channel.of([1, 'alpha'], [2, 'beta'])
    .combine(Channel.of([1, 'x'], [1, 'y'], [2, 'p']), by: 0)
    .view()
// Output: [1, alpha, x], [1, alpha, y], [2, beta, p]
```
**See also:** `cross`, `join`. `combine` produces the full Cartesian product (or filtered by key with `by`), `cross` produces combinations only for matching keys (keeping pairs nested), and `join` merges one item per key.
**Example (difference from `cross` and `join`):**
```groovy
// combine with by: merges and flattens all matching pairs
Channel.of([1, 'A'], [1, 'B'])
    .combine(Channel.of([1, 10], [1, 20]), by: 0)
    .view() // Output: [1, A, 10], [1, A, 20], [1, B, 10], [1, B, 20]

// cross: keeps pairs nested, matches by key
Channel.of([1, 'A'], [1, 'B'])
    .cross(Channel.of([1, 10], [1, 20]))
    .view() // Output: [[1, A], [1, 10]], [[1, A], [1, 20]], [[1, B], [1, 10]], [[1, B], [1, 20]]

// join: emits one merged tuple per matching item
Channel.of([1, 'A'], [1, 'B'])
    .join(Channel.of([1, 10], [1, 20]), by: 0)
    .view() // Output: [1, A, 10], [1, B, 20]
```

### join
Use `join` to enrich or correlate data from two sources, such as joining metadata with results.

- **Input:** Two channels emitting tuples or lists, with a key to match items.
- **Output:** A channel emitting joined tuples for each matching key, similar to a database join.
- **Arguments:**
    - `other` (required): The other channel to join with.
    - `by` (optional): Index, or list of indices, to use as the matching key (default: `0`).
    - `remainder` (optional): If `true`, emit unmatched items at the end instead of discarding them (default: `false`).

```groovy
Channel.of([1, 'foo'], [2, 'bar'])
    .join(Channel.of([1, 30], [2, 40]), by: 0)
    .view() // Output: [1, 'foo', 30], [2, 'bar', 40]
```
**See also:** `combine`. Use `join` for a database-style join (one output per key), while `combine` can emit multiple pairs for each key.
**Example (difference from `combine`):**
```groovy
// join emits one joined tuple per key
Channel.of([2, 'foo'], [2, 'bar'])
    .join(Channel.of([2, 100], [2, 200]), by: 0)
    .view() // Output: [2, 'foo', 100], [2, 'bar', 200]
// combine emits all possible pairs for matching keys
Channel.of([2, 'foo'], [2, 'bar'])
    .combine(Channel.of([2, 100], [2, 200]), by: 0)
    .view() // Output: [[2, 'foo'], [2, 100]], [[2, 'foo'], [2, 200]], [[2, 'bar'], [2, 100]], [[2, 'bar'], [2, 200]]
```

### cross
Use `cross` to emit pairwise combinations of two channels where items share a matching key.

- **Input:** Two channels emitting tuples, lists, or single values.
- **Output:** A channel emitting pairs (as nested lists) for items that have matching keys. By default, the key is the first element in a tuple/list, or the value itself for other types.
- **Arguments:**
    - `other` (required): The other channel to cross with.
    - `closure` (optional): A closure to extract the matching key from each item.

```groovy
// Default: matches by first element of tuples
Channel.of([1, 'alpha'], [2, 'beta'])
    .cross(Channel.of([1, 'x'], [1, 'y'], [2, 'p']))
    .view()
// Output: [[1, alpha], [1, x]], [[1, alpha], [1, y]], [[2, beta], [2, p]]

// Custom key extraction
Channel.of([1, 'alpha'], [2, 'beta'])
    .cross(Channel.of([1, 'a'], [2, 'b'])) { v -> v[1][0] }
    .view()
// Output: [[1, alpha], [1, a]], [[2, beta], [2, b]]
```
**See also:** `combine`. `cross` filters by matching keys and keeps pairs nested, while `combine` produces the full Cartesian product (unless `by` is specified, in which case it filters and flattens).
**Example (difference from `combine`):**
```groovy
// cross: only matching keys, pairs stay nested
Channel.of([1, 'A'], [2, 'B'])
    .cross(Channel.of([1, 10], [1, 20], [2, 30]))
    .view() // Output: [[1, A], [1, 10]], [[1, A], [1, 20]], [[2, B], [2, 30]]

// combine with by: only matching keys, pairs are flattened
Channel.of([1, 'A'], [2, 'B'])
    .combine(Channel.of([1, 10], [1, 20], [2, 30]), by: 0)
    .view() // Output: [1, A, 10], [1, A, 20], [2, B, 30]

// combine without by: full Cartesian product
Channel.of([1, 'A'], [2, 'B'])
    .combine(Channel.of([1, 10], [2, 20]))
    .view() // Output: [[1, A], [1, 10]], [[1, A], [2, 20]], [[2, B], [1, 10]], [[2, B], [2, 20]]
```

---

## Grouping and Batching Operators

These operators group or batch items from a channel. Use them to organize data into manageable sets for downstream processing or aggregation.

### buffer
Use `buffer` to batch items for grouped processing, such as running jobs in chunks or controlling resource usage.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting lists of items, each list containing up to the specified batch size or meeting a condition.
- **Arguments:**
    - `size` (optional): Number of items per batch.
    - `remainder` (optional): If `true`, emit any remaining items as a partial batch at the end (default: `false`).
    - `skip` (optional): Number of items to skip before starting the next batch (for sliding windows).
    - `openingCondition` (optional): Start a new batch when this condition is met (can be a value, regex, type, or closure).
    - `closingCondition` (optional): Emit the batch when this condition is met (can be a value, regex, type, or closure).

```groovy
Channel.of(1, 2, 3, 4, 5)
    .buffer(size: 2)
    .view() // Output: [1, 2], [3, 4] — trailing [5] is discarded
Channel.of(1, 2, 3, 4, 5)
    .buffer(size: 2, remainder: true)
    .view() // Output: [1, 2], [3, 4], [5]
```
**See also:** `collate`. `buffer` is more flexible and supports batching by size, time, or custom conditions.

### collate
Use `collate` to create sliding windows or overlapping groups for analysis or rolling computations.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting lists of a fixed size, optionally with overlap between groups.
- **Arguments:**
    - `size` (required): Number of items per group.
    - `step` (optional): Number of items to advance between groups (default: same as `size`).
    - `remainder` (optional): If `true`, emit any remaining items as a partial group at the end (default: `true`).

```groovy
Channel.of(1, 2, 3, 4)
    .collate(3, 1)
    .view() // Output: [1, 2, 3], [2, 3, 4], [3, 4], [4]
```
**See also:** `buffer`. `collate` is a simpler batching operator for fixed-size groups and sliding windows.

### groupTuple
Use `groupTuple` to group related data by a common identifier, such as grouping results by sample or category.

- **Input:** A channel emitting tuples or lists with a key field.
- **Output:** A channel emitting pairs of [key, grouped items] where items share the same key.
- **Arguments:**
    - `by` (optional): Index or list of indices to use as the grouping key (default: `0`).
    - `size` (optional): Number of items expected per group. If set, groups are emitted as soon as they reach this size.
    - `remainder` (optional): If `true`, incomplete groups (with fewer than `size` items) are emitted at the end (default: `false`).
    - `sort` (optional): Sorting criteria for grouped items. Can be `false` (no sort, default), `true` (natural order), `'hash'`, `'deep'`, or a custom closure/comparator.

```groovy
Channel.of([1, 'A'], [1, 'B'], [2, 'C'])
    .groupTuple(by: 0)
    .view() // Output: [1, ['A', 'B']], [2, ['C']]
```
**See also:** `transpose`. `groupTuple` groups by key, while `transpose` expands nested lists in tuples. Works on the [tuple](Inputs.md#tuple) shape shared with `tuple` inputs/outputs.

---

## Splitting and Parsing Operators

These operators split or parse structured data from files or strings into records or chunks for downstream processing. Their source channels are typically built from a [`path` input](Inputs.md#path) (e.g. `Channel.fromPath(...)`).

### splitText
Use `splitText` to split multi-line text into lines or chunks of lines.

**Example input file (`input.txt`):**
```
a
b
c
d
e
```

- **Input:** A channel emitting files or multi-line strings.
- **Output:** A channel emitting lines or chunks of lines.
- **Arguments:**
    - `by` (optional): Number of lines per chunk (default: 1).

```groovy
Channel.fromPath('input.txt')
    .splitText()
    .view() // Output: 'a', 'b', 'c', 'd', 'e'
Channel.fromPath('input.txt')
    .splitText(by: 2)
    .view() // Output: 'a\nb\n', 'c\nd\n', 'e\n'
```
**See also:** `splitCsv`, `splitFasta`.

### splitCsv
Use `splitCsv` to parse CSV-formatted text into rows (as lists or maps).

**Arguments:**
- `by`: When specified, group rows into chunks with the given size (default: none).
- `charset`: Parse the content with the specified charset, e.g. UTF-8. See the list of standard charsets for available options.
- `decompress`: When true, decompress the content using the GZIP format before processing it (default: false). Files with the `.gz` extension are decompressed automatically.
- `elem`: The index of the element to split when the source items are lists or tuples (default: first file object or first element).
- `header`: When true, the first line is used as the columns names (default: false). Can also be a list of columns names.
- `limit`: Limits the number of records to retrieve for each source item (default: no limit).
- `quote`: The character used to quote values (default: '' or "").
- `sep`: The character used to separate values (default: `,`).
- `skip`: Number of lines to ignore from the beginning when parsing the CSV text (default: 0).
- `strip`: When true, remove leading and trailing blanks from values (default: false).

**Example input file (`input.csv`):**
```
x,y
1,2
3,4
```

- **Input:** A channel emitting CSV strings or files.
- **Output:** A channel emitting lists (rows) or maps (if header is specified).
- **Arguments:**
    - `header` (optional): If true, parses the first row as column names and emits maps.
    - `skip` (optional): Number of lines to skip at the start.

```groovy
Channel.fromPath('input.csv')
    .splitCsv(header: true)
    .view() // Output: [x:1, y:2], [x:3, y:4]

Channel.fromPath('input.csv')
    .splitCsv(skip: 1)
    .view() // Output: [1, 2], [3, 4]
```

**Example (TSV file):**
Suppose you have a TSV file (`input.tsv`):
```
x	y
1	2
3	4
```
You can parse it using the `sep` argument:
```groovy
Channel.fromPath('input.tsv')
    .splitCsv(header: true, sep: '\t')
    .view() // Output: [x:1, y:2], [x:3, y:4]
```

**See also:** `splitText`.

### splitFasta
Use `splitFasta` to parse FASTA files into sequence records or chunks.

**Example input file (`sample.fa`):**
```
>seq1
ATGCATGC
>seq2
GGCCTTAA
```

- **Input:** A channel emitting FASTA files or strings.
- **Output:** A channel emitting sequence records (as maps) or text chunks.
- **Arguments:**
    - `record` (optional): If true or a map, emits parsed records with specified fields.

```groovy
Channel.fromPath('sample.fa')
    .splitFasta(record: [id: true, seqString: true])
    .view { it.id + ': ' + it.seqString }
```
**See also:** `splitFastq`.

### splitFastq
Use `splitFastq` to parse FASTQ files into sequence records or chunks.

**Example input file (`sample.fastq`):**
```
@seq1
ATGCATGC
+
IIIIIIII
@seq2
GGCCTTAA
+
JJJJJJJJ
```

- **Input:** A channel emitting FASTQ files or strings.
- **Output:** A channel emitting sequence records (as maps) or text chunks.
- **Arguments:**
    - `record` (optional): If true, emits parsed records.

```groovy
Channel.fromPath('sample.fastq')
    .splitFastq(record: true)
    .view { it.readHeader + ': ' + it.readString }
```
**See also:** `splitFasta`.

### splitJson
Use `splitJson` to parse JSON arrays or objects into individual records.

**Example input file (`input.json`):**
```
[1, 2, 3]
```
or
```
{"a": 1, "b": 2}
```

- **Input:** A channel emitting JSON strings or files.
- **Output:** A channel emitting elements of arrays or key-value pairs of objects.
- **Arguments:** None.

```groovy
Channel.fromPath('input.json')
    .splitJson()
    .view() // Output: 1, 2, 3
Channel.fromPath('input.json')
    .splitJson()
    .view() // Output: [key:a, value:1], [key:b, value:2]
```
**See also:** `splitCsv`.

---

## File and Data Persistence Operators

These operators handle the collection and writing of data to files. Use them to persist results or intermediate data for later use or external analysis.

### collectFile
Use `collectFile` to save all channel items to a file for reporting, archiving, or downstream tools.

- **Input:** A channel emitting any type of item.
- **Output:** A file (or files) containing all items.
- **Arguments:**
    - `name` (optional): Output file name. Required unless a grouping closure is given.
    - `newLine` (optional): Append a newline after each item (default: `false`).
    - `keepHeader` (optional): Keep the header line from the first file when concatenating files that share a header (default: `false`).
    - `sort` (optional): Sort items before collecting (default: `false`).
    - `storeDir` (optional): Directory to publish the collected file(s) to.

```groovy
Channel.of('foo', 'bar')
    .collectFile(name: 'output.txt', newLine: true)
// Writes 'foo\nbar\n' to output.txt

// Group into multiple files with a closure: [filename, content]
Channel.of('apple', 'banana', 'avocado')
    .collectFile { item -> ["${item[0]}.txt", item + '\n'] }
    .view { file -> "Wrote ${file.name}" }
// Writes 'apple\navocado\n' to a.txt, 'banana\n' to b.txt
```

**See also:** [Directives.md `publishDir`](Directives.md#publishdir)/[`storeDir`](Directives.md#storedir). `collectFile` writes arbitrary channel items to a file from within the workflow; `publishDir`/`storeDir` instead copy a process's declared [`path` outputs](Outputs.md#path) after the task runs.

---

## Mathematical and Statistical Operators

These operators perform mathematical computations on channel items. Use them to summarize, analyze, or reduce data streams.

### count
Use `count` to determine the size of a dataset or the number of results produced by a process.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting a single integer representing the number of items received.
- **Arguments:**
    - `filter` (optional): Only count items matching this value, regex, type, or predicate closure. If omitted, counts all items.

```groovy
Channel.of(1, 2, 3)
    .count()
    .view() // Output: 3
Channel.of(4, 1, 7, 1, 1)
    .count(1)
    .view() // Output: 3
```

### sum
Use `sum` to calculate totals, such as the sum of measurements or scores.

- **Input:** A channel emitting numeric items.
- **Output:** A channel emitting a single value representing the sum of all items.
- **Arguments:** None.

```groovy
Channel.of(1, 2, 3)
    .sum()
    .view() // Output: 6
```

### min
Use `min` to find the smallest value in a dataset, such as the lowest score or measurement.

- **Input:** A channel emitting comparable items (e.g., numbers).
- **Output:** A channel emitting the minimum value among all items.
- **Arguments:** None.

```groovy
Channel.of(5, 2, 8)
    .min()
    .view() // Output: 2
```

### max
Use `max` to find the largest value in a dataset, such as the highest score or measurement.

- **Input:** A channel emitting comparable items (e.g., numbers).
- **Output:** A channel emitting the maximum value among all items.
- **Arguments:** None.

```groovy
Channel.of(5, 2, 8)
    .max()
    .view() // Output: 8
```

---

## Testing and Debugging Operators

These operators assist in testing and debugging workflows.

### view
Use `view` to print items to the console for inspection.
- **Arguments:**
    - `closure` (optional): Custom formatting or logic for each item.

```groovy
Channel.of('foo', 'bar')
    .view() // Output: foo, bar
```

### subscribe
Use `subscribe` to perform actions on emitted items.
- **Arguments:**
    - `closure` (required): Action to perform for each item.

```groovy
Channel.of(1, 2, 3)
    .subscribe { println "Item: $it" }
// Output: Item: 1\nItem: 2\nItem: 3
```

---

## Channel Assignment and Routing Operators

These operators assign a channel to a variable or route items from one channel into multiple downstream channels.

### set
Use `set` to assign a channel to a named variable, typically at the end of a chain of operators for readability.

- **Input:** Any channel.
- **Output:** None — assigns the channel to the given variable name.
- **Arguments:**
    - `identifier` (required): Variable name to assign the channel to.

```groovy
Channel.of(10, 20, 30)
    .map { it * 2 }
    .set { doubled }

doubled.view() // Output: 20, 40, 60
```
**See also:** Plain assignment (e.g. `doubled = Channel.of(...).map { ... }`). `set` is a stylistic alternative that reads naturally at the end of a long operator chain.

### branch
Use `branch` to route each item from a channel into exactly one of several output channels, such as splitting samples by type.

- **Input:** A channel emitting any type of item.
- **Output:** Multiple channels, one per label, accessed as properties of the object passed to `set`.
- **Arguments:**
    - `closure` (required): A closure defining `label: condition` pairs. The first matching condition wins; use `true` as a final catch-all.

```groovy
Channel.of(1, 2, 3, 40, 50)
    .branch { v ->
        small: v < 10
        large: true
    }
    .set { result }

result.small.view() // Output: 1, 2, 3
result.large.view() // Output: 40, 50
```
**See also:** `multiMap`. `branch` sends each item to exactly one output channel, while `multiMap` sends every item to every output channel, each with its own transformation.

### multiMap
Use `multiMap` to split one channel into several parallel channels, each with its own transformation, such as separating a sample record into ID and file channels.

- **Input:** A channel emitting any type of item.
- **Output:** Multiple channels, one per label, accessed as properties of the object passed to `set`.
- **Arguments:**
    - `closure` (required): A closure defining `label: expression` pairs, one per output channel.

```groovy
Channel.of(1, 2, 3)
    .multiMap { v ->
        original: v
        squared: v * v
    }
    .set { result }

result.original.view() // Output: 1, 2, 3
result.squared.view()  // Output: 1, 4, 9
```
**Example (difference from `branch`):**
```groovy
// branch: each item goes to exactly one output channel
Channel.of(1, 20)
    .branch { v -> small: v < 10; large: true }
    .set { r1 }
// r1.small: 1  |  r1.large: 20

// multiMap: every item goes to every output channel, transformed differently
Channel.of(1, 20)
    .multiMap { v -> asIs: v; doubled: v * 2 }
    .set { r2 }
// r2.asIs: 1, 20  |  r2.doubled: 2, 40
```
