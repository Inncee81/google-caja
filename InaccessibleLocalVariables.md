# Inaccessible Local Variables #

## Effect ##
Some variables declared may be inaccessible, and references using those as RHS may see a value even if the variable has never been assigned.


## Background ##
The label `arguments` in a function is not assignable on most browsers.
The label in a catch block may or may not be accessible outside that block depending on browser, and may mask local variables declared in the enclosing function.


## Assumptions ##
Untrusted code can declare local variables with these names, and static checkers forgo restrictions based on whether a variable has been assigned a value.


## Versions ##
Differs


## Example ##
```
(function () {
   var arguments;
   alert('arguments === undefined: ' + (arguments === undefined));
 })();

(function () {
   var e;
   try {
     throw 1;
   } catch (e) {
   }
   alert('arguments === undefined: ' + (arguments === undefined));
 })();

(function () {
   var e = 1;
   try {
     throw 2;
   } catch (e) {
   }
   alert('e === 1 : ' + e);
 })();
```