# Introduction #

ES5/3 is an implementation of ES5 that runs on ES3 browsers. ES5/3 will take the place of Valija; we will deprecate and eventually end-of-life Valija. SES5/3, the Cajita replacement, is exactly the same with the additional feature of freezing all the accessible primordials (i.e., all the primordials other than the global object).

SES5/3 will upwards compatible from Cajita. ES5/3 will be upwards compatible from Valija, Cajita, and SES5/3.

There will be some differences between ES5/3 and ES5, detailed below. For code written with these in mind, ES5 will effectively be upwards compatible from ES5/3. SES and SES5/3 are true object-capability languages.

To try out ES5/3 in the [Caja playground](http://caja.appspot.com/), select the "ES5" radio button, type in some html with embedded ES5 scripts, and hit the "Cajole" button.

# Differences between ES5/3 and ES5 #

  * Strict mode only
  * No `eval`
  * No `Function` constructor
  * `Object.prototype` is frozen
  * The true global object is inaccessible
  * Property names must not end in two underscores
  * Does not support creating objects with null prototype
  * In a method call, the getter is called after arguments are evaluated ([V8 has the same issue](http://code.google.com/p/v8/issues/detail?id=691))
  * Whereas ES5/strict specifies no `this` coercion, ES5/3 will coerce primitive values to wrappers (as ES3 and ES5/non-strict do), and will coerce `null` and the global object to `undefined`.
  * Many coercions to string that should be implemented by `String(obj)` are instead implemented by `('' + obj)` since the latter is noticeably faster. The difference is that `valueOf` gets called first in circumstances where `toString` should have been called first.
  * Numerically named properties and `'length'`
    * All numeric own properties of the same object share one property descriptor
    * Numeric properties must be data properties
    * `'length'` is a non-configurable accessor property on arrays
    * `'length'` must not be an accessor property on non-arrays
    * Only supports bracketed numeric indexing into strings (e.g. `'foo'[1]`) if the underlying ES3 engine does
    * If we can statically determine that a variable will only contain numbers after being initialized and this variable is used as a property name before initialization (e.g., `x[i]`), then it may be converted to the property name `'NaN'` rather than the property name `'undefined'` (e.g., as if translated to `x[+i]`).
  * Invoking `Function.prototype` throws an error instead of returning undefined.
  * The `[[Construct]]` behavior on bound functions throws a "not yet implemented" complaint.
  * RegExps are not callable, and thus `(typeof` _aRegExp_`)` should return `"object"` according to the ES5 spec. Instead, it returns either `"function"` or `"object"` according to the underlying platform. Practically, this impacts Safari and Rhino.
  * Checking whether a non-whitelisted property on an object exists using the "in" keyword returns false even though assigning to the property causes an error to be thrown that says "This property is not writable."

# Differences between ES5/3 and selected ES-Harmony features #

  * Proxies
    * Proxies cannot trap numeric properties or 'length'.
    * Might not ever be implemented:
      * Proxies cannot (yet?) inherit from extensible non-proxy objects.
    * To be implemented:
      * Proxies cannot yet inherit from other proxies.
      * Non-proxies inheriting from proxies is not yet fully implemented.
      * The default set trap behavior is not yet implemented
  * "Leaky" WeakMaps
    * Membranes hold onto all references that they wrap until the membrane is revoked.

# Sources for the ES5/3 additions to Caja #

[es53.js](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/es53.js),

[ES53Rewriter.java](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/parser/quasiliteral/ES53Rewriter.java)

[ES53RewriterTest.java](http://code.google.com/p/google-caja/source/browse/trunk/tests/com/google/caja/parser/quasiliteral/ES53RewriterTest.java)