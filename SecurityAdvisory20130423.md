# Caja Security Advisory 2013/04/23 #

Caja prior to version [r5369](https://code.google.com/p/google-caja/source/detail?r=5369) may be vulnerable to takeover of the host page by guest code if run in ES5/3 mode, and may allow uncontrolled communication between guests if run in ES5 mode. If you depend on Caja for security, either upgrade to version [r5369](https://code.google.com/p/google-caja/source/detail?r=5369) or later, or backport the security patches.

## ES5 mode: `[[ThrowTypeError]]` was not defended ##

Several browsers have bugs where the function known to the ES5 spec as `[[ThrowTypeError]]` is not immutable, as it is specified to be. SES did not verify that this was correctly implemented, resulting in it being possible to add properties to the function or its .prototype.

This only affects ES5 mode. It does not make the host page vulnerable, but allows independent guests to communicate with each other.

  * Bug: https://code.google.com/p/google-caja/issues/detail?id=1661
  * Patch: https://codereview.appspot.com/8093043/
  * Committed: https://code.google.com/p/google-caja/source/detail?r=5368

## ES5/3 mode: Property virtualization was unsound ##

ES5/3 non-conformantly presented the property descriptors of certain built-in methods as accessor properties, which caused Domado's actions to defend its objects exposed to the guest to be insufficient, leaving some mutable prototype methods.

This only affects ES5/3 mode. It may allow takeover of the host page by causing Domado to misbehave.

  * Bug: https://code.google.com/p/google-caja/issues/detail?id=1668
  * Patch: https://codereview.appspot.com/7807043/
  * Committed: https://code.google.com/p/google-caja/source/detail?r=5369