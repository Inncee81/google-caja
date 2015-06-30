# arguments array allows modification of parameters #

## Effect ##
Any static checks that constrain access or modification of a functions parameters can be circumvented via the arguments array.



## Background ##
The arguments array, described in EcmaScript 2.6.2 section 10.1.8, allows access
to the called function, and the arguments it was called with.

This is often used by varargs functions.

The arguments object is an Array-like object, not an actual Array, and its storage is not separate from the local variables themselves, so assignment to its members may change actual parameters.



## Assumptions ##
The arguments array is accessible and mutable.  Security relies on statically enforced immutability of function's parameters.



## Versions ##
All



## Example ##
```
(function (a) {
  arguments[0] = 1;
  alert('a=' + a);
})(0);
```

```
function f(x) { g(); alert(x); }
function g() { f.arguments[0] = 1; }
f(0);  // alerts 1
```