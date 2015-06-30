# Caja Security Advisory 20-Feb-2009 #

Mark Miller of the Google Caja team found a serious flaw in Caja's
JSON implementation, which, if exploited, would allow an
attacker to bypass Caja's sandboxing mechanisms.

This is fixed in version [r3282](https://code.google.com/p/google-caja/source/detail?r=3282) and thereafter.

## Impact ##

On versions of Caja prior to [r3282](https://code.google.com/p/google-caja/source/detail?r=3282) an attacker can completely bypass
Caja's security mechanisms.

## Advice ##

Upgrade to a version of Caja at or after [r3282](https://code.google.com/p/google-caja/source/detail?r=3282).

## More Information ##

See http://code.google.com/p/google-caja/issues/detail?id=978 for
details of the attack and mitigation.

Thanks,

The Google Caja team.