# Introduction #

A much better and simpler code review scheme.

# Details #

  1. Check out Caja from svn.
  1. Create your patch however you want.
  1. Run `myvn snapshot`.
  1. Mark any related bugs in the issue tracker Pending.
  1. Note the review number, and see your review at http://codereview.appspot.com/mine
  1. Respond to comments, edit code and snapshot again.
  1. Loop to (3) until done.
  1. When done, commit with `myvn submit`.
  1. Mark any related bugs in the issue tracker Fixed.
  1. Close the issue on `codereview.appspot.com` by clicking "Edit Issue", clicking the "Closed" checkbox, and clicking "Update issue"