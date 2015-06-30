# Global Object Poisoning #

## Effect ##
Can lead to a plugin's method being invoked in the context of the application's object, thus allowing the method to steal references to the object itself, its fields, and its methods ; or to replace its fields and methods.  This includes methods that are frequently implicitly invoked such as toString and valueOf.

Can lead to a spurious value showing up when iterating over keys in an Object or Array.


## Background ##
Each frame gets its own version of the builtin objects `Object`, `Array`, etc.

Array and Object are implicitly referred to via the array and object constructor syntaxes and Object is the base of all prototype chains, so even if global references to `Array` and `Object` are removed from code, they are still implicitly invoked.

Many javascript operators, when applied to objects, invoke special methods to coerce the objects to primitive value.  The arithmetic and comparison operators (`==`, `<`, etc.) invoke `valueOf` with a "type hint" of number which allows `new Date(2008, 0, 1) < new Date(1970, 0, 1)` to compare chronologically.

This combination of implicit constructors, and implicitly called methods makes it non-obvious when a constructor is called, and when a method might be invoked, making javascript code subject to [confused deputy attacks](http://en.wikipedia.org/wiki/Confused_Deputy) where a piece of attached code can be attached to a `prototype` and later run on an object the attacker cannot directly reference.

[Subspace](http://www2007.org/papers/paper801.pdf) may have found a way to launder a function or object using discardable iframes and carefully choregraphed `document.domain` tricks to allow a function or object to be passed to another frame, but that only allows passing of primitive data since returning an object would reopen the attack vector.

Also see [SpiderMonkey Peculiarity](http://yura.thinkweb2.com/named-function-expressions/#spidermonkey-peculiarity) for a way in which changes to `Object.prototype` can violate the assumptions a closure makes about its containing scope.


## Assumptions ##
An attacker can modify the prototype of `Object` or `Array`.

Global reference is not necessary.  The examples below work if
```
Array.prototype
```
is replaced with
```
[].constructor.prototype
```
and `Object` similarly


## Versions ##
Works in IE 6 and Firefox.  Example 2 works in Safari, but 1 is untested in Safari.  The function scope confusion trick works in the versions of SpiderMonkey that shipped with Firefox < 2.


## Example ##
(1) Array poisoning
```
Array.prototype[4] = 'four';
var a1 = [];
alert('a1 has length ' + a1.length + ' but element 4 is ' + a1[4]);

var a2 = new Array(5);
alert('a2 has length ' + a2.length + ' and its element 4 is also ' + a2[4]);
```


(2) Object poisoning
```
Object.prototype.toString = function () {
  var name = this.name ;
  this.name = 'mud';
  return 'HA ' + name + '.  IM STEALIN UR CODEZ!';
};
var o = { name: 'Bingo' };
alert('' + o);
alert('Your name is now ' + o.name);
```

(3) Cross-frame poisoning
```
// Frame 1
var stolen;
function fromOtherFrame(o) {
  o.constructor.prototype.valueOf = function () {
    stolen = this;
    return '[Object]';
  };
}
// Frame 1 uses some mechanism like document.domain juggling
// to pass a function to another.


// Frame 2
var secretObject = ...;
fromOtherFrame({ x: 4 });
if (secretObject == '');  // implicitly calls Object.prototype.valueOf
// Frame 2 passes an object containing no sensitive information.
// Frame 1 changes the definition of Frame 2's object, so it can
// steal access to secretObject even though secretObject is never
// explicitly passed to any function, and never has any of its methods
// explicitly called.
```