# Introduction #

[Valija](http://groups.google.com/group/google-caja-discuss/browse_thread/thread/284935fa0784316d/31e476a985ae8c66?#31e476a985ae8c66) is approximately a subset of the ES5 strict mode. But existing libraries are built for "consensus JavaScript", also known as "ES3 + Reality" (hereafter "ES3R"), as represented by what browsers currently do. On this page we try to enumerate all features of ES3R not available in Valija, so that we can more quickly spot problem areas in porting old JavaScript libraries to Valija.

Neither ES5 not Valija is settled yet, and actual browser behavior is determined only by ongoing experiment, so these lists are preliminary and fluid.


# Details #

## In ES3R but not codified by ES3 ##

  * **`__proto__`**

This feature as spelled this way is not expected to ever become standard, since Mozilla uses the double underscore specifically to signal "not standard". In any case, it is not mentionable in Cajita or Valija for that reason. ES5 provides read-only access to an object's prototype by the static reflective method `Object.getPrototypeOf()`. Once we upgrade Caja to emulate the ES5 APIs, we will probably provide this as well, but currently do not.

  * **`Function.caller`**
  * **`Function.arguments`**
  * **`arguments.caller`**

ES5-non-strict does not specify them. ES5-strict effectively prohibits these by mandating that an attempt to access them throws an error. They currently don't exist in Valija, but we expect to emulate the ES5-strict behavior here as well.

## In ES3 or ES3R but not in ES5 ##

  * **crazy scoping** -- ES3 specifies various gratuitous violations of lexical scoping that are crazy enough that most browsers often ignore the spec language and do something sensible instead.
    * **'as if by the expression "`new Array()`"'** -- if such language is interpreted literally, rebinding _or shadowing_ the `Array` literal will cause, for example, array literal expressions to use the local binding to `Array` rather than the original built-in Array constructor.
    * **try/catch** -- see Deviations document
    * **nested function expressions** -- as with try/catch, ES3 specifies that the name is brought into scope by adding a new object to the scope chain. Therefore, according to the spec language, properties on `Object.prototype` will shadow _lexical variables_ of those same name within the body of the nested function.
  * **Shared mutable RegExp literal** -- ES5 specifies that each evaluation of a RegExp literal expression produce a new RegExp object. Cajita and Valija already implement this.

## In ES5, but not in ES5-strict ##

  * **`with`**
  * **`this`-promotion** -- when a function mentioning "`this`" is called as a function, or reflectively with its "thisArg" bound to null, occurrences of "`this`" in the function become bound instead to the global object. In ES5-strict, such use of "`this`" will evaluate to `undefined`. In Valija, "`this`" will currently evaluate to `cajita.USELESS` instead.
  * **`arguments.callee`**
  * **Joining of arguments array with parameter list**
  * **`delete <variable-name>`**

Taken together, this set of constraints guarantees that scoping in ES5-strict code can be statically analyzed.

## The continued adventures of block-nested declarations ##

  * **nested `var` declarations**
  * **nested `function` declarations**
  * **`const` declarations**
  * **`let` declarations**

ES5-strict doesn't provide for any of these; but all are expected in ES-Harmony-strict. Valija allows nested `var` declarations with the same meaning they have in ES3 and ES5-non-strict.

Valija allows nested `function` declarations, but not yet with the block-scoped semantics they are expected to have in ES-Harmony. Instead, Valija currently hoists the scoping of the declared function name to the beginning of the enclosing body (as with `var`s), and hoists the initialization of this variable to a function value to the beginning of the lexically enclosing block. This correctly emulates ES-Harmony's initialization but not its scoping.


## In ES3R and ES5-strict, but not in Valija ##

  * **`eval()`**
  * **`Function()` constructor**
  * **All identifiers ending in a double underscore ("`__`")**
  * **Distinction between a function's invoke behavior and its `call` or `apply` methods** -- In Valija, we translate all calls into calls to the `call` or `apply` method of the invoked thing, which need not itself be a function.
  * **Function members inherited from `Function.prototype`** --- `call`, `bind`, `apply`, and `length` are normally inherited from `Function.prototype`. In Valija, they are on each disfunction instance.

## In ES5-strict but not yet in Valija ##

  * **Reflective attribute control** -- ES5 adds reflective attribute control operations on the `Object()` constructor (including the aforementioned `Object.getPrototype()`).
  * **Getters and setters**


## Summary ##

Here is a table of features and misfeatures, and the JavaScript family that they are available in:
|                                                        | ES3         | ES3R    | ES5         | ES5-strict  | Cajita    | Valija |
|:-------------------------------------------------------|:------------|:--------|:------------|:------------|:----------|:-------|
| `__proto__`                                            | Unspecified | 3/4 Yes | Unspecified | Unspecified | No        | No     |
| `delete` variable                                      | Yes         | Yes     | Yes         | No          | No        | No     |
| aFunction`.caller`                                     | Unspecified | Yes     | Unspecified | No          | No        | No     |
| aFunction`.arguments`                                  | Unspecified | Yes     | Unspecified | No          | No        | No     |
| `arguments.caller`                                     | Unspecified | Yes     | Unspecified | No          | No        | No     |
| `arguments.callee`                                     | Yes         | Yes     | Yes         | No          | No        | No     |
| Modify meaning of `[...]` by rebinding `Array`         | Unclear     | Varies  | No          | No          | No        | No     |
| try/catch scope leakage                                | Yes         | Varies  | No          | No          | No        | No     |
| named function expression scope leakage                | Yes         | Varies  | No          | No          | No        | No     |
| Leak state between regular expressions                 | Yes         | Varies  | No          | No          | No        | No     |
| `with`                                                 | Yes         | Yes     | Yes         | No          | No        | No     |
| `this` promotes to global object                       | Yes         | Yes     | Yes         | No          | No        | No     |
| Joining `arguments` with parameter list                | Yes         | Yes     | Yes         | No          | No        | No     |
| `eval()`                                               | Yes         | Yes     | Yes         | Yes<sup>1</sup>      | No        | No     |
| `Function()` constructor                               | Yes         | Yes     | Yes         | Yes         | No        | No     |
| Identifiers ending in a double underscore ("`__`")     | Yes         | Yes     | Yes         | Yes         | No        | No     |
| Reflective attribute control                           | No          | No      | Yes         | Yes         | No        | No     |
| Getters and setters                                    | No          | 3/4 Yes | Yes         | Yes         | Future<sup>2</sup> | Future |



<sup>1</sup> But without violating static scoping

<sup>2</sup> Probably only the `get` and `set` syntax in object literals (see [Issue 999](https://code.google.com/p/google-caja/issues/detail?id=999))