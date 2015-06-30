# Null characters in URL can disguise protocols such as `javascript:`. #

## Effect ##
Unsanitized code can be embedded in comments, and conditional compilation might disable runtime assertions.


## Background ##
RFC 3986 allows the following characters in a URI scheme:
```
scheme      = ALPHA *( ALPHA / DIGIT / "+" / "-" / "." )
```

IE apparently allows, and silently removes, certain characters from URLs.

Collin Jackson reports that unicode code point 65533 is one of those.

The last 4 code points in 32b unicode are:
  * 0xfffc - Object replacement character, a placeholder
  * 0xfffd (65533) - Replacement character, represents an unknown character
  * 0xfffe - Not a character
  * 0xffff - Not a character

http://www.mozillazine.org/talkback.html?article=4078 talks about wider exploits due to null bytes %00 in URLs.



## Assumptions ##
URL html attribute not stripped of null characters,
> OR
URLs not restricted to absolute urls with a whitelisted protocol
> OR
URLs not normalized.


## Versions ##
IE


## Example ##
```
<iframe src="java&#65533;script:alert(42)"></iframe>
```