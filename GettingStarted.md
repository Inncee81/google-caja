# How do I use Caja? #

Caja turns a piece of Web content -- roughly, a snippet of HTML, CSS
and JavaScript that you would see within the `body` tag of an HTML
page -- into a Caja _module_. This module is represented as a single
JavaScript _module function_ that can be run within a Caja container.

You can try out some examples running Caja in the CajaPlayground or by visiting http://caja.appspot.com.

Caja is currently used both for securing gadgets and plain webpages.

# Caja for Gadgets #

In practice, the most common place you may need to write in Caja is
when creating Opensocial gadgets.  Some opensocial containers (such as
YAP, Shindig, iGoogle, Code Wiki and Orkut) support Caja.

To turn on caja, you require the caja feature with `<Require feature="caja" />`.

For example, here is a trivial Hello World OpenSocial gadget:

```
<?xml version="1.0" encoding="UTF-8"?>
<Module>
  <ModulePrefs title="Hello World Gadget">
    <Require feature="caja"/>
  </ModulePrefs>
  <Content type="html">
    <![CDATA[
      <div id="message"></div>
      <script type="text/javascript">
        document.getElementById('message').innerHTML = "Hello World";
      </script>
    ]]>
  </Content>
</Module>
```

You can run this gadget by loading it in an opensocial container such as Code Wiki as follows:
```
<wiki:gadget url="http://example.com/gadget.xml" height="200" border="0" /> 
```

# Hosting Caja Gadgets #

If you want to host your own gadgets or user-submitted content, see HostingModules for how to write a host page and RunningCaja for getting the cajoler and Cajita runtime set up.