# catch blocks may cause global assignment, or local scope creep #

## Effect ##
Replace global variables.


## Background ##
EcmaScript 262 says that only the Program and FunctionDeclaration constructor introduce new lexical scopes.  EcmaScript 262 section 10.1.4 says
> During execution within an execution context, the scope chain of the
> execution context is affected only by with statements (see 12.10) and
> catch clauses (see 12.14).

Section 12.10 describes with statements:
> The with statement adds a computed object to the front of the scope
> chain of the current execution context, then executes a statement with
> this augmented scope chain, then restores the scope chain.

Section 12.14 describes catch clauses
> The production Catch : catch (Identifier ) Block is evaluated as
> follows:
  1. Let C be the parameter that has been passed to this production.
  1. Create a new object as if by the expression new Object().
  1. Create a property in the object Result(2). The property's name is Identifier, value is C.value, and attributes are { DontDelete }.
  1. Add Result(2) to the front of the scope chain.
  1. Evaluate Block.
  1. Remove Result(2) from the front of the scope chain.
  1. Return Result(5).

Existing interpreters fail to implement the semantics of 12.14.  Old versions of firefox allow global assignment this way.

IE introduces it as a variable in the local scope, instead of creating a new scope (see  <a href='http://code.google.com/p/google-caja/issues/detail?id=1032'><a href='https://code.google.com/p/google-caja/issues/detail?id=032'>bug 1032</a></a>).


## Assumptions ##
There is no local variable of the same name as the caught variable already in scope, or there is one, and that variable is referenceable outside scope.



## Versions ##
Inconsistent


## Example ##
```
var a = 0;
(function () {
  try {
    throw 1;
  } catch (a) {
  }
})();
alert(a);  // alerts 1 on old FF

(function () {
  var a = 0;
  try {
    throw 1;
  } catch (a) {
  }
  alert(a);  // alerts 1 on IE 6 and 7 (IE 8 not tested)
})();
```