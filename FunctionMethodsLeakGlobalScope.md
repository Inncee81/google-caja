# Function.call or Function.apply can leak window with certain this-values. #

## Effect ##
Expose the global scope.


## Background ##
Function.call and Function.apply methods invoke the function with a specific value of this, and apply allows an array-like object to be substituted for the argument list.

They are described in EcmaScript 262 section 15.3.4.{3,4}:
  1. .3.4.3 Function.prototype.apply (thisArg, argArray)
> The apply method takes two arguments, thisArg and argArray, and
> performs a function call using the [[Call](Call.md)] property of the object.
> If the object does not have a [[Call](Call.md)] property, a TypeError exception
> is thrown.

> If thisArg is null or undefined, the called function is passed the
> global object as the this value.

> Otherwise, the called function is passed ToObject(thisArg) as the this
> value.
> ...

## Assumptions ##
Code in untrusted functions can access `this`, and can access either the call or apply methods.

`This` cannot be determined to be safe at runtime.



## Versions ##
All.


## Example ##
```
(function () { alert(this === window); }).call(null);

(function () { alert(this === window); }).call(undefined);

alert(window === ([]).sort.call());

alert(window === ([]).reverse.call());

// Firefox2 only.  [https://bugzilla.mozilla.org/show_bug.cgi?id=406337]
var o = { valueOf: function () { return null } };
(function () { alert(this === window); }).call(o);
```