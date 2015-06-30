# Caja Security Advisory 2012/01/16 #

[Revision 4374](http://code.google.com/p/google-caja/source/detail?r=4374) of Caja inadvertently introduced a change to the CSS properties of the container DIVs created by Caja, making it possible to craft guest code creating visible content outside the boundary of the double-DIV sandbox enforced by Caja. For example:

```
  <div style="margin-left:-150px;margin-top:0px;width:110px;">
    <div>Phishing content</div>
  </div>
```

## Impact ##

This vulnerability allows guest content authors to overlay and mimic the user interface of surrounding container content, and thus attempt to phish the end-user into entering into the guest content information that would normally only be revealed to the container.

## Advice ##

Upgrade to a version of Caja at or after 4761.

## More Information ##

The issue was originally reported at [issue 1422](http://code.google.com/p/google-caja/issues/detail?id=1442).

Discussion of the change is available at http://codereview.appspot.com/5540049/.