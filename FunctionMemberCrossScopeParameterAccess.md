# function object's arguments array expose arguments while call in progress #

## Effect ##
Untrusted code can steal the arguments passed to any function it can reference while that function is being called.



## Background ##
The arguments passed in the most recent uncompleted call to a function are exposed via its `arguments` property.  This is not documented in EcmaScript 262



## Assumptions ##
The arguments property of Function objects is accessible.

Untrusted code can reference a function whose parameters are sensitive.


## Versions ##
FF and IE 6 at least


## Example ##
```
function f(a) {
  g();
  alert(a);
}

function g() {
  f.arguments[0] = 1;
}

f(0)
```

Note: the above has f call g.  This is not necessary.  If f and g occur concurrently, then g can steal/change f's parameters without being called by it.  Code can run concurrently if one is the handler from an XmlHttpResponse, or can occur if there are two window's event threads -- create a new iframe, and use its setTimeout.