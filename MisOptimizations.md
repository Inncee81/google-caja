# Interpreter Optimizations change behavior in hard-to-find Corner Cases #

## Effect ##
Basic operations like the post-increment operator have subtly different semantics from the spec depending on subtle features of the code around them.


## Background ##
EcmaScript interpreter implementers try hard to match the specified semantics of the language, but there is an trade-off between (performance and time-to-market) and correctness.

When implementers are testing an interpreter, they may focus on code that web-developers are likely to write assuming that the same-origin policy will ensure that a web-developer can only screw over their own website if they find some obscure corner case where an optimization is not semantics-preserving.

Since the same-origin model has provided a reasonable safety-net, fixing these corner cases has been low-priority.



## Assumptions ##
The same-origin model cannot be used to separate untrusted code from regular code.
Other approaches rely on the defined semantics of language constructs.
Browsers fail to implement the defined semantics in certain situations.
Attackers can find these situations before they can be fixed and the fix deployed.


## Versions ##
Unknown.


## Example ##
In IE6 and IE7:
```
  'string' === (typeof ((function (x) { return x++; })('foo')));
```
but if `x` is not a parameter or local variable of the function in which the increment appears, the result is `NaN` as in
```
  'number' === (typeof ((function (x) { return (function () { return x++; })(); })('foo')));
```

See PostIncrementAndDecrementCanReturnNonNumber for more detail on this example.