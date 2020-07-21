# proposal-logical-assignment

A proposal to combine Logical Operators and Assignment Expressions:

```js
// "Or Or Equals" (or, the Mallet operator :wink:)
a ||= b;
a || (a = b);

// "And And Equals"
a &&= b;
a && (a = b);

// "QQ Equals"
a ??= b;
a ?? (a = b);
```

## Status

Current [Stage](https://tc39.es/process-document/): 4

## Champions

- Justin Ridgewell ([@jridgewell](https://github.com/jridgewell/))
- Hemanth HM ([@hemanth](https://github.com/hemanth/))

## Motivation

Convenience operators, inspired by
[Ruby's](https://docs.ruby-lang.org/en/2.5.0/syntax/assignment_rdoc.html#label-Abbreviated+Assignment).
We already have a dozen [mathematical assignment
operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Assignment_Operators),
but we don't have ones for the often used logical operators.

```js
function example(a) {
  // Default `a` to "foo"
  if (!a) {
    a = 'foo';
  }
}
```

If statements work, but terseness would be nice. Especially when dealing
with deep property assignment:

```js
function example(opts) {
  // Ok, but could trigger setter.
  opts.foo = opts.foo ?? 'bar'

  // No setter, but 'feels wrong' to write.
  opts.baz ?? (opts.baz = 'qux');
}

example({ foo: 'foo' })
```

With this proposal, we get terseness and we don't have to suffer from
setter calls:

```js
function example(opts) {
  // Setters are not needlessly called.
  opts.foo ??= 'bar'

  // No repetition of `opts.baz`.
  opts.baz ??= 'qux';
}

example({ foo: 'foo' })
```

## Semantics

The logical assignment operators function a bit differently than their
mathematical assignment friends. While math assignment operators
_always_ trigger a set operation, logical assignment embraces their
short-circuiting semantics to avoid it when possible.

```js
let x = 0;
const obj = {
  get x() {
    return x;
  },
  
  set x(value) {
    console.log('setter called');
    x = value;
  }
};

// This always logs "setter called"
obj.x += 1;
assert.equal(obj.x, 1);

// Logical operators do not call setters unnecessarily
// This will not log.
obj.x ||= 2;
assert.equal(obj.x, 1);

// But setters are called if the operator does not short circuit
// "setter called"
obj.x &&= 3;
assert.equal(obj.x, 3);
```

In most cases, the fact that the set operation is short-circuited has no
observable impact beyond performance. But when it has side effects, it
is often desirable to avoid it when appropriate. In the following
example, if the `.innerHTML` setter was triggered uselessly, it could
result in the loss of state (such as focus) that is not serialized in
HTML:

```js
document.getElementById('previewZone').innerHTML ||= '<i>Nothing to preview</i>';
```

See discussion of short-circuit semantics in [#3](https://github.com/tc39/proposal-logical-assignment/issues/3). It also highlights differences already present in mathematical assignment operators in code like `obj.deep[key++] += 1` vs `obj.deep[key] = obj.deep[key++] + 1`.

## Related

- [Ruby's logical operators](https://docs.ruby-lang.org/en/2.5.0/syntax/assignment_rdoc.html#label-Abbreviated+Assignment)
  - [Explainer on no-set semantics](http://www.rubyinside.com/what-rubys-double-pipe-or-equals-really-does-5488.html)
- [CoffeeScript](http://coffeescript.org/#try:a%20%3D%201%0Ab%20%3D%202%0A%0A%0A%23%20%22Or%20Or%20Equals%22%20(or%2C%20the%20Mallet%20operator%20%3Awink%3A)%0Aa%20%7C%7C%3D%20b%3B%0Aa%20%7C%7C%20(a%20%3D%20b)%3B%0A%0A%23%20%22And%20And%20Equals%22%0Aa%20%26%26%3D%20b%3B%0Aa%20%26%26%20(a%20%3D%20b)%3B%0A%0A%23%20Eventually....%0A%23%20%22QQ%20Equals%22%0A%23a%20%3F%3F%3D%20b%3B%0A%23a%20%3F%3F%20(a%20%3D%20b)%3B%0A)
- [C#'s null coalescing assignment](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-8.0/null-coalescing-assignment#detailed-design)
- My very first [Babel PR](https://github.com/babel/babel/pull/516) (back when it was still [6to5](https://github.com/babel/babel/tree/ecd85f53b4764ada862537aa767699814f1f1fe2)). 😄
