# Context Sensitive Keywords #

## Effect ##
Since many keywords are used around variable declaration, differing support for variables can lead to ambiguous parse trees which can lead to different scoping.


## Background ##
Different browsers support different sets of reserved keywords.  E.g. `const` can be used as a variable name in IE, but is used to mark a variable constant in Firefox.


## Assumptions ##
Rendered javascript can contain keywords that have a special meaning in some browsers, and/or rendered output contains newlines.




## Versions ##
IE at least.


## Example ##
```
this['const'] = 0;
const
alert = f();                    // looks like an assignment to self.

function f() { return alert; }  // looks like a reference to an undefined local.

alert('hello world');
```

Since `const` is on a different line than `alert`, IE will insert semicolons and interpret this as
```
this['const'] = 0;               // avoid undefined property error later.
const;                           // const now looks like an unused reference.
alert = f();                     // assigns to self.

function f() { return alert; }   // reference to a global.

alert('hello world');            // call the global function alert
```