## Effect ##

Properties that should not be readable may be read (and possibly written). It may be possible to combine this with other attack vectors such as EvalArbitraryCodeExecution to run arbitrary code.


## Assumptions ##

  * In a JavaScript subset, properties named by stringified numbers are implicitly readable (and possibly writable).
  * The subset implementation assumes that a postincrement or postdecrement expression always evaluates to a number.
  * The JavaScript interpreter incorrectly implements postincrement or postdecrement, so that it may return its argument without first coercing it to a number.


## Background ##

The postincrement and postdecrement operators are specified in ECMA-262 sections 13.3.1 and 13.3.2 to always return a number. Caja, ADsafe, and Jacaranda unconditionally allow reading of properties named by "stringified numbers", that is, strings that can be the result of ToString(ToNumber(x)) for some x.

However, in Internet Explorer, if `x` is a local variable or parameter of the current function, then `x++` or `x--` incorrectly evaluates to `x` without coercing to a number.

For example, `(function() { var x = 'foo'; return x++; })();` will return `'foo'`, when it should return `NaN`.

This means that a property access of the form `a[x++]` or `a[x--]` might be allowed under the assumption that it is accessing a stringified number property, but actually access a different property that should not have been readable (or writable when the access is a LeftHandSideExpression).

For discussion of the general problem of which this is a special case, see MisOptimizations.


## Versions ##
  * JScript in Internet Explorer 6 and 7 (version 8 has not been tested)


## Example ##

The following example uses this to get the `constructor` property of a function object (i.e. what the global `Function` normally refers to), and then evals arbitrary code:

```
  (function() {
    var c = 'constructor';
    var F = (function(){})[c++];  // Function constructor
    F('alert("toast")')();
  })();
```

Jacaranda 0.3 was vulnerable to this attack (only on IE). See also [this google-caja-discuss thread](http://groups.google.com/group/google-caja-discuss/browse_thread/thread/d85a423be80f17e9/70784a05e0f3723b#70784a05e0f3723b).