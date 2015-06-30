# Function constructor accessible via the 'constructor' property #

## Effect ##
Execution of unsanitized code in the global scope


## Background ##
As discussed at http://code.google.com/p/google-javascript-type-system/issues/detail?id=2 the Function constructor allows for execution of unsanitized code.

EcmaScript 262 section 15.3.4.1 Function.prototype.constructor
> The initial value of Function.prototype.constructor is the built-in
> Function constructor.



## Assumptions ##
The Function constructor has not been replaced with something innocuous.


## Versions ##
All


## Example ##
```
((function () {}).constructor)(
    'alert("document.cookie = " + document.cookie)')()
```