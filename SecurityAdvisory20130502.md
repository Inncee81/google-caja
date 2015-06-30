# Caja Security Advisory 2013/05/02 #

Caja prior to version [r5396](https://code.google.com/p/google-caja/source/detail?r=5396) may allow uncontrolled communication between guests, and possibly other vulnerabilities, if run in ES5/3 (non-ES5) mode. If you depend on confinement of untrusted code, either upgrade to version [r5396](https://code.google.com/p/google-caja/source/detail?r=5396) or later, or backport the security patches.

## Failing to freeze numeric properties ##

In ES5/3 mode, an object which has had number-named properties created in certain shortcut ways, which includes array literals, Function.prototype, String.prototype, RegExp.prototype, and RegExp, would not get those properties correctly made non-modifiable if the object is frozen.

This allows independent guests to communicate with each other (by modifying the shared taming-frame prototypes accessible via DOM wrappers), and may allow other attacks via unfrozen array literals (we have not analyzed whether this case occurs in Caja itself).

  * Bug: https://code.google.com/p/google-caja/issues/detail?id=1683
  * Patch: https://codereview.appspot.com/8531043/
  * Committed: https://code.google.com/p/google-caja/source/detail?r=5394

## Functions' .prototypes were undefended ##

ES5/3 failed to delete or freeze the .prototype of internally-generated functions which should have been made transitively immutable. Since those prototypes were never intended to be used, this is only a communication channel (global mutability).

  * Bug: https://code.google.com/p/google-caja/issues/detail?id=1695
  * Patch: https://codereview.appspot.com/8629044/
  * Committed: https://code.google.com/p/google-caja/source/detail?r=5396