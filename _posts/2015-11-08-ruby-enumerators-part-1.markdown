---
layout: post
title:  "Mastering Ruby Enumerators"
date:   2015-11-08 11:00:00
---

Some useful Enumerator methods that every Ruby developer should know.

### none?

The method returns true if the block never returns true for all elements.

```ruby
[ 1, 2, 3, 4, ].none? { |i| i > 5 }
=> true
```

### any?

The method returns true if the block ever returns a value other than false or nil.

```ruby
%w[ant bear cat].any? { |word| word.length >= 3 }
=> true
```

### all?
The method returns true if the block never returns false or nil.

```
%w[ant bear cat].all? { |word| word.length >= 3 }
=> true
```

### partition

Divide your collection based on the block you pass.

```
(1..6).partition { |v| v.even? }
=> [[2, 4, 6], [1, 3, 5]]
```

### reduce / inject
Combines all elements of enum by applying a binary operation, specified by a block or a symbol that names a method or operator.
```
(5..10).reduce(:+)
=> 45
(5..10).inject(:+)
=> 45
```
```
(5..10).reduce { |sum, n| sum + n }
=> 45
(5..10).inject { |sum, n| sum + n }
=> 45
```

### collect

Returns a new array with the results of running block once for every element in enum.

```
[:symbol, :symbol2].collect { |s| s.to_s }
=> ["symbol", "symbol2"]
```
```
[:symbol, :symbol2].collect(&:to_s)
=> ["symbol", "symbol2"]
```

### select

Select elements in your collection that match a criteria.
```
(1..6).select { |a| a.even? }
=> [2, 4, 6]
```
```
(1..6).select(&:even?)
=> [2, 4, 6]
```

### bsearch

For sorted arrays, use `bsearch` instead of `find`.

```
sorted_items.bsearch { |n| n == 999999864 }
```

Performance comparison:

```
sorted = (0..10_000_000).map { Random.rand(1_000_000_000)  }.sort

Benchmark.measure { sorted_items.bsearch { |n| n == 999999864 } }.total
=> 0.0

Benchmark.measure { sorted_items.find { |n| n == 999999864 } }.total
=> 0.6999999999999993
```


### map
Map a function over a collection.
```
[ [1,2,3], [5, 7, 9], [11, 12] ].map do |items|
  items.find_all(&:even?)
end
=> [[2], [], [12]]
```


### flat_map
Map a function over a collection and flatten the result by one-level.
Equivalent to use `map` and then call `flatten` in the result.

```
[ [1,2,3], [5, 7, 9], [11, 12] ].flat_map do |items|
  items.find_all(&:even?)
end
=> [2, 12]
```
