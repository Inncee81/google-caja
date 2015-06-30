# Caja Security Advisory 7-July-2009 #

The second hole documented in our [previous security advisory](SecurityAdvisory20090623.md):

> noted the risk of a known issue whereby an attacker may be able to construct a fake DOM wrapper object and possibly trick Caja into providing them with powerful objects not otherwise provided to sandboxed code. Subsequently, Felix Lee of Yahoo! discovered a method to escalate this into a full breach on Microsoft Internet Explorer versions 6 and 7.

was not successfully closed. The underlying problem is that Domita contains constructors whose purpose is to be used **internally** to construct tamed wrappers around DOM nodes, and to be available to cajoled code for use in type-testing those wrappers. However, by having access to these constructors themselves, cajoled code could call these constructors in ways that violate their assumptions.

http://code.google.com/p/google-caja/issues/detail?id=1065 explains how some constructors were still accessible. In that issue thread, Ihab demonstrated that Felix's arbitrary code execution exploit was still feasible.

This remaining vulnerability affects Caja version [r3545](https://code.google.com/p/google-caja/source/detail?r=3545) (submitted 23 Jun 2009) or later. They are both fixed in version [r3557](https://code.google.com/p/google-caja/source/detail?r=3557) and thereafter.

## Impact ##

These vulnerabilities allow attacking sandboxed code to completely bypass all Caja's protections.

## Advice ##

Upgrade to a version of Caja at or after [r3557](https://code.google.com/p/google-caja/source/detail?r=3557).

## More Information ##

See the following issue:

http://code.google.com/p/google-caja/issues/detail?id=1065