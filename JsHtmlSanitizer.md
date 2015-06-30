# Introduction #

The Caja project includes a html-sanitizer written in javascript which can be used independently of the cajoler.  You can use it to remove potentially executable javascript from a snippet of html.  To use it, first build html-sanitizer-minified.js by running `ant`.

Use a `<script>` tag to include the resulting `com/google/caja/plugin/html-sanitizer-minified.js` in your program.  To sanitize a snippet of javascript, use the `html_sanitize(htmlSnippet, urlTransformer, nameIdClassTransformer)` to sanitize your html snippet where:

  * `htmlSnippet` is the snippet you want to sanitize
  * `urlTransformer` is a function which is called on every url in `htmlSnippet`.  `javascript:` urls are removed before being passed to the urlTransformer.  The transformer allows you to _whitelist_ urls or rewrite them.  For example, you may only want to allow urls to a particular domain.

  * `nameIdClassTransformer` is a function which is called on every id, name and class in `htmlSnippet`

The return value is the html snippet with all script and style tags removed, and urls, ids, names and classes rewritten according to the transformers.

## Sanitizing CSS ##

The sanitizer removes style tags because they can include code which is interpreted as javascript on some browsers and because styles can affect the entire page, not just the snippet being sanitized.  Style attributes can be safely contained if they are sanitized.  If you'd like to sanitize style attributes (rather than style tags), you can include `com/google/caja/plugin/html-css-sanitizer-minified.js` instead.  This exposes exactly the same api as `html_sanitize` but also allows sanitized css property names and values in style attributes and rewrites any urls in inline styles using the `urlTransformer`.

# Advanced Use #

If you need more control, you can use `html.makeSaxParser` to create your own SAX style processor.  `makeSaxParser` takes as its argument, an object that contains event handlers like:
```
var mySaxParser = html.makeSaxParser(
    {
      startDoc: function (x) { /* called first before processing starts */ },
      startTag: function (tagNameLowerCase, attribs, x) {
        // called on start tags.  may modify attribs.
      },
      endTag: function (tagName, x) {
        // called on end tags.
      },
      pcdata: function (plainText, x) {
        // plainText has entities replaced with the literal value.
      },
      rcdata: function (plainText, x) {
        // contents of a TITLE, TEXTAREA, or similar tag.
      },
      cdata: function (plainText, x) {
        // contents of a SCRIPT, STYLE, XMP, or similar tag.
      },
      endDoc: function (x) {
        // called when processing finished.
      }
    });
```
After this call, `mySaxParser` is a function that takes HTML text and an arbitrary value that will be passed as the parameter `x` to the event handlers above.

# Example #
```
    <script src="html-sanitizer-minified.js"></script>
    <script>
      function urlX(url) { if(/^https?:\/\//.test(url)) { return url }}
      function idX(id) { return id }
      alert(html_sanitize('<b>hello</b><img src="http://asdf"><a href="javascript:alert(0)"><script src="http://dfd"><\/script>', urlX, idX))
    </script>
```