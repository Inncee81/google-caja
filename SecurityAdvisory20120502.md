## Background ##

[Revision 4469](https://code.google.com/p/google-caja/source/detail?r=4469) of Caja, at http://code.google.com/p/google-caja/source/detail?r=4469, introduced a change to the manner in which URLs in CSS were being rewritten by the container-specified URI Policy. Due to an oversight, URLs written inside the "style" property of HTML elements, such as the following:

```
<div style="background-image: url(http://example.com/foo.jpg)">
  ...
</div>
```

were no longer being rewritten by either the client or the server side URI Policy. This means that an attacker could introduce URLs of their choosing in style sheet declarations within a "style" attribute of an HTML element.

## Impact ##

An attacker could take advantage of this vulnerability if the client was running older versions of certain Web browsers by specifying a "javascript:" URL, as in:

```
<div style="background-image: url(javascript:alert(1);)">
  ...
</div>
```

For more information on the use of "javascript:" URLs in CSS, please see the following references:

http://dithered.chadlindstrom.ca/css_filters/nonvalidating/background_javascript_url.html
https://bugzilla.mozilla.org/show_bug.cgi?id=230134

Since use of "javascript:" URLs in CSS is a known security problem, all modern browsers disallow this construct, so even without upgrading Caja, updating to the latest version of a user's chosen browser should protect Caja users against the vulnerability.

## Advice ##

Upgrade to a version of Caja at or after [r4865](https://code.google.com/p/google-caja/source/detail?r=4865).

## More Information ##

The issue was originally reported at [issue 1459](https://code.google.com/p/google-caja/issues/detail?id=1459), at http://code.google.com/p/google-caja/issues/detail?id=1459.

Discussion of the change is available at http://codereview.appspot.com/6094044/.