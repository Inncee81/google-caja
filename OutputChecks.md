# Output Checks #

The HTML, CSS, and Javascript that we output should be as clear as simple as possible to make sure that browsers produce the same parse tree.

Below are some properties that we can assert on output.


## Intentional Newlines ##

Our source code formatter should not output any non-space tokens containing any of the characters listed in http://en.wikipedia.org/wiki/Newline
  * LF:    Line Feed, U+000A
  * CR:    Carriage Return, U+000D
  * CR+LF: CR followed by LF, U+000D followed by U+000A
  * NEL:   Next Line, U+0085
  * FF:    Form Feed, U+000C
  * LS:    Line Separator, U+2028
  * PS:    Paragraph Separator, U+2029


## Comment Free ##

We should strip all comments from the output to avoid lexing inconsistencies.  Known lexical errors in existing browsers include:
  * IE 6 allows nested block comments in CSS
  * IE allows conditional compilation comments in HTML and JS.


## String Literals should not appear to be markup or external entity references or CDATA ends ##

We should not allow `<script>` inside a string literal, since if malicious code can trick the rewriter into outputting a `</script>`, it can open a new script tag whose content starts inside what the browser thinks is a safe string constant.

Other problems arise with entity references.  If malicious code can escape a script tag, it can insert doctypes, and load external scripts.

If malicious code can escape a CDATA section in XHTML then it might be able to insert tags into the page.

All of these problems are avoided if the `<`, `<<`, `<<<`, `&`, and `&&` operators are always followed by space, and if the characters `<` and `&` are replaced with their octal equivalents (`\074` and `\046`) in string literals.



## ASCII identifiers ##
We should disallow non-ASCII identifiers until we understand browser support for identifiers, and identifier normalization.

We should also produce ASCII only output until we have an idea of the ways in which containers inline cajoled output and the encodings they use.  Ideally, we will always ship cajoled output in UTF-8 and recommend that containers only inline cajoled code in pages that are UTF-8 encoded.