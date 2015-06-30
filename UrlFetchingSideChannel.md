# Side-channels from unproxied connections leak information across closed networks #

## Effect ##

While the same-origin policy in javascript prevents data from a third-party site from being read, the existence of a host, an open http port and resources on that host can be deduced by the errors generated when accessing these hosts.  If an image is loaded from a host, the `onLoad` or `onError` events fire if the image is found or not found respectively.  On the other hand if the host is not found, no events get triggered.  It is possible to use a combination of timeout, onLoad and onError events to build a crude ping.  If an iFrame is used instead of an image, its possible to detect http servers (which will fire onLoad) and distinguish different web servers for example based on what icons are available.

The resolution of hostnames in gadgets happens in the browser.  This
means that by constructing URLs like "http://print", a gadget can
probe the internal network of the viewer of a gadget and work out if for example
print.private.example.com is actually a host or not.

A well configured uri policy protects users using only cajoled gadgets.

There are two parts to this attack which make it effective.  Firstly a
gadget is using short hostnames which get turned into fully qualified
host names according to the settings of the user - this means common
short hostnames like "www" and "print" work for a large number of private LANs
without an attacker having to customize their attack.  Secondly the
fully qualified host name resolves to an IP on the local network and
is accessible to the browser viewing the gadget but not accessible to an attacker outside the private network.

Caja's URI policy should prevent both types of ambient access by
ensuring hostname resolution, DNS resolution and fetching all happen
through a proxy that has no more authority than an attacker.  In
particular, there should be no search domains configured for the proxy
that is used.

## Background ##

Depending on the browser and OS, there are at least
two places where a short hostname gets turned into a fully qualified
host name **before** dns resolution occurs.  Several browsers will automatically
prepend `www` and append alternately `com`, `net` and `edu` to hostnames a user types in the address bar if the original resolution fails.  Unfortunately a similar mechanism also exists for all urls in the page.  If a host in a url fails to resolve, the OS will append each listed entry in a _search domain_ list in turn and try again.  For example, resolving `fudgeroonify.com` (a currently non-existent domain) fails, however, adding thinkfu.com to your search domain list, makes `fudgeroonify.com` resolvable by a browser.  Search domains are only used if the plain host fails to resolve hence `google.com` continues resolve to Google's IP rather than to ThinkFu's IP even though `google.com.thinkfu.com` exists.  Note the primary dns suffix if one is set is used first.

As a result, a machine is vulnerable to spoofing by anyone who can add subdomains to entries listed as the machine's search domains.

The relevant RFCs are 1738 and 2396.  The latter suggests:

> The rightmost
> domain label of a fully qualified domain name will never start with a
> digit, thus syntactically distinguishing domain names from IPv4
> addresses, and may be followed by a single "." if it is necessary to
> distinguish between the complete domain name and any local domain.
> To actually be "Uniform" as a resource locator, a URL hostname should
> be a fully qualified domain name.  In practice, however, the host
> component may be a local domain literal.

Neither browsers nor the OS should modify `fudgeroonify.com.` (with a
trailing dot) no matter what search domains are listed. FF on the mac
does the right thing for `fudgeroonify.com.` but not for `print.` or `www.`
or any other one word short hostname.

Note java's `URI.resolveURI()` also does not append the dot suffix to the
FQDN.  Further, not all webservers are correctly configured to serve
dot suffixed host names - for example `http://cr.yp.to vs
`http://cr.yp.to.`.

## Assumptions ##
Untrusted code is allowed to construct and send requests to arbitrary urls or the proxy is on an internal network.

## Versions ##
Mozilla/Firefox, versions not known.

## Example ##
```
<div>Host found: <div id="host"></div>
<script type="text/javascript">
  var host_found=false;
  function notify() {
    host_found=true;
  }
  function displayMessage() {
    document.getElementById("host").innerHTML = !!host_found ? "Yes" : "Unknown";
  }
</script>
<img src="http://print" onload="notify()" onerror="notify()">
<script>
  setTimeout(displayMessage, 5000);
</script></div>
```