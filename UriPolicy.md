# URI Policies #

## Goals ##

Allow server side rewriter to inline content, rewrite URLs, and the client-side runtime JS to rewrite URLs used in HTML attributes and passed to DOM APIs.

Many containers want to pass URLs through proxies that strip cookies, and verify, rewrite, or re-encode content.  These proxies will also check that the advertised mime-type matches the kind requested, so that if a URL appears in an image's `src` attribute, and is a JS/GIF polyglot, it must have an image
mime-type.

Since the URI Policy lies at the border between Caja and the container, errors in it can compromise both.  It is a goal for policies to be conservatively backwards compatible -- if a policy denies a URL in a certain context, then changes to Caja or to the URI policy interface should not make existing policies more permissive.


## Background ##

#### How can a URI policy implementation distinguish between URLs that are loaded without further user action, and distinguish the expected type of content? ####

URLs, URIs, and URNs appear in HTML attributes as parts of CSS
property values, and as arguments to Javascript APIs.

<table cellpadding='3' border='1' cellspacing='1'>
<tr>
<th>Source</th>
<th>Expected Type</th>
<th>Notes</th>
</tr>
<tr><th>In CSS</th></tr>
<tr>
<td><code>background-image: url(...)</code>, etc.</td>
<td><code>image/*</code></td>
<td>Image loaded in page</td>
</tr>
<tr>
<td><code>content: url(...)</code></td>
<td><code>text/plain</code></td>
<td>Immediately loaded textual content added to affected elements or their parent</td>
</tr>
<tr>
<td><code>cue-after: url(...)</code>, etc.</td>
<td><code>audio/*</code></td>
<td>Audio played before or after activation of affected element</td>
</tr>
<tr>
<td><code>cursor: url(...)</code></td>
<td><code>image/*</code></td>
<td>Immediately loaded cursor bitmap</td>
</tr>
<tr><th>In HTML</th></tr>
<tr>
<td><code>&lt;a href&gt;</code>, <code>&lt;area href&gt;</code></td>
<td><code>*/*</code></td>
<td>Link to external content not loaded until navigation</td>
</tr>
<tr>
<td><code>&lt;base href&gt;</code></td>
<td><code>*/*</code></td>
<td>Base URL of relative</td>
</tr>
<tr>
<td><code>&lt;body background&gt;</code></td>
<td><code>image/*</code></td>
<td>Image loaded in page</td>
</tr>
<tr>
<td><code>&lt;form action&gt;</code></td>
<td><code>*/*</code></td>
<td>Link to external content not loaded until navigation</td>
</tr>
<tr>
<td><code>&lt;frame src&gt;</code>, <code>&lt;iframe src&gt;</code></td>
<td><code>*/*</code></td>
<td>Link to immediately loaded separate document</td>
</tr>
<tr>
<td><code>&lt;head profile&gt;</code></td>
<td>unspecified</td>
<td>Either a URN or a link to external metadata that may mimic <code>&lt;meta&gt;</code> elements.</td>
</tr>
<tr>
<td><code>&lt;img src&gt;</code>, <code>&lt;input src&gt;</code></td>
<td><code>image/*</code></td>
<td>Image loaded in page</td>
</tr>
<tr>
<td><code>&lt;img longdesc&gt;</code>, etc.</td>
<td><code>text/plain</code></td>
<td>Link to plain text description</td>
</tr>
<tr>
<td><code>&lt;link href&gt;</code></td>
<td>Determined by <code>rel</code> attribute. Usually <code>text/css</code></td>
<td>URL of either immediately loaded content or late-loaded alternate content</td>
</tr>
<tr>
<td><code>&lt;object classid&gt;</code></td>
<td>None</td>
<td>URN for a type of executable content.</td>
</tr>
<tr>
<td><code>&lt;object codebase&gt;</code></td>
<td><code>*/*</code></td>
<td>Base URL for executable content.</td>
</tr>
<tr>
<td><code>&lt;object data&gt;</code></td>
<td><code>application/*</code></td>
<td>Unknown executable content.</td>
</tr>
<tr>
<td><code>&lt;q cite&gt;</code>, etc.</td>
<td><code>*/*</code></td>
<td>Link to external content not loaded until navigation</td>
</tr>
<tr>
<td><code>&lt;script src&gt;</code></td>
<td><code>text/javascript</code>, <code>text/vbscript</code></td>
<td>Link to immediately loaded and executed external script</td>
</tr>
<tr><th>In Javascript</th></tr>
<tr>
<td><code>document.implementation.createDocument("","",null).load(uri)</code></td>
<td><code>application/xml</code></td>
<td>Immediately loaded XML.</td>
</tr>
<tr>
<td><code>window.location</code></td>
<td><code>text/html</code>, <code>text/xhtml</code></td>
<td>Yields URL of the current document / receives external URL.</td>
</tr>
<tr>
<td><code>(new XMLHttpRequest).open(method, url)</code></td>
<td><code>*/*</code></td>
<td>Arbitrary textual content to fetch, often XML.</td>
</tr>
<tr>
<td><code>document.createElement('SCRIPT').src</code>, <code>document.body.style.backgroundImage</code>, etc.</td>
<td>?</td>
<td>Programmatic interfaces to HTML, CSS, and XML</td>
</tr>
<tr><th>In XML</th></tr>
<tr>
<td><code>&lt;!DOCTYPE ... SYSTEM "uri"&gt;</code></td>
<td><code>application/xml-dtd</code></td>
<td>Location of immediately loaded doctype or XML schema.</td>
</tr>
<tr>
<td><code>&lt;!ENTITY ... SYSTEM "uri"&gt;</code></td>
<td><code>application/xml</code></td>
<td>Location of immediately loaded XML fragment.</td>
</tr>
<tr>
<td><code>&lt;foo xmlns:bar&gt;</code></td>
<td><code>application/xml-dtd</code></td>
<td>Location of immediately loaded DTD or XML schema?</td>
</tr>
<tr><th>In XPath</th></tr>
<tr>
<td><code>document(uri)</code></td>
<td><code>application/xml</code></td>
<td>Location of an immediately loaded URI document.</td>
</tr>
</table>

The above can be broken down into a few broad source categories that describe when the content is fetched, and into what security context it is loaded:

  1. Not fetched.  URNs or base URLs.
  1. Loaded into a new document.  E.g. `<A HREF>`
  1. Loaded into the current document.  E.g. `<SCRIPT SRC>`

And there are a few broad types of content:

  1. Document level content.  A web page, text file, or image which can be displayed standalone by the browser.
  1. Scripts and styles which have side-effects, and which can embed other languages.
  1. Images & audio which need to be fetched but which have no side effects.  May need to be proxy to filter out buffer-overflow exploits, polyglots, etc.
  1. Data.  JSON and XML content which is fetched programatticaly.


### Likely Policies ###

Many clients will want to proxy external content to enforce well-formedness, require that the advertised mime-type and encoding match the actual mime-type and encoding, filter out images that exploit known buffer overflows, strip cookies to prevent XSRF, etc.

Clients who proxy content will likely want to whitelist specific URLs or hosts, such as an image hosting service that they trust not to serve problematic content.

Clients may want to substitute their own version of a piece of common content for certain URLs to improve consistency or caching, so to serve their own version of the jQuery library in place of any URL whose path ends with `/jquery.js`.

Some clients may want to ban all dynamically loaded scripts and styles, and others may want to pass dynamically loaded scripts and styles through a rewriting proxy.

Some clients may wish to prevent "phoning home," prevent user data from leaking by denying access to any but a whitelisted set of hosts.


### URLs and regular expressions ###

#### Should the URI policy receive URLs as strings or as objects? ####

Javascript has no builtin APIs for composing, decomposing, resolving, or manipulating URIs.  It provides a few functions for encoding and decoding URL parts, but the decoding parts are problematic since the decoding of `'+'` differs depending on where it appears.

Most JS code that deals with URLs uses regular expressions in ways that are subtly or blatantly incorrect.


### URL References vs URLs ###

#### Should the URI policy receive URL fragments? ####

RFC 3986 uses the term URI to refer to identifiers without a "fragment" such as `scheme://authority/path?query` and the term URI reference to refer to identifiers with a framgnet such as `scheme://authority/path?query#fragment`.

The HTML5 spec uses the term URL to refer to both.

Under HTTP, servers never receive the fragment from the browser.


### Non-latin characters and case folding in international domain names ###

#### Is domain name normalization the responsibility of the uri policy caller or the uri policy implementation? ####

#### How many non-malicious gadgets would break if a hostname whitelisting uri policy rejected URLs with unnormalized domains? ####

Erik van der Poel, a unicode.org contributor, says:

> The browsers implement a set of RFCs called IDNA (Internationalizing Domain Names in Applications) specified in RFCs 349{<a href='http://www.faqs.org/rfcs/rfc3490.html'>0</a>,<a href='http://www.faqs.org/rfcs/rfc3491.html'>1</a>,<a href='http://www.faqs.org/rfcs/rfc3492.html'>2</a>} and <a href='http://www.faqs.org/rfcs/rfc3454.html'>3454</a>. The IDNA process includes a Nameprep step (based on Stringprep) that involves lower-casing, case-folding, mapping to nothing (deleting) and NFKC normalization. This step is often bundled into the same API that performs the final Punycode step (xn-- followed by gibberish). These steps can fail, in which case you probably want to reject that domain name.

> If you're using Java, ICU4J has a class called <a href='http://icu-project.org/apiref/icu4j/com/ibm/icu/text/IDNA.html'>IDNA</a>.

> The browsers handle illegal Punycode names differently. MSIE7 rejects them, while Firefox allows them. Non-Latin characters are covered by IDNA too.

> One example is the soft hyphen (U+00AD), which is "mapped to nothing" in IDNA. I have come across URLs on the Web where there is a soft hyphen at a hyphenation point, e.g. `micro<U+00AD>soft.com`. MSIE7 just goes to microsoft.com, but MSIE6 goes to `micro\xC2\xADsoft.com`, so if your whitelist contains hosts at unscrupulous registries like `*.cc` and the like, your MSIE6 users might accidentally go to a site that you didn't intend to whitelist.


### Legacy URI Policies ###

#### Where are URI policies evaluated? ####

#### Will old URI policies continue to work? ####

There are currently few URI policies in production.  Those are based around two different APIs: one java interface, and a separate JS one.

```
/**
 * Specifies how the plugin resolves external resources such as scripts and
 * stylesheets.
 *
 * @author mikesamuel@gmail.com
 */
