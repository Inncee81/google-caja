# This page is deprecated.  Please see instructions on https://developers.google.com/caja/ #


The cajoler is written in Java and can be run either from the command line or a web service (of which the CajaPlayground is an instance).

## Running the cajoler yourself ##

### Obtain a Caja build ###

To cajole your input, first build Caja from source.

**Download.** The source is in [SVN](http://code.google.com/p/google-caja/source). We assume you have downloaded it to a directory called:
```
./google-caja
```

**Compile.** Ensure you have version 1.7 of Apache [Ant](http://ant.apache.org/). Then compile as follows:
```
cd google-caja/src
ant jars
```
You should now have a directory called `ant-jars`, relative to your current directory. Go back to your `google-caja` directory:
```
cd ..
```

### Command-line cajoling ###

**Plain HTML.** To cajole a plain HTML+JavaScript snippet, do the following:

```
bin/cajole_html -i <inputHtmlUrl> -o <targetJsFile>
```

where:

> _`<inputHtmlFile>`_ - The absolute URL of the input HTML snippet, such as `file:/Users/example/test.html`.

> _`<targetJsFile>`_ - The file path where you want your JavaScript module function to be stored, such as `./test-cajoled.js`.

`cajole_html` defaults to generating separate HTML and JavaScript files, which is appropriate if you are going to be embedding the HTML in a host page. If you are dynamically loading modules, or if there is no HTML in the input, then you can pass `--only_js_emitted`.

To see other available options, run `bin/cajole_html` with no arguments.

## Cajoler web service ##

The cajoling service is appropriate when you want the client (browser) to request third-party gadgets/modules to be cajoled at runtime (as opposed to choosing the content to be cajoled on your own server). The `cajita-module.js` library works well with the cajoling service.

To start the Cajoler web service, download Caja as above, build it with `ant playground`, then do `ant runserver`. You can then use the service at `http://localhost:8080/cajole`.

The parameters to the service are [somewhat documented](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/service/CajaArguments.java). A sample invocation (line breaks added):
```
  http://caja.appspot.com/cajole
      ?url=http%3A%2F%2Fwww.thinkfu.com%2Ftrivial.html
      &input-mime-type=text/html
      &output-mime-type=application/javascript
```
The `url` is fetched and used as the input. The `input-mime-type` specifies what kind of content to expect there; in particular, it is usually either a HTML file possibly with embedded JavaScript, `text/html`, or a JavaScript file, `application/javascript`.

The `output-mime-type` specifies what kind of content to return. It may be:
  * `application/javascript`; the result is a Caja module (possibly with the effect of generating HTML on load); this is suitable for use with the `cajita-module.js`/HostTools dynamic loading facilities (and is automatically used by HostTools by default).
  * `text/html`; a HTML fragment with `<script>` which is a Caja module. Use this if you want to just embed the gadget in a HTML document you're generating, and have set up the new-module handler in a previous `<script>`.
  * `application/json`; The result contains HTML and JS text as strings in a JSON object with properties `"html"`, `"js"`, and `"messages"`. Use this if you want to handle the HTML and JS separately.