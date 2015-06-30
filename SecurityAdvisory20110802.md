# Caja Security Advisory 2011/08/02 #

[Revision 4229](https://code.google.com/p/google-caja/source/detail?r=4229) introduced support for `innerText` with an incorrect
check for the editability of `script` tags.  As a result, an
attacker is able to create a script nodes containing uncajoled code.

For unrelated reasons, an earlier change at [Revision 3802](https://code.google.com/p/google-caja/source/detail?r=3802) disabled
tests that test for this case.

## Impact ##

This vulnerability allows attackers' sandboxed code to completely
bypass all Caja's protections if the container is using a version of
Caja between [revision 4229](https://code.google.com/p/google-caja/source/detail?r=4229) and 4570 by setting innerText of script
elements.

## Advice ##

Do one of the following:
  1. Best: Upgrade to a version of Caja at or after 4570.
  1. Apply the patch at http://codereview.appspot.com/download/issue4798044_1.diff to your current checkout, and rebuild.
  1. Least recommended: Revert to a version of Caja prior to 4229

## More Information ##

The issue was originally reported at
[issue 1384](http://code.google.com/p/google-caja/issues/detail?id=1384).

The patch is available at
http://codereview.appspot.com/download/issue4798044_1.diff and
discussion of the change at http://codereview.appspot.com/4798044/show.