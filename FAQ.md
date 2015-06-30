# Compile-time Errors #

**Why do I get a `FATAL_ERROR` about end tags when I use `XHTML`?**

`XHTML` is not very different from `HTML 4.01`.  However, because `XHTML` follows the rules of `XML`, it treats the characters < and & as markup, even if they reside inside a `<script>` block.  This creates a problem when you use these characters in javascript.  Although lenient browser parsers may allow these characters, to ensure your code is interpreted correctly and as you intended on all browsers, Caja requires you to escape these characters.

You can get around this conflict and make all JavaScript code compatible with `XHTML` by placing the JavaScript code within a `CDATA` section. A `CDATA` section `XHTML` starts with the characters `<![CDATA[ and ends with the characters ]]>`.  Alternatively, you can declare your pages to be written in HTML 4.01 by setting the doctype of the page to `<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">`