# Slice notation

This repository contains a proposal for adding slice notation syntax
to JavaScript. This is currently at stage 1 of the [TC39
process](https://tc39.github.io/process-document/).

Champions:

- Sathya Gunasekaran (@gsathya)
- HE Shi-Jun (@hax)


## Introduction

The slice notation provides an ergonomic alternative to the various
slice methods present on Array.prototype, TypedArray.prototype, etc.

```js
const arr = ['a', 'b', 'c', 'd'];

arr[1:3];
// → ['b', 'c']

arr.slice(1, 3);
// → ['b', 'c']
```

This notation can be used for slice operations on primitives
like Array and TypedArray.


## Motivation

```js
const arr = ['a', 'b', 'c', 'd'];
arr.slice(3);
// → ['a', 'b', 'c'] or ['d'] ?
```

In the above example, it's not immediately clear if the newly created
array is a slice from the range `0` to `3` or from `3` to `len(arr)`.

```js
const arr = ['a', 'b', 'c', 'd'];
arr.slice(1, 3);
// → ['b', 'c'] or ['b', 'c', 'd'] ?
```

Adding a second argument is also ambiguous since it's not clear if the
second argument specifies an upper bound or the length of the new
slice.

Programming language like Ruby and C++ take the length of the new
slice as the second argument, but JavaScript's slice methods take the
upper bound as the second argument.

```js
const arr = ['a', 'b', 'c', 'd'];
arr[3:];
// → ['d']

arr[1:3];
// → ['b', 'c']
```

With the new slice syntax, it's immediately clear that the lower bound
is `3` and the upper bound is `len(arr)`. It makes the intent
explicit.

The syntax is also much shorter and more ergonomic than a function
call.

## Examples

In the following text, 'length of the object' refers to the `length`
property of the object.

### Default values

The lower bound and upper bound are optional.

The default value for the lower bound is 0.

```js
const arr = ['a', 'b', 'c', 'd'];

arr[:3];
// → ['a', 'b', 'c']
```

The default value for the upper bound is the length of the object.


```js
const arr = ['a', 'b', 'c', 'd'];
arr[1:];
// → ['b', 'c', 'd']
```

Omitting all lower bound and upper bound value, produces a new copy of the object.
```js
const arr = ['a', 'b', 'c', 'd'];

arr[:];
// → ['a', 'b', 'c', 'd']
```

### Negative indices

If the lower bound is negative, then the start index is computed as
follows:

```js
start = max(lowerBound + len, 0)
```

where `len` is the length of the object.

```js
const arr = ['a', 'b', 'c', 'd'];

arr[-2:];
// → ['c', 'd']
```

In the above example, `start = max((-2 + 4), 0) = max(2, 0) = 2`.

```js
const arr = ['a', 'b', 'c', 'd'];

arr[-10:];
// → ['a', 'b', 'c', 'd']
```

In the above example, `start = max((-10 + 4), 0) = max(-6, 0) = 0`.

Similarly, if the upper bound is negative, the end index is computed
as follows:

```js
end = max(upperBound + len, 0)
```

```js
const arr = ['a', 'b', 'c', 'd'];

arr[:-2];
// → ['a', 'b']

arr[:-10];
// → []
```

These semantics exactly match the behavior of existing slice
operations.

### Out of bounds indices

Both the lower and upper bounds are capped at the length of the object.

```js
const arr = ['a', 'b', 'c', 'd'];

arr[100:];
// → []

arr[:100];
// → ['a', 'b', 'c', 'd']
```

These semantics exactly match the behavior of existing slice
operations.

## Prior art

### Python

This proposal is highly inspired by Python. Unsurprisingly, the
Python syntax for slice notation is strikingly similar:

```python
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

CoffeeScript provides a Range operator that is _inclusive_ with respect
to the upper bound.

```coffeescript
arr = [1, 2, 3, 4];
arr[1..3];
// → [2, 3, 4]
```

CoffeeScript also provides another form the Range operator that is _exclusive_ with respect
to the upper bound.

```coffeescript
arr = [1, 2, 3, 4];
arr[1...3];
// → [2, 3]
```

### Go

Go offers [slices](https://gobyexample.com/slices):

```go
arr := []int{1,2,3,4};
arr[1:3]
// → [2, 3]
```

There is also ability to *not* provide lower or upper bound:

```go
arr := []int{1,2,3,4};
arr[1:]
// → [2, 3, 4]

arr := []int{1,2,3,4};
arr[:3]
// → [1, 2, 3]
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

The Python syntax which excludes the upper bound index is
similar to the existing slice methods in JavaScript.

We could use exclusive Range operator (`...`) from CoffeeScript, but
that doesn't quite work for all cases because it's ambiguous with the
spread syntax. Example code from
[getify](https://gist.github.com/getify/49ae9a1f2a6031d40f5deb5ea25faa62):

```js
Object.defineProperty(Number.prototype,Symbol.iterator,{
  *value({ start = 0, step = 1 } = {}) {
     var inc = this > 0 ? step : -step;
     for (let i = start; Math.abs(i) <= Math.abs(this); i += inc) {
        yield i;
     }
  },
  enumerable: false,
  writable: true,
  configurable: true
});

const range = [ ...8 ];
// → [0, 1, 2, 3, 4, 5, 6, 7, 8]
```

### Why does this not use the iterator protocol?

The iterator protocol isn't restricted to index lookup making it
incompatible with this slice notation which works only on
indices.

For example, Map and Sets have iterators but we shouldn't be able to
slice them as they don't have indices.

### What about splice?

CoffeeScript allows similar syntax to be used on the left hand side of
an `AssignmentExpression` leading to splice operation.

```coffeescript
numbers = [1, 2, 3, 4]
numbers[2..4] = [7, 8]
// → [1, 2, 7, 8]
```

This feature is currently omitted to limit the scope of the proposal,
but can be incorporated in a follow on proposal.

### Why doesn't this include a step argument like Python does?

The step argument makes the slice notation ambiguous with the bind operator.

```js
const x = [2];
const arr = [1, 2, 3, 4];
arr[::x[0]];
```

Is the above creating a new array with values `[1, 3]` or is it
creating a bound method?

### Should this create a `view` over the array, instead of a creating new array?

Go creates a `slice` over the underlying array, instead of allocating a new array.

```go
arr := []int{1,2,3,4};
v = arr[1:3];
// → [2, 3]
```

Here, v is just descriptor that holds a reference to the original
array `arr`. No new array allocation is performed. See [this blog
post](https://blog.golang.org/go-slices-usage-and-internals) for more
details.

This doesn't map to any existing construct in JavaScript and this would
be a step away from how methods work in JavaScript. To make this
syntax work well within the JavaScript model, such a `view` data
structure is not included in this proposal.

### Should slice notation work on strings?

The `String.prototype.slice` method doesn't work well with unicode
characters. [This blog
post](https://mathiasbynens.be/notes/javascript-unicode) by Mathias
Bynens, explains the problem.

Given that the existing method doesn't work well, this proposal
does not add `@@slice` to `String.prototype`.

### How about combining this with `+` for append?

```js
const arr = [1, 2, 3, 4] + [5, 6];
// → [1, 2, 3, 4, 5, 6]
```

This is not included in order to keep the proposal's scope maximally
minimal.

The [operator overloading
proposal](https://github.com/keithamus/ecmascript-operator-overloading-proposal)
may be a better fit for this.

### Can you create a Range object using this syntax?

The slice notation only provides an ergonomic syntax for performing a slice
operation. 

The current slice notation doesn't preclude creating a range primitive in the 
future.

A new Range primitive is being discussed here:
https://github.com/tc39/proposal-Number.range/issues/22

### Isn't it confusing that this isn't doing property lookup?

This is actually doing a property lookup using `[[Get]]` on the
underlying object. For example,

```js
const arr = [1, 2, 3, 4];

arr[1:3];
// → [2, 3]
```

This is doing a property lookup for the keys `1` and `2`.

But, shouldn't it do a lookup for the string `'1:3'`?

```js
const arr = [1, 2, 3, 4];

arr['1:3'];
// → undefined
```

No. The slice notation makes it analogous with how keyed lookup
works. The key is first evaluated to a value and then the lookup
happens using this value.

```js
const arr = [1, 2, 3, 4];
const x = 0;

arr[x] !== arr['x'];
// → true
```

The slice notation works similarly. The notation is first evaluated to
a range of values and then each of the values are looked up.

### There are already many modes where ':' mean different things. Isn't this confusing?

Depending on context `a:b`, can mean:

- `LabelledStatement` with `a` as the label
- Property a with value b in an object literal: `{a: b }`
- ConditionalExpression: `confused ? a : b`
- Potential type systems (like TypeScript and Flow) that might make it
  to JavaScript in the future.

Is it a lot of overhead to disambiguate between modes with context?
Major mainstream programming languages like Python have all these
modes and are being used as a primary tool for teaching programming.
