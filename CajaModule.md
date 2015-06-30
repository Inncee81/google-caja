# Caja Modules #

A [Caja](http://code.google.com/p/google-caja/) module is the output of the Caja sanitizer, or [cajoler](CajaCajole.md).  It is a javascript function of the form
```
  function(___, IMPORTS___) {
    // ...   
  }
```
that (currently--we're reserving this for potential later use) returns undefined.  The code is guaranteed to obey [object capabilities](http://en.wikipedia.org/wiki/Object_capability_model) rules.

At the moment, the most common kind of Caja module is one produced from an [OpenSocial](http://code.google.com/apis/opensocial/) gadget spec, but anyone can write a container to [host Caja modules](HostingModules.md).