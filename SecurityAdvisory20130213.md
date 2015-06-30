# Caja Security Advisory 2013/02/13 #

Caja prior to version [r5288](https://code.google.com/p/google-caja/source/detail?r=5288) is vulnerable to the security issues described below.  If you depend on Caja for security, either upgrade to version [r5288](https://code.google.com/p/google-caja/source/detail?r=5288) or later, or backport the security patches below.

  1. ES5 mode is insufficiently frozen
  1. Default URI policy sometimes allows javascript: URIs

## ES5 mode is insufficiently frozen ##

Security in ES5 mode depends in part on freezing the base system objects, but a flaw in the freezing process left some objects unfrozen.

If untrusted guest code gets access to one of these unfrozen objects, it might be able to trick trusted code into running arbitrary code outside Caja's security sandbox.

This only affects ES5 mode.  It doesn't affect ES5/3 mode.

  * https://code.google.com/p/google-caja/source/detail?r=5287
  * https://code.google.com/p/google-caja/issues/detail?id=1623
  * https://codereview.appspot.com/7134050/

## Default URI policy sometimes allows javascript: URIs ##

Caja always blocks javascript: URIs, but otherwise allows the host container to whitelist/blacklist URIs in arbitrary ways.

A flaw in Caja's URI handling allows untrusted guest code to sneak javascript: URIs past Caja, and if the host container's URI policy also lets those URIs through, then the untrusted guest can execute arbitrary code in the host context, outside Caja's security sandbox.

This affects both ES5/3 mode and ES5 mode.

  * https://code.google.com/p/google-caja/source/detail?r=5288
  * https://codereview.appspot.com/7173043/