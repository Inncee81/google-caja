## Effect ##
Code that can control an `ID` or `NAME` can replace a member with DOM2 semantics with a completely different object.


## Background ##
DOM Level 2 defines a set of members that DOM nodes, node lists, etc. must implement.  Those members are relied on by code, so e.g. `document.forms[0].elements.length` is an integer describing the number of input elements in the first form on the page.

When an `HtmlCollection` contains a member with an `ID` or `NAME`, the collection has a member with that id or name.

Some browsers' DOM nodes' methods cannot be applied across instances ; `document.createElement('FORM').reset.call(document.forms[0])` fails with an exception on IE.  For properties that are not methods, there's no way to access the original version.  On Firefox, the `elements` array cannot be accessed without a screen rerender since removing the input named `elements` does not unmask `HTMLFormElements.elements` right away.



## Assumptions ##
Untrusted code can specify an ID or NAME that conflicts with a special member used by trusted code.



## Versions ##
IE and Opera allows masking of `length`.  Safari aliases IDs and NAMEs that look like array indices >= length.  Firefox, Safari, and possibly others allow masking of properties and methods on an `HTMLFormElement` such as `reset`.



## Example ##
```
    <form>
      <input name=length id=length>
      <input name=5 id=5>
      <input name=0 id=0>
      <input name=elements id=elements>
      <input name=reset id=reset>
    </form>
```