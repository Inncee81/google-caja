# URI Policy Examples #

Below are examples of interesting URI policies and how they can be implemented.

It depends on the [URI policy](http://codereview.appspot.com/1229046/show) definitions of URI effect and loader type.  As of 1 June 2010, these are pending.  (TODO: remove this caveat once the change is submitted).

TODO: rewrite these examples in JavaScript once the JS->UriPolicy bridge has been implemented.

## Whitelist ##
```
Set<URI> allowed = ...;

public String rewriteUri(
    ExternalReference u, UriEffect effect,
    LoaderType loader, String hint) {
  return allowed.contains(u.getUri()) ? u.getUri().toString() : null;
}
```

## Allow hosted images and videos from hosting services that properly set content-type based on extension ##
```
// Assume that images.foo.com will always serve the following extensions
// with the appropriate image content-type, and only serves images.
// Note that no SVGs are allowed since they contain scripts so violate
// LoaderType.MEDIA.
Set<String> imageExts = ImmutableSet.of(".jpg", ".gif", ".png");
Set<String> videoExts = ImmutableSet.of(".mpeg", ".wmv", ".ogv");

public String rewriteUri(
    ExternalReference u, UriEffect effect,
    LoaderType loader, String hint) {
  if (loader == LoaderType.MEDIA) {
    String scheme = u.getUri().getScheme();
    if ("http".equals(scheme) || "https".equals(scheme)) {
      Set<String> extensions = ImmutableSet.of();
      String domain = u.getUri().getHost();
      if ("images.foo.com".equals(domain)) {
        extensions = imageExts;
      } else if ("videos.foo.com".equals(domain)) {
        extensions = videoExts;
      }
      String path = u.getUri().getPath();
      int dot = path.lastIndexOf(path);
      if (dot >= 0 && extensions.contains(path.substring(dot))) {
        return u.getUri().toString();
      }
    }
  }
  return null;
}
```



## Display an interstitial before transition to uncajoled pages ##public String rewriteUri(
    ExternalReference u, UriEffect effect,
    LoaderType loader, String hint) {
  if (effect == UriEffect.NEW_DOCUMENT && isExternalUri(uri)) {
    return "javascript:displayWarning(uri)";
  }
}```