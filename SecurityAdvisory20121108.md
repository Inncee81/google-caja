# Caja Security Advisory 2012/11/08 #

## Description ##

### Foreign nodes ###

There are two vulnerabilities in the “foreign node” and “opaque node” facilities of our DOM virtualization, which may result in such nodes being modified or examined by guest code which should not be able to do so.

The first vulnerability is that the restriction of foreign or opaque nodes was implemented by overriding the methods to be suppressed. This can be trivally defeated in JavaScript by obtaining the unsuppressed method from another object, or from the target object's prototype chain, and invoking it on the target object. Furthermore, the **editable** flag of foreign and opaque nodes is always set to true, so they may be arbitrarily mutated with the unsuppressed methods.

The second vulnerability is that the taming of the innerHTML property getter does not respect opaque or foreign nodes, because it is based on invoking the browser's innerHTML getter and then sanitizing the result; there is no provision to respect the attribute-hiding policy of opaque nodes, and it is impossible to respect foreign nodes as that policy is node-identity-based.

### Tamed functions ###

Methods of host objects exposed to guest code via the taming layer may be applied to objects they were not intended to be ([issue 1519](https://code.google.com/p/google-caja/issues/detail?id=1519)).

### Actions outside of user interaction ###

We have decided that guest code should not be able to:

**trigger a form submission except in response to a user-generated event, or** change focus without a click or key event or similar.

## Impact ##

### Foreign nodes ###

If malicious guest code is given a DOM tree containing an opaque node (that is, an element not in our whitelist), the guest code may modify the node's attributes and children arbitrarily. This can lead to arbitrary code execution outside of the Caja sandbox, such as by the node being an 

&lt;object&gt;

 element.

If malicious guest code is given a DOM tree containing a foreign node, or a reference to a foreign node, the guest code may read or modify the node's attributes and children. This includes the above attack as well as exposing any information within the foreign node's subtree.

If guest code is loaded without a preexisting DOM tree (which is the default, e.g. when the HTML is guest-provided) and the foreign node facility (`tameNodeAsForeign`) is never used, then neither of the above cases is exploitable.

Note that foreign nodes are used by the Google APIs taming facility.

### Tamed functions ###

Host methods exposed using the taming layer can be used to alter the state of host objects they were not intended to alter. The risk of this depends on what the host methods do, but in general, an attacker may use this to take advantage of coincidences in the way host code is written to place host objects in a vulnerable state.

### Actions outside of user interaction ###

Guest code could automatically load arbitrary pages using `form.submit()` without user action.

Guest code could steal focus in response to incidental events such as `mousemove`.

## Advice ##

Upgrade to a version of Caja at or after [r5145](https://code.google.com/p/google-caja/source/detail?r=5145) as soon as possible. We do not recommend patching the vulnerabilities individually.

## More Information ##

Discussion of the changes for the foreign node vulnerability is available at http://codereview.appspot.com/6490106/ and http://codereview.appspot.com/6498123/. They were committed as [r5143](https://code.google.com/p/google-caja/source/detail?r=5143) and [r5144](https://code.google.com/p/google-caja/source/detail?r=5144).

Discussion of the changes for the host method vulnerability is available at https://codereview.appspot.com/6499089/. It was fixed in [r5082](https://code.google.com/p/google-caja/source/detail?r=5082).

The event-related vulnerabilities were fixed in [r5145](https://code.google.com/p/google-caja/source/detail?r=5145).