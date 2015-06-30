## Effect ##
Object can be observed in an inconsistent state if they can be tricked into calling out to code that causes an exception that is never caught.


## Assumptions ##
  * Sensitive code uses `try finally` to preserve its correctness.
  * Those pieces of code are reachable without control being inside the body of a `try` statement with a `catch` statement.
  * That sensitive code can be tricked into causing an exception after inside a critical section guarded by a `finally`.  Stack overflows result in exceptions.


## Versions ##
  * IE6 and possibly later.


## Examples ##
On IE 6,

```
try {
  ;
} finally {
  alert('Finally');
}
```

alerts "Finally," and

```
try {
  throw new Error();
} catch (e) {
  throw e;
} finally {
  alert('Finally');
}
```

also alerts but the below which should be semantically equivalent

```
try {
  throw new Error();
} finally {
  alert('Finally');
}
```

does not.