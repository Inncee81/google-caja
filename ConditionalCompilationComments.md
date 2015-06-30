Reported by futurama

# Conditional compilation may allow disabling of runtime checks. #

## Effect ##
Unsanitized code can be embedded in comments, and conditional compilation might disable runtime assertions.


## Background ##
IE contains a non-standard javascript extension that embeds code in comments.

According to http://msdn2.microsoft.com/en-us/library/121hztk3.aspx
> Conditional compilation allows the use of new JScript language features without sacrificing compatibility with older versions that do not support the features.

> Conditional compilation is activated by using the `@cc_on` statement, or using an `@if` or `@set` statement. Some typical uses for conditional compilation include using new features in JScript, embedding debugging support into a script, and tracing code execution.

> Always place conditional compilation code in comments, so that hosts (like Netscape Navigator) that do not understand conditional compilation will ignore it. Here is an example.

According to http://devedge-temp.mozilla.org/viewsource/2003/venkman/01/index_en.html

> The `//@JSD_EVAL` command will insert a breakpoint which is set
> to execute the script that follows without stopping and without logging the result.


## Assumptions ##
Rewritten source code includes comments without sanitizing them to remove conditioal compilation code OR verified code allows comments with conditional compilation commands OR conditional compilation is supported outside comments without being rewritten into equivalent javascript control structures.


## Versions ##
IE, Firefox w/ Venkman


## Example ##
```
/*@cc_on @*/ /*@if (1) alert(document.cookie) @end @*/
```

```
//@JSD_EVAL alert(document.cookie);
```

And CC can change tokenization arbitrarily far from the CC directive.
```
x /*@cc_on =*/ ++ /a/i.x
```
is interpreted as
```
x = ++((new RegExp('a', 'i')).x)
```
by IE's JScript interpreter but as
```
(x++) / a / (i.x)
```
on other interpreters.