# User-Agent Specific Cajoling #

## Background ##
Our supporting JS includes some code that is browser-specific ; we
back-port JSON onto older browsers, and do a lot of hacks that we
know will work on some browsers but not others.  Having multiple code
paths, only one of which will ever be live in a given environment is
wasteful in bandwidth and execution time.

E.g., in
```
if (typeof JSON === 'undefined') {
  JSON = (function () {
    // snip 600 lines of code
  })();
}
```
on any given browser, a fresh page will always have JSON present or
absent.  Since the EcmaScript 5 draft mandates JSON, the number of
browsers to which this code needs to be shipped will only decrease.

Similarly, in
```
if (window.addEventListener) {
  // snip code
} else if (window.attachEvent) {
  // snip
}
```
on any given browser, the test will always run the same way.

Further, some browsers have bugs and features that make it hard to
optimize code for them.  As these bugs are fixed, the set of browsers
for which we cannot optimize code should dwindle for any given bug.
` a[+i] = b[+i] `
can be significantly optimized on all browsers except certain older
Firefox versions.  And browsers that implement EcmaScript 5 strict mode
will require vastly fewer code transformations allowing smaller and
faster cajoled code to work on those browsers.

## Goals ##
To reduce the amount of dead code shipped with our supporting JS, to
allow optimizations on user-agents that don't have scary bugs, and to
optimize out unnecessary code branches.

## Overview ##
We want to be able to take into account, statically, knowledge about
the environment that our supporting JS and cajoled output is running
in.

First we define a file "environment-checks.js" which tests the
environment and produces a JSON object like:
```
({
  "navigator.userAgent": "FooBrowser/1.0",
  "typeof window.JSON": 'undefined',
  "!!window.addEventListener": true,
  "!!window.attachEventr": true,
  "!!(function () { 'use strict'; return this; })()": false
})
```
where each key is a snippet of javascript that has no side-effect, and
where each value is the result produced by `eval`ing that code in a
fresh execution context.

We can generate JSON files for each user-agent we care about and store
them as artifacts in the source-code repository.

## Container Assumptions ##
We assume two things about containers that load our
cajoled JS:
  1. That they never delete a member of the global object
  1. That any additions to the global object will behave as specified.  So if a container provides its own implementation of window.JSON it is not observably different from that on a user-agent with a native implementation.

See [Validating](#Validating.md) for ways these assumptions can be verified to hold for a wide range of programs.


## Code Eliminating Minifier ##
Next, we define a rewriter that reads the JSON file, and finds
expressions in conditions that match those and that contain no non-global references.
We then inline any assertions or their negations, and constant fold over comparison
and logic operators.

Finally the rewriter eliminates code paths that are inside conditionals or loops
where the condition is falsey.  We need to preserve any side effect in
the condition.

This rewriter will be exposed as an optional pass in our Minifier.  If
the Minifier has a `--user-agent` flag that specifies a user-agent
environment JSON file, it will start with a rewriting pass to
eliminate dead code before rendering the output.

## Allowing Cajoler to Optimize ##
The Cajoler receives a `PluginEnvironment` object which describes
the environment the gadget runs in.  We add a getter to the
PluginEnvironment to get the JSON environment object.

If the Array optimization wants to make sure that it is safe to allow
access to negative numeric indices instead of positive only, it can
check
```
enum UserAgentProperty {{
  NEGATIVE_INDICES_ON_FNS("void 0 !== (function () {})[-2]"),
  ...
}
...
void optimize() {
  Boolean negativeIndicesOnFns = myPluginEnvironment.getUserAgentJson().get(
      UserAgentProperty.NEGATIVE_INDICES_ON_FNS.code);  // null &rarr; don't know
  if (!Boolean.FALSE.equals(negativeIndicesOnFns)) { return; }  // abort
  // optimize
}
```

A gadget container that embeds the Cajoler can pick the appropriate
user-agent JSON based on the user agent header in the request headers.
Each JSON file should include the user agent as a property, so we will
probably have to write a function to strip out insignificant info (OS
version, locale, etc.) from the user agent string.  It is always safe
to use an empty JSON object as an environment object because an undefined
key means "don't know."

## <a>Validating</h2>
We can check that the assumptions inherent in a user-agent JSON file hold in a container, and in the presence of the cajita supporting code by doing the following:

  1. Load the container and/or cajita supporting code
  1. Run "environment-checks.js" to generate a new JSON object
  1. Compare the JSON from (2) with that from the environment file.

This validation can be done by containers to make sure they're not violating the assumptions above, and could be done in the beginning of a module to make sure that it only runs in the proper environment.

An excellent place to do this check would be in a unit-testing framework since that is supposed to exercise a large portion of the code.

Also, note that if the JSON from (2) does not match that in the environment file, then using the JSON object that contains only (key, value) pairs that are present in both, would yield a maximal set of optimization assumptions that **are** valid.

## Effect on Cachability ##
The User Agent definitions passed to the cajoler affect the cajoler output, so any cajoling service that caches the output of a module must incorporate a proxy for the user-agent into the cache key.  A good key to use would be `userAgentJson['navigator.userAgent']` along with the user's locale if the input contained message strings.