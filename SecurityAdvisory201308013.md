## Background ##

This advisory concerns implementations of the Array methods in the V8 JavaScript virtual machine (used in Google Chrome):

(a) Implementations of some Array methods are not spec compliant, allowing the caller to mutate a frozen array.

(b) Implementations of some Array methods may under certain circumstances receive an implicit "this" equal to the global window object. We thank Tung Tran <tunghack@gmail.com> for reporting this problem.

## Impact ##

(a) An attacker could, for example, mutate the Array.prototype object in the frame in which they are deployed. This can cause unintended behavior of internal Caja code which may lead to vulnerabilities.

(b) An attacker could gain access to the global window object, then take advantage of this to cause disallowed global page operations.

Either of these could lead to an arbitrary code execution breach.

## Advice ##

Upgrade to a version of Caja at or after [r5551](https://code.google.com/p/google-caja/source/detail?r=5551).


## More Information ##

(a) These issues were originally reported against the V8 codebase at:

> https://code.google.com/p/v8/issues/detail?id=2469
> https://code.google.com/p/v8/issues/detail?id=2615
> https://code.google.com/p/v8/issues/detail?id=2800

and against the Caja codebase at:

> https://code.google.com/p/google-caja/issues/detail?id=1816

Discussion of the change is at:

> https://codereview.appspot.com/11312043/

(b) This issue was originally reported against the V8 codebase at:

> https://code.google.com/p/v8/issues/detail?id=2758

for which we would like to thank Tung Tran <tunghack@gmail.com>. It was reported against the Caja codebase at:

> https://code.google.com/p/google-caja/issues/detail?id=1789

Discussion of the change is at:

> https://codereview.appspot.com/10711045/