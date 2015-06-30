The process of cajoling rewrites your html, javascript and css.  This rewriting can have an effect on the size, execution speed and user perceived latency of javascript programs.  While the cajoler tries to minimize the impact on performance, there are some additional steps a developer can take to further improve the performance of their cajoled code.

# Size #

The size of cajoled code is affected by the subset of caja that is used, by the format of the output and its encoding.  For example, a program cajoled in _valija mode_ or in _ES5/3_ must include code to read and set global variables in the outermost scope of the program.  This increases the size of the result cajoled program.  Cajoled code can be render "pretty" for readability and debugging or "minified" for succinctness and it can be `gzip` encoded for delivery to a browser since almost all browsers support receiving compressed content.

&lt;wiki:gadget url="http://www.thinkfu.com/size.xml" height="600" width="100%" border="0" /&gt;

For least network latency, a cajoled program should be rendered "minified" and delivered gzip-encoded.  If the effects of a javascript program are not expected to affect the rest of the page, a developer can avoid incurring the cost of reading and setting globals by wrapping their program in a function wrapper:

```
(function () {
  ...your program here...
})();
```

# Performance #

There are several aspects of performance that are important to someone using caja.  There is the time taken to cajole an application or gadget, the time it takes for the browser to parse the contents of the gadget, the time before the user first sees the gadget get rendered and the running time of the javascript in the gadget.  Some of these performance questions can be addressed outside of the cajoling pipeline.  For example, since the code in a gadget does not change frequently, cajoled code can be cached and thus reduce the cost of cajoling code.

## Run time Performance ##

The runtime performance of cajoled code on the client (for example in a browser) is affected primarily by the mode in which input is cajoled, the subset of caja that is used and by the use of globals.  The semantics of HTML require that variables created and set in one `<script>` block must be accessible to subsequent `<script>` blocks.  To support this legacy behavior, all opeations on global variables result in expensive read and set operations in code cajoled in _valija mode_.

There are also differences in the optimization and performance in the javascript engines used by browsers.  The following measurements were made using Rhino 1.6.7.

&lt;wiki:gadget url="http://www.thinkfu.com/time.xml?1" height="600" width="100%" border="0" /&gt;

For least runtime latency, Caja programs should be authored in ES5-strict and cajoled using ES5/3.

## Latency of initial rendering ##

Here we compare the rendering time for a page which consists of nine individual gadgets separated using iframes with a page with the same nine gadgets cajoled and inlined on the same page.

The test measures how long it takes before the `onload` event of each iframe has run and compares it to how long after a cajoled gadget run, before its `onload` event runs.

&lt;wiki:gadget url="http://www.thinkfu.com/malicious-uncajoled-url.xml" width="100%" height="350" border="0" title="Uncajoled gadgets running in iFrames" /&gt;

&lt;wiki:gadget url="http://www.thinkfu.com/malicious-cajoled-url.xml" height="350" width="100%" border="0" title="Cajoled gadgets running in inlined"  /&gt;
