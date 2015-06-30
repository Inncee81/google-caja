# typeof inconsistent for regular expressions and other intrinsics #

## Effect ##
See http://javascript.crockford.com/remedial.html.
```
('function' === typeof o) !== (o instanceof Function)
```
even ignoring different `Function`s from different frames.

## Background ##
The `typeof` operator must return `'function'` for anything that is callable.

According to section 11.4.3 of EcmaScript 262
| Type | Result |
|:-----|:-------|
| Undefined | "undefined" |
| Null | "object" |
| Boolean | "boolean" |
| Number | "number" |
| String | "string" |
| Object (native and doesn’t implement `[[Call]]`) | "object" |
| Object (native and implements `[[Call]]`)        | "function" |
| Object (host)                                  | Implementation-dependent |
so an callable object is a `"function"` and a "host object" can have any type it desires.

A host object is a javascript object that is backed by special browser or operating system code.  Most browsers' DOM trees are host objects, and plugins and extensions are often exposed as host objects.  The spec allows for
```
(typeof new ActiveXObject('Crime Fighter')) === 'batman'
&& (typeof new ActiveXObject('ComplexNumber')) === 'number'
```
so that the host object exemption means that the identity
```
(x === undefined) === ((typeof x) === 'undefined')
```
does not hold.

IE allows for some non-function callables.  On IE 6 and 7,
```
'object' === (typeof alert)
```
and similarly for many other builtins: `confirm`, `prompt`, `setTimeout`, `setInterval`, `clearTimeout`, `clearInterval`, and some of the DOM constructors such as `Image`.


## Assumptions ##
Runtime checks based on `typeof` allow access to members of functions that are not allowed on normal Objects.



## Versions ##
On Firefox,
```
'function' === (typeof /./)
'function' === (typeof alert)
```
On IE 6,
```
'object' === (typeof /./)
'object' === (typeof alert)
```

Some hosted objects return `'unknown'` as the `typeof` value, but I don't have a specific example.