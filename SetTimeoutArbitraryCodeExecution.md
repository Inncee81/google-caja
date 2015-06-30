# `setTimeout` and `setInterval` allow execution of unrewritten code in the global scope #

## Effect ##
Malicious code can execute arbitrary code in the global context.


## Background ##
`setTimeout` and `setInterval` are not part of EcmaScript 262 but are used as delayed execution mechanisms.  Each browser window keeps a `pqueue<time_t>` of operations to perform on the event thread.

Both are defined in all modern browser's window scope as functions that take a delta-time and either a function or string which is pushed onto the window's event thread queue.  If the argument is a string it is parsed as a Program and executed in the context of the window.



## Assumptions ##
`setTimeout` and/or `setInterval` are available in their original form.  The form that takes a function will allow access to the global object via `this` but does not by itself allow arbitrary code execution.


## Versions ##
All


## Example ##
```
setTimeout("alert('your cookie is ' + document.cookie)", 0);
setTimeout(
    function () { alert('your cookie is ' + this.document.cookie); }, 0);
```