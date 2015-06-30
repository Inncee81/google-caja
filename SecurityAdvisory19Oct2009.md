# Caja Security Advisory 19-October-2009 #

[Revision 3652](https://code.google.com/p/google-caja/source/detail?r=3652) introduced changes to allow `iframe` shims to work
around layout problems in older browsers, but did not update the
default HTML schemas to block uses of `iframe`s to load code.

## Impact ##

These vulnerabilities allow attacking sandboxed code to completely
bypass all Caja's protections if the container is using a version of
the HTML schemas between [revision 3652](https://code.google.com/p/google-caja/source/detail?r=3652) and 3810, and is
using a URI policy that does not reject or block by proxying URLs where
the mime-type is `text/html`.

## Advice ##

Do one of the following:
  1. Best: Upgrade to a version of Caja at or after 3810.
  1. Rollback to a revision prior to [r3652](https://code.google.com/p/google-caja/source/detail?r=3652)
  1. Apply the patch at http://codereview.appspot.com/download/issue124069_2001.diff to your current checkout, and rebuild.
  1. Change your URI policy to block or proxy URLs where the mime-type is `text/html`.

## More Information ##

The issue was originally reported at
[issue 1108](http://code.google.com/p/google-caja/issues/detail?id=1108).

The patch is available at
http://codereview.appspot.com/download/issue124069_2001.diff and
discussion of the change at http://codereview.appspot.com/124069/show.

CajaWhitelists explains how to modify HTML and CSS schemas, and
UriPolicy explains URI policies.