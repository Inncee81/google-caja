# Global scope reachable via this from functions not invoked as methods #

## Effect ##
Untrusted functions invoked naively by trusted code can steal access to the global scope.


## Background ##
EcmaScript 262 section 10.1.5 defines a Global Object which is the source of all references not satisfied by local function variables, properties of an object in an enclosing with block, or exception variables for an enclosing catch block.

When javascript is executing in multiple frames, it may be hard to determine whether an object is a global scope.

EcmaScript 5 changes the reflective `myFunction.{call,apply`} methods to not coerce null to the global scope in strict mode.  Technically, `this` is undefined when null or undefined is passed to `call` or `apply` but when code references `this`, the global object is substituted for `null` or `undefined`. In strict mode, the interpreter will not perform this coercion.


## Assumptions ##
An untrusted function that references `this` can be invoked not as a method (or in ES5, a property accessor), and not via call or apply without the global scope (or `null` or `undefined`) as the first input.

AND

Untrusted functions that reference `this` are not rewritten to abort execution if `this` is the global object.



## Versions ##
All



## Example ##
```
(function () {
   alert('your cookie is ' + this.document.cookie);
 })();

setTimeout(
    function () {
      alert('your cookie is ' + this.document.cookie);
    }, 0);
```