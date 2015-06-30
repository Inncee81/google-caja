# CSS Templates #

DOMita provides a way to manipulate the DOM, but it lacks a way to
manipulate CSS.

We can build out a tamed ElementStyle API, but parsing and sanitizing
CSS is much more difficult than HTML, since it is a much larger
language.

Since we already have a full server-side CSS parser, I propose we
resurrects the CSS Templating scheme that was built into the prototype
Cajoler.


## Existing CSS Template Syntax ##
The existing CSS Template syntax uses annotations to specify

```
  @template("myTemplate");
  @param("x0");
  @param("y0");
  @param("x1");
  @param("y1");

  position: absolute;
  left:     $(x0)px;
  top:      $(y0)px;
  width:    $(x1 - x0)px;
  height:   $(y1 - y0)px;
```

This scheme adds one production to the CSS 2.1 grammar:
```
  substitution  ::==  '$' '(' <js-body>* ')' <unit>?
  js-body       ::==  '(' <js-body> ')'
                   |  <js-token-not-parenthesis>
```
that can be used in place of any literal, where `<unit>` is any
numeric suffix like "cm", "in", "px", etc.

The CSS Template parser sees this, and runs the CSSValidator as usual
to mark each primitive value or substitution with a
ContentTypeProperty (e.g. left), and a ContentType (e.g. LENGTH).

The CSSRewriter compiles the template into a javascript function:
```
  function myTemplate(x0, y0, x1, y1) {
    return blessCss___(
        'position', 'absolute',
        'left', cssNumber___(x0) + 'px',
        'top', cssNumber___(y0) + 'px',
        'width', cssNumber___(x1 - x0) + 'px',
        'height: ', cssNumber___(y1 - y0) + 'px');
  }
```
that returns an immutable set of property keys and values that can be treated soecially by privileged DOM manipulation code, thus moving runtime checks to the compiler.



## Updated Syntax ##

To be more consistent with string interpolation syntax, we'll change
the substitution production to use curly brackets instead of
parentheses:
```
  substitution  ::==  '$' '{' <js-body>* '}' <unit>?
  js-body       ::==  '{' <js-body> '}'
                   |  <js-token-not-curlies>
```
and rewrite the following syntax:
```
  myHtmlElement.setStyle(eval(Template(
      "position: absolute; left: ${x0}px; top: ${y0}px;"
      + " width: ${x1-x0}px; height: ${y1-y0}px", content_type.CSS)));
```
to something that binds x0, y0, x1, and y1 in the local scope.


## Style Deltas ##

Many users will want to make incremental changes to style.  We can
use he list of key value pairs above to modify only the style properties
mentioned when caja code does
```
myHtmlElement.updateStyle(...);
```