# Caja Security Advisory 2012/09/19 #

There is an arbitrary JS execution vulnerability in Caja versions from [revision 4576](https://code.google.com/p/google-caja/source/detail?r=4576) to before [revision 5054](https://code.google.com/p/google-caja/source/detail?r=5054). Our taming of `setTimeout` and `setInterval` allowed passing objects which would be interpreted as strings of JavaScript code, thus executing JS in the host page's context.

Versions of Domado (then Domita) before [revision 4576](https://code.google.com/p/google-caja/source/detail?r=4576) always created a wrapper function around the callback provided to our tame setTimeout before passing it to the browser's setTimeout; [r4576](https://code.google.com/p/google-caja/source/detail?r=4576) removed this wrapper as part of the transition from uncajoled Domita to cajoled Domado, but included a test that the callback was not a string. The test should have been that the callback _is_ a _function_, and [r5054](https://code.google.com/p/google-caja/source/detail?r=5054) corrects that mistake as well as always using a wrapper function.

## Impact ##

Between [revision 4576](https://code.google.com/p/google-caja/source/detail?r=4576) and [revision 5054](https://code.google.com/p/google-caja/source/detail?r=5054), all users of our DOM sandbox are vulnerable. If you are using `caja.js` to load HTML, this means you.

## Advice ##

Do one of the following, in order of preference:

  1. Upgrade to a version of Caja at or after [r5054](https://code.google.com/p/google-caja/source/detail?r=5054).
  1. Apply the patch available at https://codereview.appspot.com/download/issue6535045_1_2.diff .
  1. Disable use of Caja until you can do one of the above.

## More Information ##

The vulnerability was reported as [issue 1532](https://code.google.com/p/google-caja/issues/detail?id=1532).
The fix was reviewed at https://codereview.appspot.com/6535045/ and committed as [revision 5054](https://code.google.com/p/google-caja/source/detail?r=5054).

Shindig, which uses Caja, was updated to incorporate the fix at their [revision 1387413](http://svn.apache.org/viewvc?view=revision&revision=1387413).