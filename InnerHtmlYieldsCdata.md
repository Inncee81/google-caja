# xml, plaintext, and other elements' innerHTML cannot be safely inserted into another element's innerHTML #

## Effect ##
If an implementation allows injecting of innerHTML from one element into another, the deprecated XMP and LISTING tags' content is treated as CDATA, and so their innerHTML is not escaped as normal.


## Background ##
http://www.freesoft.org/CIE/RFC/1866/37.htm describes the XMP and LISTING tags' content as preformatted text.  The HTML 3.2 spec defines their content as %literal.

The full list as defined in [HTML3.2](http://www.w3.org/TR/REC-html32) is:
  * listing
  * plaintext
  * xmp

Also problematic are
  * option
  * script
  * style
  * textarea
  * title
which allow no tag content, so many browsers interpret what appears to be tag content as PCDATA.


## Assumptions ##
The document is an HTML document, not XHTML.

An implementation allows innerHTML to be extracted from one DOM node to be injected into another without rewriting.



## Versions ##
All


## Example ##
```
// Does not require programmatic creation of a script tag
var xmp = document.createElement('xmp');
xmp.appendChild(
    document.createTextNode(
        "<script>alert('cookie=' + document.cookie)</script>"));
// If the implementation assumes this is safe
var html = xmp.innerHTML;
// then it might allow this
document.writeln(html);
```