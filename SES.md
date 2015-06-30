_All mentions of "ES5" below actually refer to EcmaScript 5.1_



# Introduction #

SES is mostly defined as a fail-stop subset of ES5. SES should compatibly run all ES5 code that follows recognized ES5 best practices. The SES restrictions support the writing of defensively consistent abstractions -- object abstractions that can defend their integrity while being exposed to untrusted but confined objects.

This page is for hard core developers, to document precisely the differences between the various forms of SES, and how these differ from ES5. Introductory and tutorial material will appear elsewhere. _**where?**_

# ES6 extras: `WeakMaps` #

There is one crucial way in which SES is more than a subset of ES5 though still a subset of ES6 -- the inclusion of the [WeakMap abstraction](http://wiki.ecmascript.org/doku.php?id=harmony:weak_maps) proposed for EcmaScript 6. Over time, as more of ES6 settles down and becomes available in JavaScript implementations, we expect to make more of these features available within SES as well. Note that `WeakMap` is always available in SES, whether or not it is available yet on the underlying JavaScript platform. (When it does not derive from platform provided `WeakMap`s, the [emulated WeakMaps](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/ses/WeakMap.js) are necessarily leaky, but this is technically a quality-of-implementation issue rather than a correctness issue.)

# Idealized SES vs ES5 vs ES5 best practices #

## Strict and Non-Strict Code ##

By default, ES5 code is non-strict, which is compatible enough with legacy ES3 code in exhange for preserving most of the semantic minefields that make ES3 programming hell. For example, ES5 non-strict code, like ES3 code, is not lexically scoped, and in ways almost no one understood. Non-strict failed assignments are silent, allowing control flow to proceed on paths that assumed success.

So ES5 best practice is to always [opt-in to strict mode](http://wiki.ecmascript.org/doku.php?id=conventions:avoid_strictness_contagion).

SES enforces that all SES code is strict, i.e., as if it had opted into strict mode, whether it does so explicitly or not. For ES5 code that follows ES5 best practices, this enforcement of strict code makes no difference.

## Whitelisted Primordial Properties ##

The notorious [Chapter 16 exemptions](http://es5.github.com/#x16) of the ES5 spec says that

> An implementation may provide additional types, values, objects, **_properties_**, and functions
> beyond those described in this specification. This may cause constructs (such as looking up
> a variable in the global scope) to have implementation-defined behaviour instead of throwing
> an error (such as ReferenceError).
_emphasis added_

Implementations do indeed make use of this freedom and provide all sorts of non-standard properties, many of which are semantically problematic (e.g., `RegExp.prototype.compile`, `RegExp.leftContext`), and that often differ from browser to browser (e.g., `__proto__`). Nevertheless, some properties that are not _de jure_ standards are unproblematic and universal, and have come to be recognized as harmless _de facto_ standards (e.g., `String.prototype.anchor`).

ES5 best practice limits code to relying only on such _de jure_ or harmless _de facto_ properties. While best practice code may attempt to use non-standard features when present, all such uses should be conditioned on feature tests for the presence of these properties, and the code should continue to work correctly in their absence.

SES is partially specified by its [whitelist](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/ses/whitelist.js), which includes all ES5 standard properties and those _de facto_ properties that we judge to be safe. SES guarantees the absence of any primordial properties that are absent from its whitelist. For ES5 code that follows ES5 best practices, these absences are unproblematic.

## Whitelisted Global Variables ##

Likewise, ES5 allows the global object to contain arbitrary properties, thereby defining arbitrary new global variables of the same names. The [SES whitelist](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/ses/whitelist.js) also defines what global variables are visible to SES code. The whitelisted globals include all the ES5 standard globals, the harmless _de facto_ standard **[escape](http://es5.github.com/#B.2.1)** and **[unescape](http://es5.github.com/#B.2.2)**, **`WeakMap`** as mentioned above, **[StringMap](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/ses/StringMap.js)**, which is a special case of the [proposed ES6 Map](http://wiki.ecmascript.org/doku.php?id=harmony:simple_maps_and_sets), and (for now) the global **`cajaVM`**, which provides a collection of other SES and Caja specific additional APIs. Notably absent from SES itself are authority providing globals such as `document`, `window`, or `location`.

We do not intend to propose `cajaVM` itself be part of the future _de jure_ Ecma SES standard. Any functionality on `cajaVM` we wish to standardize, we will relocate and possibly refactor before proposing as a _de jure_ standard.

To write _platform independent_ JavaScript code, i.e., code that is independent of whether its hosting environment is a browser, server, or whatever, ES5 best practice is to not rely on any non-standard globals. As with non-standard properties, best practice code may attempt to use such non-standard globals if present, but should condition such usage on feature testing whether those variables are present and have a value other than `undefined`, such as by testing
```
    if (typeof someConventionalGlobal === 'undefined') {
      // If someConventionalGlobal is either absent or has value undefined
      // Fallback behavior may simply be to terminate with an informative diagnostic
    } else {
      // If someConventionalGlobal is both present and has a non-undefined value
      // we may use it, assuming it has its conventional meaning
    }
```

Beyond the whitelisted globals, SES code may see additional global variable names depending on how it is spawned, including names like `document`, etc, which are bound to values emulating the traditional browser environment. _**We need to explain these spawning and scoping options somewhere**_

For ES5 best practice code that assumes (without testing) a hosting environment with additional globals, the SES code setting up the spawning environment can endow it with an emulation of the additional globals assumed by code expecting to run in that environment. Caja's [Domado library](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/plugin/domado.js) provides just such a safe emulation of the browser environment.

## Only Standard Syntax ##

The notorious [Chapter 16 exemptions](http://es5.github.com/#x16) of the ES5 spec also says that

> An implementation may extend program syntax and regular expression pattern or flag
> syntax. To permit this, all operations (such as calling eval, using a regular expression
> literal, or using the Function or RegExp constructor) that are allowed to throw SyntaxError
> are permitted to exhibit implementation-defined behaviour instead of throwing
> SyntaxError when they encounter an implementation-defined extension to the program
> syntax or regular expression pattern or flag syntax.

When this additional syntax adds only sugar whose meaning is equivalent to code that could have been written in the standard language, it presents no great problem. However, this exemption allows non-standard syntax that could express non-standard semantics, including semantics that is [impossible to secure](https://bugzilla.mozilla.org/show_bug.cgi?id=695579). Whereas programs can feature test for additional non-standard properties, as above, it is much more difficult (though possible using `eval`) to feature test for non-standard syntax.

ES5 best practice is to write code that uses only standard syntaxes.

SES accepts only standard syntax, or at most those non-syntactic elements of the underlying platform that are morally equivalent to sugar. For ES5 code that follows ES5 best practices, this restriction makes no difference.

Note that the parser-free _Target-SES_ implementation explained below cannot prohibit non-standard syntaxes, but rather simply refuse to run if it detects the acceptance of specific non-standard syntaxes that it knows to worry about, [such as E4X](https://bugzilla.mozilla.org/show_bug.cgi?id=695577).

## No Monkey Patching Primordials ##

In ES5, almost all specified properties of build in primordial objects are specified to start in the writable and configurable state. And the built in primordial objects are themselves specified to start in the extensible state. Since all code loaded into a context (frame) implicitly shares much of this primordial state, this implicit access to shared mutable state represents an un-granted communications channel. Worse, it allows any code in that context to engage in _prototype poisoning_, replacing inherited utility methods, like Function.prototype.call, whose meaning other code relies on for its integrity.

SES, to enforce object-capability rules, must prevent all un-granted communications channels. So in SES, all built-in primordial objects are transitively frozen, which would ideally make them immutable. However, there are three remaining sources of primordial mutability left, which we enumerate below.

Some old libraries, like _Prototype_, made use of this pervasive mutability to install its own extension, e.g., to `Array.prototype`. However, these extensions would often conflict with each other, especially if two versions of the same monkey patching library were loaded into the same context. So ES3 and ES5 best practice is to avoid mutating (or monkey patching) any built-in primordial objects. So in this regard, ES5 best practice is consistent with frozen primordial state.

Unfortunately, due to a [mistake made when specifying ES5](http://wiki.ecmascript.org/doku.php?id=strawman:fixing_override_mistake), there is one bit of ES5 best practice which would break given naively frozen primordials. Suffice it to say that we specify SES to not suffer from [mistake made when specifying ES5](http://wiki.ecmascript.org/doku.php?id=strawman:fixing_override_mistake), and so not be in conflict with ES5 best practices. And on conforming ES5 platforms, we implement a reasonable approximation of this spec to have only a minor impact on code following ES5 best practices. We return to this topic at [Tamper Proofing vs Freezing](http://code.google.com/p/google-caja/wiki/SES#Tamper_Proofing_vs_Freezing) below.

### Current date and time ###

A no-argument call to the `Date` constructor, `new Date()`, as well as a call to `Date.now()` gives access access to the current date and time, which is an un-granted read-only access to be effected by state change of the outside world (the advance of time). Further, but providing undeniable access to time, we widen the information leakage possible through covert timing channels. Since SES makes no claims to prevent or even inhibit covert channels, and since this channel is really read-only, SES leaves it undisturbed, which is technically in violation of object-capability rules.

### `Math.random()` ###

`Math.random()` is a standard primordial method that provides a new random-enough number (between 0.0 and 1.0) each time it is called. This is clearly not immutable, in that each call is clearly mutating some hidden state. Were it cryptographically pseudo-random, then to any computation that does not know the hidden state, the behavior of `Math.random()` would not be observably different (for feasible observations) from a source of true random numbers, e.g., from tapping a source of physical entropy. Such a source of physical entropy need not be affected in any way by a call to Math.random(), and so we could safely consider it a read-only channel, like Date.

However, `Math.random()` is not specified to provide such high quality entropy, and actually implementations are know to provide low enough entropy to lead to actual security problems. But again, since SES makes no claims to prevent or inhibit covert channels, SES currently does not tighten this spec, and current SES implementations do not replace `Math.random()` with a higher quality source.

### Accidentally Unfreezable Mutable  State ###

Even though `Date.prototype instanceof Date` returns `false`, nevertheless, `Date.prototype` is itself specified to be a Date object. (In spec language, its `[[Class]]` is `"Date"`.) Doing `Object.freeze(Date.prototype)` as well as freezing all the methods found there does not suppress all implicitly shared mutable state. Date objects have an internal property, not freezable with `Object.freeze`, that can be accessed only by the built-in Date setting methods found on `Date.prototype`.

We specify SES so that using any of the Date setting method on a Date prototype from any context (frame) causes no mutation and instead throws a TypeError. On conforming ES5 platforms, the SES implementation realizes this spec by monkey patching the Date setting methods. On IE10, the Date setting methods indeed cause no mutation, but instead of throwing a TypeError, they fail silently. Since the IE10 behavior is SES-safe, we currently do not monkey patch these to throw a TypeError on IE, but may change out mind.


## No Mutable Globals ##



## Script Code vs Eval Code ##

## Direct and Indirect `eval`, and the `Function` Constructor ##

## Powerless thrown Errors, etc. ##

# SES-on-ES5 vs Idealized SES #

## Tamper Proofing vs Freezing ##

The following code does not violate any ES5 best practice:

```
  function Point(x, y) {
    this.x = x;
    this.y = y;
  }
  Point.prototype.toString = function() {
    return '<' + x + ',' + y + '>';
  };
```

However, if SES simply freezes the ES5 primordials, then the above code would fail. The reason being that ES5 specifies that an attempt to override a non-writable data property with assignment fails, and we were assuming that `Point.prototype.toString` was made simply non-writable non-configurable, by freezing `Point.prototype`. So instead of simply transitively "freezing" all the primordials, we instead "tamper proof" them. Tamper proofing differs from freezing as necessary so that SES does accept the above code, in order to avoid rejecting code obeying ES5 best practices.

To do so on browsers which correctly implement the ES5 spec, SES does this by classifying objects into "prototypical" and everything else. We make the pragmatic assumption that only properties of prototypical objects need to be overridable by assignment. SES makes these properties into accessor (getter/setter) properties whose setter accepts the above code. For these properties, it emulates the situation where [ES5 Override Mistake](http://wiki.ecmascript.org/doku.php?id=strawman:fixing_override_mistake) is fixed.

# SES vs CES (Confined EcmaScript) #

SES will support spawning a JavaScript context (frame) running with a weaker kind of safety, in order to be more compatible with legacy JavaScript. The rules in this other frame are a variant of SES we call CES, or Confined EcmaScript. The difference is that in CES the built-in primordial state is not frozen. As a result, legacy code which monkey patches their built in primordials, such as the Prototype library, should still be able to run. However, this code cannot feasibly defend its own integrity against prototype poisoning. Which should be fine, since the only code that should be run under CES is legacy pre-SES code that is ignorant of the possibility of defending its integrity anyway.

The safety that is preserved is that the CES context as a whole is confined, and so can only effect the world outside itself according to what authority its spawner explicit grants it. The spawner (itself presumably in SES) is in a good position to defend itself from the CES frame.

Whereas in SES, the unit of protection is the individual object, in CES the unit of protection is the context (frame) as a whole. The granularity is like an HTML5 sandboxed unique origin iframe, but the security is much stronger.

# Source-SES vs Target-SES #

On an ES5 platform, the SES initialization results in a object-capability system that can accept untrusted code without parsing or translation, and safely run it confined under object-capability rules. However, because of the means we use to secure it without parsing or translation, the language in which this untranslated code is written differs from SES in several minor ways, which we enumerate below.

When we distinguish this untranslated almost-SES language from SES, we refer to the untranslated language as _Target-SES_ and real SES as _Source-SES_. At [expandProgramToExpr.js](http://code.google.com/p/google-caja/source/browse/trunk/experimental/src/com/google/caja/ses/expandProgramToExpr.js?r=4754) is pseudo-code explaining our translation from Source-SES to Target-SES. If you are using this translator in order to write in Source-SES, _and_ if all code of concern that you don't trust is also required to pass through the Source-SES to Target-SES translator, then you can ignore these differences. Otherwise, they are relevant.

The SES implementation also provides an optional "mitigateGotchas stage" that uses a parser and rewriter to minimize these differences between _Source-SES_ and _Target-SES_ where possible.  We note in the relevant sections below where such mitigation is performed and it's limitations.  TODO(jasvir): Flesh out the actual mitigation.

## Completion Value ##

[ES5's eval](http://es5.github.com/#x15.1.2.1) is specified to evaluate a [Program](http://es5.github.com/#x14) and to return the Program's [completion value](http://es5.github.com/#x8.9). SES instead specifies the [completion value reform](http://wiki.ecmascript.org/doku.php?id=harmony:completion_reform) specified for ES6. However, this is impossible to emulate in Target-SES without parsing or translation. Instead, Target-SES's `eval` makes the following compromise:

If the source test happens to parse as an Expression, then `eval` returns the value this expression evaluates to. Otherwise, it just evaluates the program and returns `undefined`.

## eval, Function ##

Since the functions bound to `eval` and `Function` are first class values, it would create confusion for these to seem to be different values in Source-SES and Target-SES. We specify these to accept Source-SES, requiring a client side translator. However, since we do not yet have a client-side Source-SES to Target-SES translator, our current implementation has both of these accept Target-SES.

## Top Level Declarations ##

In SES and ES5-strict, a [Program](http://es5.github.com/#x14) may be evaluated as _global code_ or as _eval code_. Practically, in a browser, scripts as brought in by a script tag are global code. A top level variable declaration has two different meaning for these two types of evaluation. In a script, a top level declaration adds a property of that same name to the global object, so that two scripts in the same context (frame) can use top level variable declarations for inter-script linkage. By contrast, a top level declaration in strict eval code is local to that program and does not effect the global object.

Source-SES and Target-SES operate likewise (but for one complexity explained below). Further, SES script code sees the full global environment and the real global object, and so wields the full authority of its frame. Thus, SES script code should only bring in untrusted code as SES eval code, to be evaluated by one of the SES safe evaluators, including `eval` and `Function`.

Unfortunately, the situation is not quite so simple. SES is typically used alongside DOM virtualization libraries like Domado. These must translate untrusted html into safe html. To do so, it must translate untrusted scripts embedded in html into safely confined scripts. Only SES eval code is safely confined, which would break the linkage between scripts on the html page. Instead, we translate these embedded scripts into calls to `cajaVM.compileExpr(src)(imports)`, where the `imports` argument represents the pseudo-global that these sources can use for inter-script linkage. For Source-SES, that's the end of the story, and there is no observed incompatibility because of this issue.

For Target-SES, the problem is that we know of no technique for feasibly and safely implementing compileExpr without parsing or translation that allows top level declarations to be used in this way. Instead, to write code that works as both regular JavaScript script code and as Target-SES script code, use the following pattern:

```
(function(global) {
  "use strict";

  global.foo = 8; // as if a top-level "var foo = 8;"

  .... foo + 3 ....  // foo is now a property on imports and in the "global" scope for this script.

})(this);
```

The mitigateGotchas stage implemented by SES rewrites variable declarations of the form:

`var x = expression;` to `this.x = expression;`
`var x; ` to `this.x = this.x`

Variable declaration in top-level for..in loops, however, are not rewritten into top-level variable declarations and thus remain a gotcha.

## typeof _variable_ ##

In ES5, in most way, `typeof` acts as a unary operator -- the operand expression is evaluated to a value, and then `typeof` examines that value. If the evaluation of the operand throws an error, the throw propagates and the `typeof` work is skipped. However, ES5 and Source-SES make one special case: If the operand is a simple variable name which names a non-existent variable, i.e., if there is no corresponding variable definition or global property, then the variable name as expression would normally throw a `ReferenceError`. This is indeed its behavior under Target-SES because we have no other choice. But under ES5 and Source-SES, in this special circumstance, no error is thrown and the `typeof` expression returns the string `"undefined"`.

To write `typeof` tests that operate the same way under ES5, Source-SES, and Target-SES, using the above framework giving us a `global` binding, replace "`typeof foo`" with "`typeof global.foo`".

```
(function(global) {
  "use strict";

  ... typeof global.foo ... // like conventional "typeof foo"

})(this);
```

## `this`-binding of Global Function Calls ##

In ES3 and ES5-non-strict, when a global function is called as a function, its `this`-binding is the global object. In ES5-strict and in Source-SES, its `this`-binding is `undefined`. Unfortunately, the Target-SES behavior is more like the ES3 and ES5-non-strict behavior, but the global in question is the scope object for that Target-SES evaluation.

To write function calls that operate the same way under ES5-strict, Source-SES, and Target-SES, always wrap the function to be called in an otherwise-useless comma expression:

```
    ... (1,foo)() ... // like conventional "foo()"
```

## Safety of Thrown Values ##


In Source-SES, when some thrown value is caught, the value as caught is guaranteed to be immutable and powerless. If the Source-SES runtime cannot verify that the throws value is immutable, it will derive an informative immutable value from the thrown value (such as by `toString`-ing the thrown value) and use it instead as the caught value. However, in Target-SES we are not in a position to intervene in the try/catch logic, and so cannot sanitize caught values.

None of the other differences between Source-SES and Target-SES creates any significant insecurity. Mostly they are just annoyances because they make Target-SES less compatible with ES5. However, this issue is different. The Target-SES behavior does not violate object-capability formal properties. But experience shows that in code reviews, it is infeasible for reviewers -- no matter how much they try to be vigilant -- to remember to think about what may escape on the exceptional control flow pathway. The solution adopted by several other object-capability languages is the same as Source-SES -- prevent anything from being transmitted along this pathway that would enable the catcher to cause additional effects. However, we don't have this option in Target-SES.

This leaves us with a hard choice: Do we allow untrusted code to be loaded and run as Target-SES code, or do we insist that all untrusted code be Source-SES code and be translated by the Source-SES to Target-SES translator? Currently, since we have not yet built this translator, we have no choice. By the time we have a choice, we may have a legacy of Target-SES code.