# Uncaught Exception Handling #

## Background ##
In normal HTML pages, exceptions that bubble out of a `<script>` element are trapped and invoke `window.onerror` but do not abort execution of subsequent `<script>` elements.

According to [HTML5](http://www.w3.org/TR/html5/web-browsers.html#runtime-script-errors)

> Whenever a runtime script error occurs in one of the scripts associated with the
> document, the value of the `onerror` event handler DOM attribute of the
> `Window` object must be processed, as follows:

> If the value is a function
> > The function referenced by the `onerror` attribute must be invoked with
> > three arguments, before notifying the user of the error.
> > The three arguments passed to the function are all `DOMString`s; the first
> > must give the message that the UA is considering reporting, the second must give
> > the URI to the resource in which the error occurred, and the third must give the
> > line number in that resource on which the error occurred.
> > If the function returns `false`, then the error should not be reported to the
> > user. Otherwise, if the function returns another value (or does not return at
> > all), the error should be reported to the user.
> > Any exceptions thrown or errors caused by this function must be reported to the
> > user immediately after the error that the function was called for, without
> > calling the function again.


> If the value is `null`
> > The error should not reported to the user.


> If the value is anything else
> > The error should be reported to the user.


> The initial value of `onerror` must be `undefined`

A gadget is a chunk of HTML containing zero or more `<script>` tags interleaved with HTML.  The intervening chunk of HTML and CSS are compiled to javascript that has the same effect.



## Goals ##

Emulate existing `onerror` mechanisms to allow gadgets to encapsulate code and define their own error recovery mechanisms, but also allow containers hooks to detect and log gadget failures.  The last is especially important in debug mode.



## Implementation ##

The `HtmlCompiler` which extracts `script` tags from HTML wraps the script body in an exception handler like
```
  try {
    ...
  } catch (ex___) {
    ___.getNewModuleHandler().handleUncaughtException(
        ex___, onerror, 'file.html', lineNumber);
  }
```
If `onerror` is defined locally as by `var onerror = null;` or `function onerror(message, sourceUri, lineNumber) {...} `, `onerror` will be that version.  Otherwise, `onerror` will end up being loaded from `IMPORTS___` before any of the `try` blocks can be entered.  This latter case allows a container to define a default implementation.

If the container wants to maintain any invariants it can provide a new module handler that overrides `handleUncaughtException`.

The default ES5/3 `handleUncaughtException` implementation does
  1. Cast the exception to a string if it's an object.  This is a work around to avoid trapping uncatchable exceptions such as stack overflows.
  1. Extract the error message from `ex` ensuring it is a string.
  1. `var result = (typeof onerror) === 'function' ? onerror(message, sourceUri, lineNumber) : onerror !== null;`
  1. If `result !== false` invoke `caja.log` with the error message, sourceUri, and lineNumber.

## Open Questions ##
Does providing the source file and line number lead to non-determinism?