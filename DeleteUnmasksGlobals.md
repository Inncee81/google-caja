# Delete can unmask globals #

## Effect ##
If a rewriter implementation relies on with blocks to mask global references, such as by iterating all global references, and then creating an object with a property for each, and defining all untrusted code inside a with statement, the untrusted code can escape containment using delete.



## Background ##
The `with` construct takes an expression, and any reference R in its body not satisfied by an interior declaration is interpreted as `obj[R]` if `(R in obj)`.



## Assumptions ##
  * Globals are hidden by using the `with` construct; and
  * Either deletes of unqualified references are not prevented, or the masking object is aliased by this or some other reference that can have properties deleted from it.


## Versions ##
All



## Example ##
If the untrusted code
```
  delete document;
  alert('your cookie is ' + document.cookie);
```
is naively implemented as
```
  with ({ document: null }) {
    delete document;
    alert('your cookie is ' + document.cookie);
  }
```