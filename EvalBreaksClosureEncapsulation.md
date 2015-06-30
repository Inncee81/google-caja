# Eval Breaks Closure Encapsulation #

## Effect ##
Private state in a closure can be read and modified on SpiderMonkey.


## Background ##
Each EcmaScript function instance has a scope chain that is used to resolve free variables.  This scope chain includes variables local to enclosing functions.

For example, in the code below
```
function counter(i) {
  return function () { return ++i; }
}
```
counter defines a local variable `i`.  It contains an anonymous function that references `i` and that function can continue to read and set `i` even after `counter` has returned.  In this case, the inner function's scope chain is `[counter(i), global scope]`, which means that variables not defined in the inner function will first be resolved by checking `counter(i)`'s scope, and if not defined there, will look in the global scope.

Many programmers assume that once a function has returned, its local variables are only readable/settable by function instances defined in that closure, and so closures are used to protect data.

`eval` normally uses the current scope chain to resolve free variables in the script being evaluated.

SpiderMonkey breaks this assumption.  The `eval` function takes a second optional argument.  If supplied, that function's scope chain is used to resolve free variables in the script being evaluated instead of the current scope chain.


## Assumptions ##
Untrusted code can reference a function whose scope chain contains sensitive information AND that code can call `eval` with 2 parameters.


## Versions ##
Firefox 2 and 3, but this feature is slated to be removed in Firefox [3.7](https://bugzilla.mozilla.org/show_bug.cgi?id=531675) as it is widely recognized as a bug.


## Example ##// Assumed to generate unique serial numbers.
function counter(i) {
  return function () { return ++i; };
}

var myCounter = counter(0);
alert(myCounter());  // => 1
eval('i = 4', myCounter);
alert(myCounter());  // => 5 on Firefox 2, 2 on other browsers```