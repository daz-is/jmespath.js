# jmespath.js

[![Build Status](https://travis-ci.org/daz-is/jmespath.js.png?branch=publish)](https://travis-ci.org/daz-is/jmespath.js)

## Install

    npm install --save @daz.is/jmespath

## FORK

NB: This is a fork of the original JavaScript implementation 
of JMES Path. The original is now believed to be unmaintained,
so this fork adds several useful features:

- define custom functions
- cache parsed expressions 
- access the root element using $

## About

jmespath.js is a javascript implementation of JMESPath,
which is a query language for JSON.  It will take a JSON
document and transform it into another JSON document
through a JMESPath expression.

Using jmespath.js is really easy.  There's a single function
you use, `jmespath.search`:

```js
var jmespath = require('jmespath');
jmespath.search({foo: {bar: {baz: [0, 1, 2, 3, 4]}}}, "foo.bar.baz[2]")

// output = 2
```

In the example we gave the ``search`` function input data of
`{foo: {bar: {baz: [0, 1, 2, 3, 4]}}}` as well as the JMESPath
expression `foo.bar.baz[2]`, and the `search` function evaluated
the expression against the input data to produce the result ``2``.

The JMESPath language can do a lot more than select an element
from a list.  Here are a few more examples:

```js
jmespath.search({foo: {bar: {baz: [0, 1, 2, 3, 4]}}}, "foo.bar")

// { baz: [ 0, 1, 2, 3, 4 ] }

jmespath.search({"foo": [{"first": "a", "last": "b"},
                           {"first": "c", "last": "d"}]},
                  "foo[*].first")

// [ 'a', 'c' ]

jmespath.search({"foo": [{"age": 20}, {"age": 25},
                           {"age": 30}, {"age": 35},
                           {"age": 40}]},
                  "foo[?age > `30`]")

// [ { age: 35 }, { age: 40 } ]
```

## Adding custom functions

Custom functions can be added to the JMESPath runtime by using the
`decorate()` function:

```js
function customFunc(resolvedArgs) {
  return resolvedArgs[0] + 99;
}
var extraFunctions = {
  custom: {_func: customFunc, _signature: [{types: [jmespath.types.TYPE_NUMBER]}]},
};
jmespath.decorate(extraFunctions);
```

The value returned by the decorate function is a curried function
(takes arguments one at a time) that takes the search expression 
first and then the data to search against as the second parameter:

```js
var value = jmespath.decorate(extraFunctions)('custom(`1`)')({})
// value = 100
```

Because the return value from `decorate()` is a curried function
the result of compiling the expression can be cached and run 
multiple times against different data:

```js
var expr = jmespath.decorate({})('a');
// expr is now a cached compiled version of the search expression
var value = expr({ a: 1 });
assert.strictEqual(value, 1);
value = expr({ a: 2 });
assert.strictEqual(value, 2);
```

## Key-only MultiSelect Hashes

I have changed the grammar slightly to allow multi-select hashes to 
be specified using only the keys.

```js
var data = { one: 'first',  two: 'second' };
var result = jmespath.search(data, '{ one }');
strictDeepEqual(result, { one: 'first' });
```

## Optional arguments to functions

The latest version (v1.3) adds the ability for functions 
(including custom functions added via `decorate`) to have 
optional arguments. To activate this, set `optional: true`
in the function signature. 

## JMESPath+

For more examples of custom functions, and intregrating Lodash, 
see [JMESPath+](https://github.com/daz-is/jmespath-plus).

## More Resources

The example above only show a small amount of what
a JMESPath expression can do.  If you want to take a
tour of the language, the *best* place to go is the
[JMESPath Tutorial](http://jmespath.org/tutorial.html).

One of the best things about JMESPath is that it is
implemented in many different programming languages including
python, ruby, php, lua, etc.  To see a complete list of libraries,
check out the [JMESPath libraries page](http://jmespath.org/libraries.html).

And finally, the full JMESPath specification can be found
on the [JMESPath site](http://jmespath.org/specification.html).
