# Object.eval allows execution of unsanitized code on Firefox. #

## Effect ##
Execution of arbitrary code.



## Background ##
According to http://developer.mozilla.org/en/docs/Core_JavaScript_1.5_Reference:Global_Objects:Object, Object.eval is a deprecated method of all Objects.



## Assumptions ##
Any object is accessible, and the eval property is accessible.


## Versions ##
Firefox <= 2.0.0.12.  Fixed in https://bugzilla.mozilla.org/show_bug.cgi?id=382509


## Example ##
```
({}).eval('alert("Your cookie is " + document.cookie)')
```