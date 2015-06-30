# HTML Tags in Javascript Strings can allow Unsanitized Script Execution #

## Effect ##
Unsanitized code can be executed in the global scope


## Background ##
Javascript can be embedded in HTML, or served in separate files.

In HTML, there is one way
```
<script>
// Script proceeds until the string "</script" is seen.
// Strictly it is allowed to end when "</" is seen, but all browsers support the looser
// behavior.
</script>
```
If the comment markers {<!-- ... -->} are seen around the contents of a script tag, they are ignored.  HTML 4 does not actually treat the content of the comment as a comment since the SCRIPT tag's content is specified as CDATA.

In XHTML, a script tag is not always CDATA like HTML, but may be composed from a series of Text nodes and CDATA sections
```
<script>
1 &lt; 2;  // Have to escape HTML special characters
<![CDATA[[
3 < 4;  // Now I don't
]]>
5 &lt; 6;  // Now I do again
</script>
```

http://www.w3.org/TR/1998/REC-xml-19980210#sec-entity-decl describes external entity definitions.
External entities can be used in XHTML to perform javascript imports:
```
<!DOCTYPE ...
  <!ENTITY some-source SYSTEM "my-script.js">
>
<script>
&my-script;
</script>
```


## Assumptions ##
Rewritten javascript allows any of the operators `<`, `<<`, `<<<`, `&`, or `&&` to be followed immediately by an identifier, a slash (`/`), an open square bracket (`[`), or a bang `!`.

OR

The characters `<` and `&` in string literals are output without escaping, and string literals are not split into a concatenation after these characters.


## Versions ##
All


## Example ##
```
'</script><script>alert("hello world");//'
```
when embedded in an HTML page naively.

```
"]]]>&quot;; alert('hello world');//<script>//"
```
when embedded in an XHTML page as
```
<script><![CDATA["]]]>&quot;; alert('hello world');//<script>//"]]></script>
```

```
var script;
1</script />2;
// Can now insert arbitrary html into the document
var iframe, src;
1<script ;
src="foo" > "bar";
```
when embedded in normal HTML.  Note, no tag beginning or end appear inside string literals.

Escaping script enables any number of external entities as described above.