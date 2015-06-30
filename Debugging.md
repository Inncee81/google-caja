Caja performs a server-side translation of your HTML, CSS and
JavaScript.  It also includes some client-side runtime support.

# Compile-time errors #

Caja compile time errors include information about what errors
occured, the url of the offending content as well as a line number and
offset.

For example, suppose you try to cajole the following piece of code:
```
<script>
var top;
with(top) { location = "http://www.evil.com" }
</script>
```

You get the following messages:
```
LINT http://www.example.com/index.html:3+48: Semicolon inserted:
ERROR http://www.example.com/index.html:3+1 - 50: "with" blocks are not allowed:
```

The first message is of type `LINT`.  These do not stop the
compilation from proceeding and producing output but inform you of
choices the cajoler has made about how to recover from a possible
problem in the input.  In this case, the cajoler has inserted a
semicolon on Line 3, Column 48.  According to the JavaScript
specification, it is not an error to leave out semicolons at the end
of the line.  However, under some circumstances you may be surprised
about where JavaScript thinks a semicolon should go.

The second message is of type `ERROR`.  These do not stop the
compilation from proceeding to discover more context around the error
but the cajoler will not be able to produce usable output.  In this
case, the cajoler has come across a `with` block on Line 3, starting
at Column 1 and continuing to Column 50.  `with`statements modify
dynamic scope and are often misunderstood and a common source of
errors.  They are disallowed in Caja.  To correct this error, you must
rewrite this block of code to avoid the `with` statement.

Here are some types of messages you may see:
  * `LOG` info about the internal progress of the Cajoler
  * `LINT`/`WARNING` a possible problem in an input source file
  * `CRITICAL_WARNING` a serious problem but cajoler can still produce usable output
  * `ERROR`/`FATAL_ERROR` cajoler cannot produce usable output

# Run-time errors #

Some types of errors in your program won't get discovered until
runtime.  On browsers that support a console, runtime errors are
logged to the console configued by the container.

For example, suppose you try to cajole and run the following piece of code:
```
<script>
top.location = "http://www.evil.com";
</script>
```

You will see no compile errors, however, your browser page will
probably not redirect as you expect.  In your JavaScript console, you
get the following message:

```
http://www.example.com/index.html:2: Result of expression 'top' [undefined] is not an object.
```

This error means on Line 2, the code was expecting the expression
`top` to be an `Object` but it turned out to be undefined.  This
happens because by default cajoled code is not granted access to _any_
browser object other than the DOM.  As a result variables such as
`window`, `top` and `navigator` which you may expect to be defined
when you are running JavaScript in a browser will all be undefined.

Here's another trivial piece of code with an error:
```
<script>
  function apply(func, val) {
    return func(val);
  }
  apply(1, alert);
</script>
```

As you would expect in your JavaScript console, you get the following message:
```
http://www.example.com/index.html:2: expected function instead of number:1
```

Caja checks that if you try to call an object as a function, that it
is in fact a function and that the gadget is allowed to call it before
it is called.

Some containers and browsers may not provide you with adequate
information about the error messages your application generates.  If
you are able to narrow the problematic section of your code, you can
may be able to get more detailed error messages by testing snippets on
our testbeds([1](http://www.cajadores.com/demos/testbed/)
[2](http://caja.appspot.com)).

# Advanced Debugging #

With larger and more sophisticated applications, it is helpful to
delve deeper into the cajoled code to understand why a cajoled
application is misbehaving.

Here are some tips that you may find helpful:

  * [Squarefree Shell](http://www.squarefree.com/shell/shell.html) provides a very useful [bookmarklet](https://www.squarefree.com/bookmarklets/webdevel.html) which you can load in the same frame as a cajoled application.  In the shell, you can access what a cajoled gadget perceives its global scope to be as follows:
```
  var gadgetGlobal = ___.getNewModuleHandler().getImports().outers;
```

  * You can test if some property of an object is readable, settable or callable by the gadget with `___.canRead(obj, property)`, `___.canSet(obj, property)` and `___.canCall(obj, property)` respectively.

  * Other [internal properties](InternalProperties.md) can also help identify why a cajoled program behaves differently from an uncajoled one

  * By default, many containers will minify cajoled output for efficient delivery.  This is not ideal for debugging. If possible, request debug mode from the container or use the testbed for debugging.

  * If you are using Firefox, install the Firebug extension.  Caja dumps useful logs and stack traces in the console.

  * If the container has made the `alert` function or the `console.log` functions available, they provide very useful feedback.