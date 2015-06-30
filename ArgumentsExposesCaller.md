# arguments Array and function object expose caller #

## Effect ##
An untrusted function can steal a reference to a trusted caller function which it can later invoke.


## Background ##
http://developer.mozilla.org/en/docs/Core_JavaScript_1.5_Reference:Objects:Function:call describes both caller and the caller property of Functions.

The members of function objects are described in EcnaScript 262 Section 15.3.5.
In addition to these, many implementations expose properties `caller`, and `__caller__`.

Many implementations also expose caller as part of the `arguments` Array.



## Assumptions ##
Untrusted code can call a trusted function to escalate privileges if it holds a reference to it.

Untrusted code can access the currently executing function.  This can normally be done at runtime via arguments.callee.

Calls of untrusted functions are not passed through a launderer function which recurses to itself as in
```
  function launderer(fn, varargs) {
    if (arguments.caller !== arguments.callee) {
      arguments.callee.apply(this, arguments);
    } else {
      arguments[0].apply(
          this, [].splice.call(arguments, 1, arguments.length));
    }
  }
```
This laundering scheme does not work if `__caller__` is exposed.  `__caller__` was removed from Firefox.



## Versions ##
At least on FF and IE.  Only old versions of Firefox expose `__caller__`



## Example ##
```
function untrusted() {
  alert('got function ' + untrusted.caller + ' : '
        + arguments.callee.caller.arguments[0]);
}

(function trusted() { untrusted(); })(4);
```