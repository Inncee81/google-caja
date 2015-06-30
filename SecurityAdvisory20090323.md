# Caja Security Advisory 23-Mar-2009 #

Ben Laurie of the Google Caja team found a serious flaw in Caja's source code translator, which would allow an attacker to bypass Caja's sandboxing mechanisms.

This is fixed in version [r3349](https://code.google.com/p/google-caja/source/detail?r=3349) and thereafter.

## Impact ##

On versions of Caja prior to [r3349](https://code.google.com/p/google-caja/source/detail?r=3349) an attacker can completely bypass
Caja's security mechanisms.

## Advice ##

Upgrade to a version of Caja at or after [r3349](https://code.google.com/p/google-caja/source/detail?r=3349).

## More Information ##

See http://code.google.com/p/google-caja/issues/detail?id=1002 for
details of the attack and mitigation.

Thanks,

The Google Caja team.