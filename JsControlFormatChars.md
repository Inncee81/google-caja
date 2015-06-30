# Control Format Characters #

## Effect ##
Can confuse the lexer causing executable code to be interpreted as the content of strings or comments, allowing an attacker to slip arbitrary unsafe code past a rewriter  or verifier.


## Background ##
According to EcmaScript
### 7.1 Unicode Format-Control Characters ###
> The Unicode format-control characters (i.e., the characters in category "Cf" in the Unicode Character Database such as LEFT-TO-RIGHT MARK or RIGHT-TO-LEFT MARK) are control codes used to control the formatting of a range of text in the absence of higher-level protocols for this (such as mark-up languages).
> It is useful to allow these in source text to facilitate editing and display.
> The format control characters can occur anywhere in the source text of an ECMAScript program. These characters are removed from the source text before applying the lexical grammar. Since these characters are removed before processing string and regular expression literals, one must use a. Unicode escape sequence (see 7.6) to include a Unicode format-control character inside a string or regular expression literal.

The latest version of ICU specifies 0xad, 0x600-0x603, 0x70f,  0x17b4-0x17b5, 0x200c-0x200f, 0x202a-0x202e, 0x2060-0x206f, 0xfeff, 0xfff9-0xfffb, 0x1d173-0x1d17a, 0xe0001, 0xe0020-0xe007f as `[:Cf:]`.

~~Both ES3.1 and ES4.0 are making sure that all codepoints allowed in string and regular expression literals are treated as significant, though as of this writing it is unclear whether they will ignore the default ignorable set outside literals.~~ The ES5 draft treat all format control characters as lexically significant.  It redefines a few codepoints (e.g. the BOM) as whitespace, but these are still lexically significant, not ignored.



## Assumptions ##
The Javascript lexer does not error out on input that contains `[:Cf:]` chars in javascript or a parse tree does not strip or escape `[:Cf:]` characters.

Note: it is <b>not</b> sufficient to strip `[:Cf:]` chars before parsing.



## Versions ##
Firefox strips codepoints 0x200c-0x200f, 0x202a-0x202e, and 0x206a-0x206f.  IE, Safari, and Opera do not, but strip some control characters or characters in the 0xff?? codepage.



## Example ##
In the below, Firefox and any standards compliant interpreter, strips the &#x200D; out before lexing, so it sees `/&#x200D;/` is interpreted as a line comment.  The line comment defuses the block comment, so the `alert('hi');` is treated as real code.
```
<html>
  <body onload="

      /&#x200D;/.test(''); /*
      alert('hi');
      // */

      ">
  </body>
</html>
```

The below cobines the first trick with another to produce javascript that alerts one way on interpreters that strip `[:Cf:]`, and another on interpreters that don't.
```
<html>
  <body onload="

      /&#x200D;*/.test(''); alert('Does not Strip [:Cf:]'); /**/
      /**&#x200D;/ alert('Strips [:Cf:]'); /**/

      ">
  </body>
</html>
```

It is probably posible to hide code in strings by putting a `[:Cf:]` char between a backslash (\) and a quote ("), to construct a case that works on all browsers, but I have not done so.