Reported by Gareth

# `DOCTYPES` allow injection of unsanitized markup #

## Effect ##
Unsanitized markup and unsanitized script execution


## Background ##
XML allows [definition of entities](http://www.w3.org/TR/REC-xml/#NT-EntityDecl) via `<!ENTITY ...>` declarations inside a `<!DOCTYPE ...>` block.

`DOCTYPE`s commonly appear in HTML pages since they provide the browser with hints about content-type, character encoding, and whether or not to render in quirks or standards-compliance mode.

Other problems with entity definitions are described at XsrfViaXxe.


## Assumptions ##
Untrusted parties can specify entity declarations to include in the DOCTYPE of an HTML or XHTML file and they can generate markup using those entities.
> or
User defined entities exist and they can be spliced together to construct unsafe markup.


## Versions ##
Firefox and possibly others.


## Example ##
```
<!DOCTYPE html
  PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"
[
  <!ENTITY inject "&#60;script&#62;alert(1)&#60;/script&#62;">
]>
<html xmlns="http://www.w3.org/1999/xhtml">
  <head><title/></head>
  <body>
    &inject;
  </body>
</html>
```