public interface PluginEnvironment {

  /**
   * Loads an externally resource such as the src of a script tags or
   * a stylesheet.
   *
   * @return null if it could not be loaded.
   */
  CharProducer loadExternalResource(ExternalReference ref, String mimeType);

  /**
   * May be overridden to apply a URI policy and return a URI that enforces that
   * policy.
   *
   * @return null if the URI cannot be made safe.
   */
  String rewriteUri(ExternalReference uri, String mimeType);
}
```

and

```
@param {Object} uriCallback an object like {
  rewrite: function (uri, mimeType) { return safeUri }
}.
The rewrite function should be idempotent to allow rewritten HTML
to be reinjected.
```

## Design ##

### Decisions ###

#### How can a URI policy implementation distinguish between URLs that are loaded without further user action, and distinguish the expected type of content? ####

Our API will expose the distinctions described above as hints: URNs vs. immediate load in same document vs. eventual load in new document, document level content vs. side effecting includes, audiovisual content, data.

#### Should the URI policy receive URL fragments? ####

No compelling reason to deny that data.  There has been at least <a href='http://www.owasp.org/images/4/4b/OWASP_IL_The_Universal_XSS_PDF_Vulnerability.pdf'>one attack</a> that exploited data in fragments.

#### Should the URI policy receive URLs as strings or as objects? ####

#### Is domain name normalization the responsibility of the uri policy caller or the uri policy implementation? ####

This problem can be solved with library support, but because of IDNA and encoding issues, all uri policy callers will have to normalize the input URL anyway.

#### How many non-malicious gadgets would break if a hostname whitelisting uri policy rejected URLs with unnormalized domains? ####

I believe the number is small.  There are widely used international domain names and IDNA is hard to implement in javascript since it requires full NFKC normalization.  But we can normalize all URLs that appear in HTML and CSS server-side, and most other URLs generated by JS are derived from those URLs.

#### How do we make sure that rewritten URLs in innerHTML can be extracted and reinjected? ####

We can either require that URI policies be idempotent in the (∀ x∈dom(f), f(x)=f(f(x))) sense, or that URL rewriting be reversible.  Idempotence is simple to test for and is easier to implement, so we require URI policies to be idempotent.

#### Should the URI Policy design make specific allowances for memoization? ####

No.  The URI policy API should be designed in such a way that a generic memoizing implementation can wrap a non-memoizing implementation if memoization turns out to be a significant concern.  That should be possible if the URI policy takes immutable inputs, produces immutable results, and is stateless, this should be possible.

#### Where are URI policies evaluated? ####

Callers need to invoke the URI policy from both server-side java, and from client-side javascript.  There are two ways that these can be unified - either the policies are authored in Java and the client javascript is generated from the java class.  Alternatively, we can use Rhino or another server-side JS interpreter to interpret a policy implemented in JS in the cajoler.  The latter is preferred since a cajoling service that cajoles output for a variety of containers may need to supply different policies to the service.

The policy might not be entirely trusted code and may need to itself be cajoled before it is run or it may needed to be sandboxed in some other way.

#### Is it the responsibility of the policy or the caller to resolve relative URLs? ####

The caller.  Only the caller has enough information to resolve URLs correctly.  E.g., the HTML rewriter will want to resolve URLs relative to the input HTML's source or `<base href>`, pass the URL to the policy, and then relativize the URL against the URL the gadget will be served under.

#### Will old URI policies continue to work? ####

Maybe, but we should work with the few existing policy authors to rework them quickly.

#### What is responsible for fetching scripts and styles so the Cajoler can inline them? ####

The URI policy will no longer be responsible for this.  It doesn't need to be involved in URL fetching in the browser, so we will separate out URL loading into a separate concern: a java `interface UrlGetter` that can GET content from a URL that appears statically in HTML or CSS.

### Definition ###

A URI policy is a mapping from absolute normalized† URI references plus context hints to URI references or the special DENY value.

URI policies are implemented as Javascript (or Cajita) objects with the API below.

Context hints come in several flavors:

  * OTHER\_DOCUMENT -- will be loaded in another document.
  * NO\_DOCUMENT -- a URN which does not point to content.
  * REQUIRES\_USER\_INTERACTION -- will not be loaded without user interaction such as a link click.  E.g., a `<A HREF>` would have this set, but `<IMG SRC>` would not.
  * NO\_SIDE\_EFFECT -- on for content which cannot cause other network loads or programatically change the current document.  JS may execute, CSS may contain JS.  XML may fetch arbitrary other URLs via DOCTYPEs and external references.  HTML may contain scripts.
  * TEXT -- textual content which is unlikely to exploit buffer overflows or other browser flaws. Not set for image or sound URLs.
  * The expected mime-type(s) which can be used by a filtering proxy to match against actual `Content-Type` headers.


† - URL normalization on the Cajoler is reliable, but is
best-effort on the client.


### API ###

The API is a javascript object which has a single public method which dispatches to other methods.  If an implementor neglects to implement one of these, e.g. `rewriteScriptUrl`, then they cannot suffer compromises due to a failure to properly proxy dynamically loaded script sources.  If none of the specific handlers are applicable, then it tries `rewriteOther` which can be used to log the fact that a URL could not be rewritten, or try some best-effort but extra-paranoid proxying.

```
{
  rewriteUrl: function (absUrl, hints) {  // final public
    // Dispatch to other handlers based on hints.
  },
  rewriteScriptUrl: function (absUrl, hints),  // abstract protected
  rewriteStylesheetUrl: function (absUrl, hints),  // abstract protected
  rewriteAudioVisualUrl: function (absUrl, hints),  // abstract protected
  rewriteDocumentUrl: function (absUrl, hints),  // abstract protected
  rewriteObjectUrl: function (absUrl, hints),  // abstract protected
  rewriteUrn: function (absUrl, hints),  // abstract protected
  rewriteOther: function (absUrl, hints),  // abstract protected
}
```

### Supporting Code ###

Javascript URI library that does resolution.

Java library for API normalization.


### Advice for Policy Implementors ###

Do not black-list.  Domain black-listing is unreliable because of numeric IPAs, open redirectors, and the difficulty of normalizing host-names on the browser.

Use the `NormalizedUri` class to normalize any URL parts that you wish to use in whitelists.  All whitelists of host-names should only contain IDNA normalized host-names.

Do not fetch the URL speculatively before deciding whether to allow it or not since that might enable XSRF attacks.

White-list protocols.  Be wary of `javascript:`, `widget:` and anything not `http:` or `https:` or `mailto:`.

Check the context hints.  If you don't have enough hints, DENY.

Be careful with `rewriteOther`.  If you implement it in such a way that it doesn't deny anything, you must be careful to stay apprised of changes to the URI policy API.

Be careful of the encoding of text documents.  The encoding of a document often affects the encoding of % encoded octets of URLs in that document, so it's a good idea to re-encode text as UTF-8.

Don't use regular expressions to decompose URLs.  If you need to whitelist a particular domain and protocol, look at the domain and protocol fields individually.
White-listing by regular expressions tends to be vulnerable to <a href='http://secunia.com/advisories/10395/'>URL spoofing</a>.

Almost all urls should be rewritten to be fetched by a proxy.  The proxy ought to have the same level of amount of access as the authors of the gadgets ie. if gadgets are fetched from the internet, urls ought to be rewritten to use a public proxy to prevent gadgets from scanning internal networks via url fetching errors.

If a url must be fetched without proxying, the host name ought to be fully qualified and terminated with a dot suffix (http://www.example.com.).  If such a precaution is not taken, a gadget can be used to probe an internal network.  Also, consider rejecting any URLs with non-standard ports, e.g. http to port 22.