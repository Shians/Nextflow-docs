## 🔄 Transformation Operators

These operators apply functions to each item in a channel, producing a new channel with transformed data. Use them to modify, aggregate, or restructure data as it flows through your pipeline.

### map
- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting the result of applying a function to each input item.
- **Use case:** Transform each item in a channel, such as converting formats, scaling values, or extracting fields.
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
- **Input:** A channel emitting collections or items that can be expanded.
- **Output:** A channel emitting each element from the collections, flattened into a single stream.
- **Use case:** Expand nested lists or split items into multiple outputs for further processing.
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
- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting a single list containing all input items.
- **Use case:** Gather all items into a single collection for summary operations or final output.
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
- **Input:** A channel emitting nested collections.
- **Output:** A channel emitting all elements from the nested collections as a flat stream.
- **Use case:** Remove one level of nesting from channel items, useful after aggregation or grouping.
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
- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting a single list of all items, similar to `collect`.
- **Use case:** Collect all items for batch processing or output as a single list.
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
- **Input:** A channel emitting comparable items.
- **Output:** A channel emitting a single sorted list of all items.
- **Use case:** Collect and sort all items, such as for reporting or ordered output.
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

## 🧹 Filtering Operators

These operators filter items in a channel based on specified conditions. Use them to select, limit, or exclude data as it flows through your pipeline.

### filter
- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting only items that match the given predicate (closure, regex, or type).
- **Use case:** Select items that meet specific criteria, such as filtering by value, pattern, or type.
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
- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting only the first N items.
- **Use case:** Limit the number of items processed downstream, such as for sampling or testing.
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
- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting all items except the first N.
- **Use case:** Ignore a fixed number of initial items, such as skipping headers or warm-up data.
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

## 🔗 Combining Operators

These operators combine data from multiple channels. Use them to merge, join, or relate data streams for complex workflows.

### mix
- **Input:** Two or more channels emitting any type of item.
- **Output:** A single channel emitting all items from the input channels, in the order they become available.
- **Use case:** Merge multiple data sources into a single stream for unified processing.
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
- **Input:** Two or more channels emitting any type of item.
- **Output:** A single channel emitting all items from the first channel, then all items from the next, and so on.
- **Use case:** Chain channels together in a specific order, such as appending results or logs.
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
- **Input:** Two channels emitting tuples or lists, with a key to match items.
- **Output:** A channel emitting pairs of items from both channels that share the same key.
- **Use case:** Relate or synchronize data from two sources based on a common identifier.
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
- **Input:** Two channels emitting tuples or lists, with a key to match items.
- **Output:** A channel emitting joined tuples for each matching key, similar to a database join.
- **Use case:** Enrich or correlate data from two sources, such as joining metadata with results.
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
- **Input:** Two channels emitting any type of item.
- **Output:** A channel emitting the Cartesian product (all possible pairs) of items from both channels.
- **Use case:** Generate all combinations of two datasets, such as parameter sweeps or matrix builds.
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

## 📦 Grouping and Batching Operators

These operators group or batch items from a channel. Use them to organize data into manageable sets for downstream processing or aggregation.

### buffer
- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting lists of items, each list containing up to the specified batch size or meeting a condition.
- **Use case:** Batch items for grouped processing, such as running jobs in chunks or controlling resource usage.
```groovy
Channel.of(1, 2, 3, 4, 5)
  .buffer(size: 2)
  .view() // Output: [1, 2], [3, 4], [5]
```

### collate
- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting lists of a fixed size, optionally with overlap between groups.
- **Use case:** Create sliding windows or overlapping groups for analysis or rolling computations.
```groovy
Channel.of(1, 2, 3, 4)
  .collate(3, 1)
  .view() // Output: [1, 2, 3], [2, 3, 4]
```

### groupTuple
- **Input:** A channel emitting tuples or lists with a key field.
- **Output:** A channel emitting pairs of [key, grouped items] where items share the same key.
- **Use case:** Group related data by a common identifier, such as grouping results by sample or category.
```groovy
Channel.of([1, 'A'], [1, 'B'], [2, 'C'])
  .groupTuple(by: 0)
  .view() // Output: [1, [[1, 'A'], [1, 'B']]], [2, [[2, 'C']]]
```

---

## 📁 File and Data Persistence Operators

These operators handle the collection and writing of data to files. Use them to persist results or intermediate data for later use or external analysis.

### collectFile
- **Input:** A channel emitting any type of item.
- **Output:** A file containing all items, typically one per line or as specified.
- **Use case:** Save all channel items to a file for reporting, archiving, or downstream tools.
```groovy
Channel.of('foo', 'bar')
  .collectFile(name: 'output.txt')
// Writes 'foo' and 'bar' to output.txt
```

### save
- **Input:** A channel emitting any type of item.
- **Output:** A file at the specified path containing all items.
- **Use case:** Persist results or logs to a specific file location for reproducibility or sharing.
```groovy
Channel.of(1, 2, 3)
  .save('numbers.txt')
// Writes 1, 2, 3 to numbers.txt
```

---

## 🧮 Mathematical and Statistical Operators

These operators perform mathematical computations on channel items. Use them to summarize, analyze, or reduce data streams.

### count
- **Input:** A channel emitting any type of item.
- **Output:** A channel emitting a single integer representing the number of items received.
- **Use case:** Determine the size of a dataset or the number of results produced by a process.
```groovy
Channel.of(1, 2, 3)
  .count()
  .view() // Output: 3
```

### sum
- **Input:** A channel emitting numeric items.
- **Output:** A channel emitting a single value representing the sum of all items.
- **Use case:** Calculate totals, such as the sum of measurements or scores.
```groovy
Channel.of(1, 2, 3)
  .sum()
  .view() // Output: 6
```

### min
- **Input:** A channel emitting comparable items (e.g., numbers).
- **Output:** A channel emitting the minimum value among all items.
- **Use case:** Find the smallest value in a dataset, such as the lowest score or measurement.
```groovy
Channel.of(5, 2, 8)
  .min()
  .view() // Output: 2
```

### max
- **Input:** A channel emitting comparable items (e.g., numbers).
- **Output:** A channel emitting the maximum value among all items.
- **Use case:** Find the largest value in a dataset, such as the highest score or measurement.
```groovy
Channel.of(5, 2, 8)
  .max()
  .view() // Output: 8
```

### mean
- **Input:** A channel emitting numeric items.
- **Output:** A channel emitting the average (mean) value of all items.
- **Use case:** Calculate the average of a set of values, such as mean coverage or expression.
```groovy
Channel.of(2, 4, 6)
  .mean()
  .view() // Output: 4.0
```

---

## 🧪 Testing and Debugging Operators

These operators assist in testing and debugging workflows.

### view
Prints items to the console for inspection.
```groovy
Channel.of('foo', 'bar')
  .view() // Output: foo, bar
```

### subscribe
Subscribes to a channel to perform actions on emitted items.
```groovy
Channel.of(1, 2, 3)
  .subscribe { println "Item: $it" }
// Output: Item: 1\nItem: 2\nItem: 3
```
