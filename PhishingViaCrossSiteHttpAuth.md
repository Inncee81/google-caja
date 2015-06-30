# Phishing Via Cross-Site HTTP Authorization #

## Effect ##
An attacker can arrange to have the browser display a credible-looking HTTP authorization dialog in the context of some site they do not control. A naïve user may then enter into that dialog their username and password for that site, transmitting that information to the attacker.

## Background ##
The HTTP protocol allows a server to respond to a request with a demand for authentication. In the case of basic access authentication http://tools.ietf.org/html/rfc1945#section-11.1, the credentials needed are a username and password.

Browsers are designed so that, if they encounter a resource demanding such authentication, they display a dialog asking for a username and password. This dialog does not show any information about the specific resource on a page which triggered it. The only contextual information in the dialog is a "Realm", which is a string supplied by the server of the resource.

In the attack scenario, the user has an account on site `example.com` and is browsing content there. An attacker, who controls content on `evil.com` arranges for the inclusion, as part of the `example.com` content seen by the user, of (say) an `img` tag with a `src` from the attacker's domain:

```
<img src="evil.com/pony.jpg"/>
```

For example, if `example.com` is an email UI, the attacker may send the user an email containing the above HTML. On the face of it, this content seems like harmless static HTML presenting no XSS risks, and an HTML sanitizer on `example.com` might validly choose to allow it.

The attacker arranges for the HTTP response for `evil.com/pony.jpg` to include a request for Basic HTTP authentication. The result is that the user of `example.com` sees a dialog that _looks_ like it came from something on `example.com`, saying (depending on the browser):

<pre>
+----------------------------------+<br>
| Authorization for example.com:   |<br>
| Username: ...................... |<br>
| Password: ...................... |<br>
+----------------------------------+<br>
</pre>

A naïve user, at that point, may enter their credentials for `example.com`. These credentials are sent back to the attacker's server at `evil.com`.


## Assumptions ##
A site displays third-party HTML content
AND the site does not proxy the images (and other linked items) in the content

## Versions ##
All

## Example ##
See above.