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
    - `closure` (optional): Custom aggregation logic. If omitted, collects items into a list.

```groovy
Channel.of(1, 2, 3)
    .collect()
    .view() // Output: [1, 2, 3]
```
**See also:** `toList`. Both aggregate all items into a list, but `collect` is more general and can be customized for other types of aggregation.
**Example (difference from `toList`):**
```groovy
// collect can be customized for other aggregations
Channel.of(1, 2, 3)
    .collect { acc, val -> acc + val * 2 }
    .view() // Output: [2, 4, 6]
// toList always collects all items into a list
Channel.of(1, 2, 3)
    .toList()
    .view() // Output: [1, 2, 3]
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

---

## Combining Operators

These operators combine data from multiple channels. Use them to merge, join, or relate data streams for complex workflows.

### mix
Use `mix` to merge multiple data sources into a single stream for unified processing.

- **Input:** Two or more channels emitting any type of item.
- **Output:** A single channel emitting all items from the input channels, in the order they become available.
- **Arguments:**
    - `channels` (required): Two or more channels to merge.

```groovy
Channel
    .mix(Channel.of(1, 2), Channel.of(3, 4))
    .view() // Output: 1, 2, 3, 4 (order may vary)
```
**See also:** `concat`. Both merge multiple channels, but `mix` emits items as soon as they are available from any input, while `concat` emits all items from one channel before moving to the next.
**Example (difference from `concat`):**
```groovy
// mix interleaves items as they arrive
Channel.mix(Channel.of(1, 2), Channel.of(10, 20))
    .view() // Output: 1, 10, 2, 20 (order may vary)
// concat preserves channel order
Channel.concat(Channel.of(1, 2), Channel.of(10, 20))
    .view() // Output: 1, 2, 10, 20
```

### concat
Use `concat` to chain channels together in a specific order, such as appending results or logs.

- **Input:** Two or more channels emitting any type of item.
- **Output:** A single channel emitting all items from the first channel, then all items from the next, and so on.
- **Arguments:**
    - `channels` (required): Two or more channels to concatenate.

```groovy
Channel
    .concat(Channel.of('a', 'b'), Channel.of('c', 'd'))
    .view() // Output: 'a', 'b', 'c', 'd'
```
**See also:** `mix`. Use `concat` when you need to preserve the order of channels, unlike `mix` which interleaves items as they arrive.
**Example (difference from `mix`):**
```groovy
// concat emits all items from the first channel, then the next
Channel.concat(Channel.of('a', 'b'), Channel.of('x', 'y'))
    .view() // Output: 'a', 'b', 'x', 'y'
// mix interleaves items as they become available
Channel.mix(Channel.of('a', 'b'), Channel.of('x', 'y'))
    .view() // Output: 'a', 'x', 'b', 'y' (order may vary)
```

### combine
Use `combine` to relate or synchronize data from two sources based on a common identifier.

- **Input:** Two channels emitting tuples or lists, with a key to match items.
- **Output:** A channel emitting pairs of items from both channels that share the same key.
- **Arguments:**
    - `other` (required): The other channel to combine with.
    - `by` (required): Index or closure to specify the key for matching.

```groovy
Channel.of([1, 'A'], [2, 'B'])
    .combine(Channel.of([1, 10], [2, 20]), by: 0)
    .view() // Output: [[1, 'A'], [1, 10]], [[2, 'B'], [2, 20]]
```
**See also:** `join`. Both relate items from two channels by key, but `combine` emits all possible pairs for matching keys, while `join` emits a single joined tuple per key.
**Example (difference from `join`):**
```groovy
// combine emits all pairs for matching keys
Channel.of([1, 'A'], [1, 'B'])
    .combine(Channel.of([1, 10], [1, 20]), by: 0)
    .view() // Output: [[1, 'A'], [1, 10]], [[1, 'A'], [1, 20]], [[1, 'B'], [1, 10]], [[1, 'B'], [1, 20]]
// join emits a single joined tuple per key
Channel.of([1, 'A'], [1, 'B'])
    .join(Channel.of([1, 10], [1, 20]), by: 0)
    .view() // Output: [1, 'A', 10], [1, 'B', 20]
