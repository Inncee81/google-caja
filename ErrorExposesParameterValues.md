# Error Exposes Parameter Values #

## Effect ##
A function can steal secrets passed to its caller but not directly passed to it.


## Background ##
Any value can be thrown, but the `Error` class is most frequently used, and is the base class for interpreter Errors.

According to section 15.11 of ES 262:
> S15.11 **Error** **Objects**

> Instances of Error objects are thrown as exceptions when runtime errors occur. The Error objects may also serve as base objects for user-defined exception classes.

When `Error`s are converted to a string, the result is implementation-dependent.  Some interpreters (e.g. SpiderMonkey and Opera's implementation) add members to `Error` that include stack trace information.  That stack may include not just names, and source files and line numbers, but also parameter values.


## Assumptions ##
(Stack traces are readable by untrusted code, either by directly reading properties such as `anError.{stack,message,stacktrace} `, OR by implicit conversion if the result of `anError.toString()` or `anError.valueOf()` contains a stack trace)

AND

(Stack traces include source filenames containing secrets, as in `foo.js?password=hello`, OR
stack traces include privileged stack frames with actual parameter values).


## Versions ##
Works in Firefox (`.stack`) and Opera (`.message` and `.stacktrace`). Both add stack traces that include parameter values.

On Opera, the result of `anError.toString()` contains the contents of `anError.message`. This can be suppressed using:
```
// not production-quality code
Error.prototype.toString = function() { return ''+this.name+': '+this.message.split('\n')[0]; };
```

Note that Opera only adds the `message` and `stacktrace` properties when an Error object is thrown, whereas Firefox adds the `stack` property when an Error object is constructed.

Addition of a stack trace to the `message` property on Opera does not always happen consistently; it appears to happen for system-thrown errors (such as ReferenceError), but not for all errors thrown by user code.

Opera's `stacktrace` property is controlled by a user preference that is disabled by default. Its value when disabled is the string "n/a; see 'opera:config#UserPrefs|Exceptions Have Stacktrace'". Setting that preference does enable it. There is no obvious preference to disable the `message` property. (Tested on Opera 9.64.)


## Example (Firefox) ##
```
// President Skroob knows his luggage combination.
function skroob(luggageCombination) {
  // He orders Dark Helmet around but does not give
  // Dark Helmet his luggage combination.
  // Only an idiot would do that.
  darkHelmet();
}

// Dark Helmet can deduce Skroob's luggage combination 
// without ever receiving it as input and without it
// being stored in an object property.
function darkHelmet() {
  var combo = Number((new Error).stack.match(/skroob\((\d+)\)/)[1]);
  alert('Only an idiot would use that combination!: ' + combo);
}

skroob(1234);
```