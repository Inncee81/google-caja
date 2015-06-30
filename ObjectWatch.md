# Object.watch allows stealing and poisoning of otherwise restricted data #

## Effect ##
If static or runtime checks prevent access to certain properties, then on Firefox, malicious code can still access those properties by using Object.watch.


## Background ##
http://developer.mozilla.org/en/docs/Core_JavaScript_1.5_Reference:Global_Objects:Object:watch defines Object.watch as a property of all javascript Objects that allows a client to watch a particular property of a particular Object and receive notifications when it changes, and possible modify the value set.


## Assumptions ##
`Object.watch` is callable by client code.


## Versions ##
Firefox and possibly others.  Not IE.



## Example ##
```
// Untrusted code need never access private directly to observe and
// modify private fields of a mutable object
function untrusted(o) {
  o.watch(
      'private_',
      function (obj, oldval, newval) {
        alert('untrusted got oldval ' + oldval + ' and newval ' + newval);
        return 'poisoned';  // substitute a bogus value
      });
}

// Trusted code
var o = { private_: 'old' };
untrusted(o);
o.private_ = 'new';
alert('private is now ' + o.private_);
```