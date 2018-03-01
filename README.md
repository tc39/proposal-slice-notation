# Slice notation

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

This notation can be used for slice operations on primitives like
String and any object that provides indexed access using `[[Get]]`
like Array and TypedArray.

The length used for these operations is the `length` property of the
object.

```js
const obj = { 0: 1, 1: 2, 2: 3, 3: 4, length: 4 };
obj[1:3];
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

## Examples

In the following text, 'length of the object' refers to the `length`
property of the object.

### Default values

The lower bound, upper bound and the step argument are all optional.

The default value for the lower bound is 0.

```
const arr = [1,  2, 3, 4];
arr[:3:1];
// → [1, 2, 3]
```

The default value for the upper bound is the length of the object.

```
const arr = [1,  2, 3, 4];
arr[1::1];
// → [2, 3, 4]
```

The default value for the step argument is 1.
```
const arr = [1,  2, 3, 4];

arr[1:];
// → [2, 3, 4]

arr[:3];
// → [1, 2, 3]

arr[1::2];
// → [2, 4]

arr[:3:2];
// → [1, 3]
```

Omitting all lower bound and upper bound value, produces a new copy of the object.
```
const arr = [1, 2, 3, 4];

arr[:];
// → [1, 2, 3, 4]

arr[::];
// → [1, 2, 3, 4]
```

### Negative indices

If the lower bound is negative, then the start index is computed as
follows:

```
start = max(lowerBound + len, 0)
```

where `len` is the length of the object.

```
const arr = [1, 2, 3, 4];

arr[-2:];
// → [3, 4]
```

In the above example, `start = max((-2 + 4), 0) = max(2, 0) = 2`.

```
const arr = [1, 2, 3, 4];

arr[-10:];
// → [1, 2, 3, 4]
```

In the above example, `start = max((-10 + 4), 0) = max(-6, 0) = 0`.

Similarly, if the upper bound is negative, the end index is computed
as follows:

```
end = max(upperBound + len, 0)
```

```
const arr = [1, 2, 3, 4];

arr[:-2];
// → [1, 2]

arr[:-10];
// → []
```

These semantics exactly match the behavior of existing slice
operations.

If the step argument is negative, then the object is traversed in
reverse.

```
const arr = [1, 2, 3, 4];

arr[::-1];
// → [4, 3, 2, 1]
```

### Out of bounds indices

Both the lower and upper bounds are capped at the length of the object.

```
const arr = [1, 2, 3, 4];

arr[100:];
// → []

arr[:100];
// → [1, 2, 3, 4]
```

These semantics exactly match the behavior of existing slice
operations.

## Prior art

### Python

This proposal is highly inspired by the Python. Unsurprisingly, the
Python syntax for slice notation is strikingly similar:

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

### CoffeeScript

CoffeeScript provides a Range operator that is inclusive with respect
to the upper bound.

```coffeescript
arr = [1, 2, 3, 4];
arr[1..3];
// → [2, 3, 4]
```

CoffeeScript also provides another form the Range operator that does
not include the upper bound.

```coffeescript
arr = [1, 2, 3, 4];
arr[1...3];
// → [2, 3]
```

### Ruby

Ruby seems to have two different ways to get a slice:

* Using a Range:

```ruby
arr = [1, 2, 3, 4];
arr[1..3];
// → [2, 3, 4]
```

This is similar to CoffeeScript. The `1..3` produces a Range object
which defines the set of indices to be sliced out.

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

### Why pick the Python syntax over the Ruby/CoffeeScript syntax?

The Python syntax allows us to provide an optional step argument.

Also, the Python syntax which excludes the upper bound index is
similar to the existing slice methods in JavaScript. Admittedly, this
is a weak argument as we could use exlusive Range operator (`...`)
from CoffeeScript.

### Why does this not use the iterator protocol?

The iterator protocol because isn't restricted to index lookup, which
makes it incompatible with this slice notation which works only on
indices.

For example, Map and Sets have iterators but we shouldn't be able to
slice them as they don't have indices.

### What about splice?

CoffeeScript allows similar syntax to be used on the LHS of an
AssignmentExpression leading to splice operation.

```coffeescript
numbers = [1, 2, 3, 4]
numbers[2..4] = [7, 8]
// → [1, 2, 7, 8]
```

This doesn't work with Strings as they are immutable, but could be
made to work with any object using a `[Set]]` operation.

This feature is currently omitted to limit the scope of the proposal,
but can be incorporated in a follow on proposal.

### Doesn't the bind operator have similar syntax?

Unfortunately, yes. The ambiguity arrises from this production:

```
const x = [2];
const arr = [1, 2, 3, 4];
arr[::x[0]];
```

Is the above creating a new array with values `[1, 3]` or is it
creating a bound method?
