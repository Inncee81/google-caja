# Caja Security Advisory 23-Jun-2009 #

  1. Felix Lee of Yahoo! found a flaw in Caja's wrappers of the browser DOM API that could allow an attacker to execute arbitrary code with full access to the containing page.
  1. In the process, Mark Miller of Google noted the risk of a known issue whereby an attacker may be able to construct a fake DOM wrapper object and possibly trick Caja into providing them with powerful objects not otherwise provided to sandboxed code. Subsequently, Felix Lee of Yahoo! discovered a method to escalate this into a full breach on Microsoft Internet Explorer versions 6 and 7.

Both vulnerabilities affect Caja version [r3132](https://code.google.com/p/google-caja/source/detail?r=3132) (submitted Dec 12,
2008) or later. They are both fixed in version [r3545](https://code.google.com/p/google-caja/source/detail?r=3545) and thereafter.

## Impact ##

These vulnerabilities allow attacking sandboxed code to completely
bypass all Caja's protections.

## Advice ##

Upgrade to a version of Caja at or after [r3545](https://code.google.com/p/google-caja/source/detail?r=3545).

## More Information ##

See the following issues:

> http://code.google.com/p/google-caja/issues/detail?id=1043

> http://code.google.com/p/google-caja/issues/detail?id=1045

for details of the vulnerabilities.

Thanks,

The Google Caja team.