# `__parent__` Circumvents Scoping #

## Effect ##
The `__parent__` property circumvents normal scoping, allowing covert access to masked properties, possibly including protected globals such as `eval`.


## Background ##
In javascript 1.2 `__parent__` was defined to provide greater reflection.  It was removed from javascript 1.3, but many interpreters (including Firefox's) still implement it.  Mozilla has identifies it as a [security risk](http://devedge-temp.mozilla.org/library/manuals/2000/javascript/1.3/guide/sec.html#1015368) since it can leak references to XPCOM objects.

Javascript uses the `__parent__` chain to resolve global references in much the same way it uses the {{{proto} prototype chain to resolve object property references.


## Assumptions ##
Security is enforced by restricting reference to outer scopes and the `__parent__` property is accessible.


## Versions ##
Firefox.


## Examples ##
```
(function () {
  var alert = null;  // boilerplate that masks global alert

  (function () {  // untrusted code that can't access alert directly
    ({}).__parent__.alert('hello');
  })();

})();
```