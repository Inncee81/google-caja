# Any frame can redirect any nameable frame to any URL #

## Effect ##
A frame can redirect another frame to a phishing page.
If a frame knows the likely contents of an existing frame, it can redirect that frame to a malicious website that purports to be the same frame.

For example, if `galactic-mega-bank.com` embeds an iframe to `evil.org` then `evil.org` could redirect its parent frame to a reasonable simulcrum of `galactic-mega-bank.com` and trick the user into divulging their bank account password.

Such techniques can also be used to initiate "drive-by-downloads" wherein a malicious website continuously redirects to a website that causes the browser to pop-up a download for malware.  The pop-up is modal; it prevents the normal use of the browser until the user accepts or rejects the download.  Users may initially reject the download, but the site can continually redirect until the user accepts, effectively hijacking the browser.

## Background ##
The HTTP Location header allows a server to tell the browser to instead look for a site at a different URL.  The HTTP-EQUIV element can have the same effect but can be embedded in HTML.

From JavaScript, `document.location` can be set to achieve the same effect, a navigation of the current frame.

But the `location` member is available on all frames and is specifically exempted from the same-origin policy.  So any frame can navigate any other frame which it can reference.

There are some other members of the JavaScript frame object which are specifically exempted from the same-origin policy, and these make it easy for one frame to reference another.  The `parent`, `top`, and `frames` members allow access to respectively the parent of the given frame, the topmost frame in the tab containing the given frame, and frames directly contained in the given frame.

Further, the `window.open` method allows a frame to open a URL in a frame with a particular name.


## Assumptions ##
JavaScript can access an unattenuated frame object or an attenuated one with a settable location object that does not filter the value set AND that the ability to set the location is not contingent on user interaction.

## Versions ##
All.

## Example ##
```
window.top.location = 'http://evil.org/';
```