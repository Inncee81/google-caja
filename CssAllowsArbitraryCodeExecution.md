# CSS allows execution of unsanitized javascript? #

## Effect ##
Crafted CSS stylesheets can execute unsanitized javascript in the global scope on some browsers.



## Background ##
CSS includes several mechanisms for changing the surrounding markup and executing expressions.

IE has an extension that allows execution of arbitrary javascript.  The `expression` property is described at http://msdn2.microsoft.com/en-us/library/ms537634.aspx

> Using the power of dynamic properties, it is now possible to declare property values not only as constants, but also as formulas.
> ...
> For scripting, a dynamic property can be any legal JScript or Microsoft Visual Basic Scripting Edition (VBScript) statement.

http://msdn2.microsoft.com/en-us/library/ms533503.aspx
> `binding` allows binding to externally specified scripts

http://developer.mozilla.org/en/docs/CSS:-moz-binding &
http://developer.mozilla.org/en/docs/XBL:XBL_1.0_Reference:Elements#binding
> `-moz-binding` allows binding via the XML interface (also using data: URLs)

The `content` property allows specifying text that is included in the DOM but it is unknown whether this is exploitable if controllable by an attacker.


## Assumptions ##
Untrusted code can generate style elements or style attributes or otherwise add arbitrary CSS rules and create DOM elements that trigger those rules.


## Versions ##
IE 5 and later (but [not IE 8 or later in "standards mode"](http://blogs.msdn.com/ie/archive/2008/10/16/ending-expressions.aspx)).

Mozilla/Firefox, versions not known.


## Example ##
```
<div id='oDiv' style='left:expression(alert("hello"), 0)'>
Example DIV
</div>
```

```
node.style.cssText = 'left:expression(alert("hello"), 0)';
```

```
<input style='-moz-binding: url("http://www.mozilla.org/xbl/htmlBindings.xml#checkbox");'>
```

```
div {
  -moz-binding: url(data:text/xml;charset=utf-8,%3C%3Fxml%20version%3D%221.0%22%3F%3E%0A%3Cbindings%20id%3D%22xbltestBindings%22%20xmlns%3D%22http%3A//www.mozilla.org/xbl%22%3E%0A%20%20%3Cbinding%20id%3D%22xbltest%22%3E%3Ccontent%3EPASS%3C/content%3E%3C/binding%3E%0A%3C/bindings%3E%0A);
}
```

```
node.style.MozBinding = 'url("http://www.mozilla.org/xbl/htmlBindings.xml#checkbox")';
```

```
<ul>
  <li style="behavior:url(a1.htc) url(a2.htc)">List Item</li>
</ul>
```