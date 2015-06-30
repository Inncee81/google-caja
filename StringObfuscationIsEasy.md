# String Obfuscation is Easy #

## Effect ##
Approaches that rely on statically detecting code for other languages in string literals are easy to defeat.


## Background ##
There are literally an infinite number of ways to encode a string in any turing complete language that can compose a string from smaller elements.  It is trivial to construct a turing machine that outputs a fixed bit string and then exits, and that turing machine can be emulated by a universal turing machine, and a universal turing machine can emulate a universal turing machine that outputs a string, ad infinitum.

Given that there are an infinite number of ways of producing the same string, any approach that tries to statically prove that certain strings cannot be produced by a computation by analyzing only string constants is doomed to fail.

Javascript's `[]` operator allows a string constant to be converted to an object reference -- `window['eval']` is the same as `window.eval`.


## Assumptions ##
The language is Turing complete and it allows composing of strings from smaller elements and strings can be converted to unsafe references.


## Example ##
An early draft of ADSafe JS allowed the square bracket operator to receive strings.

It disallowed
```
(function () {
  (new ((function () {})['constructor'])('alert("hello")'))();
})();
```
because `constructor` was disallowed but allowed
```
(function () {
  var s = 'cons';
  s += 'tructor';
  (new ((function () {})[s])('alert("hello")'))();
})();
```

ADSafe solved this problem by disallowing the square bracket operator, but providing an alternative that only allowed numeric indices, breaking the "strings can be converted to unsafe references" assumption above.