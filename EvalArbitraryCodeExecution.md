# Eval and Function Constructor allow Execution of Unrewritten Javascript #

## Effect ##
Execute arbitrary code with access to the global environment, and the local members of the stack frame in which it is called.


## Background ##
`eval` is described at ES3 15.1.2.1, and parses its argument as a Program, and executes it in its caller's environment.

The `Function` constructor allows creation of a function given a string body.  It will execute in the global scope as described at ES3 section 15.3.2.1

EcmaScript 262 specifically requires that the global eval method be assignable, so it can be replaced.  The Function constructor is available via the 'constructor' property of any function object.

EcmaScript 5 attenuates `eval` somewhat: it has an `eval` function that may be aliased, but calls to this "indirect eval" always resolve references in the global scope. So, if `Function` refers to the built-in Function constructor and `evil` is an alias to `eval`, `evil('(' + x + ')') ≡ (new Function('return ' + x))()` for all strings x that are well formed javascript expressions.


## Assumptions ##
`window.eval` and/or the `Function` constructor are accessible and callable.  The function constructor is available if any function is available and a function's `constructor` property is readable.


## Versions ##
All interpreters that obey the referenced sections of EcmaScript.


## Example ##
```
eval('alert("your cookie is " + document.cookie)');

(new Function('alert("your cookie is " + document.cookie)'))();
```