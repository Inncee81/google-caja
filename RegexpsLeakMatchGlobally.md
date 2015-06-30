# Regular Expressions leak matched text #
Reported by Jasvir

## Effect ##
Secrets can be stolen by untrusted code.


## Background ##
EcmasScript 262 S 15.10.6.2 describes `RegExp.prototype.exec` and other parts of 262 define `String.prototype.match`, `RegExp.prototype.test` and other core functions in terms of `RegExp.prototype.exec`.

According to the spec, `RegExp.prototype.exec` should match against the string `'undefined'` if no parameter is passed in, but that is not what happens in some implementations.  When no value is passed in, the last string matched by **any** regular expression instance's `exec` method is used.


## Assumptions ##
An attacker can cause a regular expression matching function to be called without argument and can observe the result.
AND
Sensitive data is not laundered after being matched.


## Versions ##
Works in Firefox 2 but not in IE 6.  Others not tested.


## Example ##
```
// Privileged code
(function () {
  var queryString = document.location.search;  // Assume it's "?password=1234"

  function params() {
    return queryString.split(/[&?]/g);
  }

  if (params()[0] === 'debug=on') {
    // ...
  }
})();

// Unprivileged code without direct access to document.location
(function () {
  alert(/.*/.exec());
})();
```

But regexps can be laundered as long as no untrusted code is executed while a match is in progress.
```
function params() {
  try {
    return queryString.split(/[&?]/g);
  } finally {
    /^/.exec('NOTHING TO SEE HERE.  MOVE ALONG.');
  }
}
```