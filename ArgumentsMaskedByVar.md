# function arguments array masked by var arguments on Opera #

See http://my.opera.com/hallvors/blog/?tag=javascript&startidx=24&nodaylimit=1.

## Effect ##
Runtime checks that rely on arguments for arity checking or for forcing recursion can be defeated by a local variable definition.  These checks will work on Firefox and IE, but will fail on Opera.



## Background ##
The function body has an implied reference to `arguments` that allows access to function arguments, and the function itself.

From Ecmascript 262 10.1.8,
> When control enters an execution context for function code, an
> arguments object is created and initialised as follows:
> ...
  * A property is created with name callee and property attributes { DontEnum }. The initial value of this property is the Function object being executed. This allows anonymous functions to be recursive.
  * A property is created with name length and property attributes { DontEnum }. The initial value of this property is the number of actual parameter values supplied by the caller.
  * ....  In the case when arg is less than the number of formal parameters for the Function object, this property shares its value with the corresponding property of the activation object. This means that changing this property changes the corresponding property of the activation object and vice versa.


## Assumptions ##
Untrusted code is allowed to declare a var with name `arguments` and runtime security checks use arguments to check parameters, or to force recursion to callee.




## Versions ##
Arguments masking is only a known issue on Opera, and arguments on IE 6 is not consistent with the spec.  Specifically, on IE 6, arguments.callee is not available.


## Example ##
```
(function (a, b, c) {
   var arguments = [1, 1, 1];
   alert('arguments[0] === ' + arguments[0] + ', a=' + a);
   arguments[0] = 2;
   alert('arguments[0] === ' + arguments[0] + ', a=' + a);
 })(0, 0, 0);
```