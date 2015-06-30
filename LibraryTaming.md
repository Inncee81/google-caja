See NiceNeighbor for background.

# Introduction #

Ideally any library you use will be included prior to Cajoling, and so
will be cajoled and thus not need to be tamed.

However, libraries sometimes provide facilities which cannot be
provided using the safe subset of Javascript that Caja provides. In
that case, the library will have to be run outside the Cajoled code,
within your container, and an interface to the library provided to
Cajoled code. This process is known as "taming".

# Taming #

## Exposing The Library ##

First of all, the Cajoled code needs some means to access the
library. In your container you should have some code looking something
like this

```
  var imports = ___.copy(___.sharedImports);
  imports.outers = imports;
  imports.$v = ___.asFunc(valijaMaker)(imports);
  ___.getNewModuleHandler().setImports(imports);
```

Anything you attach to `imports` becomes a "global" variable in the
scope of the Cajoled gadget (remembering that Caja rewrites globals to
become properties of the `imports` object).

So, let's say that your library is exposed to your container through
some object called `mylibrary`. The simplest thing to do is to expose
that object like so

```
imports.mylibrary = mylibrary;
```

Now the Cajoled code can see the library, but it does not necessarily
have access to its properties yet. Numeric properties and "`length`"
are always accessible. Properties that end in double underscore
("`__`") are "unmentionable", and so are never accessible. If
`mylibrary` is a record or array (see CajitaValues for definitions),
then all its mentionable properties are accessible. Otherwise,
mentionable properties must be individually granted in order to be
accessible from cajoled code.

## Taming simple properties ##

Suppose your library exposes a property called `someProperty`. You
can grant a cajoled program read access to `someProperty` like this:

```
___.grantRead(mylibrary, 'someProperty');
```

This would allow the Cajoled code to read `mylibrary.someProperty` but
not to write or execute it. Similarly you can grant write access with
`___.grantSet(object, 'property')`. Assuming that `object[property]`
is a simple-function (a function that does not mention `this`) you
grant call and read access together with
`___.grantFunc(object,'property')`. Otherwise, you must use one of the
`___.grant*Method(object, 'property')` calls explained at
NiceNeighbor.

Constructors are are marked as such with
`___.markCtor(class, parent,'classname')`. This enables cajoled code
to call them with `new`. Stand-alone simple-functions are marked as
such with
`___.markFuncFreeze(func, 'name')`. This enables cajoled code to call
them as functions.

## Taming callbacks and functions as arguments ##

Some functions which you want to tame take other functions as
parameters or return functions as results. Taming these functions is
more involved. Suppose you want to tame the following function in
your library:

```
  function registerCallback(callback) {
    var result = someOperation();
    callback(result);
  }
```

When a cajoled program calls `registerCallback`, the `callback`
function it passes as an argument has already been cajoled. Caja
cajoles functions to an object or function which has three properties,
`call`, `bind` and `apply`. Remember that your taming code is not
cajoled and thus wields an uncontrolled amount of authority. In
particular, the default binding for occurrences of `this` is the
global object `window`, and naively granting `callback` access to the
global object will allow a cajoled program to break out of its sandbox
completely.

The safe and recommended taming of a callback function like the one
above would be to bind occurrences of `this` inside `callback` to
something that was useless and granted no additional authority. The
caja library provides one such object called `___.USELESS`. Suppose
the `callback` function expects one argument.  We can tame is as
follows:

```
   function tameRegisterCallack(cajoledCallback) {
     function rawCallback(result) {
       return ___.callPub(cajoledCallback, 'call', [___.USELESS, result]);
     }
     return registerCallback(rawCallback);
   }
   ___.markFuncFreez(tameRegisterCallack);
```

## Taming functions as returns values ##

Similarly, your library function may return a function itself. For example:
```
   function makeAdder() {
     return function(s) { return s+1; }  
   }
```

In this case, taming your library by marking `makeAdder` callable is
not sufficient because the function it returns will not itself be
callable by a cajoled program. The solution is to tame `makeAdder` by
marking the result it returns as callable:

```
  function tameMakeAdder() {
    return ___.markFuncFreeze(makeAdder());
  }
  ___.markFuncFreeze(tameMakeAddr);
```

## Simplified taming of libraries ##

As you can see above, if a large number of functions in the library you are taming take functions as arguments or return other functions, taming a library can become very tedious and error prone.  Caja provides a simpler means of taming such functions using `tamesTo`, `tamesToSelf`, `tame` and `untame`.

The `tamesTo` function allows you to associate an untamed function (which we call _feral_ functions) with their _tamed_ counterpart and vice versa. A feral object is one safe to make accessible to trusted but possibly innocent uncajoled code. A _tame_ object is one safe to make accessible to untrusted cajoled code. Most tamed objects should be both feral and tame and are safe to be accessed from both the feral and tame worlds.  An object can be marked safe to be accessed by both feral and tame worlds using `tamesToSelf`.

Once feral and tamed objects have been paired, a container writer can easily tame a function using `tame`.  The function call `tame(f)` returns an object which can be safely passed to cajoled code.  In particular, marked functions will be wrapped such that their arguments are untamed before the original function is called and its result tamed before it is returned.

For example `makeAdder` can be tamed just by marking it as a function and calling `tame`:

```
   function makeAdder() {
     return function(s) { return s+1; }  
   }
   ___.markFunc(makeAdder);
   tameAdder = ___.tame(makeAdder);
```

## Caveats while taming libraries ##

Obviously because the library you have exposed is not cajoled, it
could do anything. So, you must ensure that it doesn't do anything
that would allow the cajoled code to break out of its sandbox.

Firstly, only grant permissions to things you absolutely have to grant
permissions to.

Secondly, anything you do grant permission to has to be carefully
reviewed to make sure it is safe. Some things to avoid are

  * Use of `eval()` on anything controlled by the caller.
    * If `eval()` is really necessary then you must construct what is evaled very carefully to ensure it cannot be abused by the caller.
  * Setting `innerHTML` on DOM nodes. Any caller-controlled input must be sanitised to avoid the use of script tags and the like. The easiest way to do this is to use the HTML sanitizer supplied with Caja _need reference for how to do this_.

Sometimes the easiest way to deal with these issues (especially if the
library is a third-party library) is to wrap the dangerous functions
in functions of your own that sanitize or otherwise restrict the
inputs to the "real" functions.