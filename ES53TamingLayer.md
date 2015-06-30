# Introduction #

This is a quick description of some aspects of the ES5/3 taming layer. This layer is implemented in [es53.js](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/es53.js).

# General design #

Our taming layer is an objective membrane. It is a _membrane_ because it wraps the entire API reachable from a tamed object, allowing functions in that API to be called but unwrapping and wrapping, respectively, their arguments and return value. It is _objective_ because, to each feral object, it assigns exactly one tamed object, and all Caja clients of the feral object receive the same tamed object.

By way of nomenclature, each tamed feral object has a _tamed twin_, and each tamed object has a _feral twin_ from which it was created. These are bidirectionally maintained using the hidden properties `TAMED_TWIN___` and `FERAL_TWIN___`, respectively.

To a first approximation, all tamed twins are proxies, implemented using the ES5/3 generic handlers. A corollary is that the relationships between objects in a graph are all represented in the _feral_ side. The handlers on the tamed twins navigate to the feral objects via `FERAL_TWIN___` to do their work. When needing to return the taming of a feral object to the Caja code, the taming layer returns the existing `TAMED_TWIN___` or, if no such exists, lazily create, store and return it.

![http://google-caja.googlecode.com/svn/trunk/doc/images/tameAndFeralObjects.png](http://google-caja.googlecode.com/svn/trunk/doc/images/tameAndFeralObjects.png)

A limitation of our approach is the fact that we cannot uniformly intercept access to numeric indices (as in, `a[3]`). This is because our compiler provides a fastpath for As a result, our tamed twins uniformly return `undefined` as the value of any numeric property, and throw upon any attempt to assign to any numeric property. The only exception is arrays, which are discussed below.

# Taxonomy of feral objects #

The following is a brief taxonomy of feral objects and a description of the ways they are tamed:

## Primitives ##

Things that are `typeof` something other than `'object'` or `'function'` are considered primitives and are tamed and untamed to themselves.

## Records ##

A record is an object that directly inherits from the `Object.prototype` of the frame in which it is defined. A record is tamed to a proxy through which read-write access to the feral object is permitted, unless the application has marked the record as read-only.

## Arrays ##

Arrays are always passed by copy as they cross the taming boundary via function calls. Clearly, this is a compromise. Since proxies cannot intercept numeric indices, arrays are the only kind tamed object for which numeric indices are supported. To implement the correct behavior, where items assigned as values in an array are tamed when moving across the boundary towards Caja code, and un-tamed when moving in the other direction, and yet have the proper behavior if one side holds on to an array they got from the other side, we must implement them as pass-by-copy.

## Functions ##

The `TAMED_TWIN___` of a function is just another function which forwards to the feral one, un-taming the arguments passed and taming the return value. Depending on how the application asked for a function to be tamed, the function may or may not be an xo4a, in other words, it may or may not receive the `this` value passed by the Caja caller.

## Constructors ##

Constructors must be marked by the application to be tamed as such. To tame them, we build a shadow pair of a tame constructor (which is a function) and a tame prototype (which is a proxy). We also construct an "instantiator" function, with the appropriate `prototype` property, which allows us to construct new feral instances reflectively.

![http://google-caja.googlecode.com/svn/trunk/doc/images/constructorTamingES53.png](http://google-caja.googlecode.com/svn/trunk/doc/images/constructorTamingES53.png)

The result of this arrangement is that Caja code perceives the tamed objects to be properly `instanceof` the `TAMED_TWIN___` of the constructor.