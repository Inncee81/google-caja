# Caja Security Advisory 2013/01/22 #

## Description ##

This is a combined advisory for three unrelated vulnerabilities.

### Exposed taming constructors ###

In some cases, the value of the `.constructor` property of a virtual DOM object was a function (a "taming constructor") which, in ES5/3 mode, could be applied to another object in order to defeat its encapsulation.

### measureText not tamed ###

The result of `<canvas>.getContext('2d').measureText(...)` was not tamed and therefore a host object, which, in ES5 mode, results in complete vulnerability.

### String code execution in window.onload ###

If a string were assigned to the virtual `window.onload` property, then it would be executed as JavaScript in the host page.

## Impact ##

Each of these vulnerabilities can be used to cause arbitrary effects on the host page.

## Advice ##

Upgrade to a version of Caja at or after [r5223](https://code.google.com/p/google-caja/source/detail?r=5223), or apply the patches from the mentioned reviews or commits below, as soon as possible.

## More Information ##

Discussion of the changes for the exposure of taming constructors is available at https://codereview.appspot.com/6843096/. It was committed as [r5220](https://code.google.com/p/google-caja/source/detail?r=5220).

The measureText issue was reported publicly, not as a security bug, as [Issue 1598](http://code.google.com/p/google-caja/issues/detail?id=1598). Discussion of the changes is available at https://codereview.appspot.com/6902057/. It was committed as [r5222](https://code.google.com/p/google-caja/source/detail?r=5222).

The window.onload issue was recorded as [Issue 1612](https://code.google.com/p/google-caja/issues/detail?id=1612). Discussion of the changes is available at https://codereview.appspot.com/7001045/. It was committed as [r5223](https://code.google.com/p/google-caja/source/detail?r=5223).