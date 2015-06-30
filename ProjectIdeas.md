# Project Ideas for Potential Contributors #
  * Figure out what prevents popular JavaScript libraries from cajoling (like [jQuery](http://jquery.com/), [Prototype](http://www.prototypejs.org/), and others).
    * Either improve the coverage of our [DOM wrappers](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/plugin/domita.js) or submit patches to the libraries.
    * Contributor must know JavaScript.
    * Contributor must be comfortable working with the Document Object Model in web browsers.
    * If the problems are insurmountable (unlikely, but possible) then the popular libraries will need [taming](LibraryTaming.md).
  * Create an [Auditor](http://docs.google.com/viewer?a=v&q=cache:1VMOJxtnCkUJ:people.ischool.berkeley.edu/~ping/auditors/auditors.pdf&hl=en&gl=us&sig=AHIEtbRjmisE1ZPikrpSlNGdN4aZ_4I62Q) framework for SES code
    * Contributor must know JavaScript.
    * An understanding of object-capabilities principles would be helpful.
    * One auditor should be able to prove that certain objects are immutable
  * Static analysis and optimization
    * Plenty of places optimization could occur
      * removal of temporary JavaScript variables
      * removal of type checks and other runtime checks
      * improving the speed of the rewriter
      * etc.
    * Contributor must know Java, and it would be better if they knew JavaScript
  * Ambient-authority-to-capability membrane
    * Take a username and password for a site and attenuate that authority so it can be doled out in smaller pieces
      * Given the URL of a webpage and authentication info, rewrite the page so that all URLs point back to the rewriter (a membrane)
      * Remove all content from the page except the whitelisted parts, allowing finer divisions of authority than not all-or-nothing
    * Contributor must know object-capabilities principles.
  * [Shahar](http://code.google.com/p/google-caja-shahar/wiki/ShaharIntroduction)-related [project ideas](http://code.google.com/p/google-caja-shahar/wiki/ProjectIdeas)
  * Bridge building between [Joe-E](http://code.google.com/p/joe-e/), [GWT](http://code.google.com/webtoolkit/), and Caja
    * For example, [Identify the intersection of GWT and Joe-E](https://groups.google.com/group/google-caja-discuss/msg/2e6ca497d2cb450d), and adapt GWT to compile it to Caja-compatible, Joe-E-security preserving JavaScript.
  * [Suggest something!](http://groups.google.com/group/google-caja-discuss/post)
  * Tech writing doesn't qualify for a GSoC project, but if you're a tech writer, we'd love to have you write documentation!  See DocumentationIdeas.
