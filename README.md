proposal-logical-assignment
===========================

A Stage 1 proposal to combine Logical Operators and Assignment Expressions:

```js
// "Or Or Equals" (or, the Mallet operator :wink:)
a ||= b;
a || (a = b);

// "And And Equals"
a &&= b;
a && (a = b);

// Eventually....
// "QQ Equals"
a ??= b;
a ?? (a = b);
```

Motivation
----------

Convenience operators, inspired by [Ruby's](https://docs.ruby-lang.org/en/2.5.0/syntax/assignment_rdoc.html#label-Abbreviated+Assignment). We already have a dozen [mathematical assignment operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Assignment_Operators), but we don't have ones for the often used logical operators.

```js
function example(a = b) {
  // Default assignment only works for `undefined`.
  // But I want any falsey to deafult
  if (!a) {
    a = b;
  }
}

function numeric(a = b) {
  // Maybe I want to keep numeric 0, but default nullish
  if (a == null) {
    a = b;
  }
}
```

If statements work, but terseness would be nice.

```js
function example(a = b) {
  // Ok, but it triggers setter.
  a = a || b;

  // No setter, but sometimes flagged as a lint error!
  a || (a = b);
}
```

With this, we get terseness and we don't have to suffer from setter calls.

Semantics
---------

The logical assignment operators function a bit differently than their mathematical assignment friends. While math assignment operators _always_ trigger a set operation, logical assignment embraces their short-circuiting semantics to avoid it when possible.

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

Related
-------

- [Ruby's logical operators](https://docs.ruby-lang.org/en/2.5.0/syntax/assignment_rdoc.html#label-Abbreviated+Assignment)
  - [Explainer on no-set semantics](http://www.rubyinside.com/what-rubys-double-pipe-or-equals-really-does-5488.html)
- [CoffeeScript](http://coffeescript.org/#try:a%20%3D%201%0Ab%20%3D%202%0A%0A%0A%23%20%22Or%20Or%20Equals%22%20(or%2C%20the%20Mallet%20operator%20%3Awink%3A)%0Aa%20%7C%7C%3D%20b%3B%0Aa%20%7C%7C%20(a%20%3D%20b)%3B%0A%0A%23%20%22And%20And%20Equals%22%0Aa%20%26%26%3D%20b%3B%0Aa%20%26%26%20(a%20%3D%20b)%3B%0A%0A%23%20Eventually....%0A%23%20%22QQ%20Equals%22%0A%23a%20%3F%3F%3D%20b%3B%0A%23a%20%3F%3F%20(a%20%3D%20b)%3B%0A)
- My very first [Babel PR](https://github.com/babel/babel/pull/516) (back when it was still [6to5](https://github.com/babel/babel/tree/ecd85f53b4764ada862537aa767699814f1f1fe2)). 😄
