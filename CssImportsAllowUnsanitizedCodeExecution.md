Reported by Michal Zalewski

# `@import` can import unsanitized CSS #

## Effect ##
Allows import of arbitrary CSS which can execute arbitrary javascript.


## Background ##
In a CSS file `<style>` tag, an `@import` declaration can be used to import styles from an external source specified by a URL relative to the URL of the CSS or HTML file containing the `@import`.

The `data:` protocol can be used to specify inline content in some browsers.


## Assumptions ##
`@import` is allowed in style tags or third party CSS files and is not restricted to known safe CSS.


## Versions ##
All


## Example ##
Expression is IE specific, but the same could be applied to `-moz-binding` in Firefox.  IE 7 does not support `data:` urls.
```
<style>
@import "data:text/css;charset=UTF-8,p%7Bwidth%3Aexpression%28alert%281%29%29%7D";
</style>
```

An `@import` with a HTTP url will also allow importing of unsanitized CSS.

IE has an [undocumented extension](http://www.securityfocus.com/archive/1/372659) that serves as a shorthand for `@import`:
```
    <style><!--@\ "http://www.finjan.com/mcrc/file.css";--></style>
```