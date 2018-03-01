# slice notation

This repository contains a proposal for adding slice notation syntax
to JavaScript. This is currently at stage 0 of the [TC39
process](https://tc39.github.io/process-document/).

## Motivation

The slice notation provides an ergonomic alternative to the various
slice methods present on Array.prototype, String.prototype, etc.

```js
const arr = [1, 2, 3, 4];

arr[1:3];
// → [2, 3]

arr.slice(1, 3);
// → [2, 3]
```

The slice notation can be used for slice operations on primitives like
String and any object that provides indexed access using `[[Get]]`
like Array and TypedArray.

```js
const obj = { 0: 1, 1: 2, 2: 3, 3: 4 }
obj[1:3]
// → [2, 3]
```

The slice notation extends the slice operations by accepting an
optional step argument. The step argument is set to 1 if not
provided.

```js
const arr = [1, 2, 3, 4];
arr[1:4:2]
// → [2, 4]
```

## Prior art

### Python

This proposal is highly inspired by the Python syntax and
unsurprisingly, Python's syntax is the same:

```
slicing      ::=  primary "[" slice_list "]"
slice_list   ::=  slice_item ("," slice_item)* [","]
slice_item   ::=  expression | proper_slice
proper_slice ::=  [lower_bound] ":" [upper_bound] [ ":" [stride] ]
lower_bound  ::=  expression
upper_bound  ::=  expression
stride       ::=  expression
```

Examples:

```python
arr = [1, 2, 3, 4];

arr[1:3];
// → [2, 3]

arr[1:4:2]
// → [2, 4]
```

### Ruby

Ruby seems to have two different ways to get a slice:

* Using a Range object:

```ruby
arr = [1, 2, 3, 4];
arr[1..3];
// → [2, 3, 4]
```

The `1..3` produces a Range object which defines the set of indices to
be sliced out.

The rest/spread operator in ECMAScript is very similar to the range
operator, which could potentially cause confusion to developers as
they could easily forget a `.` leading to a syntax error.

* Using the comma operator:
```ruby
arr = [1, 2, 3, 4];
arr[1, 3];
// → [2, 3, 4]
```

The difference here is that the second argument is actually the length
of the new slice, not the upper bound index.

This is currently valid ECMAScript syntax which makes this a non
starter.

```js
const s = 'foobar'
s[1, 3]
// → 'b'
```

## FAQ

### Why does this not use the iterator protocol?

It doesn't make sense to use the iterator protocol because the
iterator protocol isn't restricted to index lookup. For example, Map
and Sets have iterators but we shouldn't be able to slice them as they
don't have indices.
