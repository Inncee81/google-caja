# ActiveXObject, document.createElement, document allow arbitrary code execution #

## Effect ##
Execute unsanitized code in the global context.


## Background ##
By creating a script tag, or setting the src of an existing script tag, untrusted code can cause the browser to load an execute javascript.

By accessing an ActiveXObject or plugin, untrusted code might be able to escape the bounds of a normal webpage to access the file system and devices.

http://www.w3.org/TR/REC-DOM-Level-1/ecma-script-language-binding.html defines the Node and Document interfaces.

All browsers define a document property of the global object which implements the Document interface and so allows creating of script tags.

Many document objects also include nodes that correspond to plugins.

The ActiveXObject constructor allows creation of ActiveXObjects which allow interaction with the operating system on IE.

The document node is accessible from any DOM node via the parent property.


## Assumptions ##
Untrusted code can access ActiveXObject or document.createElement, or any DOM element.



## Versions ##
All browsers.



## Example ##
```
var script = document.createElement('script');
script.appendChild(
    document.createTextNode(
        'alert("Your cookie = " + document.cookie)'));
document.body.appendChild(script);
```