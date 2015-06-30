# Code on this page uses a deprecated api #


# Introduction #

The corkboard demo is an example of using Caja to sandbox, not explicitly authored "gadgets", but rather whatever random HTML a user of your site types in (e.g. in blog comments, forum posts, user profiles, ...).

It is intended to be a very basic, easy to emulate example, and demonstrate a style of Caja integration which is completely independent of your server platform.

The corkboard is written in Python and runs on Google App Engine; in fact, it is directly derived from the [Google App Engine Python tutorial](http://code.google.com/appengine/docs/python/gettingstarted/).

# Source #

The source may be found at [src/com/google/caja/demos/corkboard/](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/demos/corkboard/).

# Getting here from there #

If you'd like to follow along with the original invention of the corkboard demo, here's how!

## Getting started ##

First, do the [Google App Engine Python tutorial](http://code.google.com/appengine/docs/python/gettingstarted/), which produces a “guestbook” site. As it stands, each guestbook Greeting has some content, which is treated as plain text, and therefore escaped for insertion in the HTML document; that's `{{ greeting.content|escape }}`. What we want to do instead is cajole this content and then insert the cajoled HTML.

## Adding the cajoler ##

All of the talking-to-the-cajoler logic is in [cajole.py](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/demos/corkboard/cajole.py) in the corkboard demo. This module (which stands alone except for some easily-replaced dependencies on App Engine, so you could drop it into your own application if you want) provides a function `cajole(html)` which returns a dict:

```
{
  "html": "...cajoled HTML...",
  "js": "...cajoled JS module...",
  "log: ...cajoler's report...
  "error": (is True if there was an error, else missing)
}
```

So all you have to do to get HTML working is feed `cajole(greeting.content)` in as a template variable, say `cajoled`, and embed `{{ cajoled.html }}` — note the lack of `|escape`. (In the corkboard demo we instead add a method `cajole()` to `Posting` (our version of `Greeting`) which does the cajoling, so we write `{{ posting.cajole.html }}`.)

In order to make sure the HTML's styles can't place things elsewhere on the page, two steps are needed. First, add `class="vdoc-body___"` to the `<blockquote>` around the content. Then link to the Caja gadget stylesheet: `<link rel="stylesheet" href="http://caja.appspot.com/caja-gadget.css">`.

> _(If this is a production site, please use your own server instead; this goes for everywhere `caja.appspot.com` is mentioned. In the corkboard demo all such mentions are replaced by a variable `cajaServer`.)_

This takes care of the HTML. You could stop here if you liked, and have a perfectly good HTML sanitizer for your user-generated content. But it's overkill (you can use the `html-sanitizer` JS library we provide instead) unless we're going to support JavaScript too.

## Adding JavaScript ##

First, link to the Caja loader:
```
<script src="http://caja.appspot.com/caja.js" type="text/javascript"></script>
```

Now, we have a bit of a problem. For reasons which are out of the scope of this discussion, the Caja runtime will not be available immediately, but is deferred until after the page loads and another network roundtrip — so we can't just take `cajole(greeting.content)["js"]` and stuff it in a `<script>` block. Also, the cajoled code is actually evaluated in a hidden `<iframe>` rather than the host page. So, instead, each script is embedded as a string and stored for later evaluation.

```
    <blockquote class="vdoc-body___"
                id="gc-{{ forloop.counter }}">
      {{ greeting.cajole.html }}
    </blockquote>
    <script type="text/javascript">
      registerForScript("gc-{{ forloop.counter }}",
        "{{ greeting.cajole.js|escapejs|escape }}"
      );
    </script>
```

Each greeting is given a separate id (using the loop counter provided by the template system), which is used to associate the script with the element later. (TODO(kpreid): |escape is wrong here since the content is CDATA. Might not matter. Should fix app.)

`registerForScript` is defined in the file `[static/embedded-scripts.js](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/demos/corkboard/static/embedded-scripts.js), which both loads the Caja runtime and attaches the embedded scripts to the elements. This is its source:

```
var registerForScript, loadScripts;
(function () {
  var scriptHooks = [];

  registerForScript = function (vdocId, moduleText) {
    scriptHooks.push([vdocId, moduleText]);
  }

  function go(caja) {
    for (var i = 0; i < scriptHooks.length; i++) {
      var id         = scriptHooks[i][0];
      var moduleText = scriptHooks[i][1];
      var sandbox = new caja.hostTools.Sandbox();
      sandbox.attach(document.getElementById(id));
      sandbox.runCajoledModuleString(moduleText);
    }
    scriptHooks = [];
  }
  
  loadScripts = function (server) {
    loadCaja(go, {
      debug: true,
      cajaServer: server
    });
  }
})();
```
(Of course, set `debug` to `false` for production code.) It needs to be invoked when the page is loaded, and this is done by an `onload` in the page:
```
<body onload="loadScripts('http://caja.appspot.com/')">
```

And that's it!