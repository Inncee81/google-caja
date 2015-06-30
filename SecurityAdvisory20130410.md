# Caja Security Advisory 2013/04/10 #

Caja prior to version [r5341](https://code.google.com/p/google-caja/source/detail?r=5341) is vulnerable to takeover of the host page by guest code if run in ES5 mode. If you depend on Caja for security, either upgrade to version [r5341](https://code.google.com/p/google-caja/source/detail?r=5341) or later, or backport the security patch.

## Details ##

Security in ES5 mode depends on none of the host page's objects being accessible by guest code (as each of them conveys access to the host's Object.prototype which may be mutated), but Caja neglected to prevent exceptions thrown by the browser's DOM API from reaching the guest code.

This only affects ES5 mode.  It doesn't affect ES5/3 mode.

  * Bug: https://code.google.com/p/google-caja/issues/detail?id=1607
  * Patch: https://codereview.appspot.com/7182049/
  * Committed: https://code.google.com/p/google-caja/source/detail?r=5341