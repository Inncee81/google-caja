# Browser History Mining #

## Effect ##
Malicious code can check whether a user has visited certain websites.  If malicious code wanted to target a phishing attack, they could figure out which bank websites a user has visited, and make their phishing attacks more plausible.

## Background ##
The browser keeps track of which URLs users visit, and uses this information for URL bar auto-completion, and to color links differently.

CSS2.1 and later allow webpages control of link styling.  This affects all links not just links to sites in the same origin.
```
  a:visited { color: purple }
  a:link { color: blue }
```
This can be extended to not only affect color/voice/volume but also affect layout:
```
  a:visited { font-size: 300%; display: block }
  a:link { font-size: 50%; display: inline }
```

JavaScript can inspect both computed styles via `window.getComputedStyle` and can inspect layout information via `HTMLElement.offsetLeft`, `offsetWidth`, etc.

Some CSS styles like `color` and various aural properties used by screen readers are commonly used to differentiate visited and unvisited links and the inability to use those styles might cause significant usability problems.  Indeed, users may use link styles to descide whether a link goes to a site that they trust.

[This variation](https://bugzilla.mozilla.org/show_bug.cgi?id=147777#c132) is not currently prevented by Caja:
<blockquote>
Let's say a web page shows N hyperlinks that all say "Click here to continue." The unvisited links are styled to blend in with the background so the user can't see them. The visited links are visible because of the visited link styling, so the user only see the visited ones. Then the attacker can find out where the user's been by which link they click on.<br>
</blockquote>

## Assumptions ##
Untrusted code can specify a link to a target URL
AND (untrusted code can specify a URL to a site they control in a CSS style that is loaded contingent on whether that target URL is visited
> OR (untrusted code can make the layout of the page differ depending on whether the target URL was visited)
> OR (untrusted code can access the computed style of a CSS property that differs between visited and unvisited links).

## Versions ##
All

## Example ##
See http://jeremiahgrossman.blogspot.com/2006/08/i-know-where-youve-been.html for examples.