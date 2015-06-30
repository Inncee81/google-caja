# Object.toSource and uneval allow access to private fields #

## Effect ##
Untrusted code that can reference an object can get access to private fields by s
erializing it using uneval and then parsing the resulting string.



## Background ##
`Object.toSource` and `uneval` are non-standard members of Object.

[toSource](http://developer.mozilla.org/en/docs/Core_JavaScript_1.5_Reference:Global_Objects:Object:toSource) is defined in spidermonkey's JS1.3+ interpreters and provides a mechanism similar to Python's `__repr__` method to allow object's to serialize themselves to code that can be evaled, so `(new Date(2000, 0, 1)).toSource() === '(new Date(946713600000))'`.

`uneval` does the same.

The act of serializing an object can expose it's private state.  Serializing a function using either of these methods does not expose private variable state as evidenced by
```
  var ctr = (function (idx) { return function () { return ++idx; }; })(0);
  ctr();
  ctr();
  ctr.toSource();
```


## Assumptions ##
uneval and Object.toSource are accessible to untrusted code.


## Versions ##
Firefox and possibly others.


## Example ##
```
// Untrusted code
function untrusted(o) {
  // untrusted need not attempt to access private_ directly
  var privateValue = o.toSource().match(/private_:\s*(\d+)/)[1] * 1;
  alert('private value is ' + privateValue);
}

var o = { private_: 4 }
untrusted(o);
```