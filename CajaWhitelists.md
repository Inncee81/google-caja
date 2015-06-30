# Whitelist File Format and Schema #

## Background ##

Caja uses white-lists to approve HTML tags, HTML attributes, and CSS properties.  These white-lists were hard-coded in java files.

The white-list is a table of white-listed item.  Each row in the table includes a key (variously an HTML element name, HTML attribute name, or CSS property name), and some information about the item's content.

Although we believe the default white-lists are fairly comprehensive, clients that do their own preprocessing of HTML & CSS before cajoling may want to white-list constructs which they know are safe, but which Caja cannot prove safe for arbitrary input.

It may become necessary to support different element definitions for HTML4 vs XHTML.


## Builtin WhiteLists ##
  * [resource:///com/google/caja/html/html4-elements.json](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/lang/html/html4-elements.json) [defs](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/lang/html/html4-elements-defs.json) [whitelist](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/lang/html/html4-elements-whitelist.json)
  * [resource:///com/google/caja/html/html4-attributes.json](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/lang/html/html4-attributes.json) [defs](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/lang/html/html4-attributes-defs.json) [whitelist](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/lang/html/html4-attributes-whitelist.json)
  * [resource:///com/google/caja/lang/css/css21.json](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/lang/css/css21.json) [defs](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/lang/css/css21-defs.json) [whitelist](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/lang/css/css21-whitelist.json)





## WhiteList File Format Overview ##

A white-list is a [JSON file](http://www.json.org/) like:
```
{
  "description":
      "Extends the default HTML tag WhiteList to allow <OBJECT> and deny <IFRAME>",

  "inherits": [
    { "src": "resource:///com/google/caja/html/tags.json" }
  ],

  "allowed": [
    { "key": "OBJECT" }
  ],

  "denied": [
    { "key": "IFRAME" }
  ],

  "types": [
    { "key": "OBJECT", "optionalEnd": false, "empty": false }
  ]
}
```

We use JSON, because, unlike XML, parsing it cannot open arbitrary network connections, and it is more extensible since it doesn't make a hard distinction between elements, which can be extended, and attributes, which can't.


## Elements of a WhiteList ##

The example above includes four elements, `inherits`, `allowed`, `denied`, and `types`.

The cajoler examines the `inherits` and loads those.  The `inherits` `src` can be either a `file://...` URL, or a special {{{resource://...} URL which is resolved relative to the Cajoler's class-path.

A WhiteList has the form
```
    interface WhiteList {{{
      Set<String> allowedItems();
      Map<String, TypeDefinition> typeDefinitions();

      interface TypeDefinition {
        Object get(String key);
      }
    }
```

The cajoler loads a white-list using the following algorithm:
  * Create an empty white-list W
  * For each `inherits`
    * Fetch its URL -- Abort on failure
    * Load it using this algorithm
    * Add it to the list of loaded white-lists
  * For each loaded white-list LW
    * Add LW's `allowed` to W
    * Add LW's `types` to W
  * For each loaded white-list LW
    * Remove any items in W `denied` in LW's
  * For each `allowed`
    * Add an item to W.
  * For each `denied`
    * Remove any item in W with the same key.
  * For each `types`
    * Remove any type definition from W with the same key.
  * For each `types`
    * Add a type definition to W
  * If there are type definitions in W with the same key, and the same value, remove all but 1.
  * If there exist any two distinct type definitions in W with the same key, mark W invalid.
  * Return W.

This algorithm preserves the properties that
  * Any item not mentioned in either the start white-list or in inherited white-lists is **denied**.
  * Any item denied by a start is denied.
  * Any item allowed by the start white-list and not also denied is **allowed**.
  * Any item denied by an inherited white-list that is not allowed by the start white-list is denied.
  * Any item allowed by an inherited white-list and not denied by any inherited white-list or by the start white-list is allowed.
  * If the start white-list defines a type for an item, then that type is used for the item.
  * If the start white-list defines an ambiguous type, then processing aborts.
  * If the start white-list defines a type for an item, and inherited white-lists define ambiguous types for an item, then the start white-list's definition is used, and the start white-list is valid.

This allows white-lists to be used in several distinct ways:
  * As a schema -- a white-list that includes only type definitions and neither allows nor denies items may be inherited for its definitions.
  * As a white-list -- a white-list may inherit definitions from a schema and then pick and choose items to allow.
  * As an override -- a white-list may inherit types and definitions from other white-lists and then choose to override allows and definitions made in the inherited lists.

## Type information in WhiteLists ##
For HTML Elements, we need the following type information:
  * `empty` - does the element not contain any other elements?  `<INPUT>`, `<BR>`, and `<HR>` are examples of empty elements in HTML 4.
  * `optionalEnd` - does the element require an end tag?  `<P>` elements do not require end tags in HTML 4, but `<TABLE>` elements do.

For HTML Attributes, we need the following type information:
  * `type` - one of the values in `com.google.caja.html.HTML.Type`: `NONE`, `SCRIPT`, `STYLE`, `URI`, `ID`, `IDREF`, `NMTOKEN`, `NMTOKENS`, or `FRAME`.

For CSS Properties, we need the following type information:
  * `signature` - a property signature as described and "Value" in the [CSS2.1 spec](http://www.w3.org/TR/CSS21/about.html#property-defs)



## Specifying WhiteLists at the Command Line ##
From usage:
```
 --css_prop_schema         A file: or resource: URI
                           of the CSS Property Whitelist to use.
 --html_attrib_schema      A file: or resource: URI
                           of the HTML attribute Whitelist to use.
 --html_property_schema    A file: or resource: URI
                           of the HTML element Whitelist to use.
```