# Caja Security Advisory 2013-11-21 #

## Background ##

JavaScript parsers differ on whether they interpret escaped sequences of letters spelling a reserved word, such as "de\u006Cete", as an identifier or a reserved word.

This can result in Caja and the browser having different notions of how a specific program parses; additionally, Caja's code generator would take the parse tree of such a program and emit text which did not have the same interpretation when parsed.

## Impact ##

No specific exploits of this inconsistency are known, but we feel that the risk that one which leads to unsandboxed code execution might be possible is significant.

## Advice ##

Upgrade to a version of Caja at or after [r5632](https://code.google.com/p/google-caja/source/detail?r=5632).

## More Information ##

This issue was originally reported at:

> https://code.google.com/p/google-caja/issues/detail?id=1867

Discussion of the change is at:

> https://codereview.appspot.com/19560044/

The effect of the change is to reject all programs which contain the problematic escapes. This conservative policy will likely be in place until such time as all supported browsers conform to the ECMAScript specification in their interpretation of such programs.