```

### join
Use `join` to enrich or correlate data from two sources, such as joining metadata with results.

- **Input:** Two channels emitting tuples or lists, with a key to match items.
- **Output:** A channel emitting joined tuples for each matching key, similar to a database join.
- **Arguments:**
    - `other` (required): The other channel to join with.
    - `by` (required): Index or closure to specify the key for matching.

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
Use `cross` to generate all combinations of two datasets, such as parameter sweeps or matrix builds.

- **Input:** Two channels emitting any type of item.
- **Output:** A channel emitting the Cartesian product (all possible pairs) of items from both channels.
- **Arguments:**
    - `other` (required): The other channel to cross with.

```groovy
Channel.of([1, 'A'], [2, 'B'])
    .cross(Channel.of([1, 10], [2, 20]))
    .view() // Output: [[1, 'A'], [1, 10]], [[2, 'B'], [2, 20]]
```
**See also:** `combine`. `cross` produces the full Cartesian product of two channels, while `combine` only pairs items with matching keys.
**Example (difference from `combine`):**
```groovy
// cross produces all possible pairs (Cartesian product)
Channel.of(1, 2)
    .cross(Channel.of('a', 'b'))
    .view() // Output: [1, 'a'], [1, 'b'], [2, 'a'], [2, 'b']
// combine only pairs items with matching keys
Channel.of([1, 'A'], [2, 'B'])
    .combine(Channel.of([1, 10], [2, 20]), by: 0)
    .view() // Output: [[1, 'A'], [1, 10]], [[2, 'B'], [2, 20]]
```

---

## Grouping and Batching Operators

These operators group or batch items from a channel. Use them to organize data into manageable sets for downstream processing or aggregation.

### buffer
Use `buffer` to batch items for grouped processing, such as running jobs in chunks or controlling resource usage.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting lists of items, each list containing up to the specified batch size or meeting a condition.
- **Arguments:**
    - `size` (optional): Maximum number of items per batch.
    - `timeout` (optional): Maximum time to wait before emitting a batch.

```groovy
Channel.of(1, 2, 3, 4, 5)
    .buffer(size: 2)
    .view() // Output: [1, 2], [3, 4], [5]
```

### collate
Use `collate` to create sliding windows or overlapping groups for analysis or rolling computations.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting lists of a fixed size, optionally with overlap between groups.
- **Arguments:**
    - `size` (required): Number of items per group.
    - `step` (optional): Number of items to advance between groups (default: same as `size`).

```groovy
Channel.of(1, 2, 3, 4)
    .collate(3, 1)
    .view() // Output: [1, 2, 3], [2, 3, 4]
```

### groupTuple
Use `groupTuple` to group related data by a common identifier, such as grouping results by sample or category.

- **Input:** A channel emitting tuples or lists with a key field.
- **Output:** A channel emitting pairs of [key, grouped items] where items share the same key.
- **Arguments:**
    - `by` (required): Index or closure to specify the key for grouping.

```groovy
Channel.of([1, 'A'], [1, 'B'], [2, 'C'])
    .groupTuple(by: 0)
    .view() // Output: [1, [[1, 'A'], [1, 'B']]], [2, [[2, 'C']]]
```

---

## Splitting and Parsing Operators

These operators split or parse structured data from files or strings into records or chunks for downstream processing.

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
- **Output:** A file containing all items, typically one per line or as specified.
- **Arguments:**
    - `name` (required): Output file name.
    - `mode` (optional): File write mode (e.g., 'overwrite', 'append').

```groovy
Channel.of('foo', 'bar')
    .collectFile(name: 'output.txt')
// Writes 'foo' and 'bar' to output.txt
```

### save
Use `save` to persist results or logs to a specific file location for reproducibility or sharing.

- **Input:** A channel emitting any type of item.
- **Output:** A file at the specified path containing all items.
- **Arguments:**
    - `path` (required): Output file path.

```groovy
Channel.of(1, 2, 3)
    .save('numbers.txt')
// Writes 1, 2, 3 to numbers.txt
```

---

## Mathematical and Statistical Operators

These operators perform mathematical computations on channel items. Use them to summarize, analyze, or reduce data streams.

### count
Use `count` to determine the size of a dataset or the number of results produced by a process.

- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting a single integer representing the number of items received.
- **Arguments:** None.

```groovy
Channel.of(1, 2, 3)
    .count()
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

### mean
Use `mean` to calculate the average of a set of values, such as mean coverage or expression.

- **Input:** A channel emitting numeric items.
- **Output:** A channel emitting the average (mean) value of all items.
- **Arguments:** None.

```groovy
Channel.of(2, 4, 6)
    .mean()
    .view() // Output: 4.0
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
