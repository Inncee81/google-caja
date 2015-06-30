## Background ##

Caja and SES had several vulnerabilities with a variety of causes.
  * Browser progress implementing EcmaScript 6
  * A recent change in the EcmaScript 6 spec
  * Bugs in browsers

(SES is [Secure EcmaScript](https://code.google.com/p/google-caja/wiki/SES), the component of modern Caja that deals with JavaScript. It is also independently useful outside the browser.)

### Browser progress implementing EcmaScript 6 ###

The latest draft standard of JavaScript is the 6th edition of
the EcmaScript standard, also known as EcmaScript 2015. Here, we will
refer to that as ES6, and to the latest official standard EcmaScript, the 5th edition,
as ES5.

Caja's taming of JavaScript is based on a whitelist, which lists all
the objects, properties, and methods of the JavaScript API that we
decide are safe to expose. This includes all of the ES5 API and some
of the ES6 API as they get deployed and we deem them safe. Until ES6
deployments are more mature, Caja's focus for now remains to provide a
safe subset of ES5.

We had assumed that elements of ES6 we omit from our whitelist were
also elements whose taming we could postpone reasoning about. We
missed two avenues by which elements not on our whitelist could
nevertheless be reached:

  1. An object on our whitelist might directly inherit from an object not on our whitelist, enabling the non-whitelisted object to be reached by prototype traversal. But we forgot to test these inheritance links against our whitelist during initialization, in order to catch such omissions.
  1. Besides API, which our whitelist handles, ES6 also introduces new syntax by which new ES6 objects are reachable. Specifically, ES6's _generator function_ syntax, `function*(){} ` evaluates to a new generator function object which directly inherits from the new `%Generator%` object. A variety of other new objects are reachable from `%Generator%`, about which more later. A goal of modern Caja is to avoid the need for parsing, so we must remain safe as such new syntax is introduced.

We whitelisted the typed-array classes initially according to the
Khronos specification. By the time they became the ES6 typed-array
specification, all these new typed-array classes directly inherit from
a new `%TypedArray%` superclass. Because of our first mistake
above, we missed that added superclass.

Another new ES6 syntactic construct by which a whole menagerie of new
objects would become reachable is the `import` syntax of ES6
modules. An ES6 `import` statement may only occur in module source
text. We are currently safe from these because untrusted code only
enters a SES environment through some variant of `eval` or the
`Function` constructor, neither of which may accept module
syntax. As a precaution, this release also tests to ensure that
`import` statements are indeed safely rejected.


### A recent change in the EcmaScript 6 spec ###

In ES5, if `Object.prototype.toString.call(x)` evaluates to, for
example, `"[object Date]"`, we could be sure that the value of x
was a genuine Date object, an instance of the builtin Date data
type. This was, by design, a reliable branding mechanism, and Caja's
implementation depended on this reliability in a few places.

Unfortunately, it is a non-extensible branding mechanism, useful for
distinguishing some built in types but useless for distinguishing
anything else. ES6 had adopted a compromise to preserve the security
it had in ES5 while nevertheless allowing some extensibility for
cosmetic, not security, purposes. After reviewing how Caja was using
it, we decided we could switch to other mechanisms that would remain
reliable. The spec could then standardize on a non-kludgy
extensibility mechanism.

We have now switched to other mechanisms when needed for secure
branding, so we are no longer endangered by browsers implementing this
new spec.



### Bugs in browsers ###

Mozilla shared their undisclosed
[SpiderMonkey (Firefox) extensibility bug 1125389](https://bugzilla.mozilla.org/show_bug.cgi?id=1125389)
with us, that allows an attacker to breach Caja's isolation
guarantees, enabling two confined instances of code to communicate
with each other without permission. The bug is that non-extensible
objects can be made extensible, and then extended, by use of a
delicate coding pattern triggering a flaw in SpiderMonkey's
optimization logic. Mozilla also gave us enough information so that we
could both test for the existence of this vulnerability, and repair it
in Caja when it occurs. They have reviewed both our test and repair
and confirmed that it does cope with their issue.

We reported undisclosed
[V8 (Chrome, Opera) non-configurability bug 3902](https://code.google.com/p/v8/issues/detail?id=3902)
to V8 and Chrome teams, where the `%Generator%.constructor`
property was a non-configurable, non-writable data property, whereas
ES6 specifies that it be a configurable, non-writable data
property. This is severe for Caja because, without parsing,
`%Generator%` is necessarily reachable by the generator function
syntax, as explained above, and the initial value of
`%Generator%.constructor` is the original
`%GeneratorFunction%` constructor, which Caja must deny to
confined code. The `%GeneratorFunction%` constructor is for
generator functions what the `Function` constructor is for normal
functions: The `%GeneratorFunction%` constructor creates a new
generator function from string source code for the function's body,
where this source code executes in the genuine global scope. As a
result, without parsing, we could not prevent the expression
```
  (function*(){}).constructor('yield window;')().next().value 
```
from evaluating to the genuine global (window) object, thereby
providing access to it.

We reported undisclosed
[JavaScriptCore (WebKit, Safari) \_\_proto\_\_ bug 141865](https://bugs.webkit.org/show_bug.cgi?id=141865)
to the WebKit team, which provides another avenue for obtaining the global
object. ES6 makes `__proto__` into an officially recognized
accessor property, whose getter and setter manipulate the inheritance
chain. `foo.__proto__` invokes the getter on `foo`, returning
the object that `foo` directly inherits from. Like any other
function, this getter can be extracted and directly supplied with any
other value as its this-binding. When applied to `undefined`, it
mistakenly acts as a legacy sloppy function does, coercing the `undefined` to the global
object. (A sloppy function is a non-strict function
coded in JavaScript.) As a consequence
```
  (1,Object.getOwnPropertyDescriptor(Object.prototype, '__proto__').get)();
```
evaluates to the prototype of the global (window) object. Instead,
this expression must throw, as it does on all other browsers.

We first reported public bug
https://bugs.webkit.org/show_bug.cgi?id=141871 to WebKit (Safari,
JavaScriptCore) when had we underestimated the extent of the problem
whose symptom we observed. Once we realized the deeper problem, we
followed up by reporting undisclosed
[JavaScriptCore (WebKit, Safari) throw-thaw bug 141878](https://bugs.webkit.org/show_bug.cgi?id=141878)
to the WebKit team. The issue is that throwing a frozen object causes
it to lose its frozen status, and for the underlying JavaScript
engine to add some properties to it. The problem is that the
`stack` property is added as a configurable writable data
property, to which any value can be assigned.
```
  var o = Object.freeze([]), leak = {}; 
  try { throw o; } catch (_) {}; 
  o.stack = leak;
```
This opens up the possibility of an arbitrary capability
leak between two compartments that should have been isolated because
they shared only transitively frozen objects.



## Impact and Advice ##

The overall impact of these bugs taken together is that Caja prior to
[r5717](https://code.google.com/p/google-caja/source/detail?r=5717), when run on browsers recent enough to have implemented any of
the triggering ES6 features (generators, typed arrays, `__proto__`
as an accessor) should be assumed vulnerable to a full breach. If the
`%GeneratorFunction%` constructor is reachable, a full breach is
trivial.

Our advice is therefore to immediately upgrade to at least Caja
[r5717](https://code.google.com/p/google-caja/source/detail?r=5717). Once you do, the remaining impact is:

On Firefox 35 and possibly some earlier, the Firefox vulnerability is
safely detected and repaired, but at some performance
cost. Fortunately, the current release is Firefox 36, which due to
Mozilla's quick action is not vulnerable, and so runs Caja without penalty.

On Chrome 40 / Opera 27 and possibly earlier, Caja detects the
non-configurability of `%Generator%.constructor` that prevents
Caja from repairing the situation. This causes Caja to fail to start
in ES5 (SES) mode. Those Caja installations configured to fall back to
ES5/3, the Caja translator to EcmaScript 3, will do so. The others
will fail to run Caja, which is their only way to fail
safe. Fortunately, the current releases are Chrome 41 and Opera 28,
which due to the V8, Chrome, and Opera team's quick action are not vulnerable,
and so succeed at running Caja in ES5 mode without penalty.

Note that Apps Script is no longer configured to automatically fall
back to ES5/3 (EMULATED) mode, as there were too many annoying
incompatibilities with ES5 (SES,NATIVE) mode.  Instead, a user only
gets EMULATED mode now when the developer explicitly sets EMULATED
mode. When NATIVE mode fails, the user is instead prompted to upgrade
their browser. This remains an issue for Chrome 40 and Opera 27.

On both Safari 7 (the Safari of Mac OSX 10.9 Mavericks) and Safari 8
(the Safari of Mac OSX 10.10 Yosemite), Caja detects and repairs both
Safari bugs. The repair for the throw-thaw bug is to preemptively add
the properties that throw/catch would add prior to freezing an
object. This includes all the reachable primordial objects -- the
object like `Object.prototype` that exist in a frame before code starts
running -- since SES must freeze these during SES initialization. This
has both performance costs and unavoidable namespace pollution costs.
Modulo these costs, this release of Caja should run fine on modern
Safari in ES5 (SES) mode.


## More Information ##

For all the links below you cannot yet follow, we have requested that
they be disclosed, so please try again in a few days.


### SpiderMonkey (Firefox) extensibility [bug 1125389](https://code.google.com/p/google-caja/issues/detail?id=125389) ###

  * https://bugzilla.mozilla.org/show_bug.cgi?id=1125389
  * https://code.google.com/p/google-caja/issues/detail?id=1954
  * https://codereview.appspot.com/202040043/

We thank Mozilla both for fixing this bug quickly, and also for
accelerating the release of this fix into Firefox 36, which is now the
official release. We also thank Mozilla for keeping this vulnerability
quiet until we could deploy this release of Caja.


### V8 (Chrome, Opera) non-configurability [bug 3902](https://code.google.com/p/google-caja/issues/detail?id=902) ###

  * https://code.google.com/p/v8/issues/detail?id=3902
  * https://code.google.com/p/chromium/issues/detail?id=460145
  * https://code.google.com/p/google-caja/issues/detail?id=1953
  * https://codereview.appspot.com/202030043/
  * https://codereview.chromium.org/950613002
  * **Fallout for Apps Script**
    * https://code.google.com/p/google-apps-script-issues/issues/detail?id=4866
    * https://code.google.com/p/google-caja/issues/detail?id=1957
    * https://code.google.com/p/google-caja/issues/detail?id=1958

We thank the V8 and Chrome teams for fixing this bug quickly, and also
for accelerating the release of this fix into Chrome 41, which is now
the official release. The V8 changes have also been incorporated into
the current Opera release, Opera 28. We thank the Apps Script team for
pushing the corresponding Caja fixes out quickly. We also thank the
V8, Chrome, Opera, and Apps Script teams for keeping this
vulnerability quiet until we could deploy this release of Caja.


### JavaScriptCore (WebKit, Safari) `__proto__` [bug 141865](https://code.google.com/p/google-caja/issues/detail?id=41865) ###

  * https://bugs.webkit.org/show_bug.cgi?id=141865
  * https://codereview.appspot.com/202030043/ (again)

We thank the WebKit, Safari, and JavaScriptCore teams for keeping
these vulnerabilities quiet until we could deploy this release of
Caja.


### JavaScriptCore (WebKit, Safari) throw-thaw [bug 141878](https://code.google.com/p/google-caja/issues/detail?id=41878) ###

  * https://bugs.webkit.org/show_bug.cgi?id=141878
  * https://bugs.webkit.org/show_bug.cgi?id=141871
  * https://codereview.appspot.com/202030043/ (again)
  * https://codereview.appspot.com/214110043/

We thank the WebKit, Safari, and JavaScriptCore teams for keeping
these vulnerabilities quiet until we could deploy this release of
Caja.


### EcmaScript 2015 (ES6) Object.prototype.toString spec change ###

  * https://esdiscuss.org/topic/tostringtag-spoofing-for-null-and-undefined
  * https://code.google.com/p/v8/issues/detail?id=3502
  * https://code.google.com/p/google-caja/issues/detail?id=1955
  * https://codereview.appspot.com/202140043/

We thank all makers of JavaScript engines and browsers for delaying
deployment of an implementation of the new cleaner ES6
`Object.prototype.toString.call(x)` spec until we could deploy
this release, so we will not be vulnerable when this ES6 change is
deployed.