## Background ##

The so-called “Rosetta Flash” vulnerability can occur when a web server allows the attacker to control the first bytes of the response, even if they are limited to being ASCII alphanumeric characters. The response can be made to be interpreted as Flash content, allowing the attacker to execute Flash code in the served origin/domain. See http://miki.it/blog/2014/7/8/abusing-jsonp-with-rosetta-flash/ for more information about the technical details of the vulnerability.

The Caja cajoling service/web proxy servlet can return JSONP responses and therefore is at risk.

## Impact ##

Domains hosting the Caja cajoling service servlet may be vulnerable, resulting in a bypass of the same-origin policy (equivalent to XSS), if the version of Flash in use is older than version 14.0.0.145.

## Advice ##

If you are using any Java servlets provided by Caja, upgrade to a version of Caja at or after [r5698](https://code.google.com/p/google-caja/source/detail?r=5698).

## More Information ##

Discussion of the issue and the changes may be found at:

  * https://code.google.com/p/google-caja/issues/detail?id=1923
  * https://codereview.appspot.com/118640043/
  * https://codereview.appspot.com/117650043/