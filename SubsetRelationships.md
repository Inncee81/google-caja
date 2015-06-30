# Caja Today #

JavaScript and Caja are jointly in transition. The language in the middle of the digram below is Cajita, the Caja kernel language. Cajita is a genuine [object-capability language](http://en.wikipedia.org/wiki/Object-capability_model) we discovered in a subset of JavaScript. As an object capability language, Cajita omits all of JavaScript's mutable static state. In other words, all global implicitly accessible objects, like `Object.prototype`, are immutable. Cajita strengthens JavaScript's one encapsulation mechanism -- lexical variable capture by closures -- into a true security boundary. To make defensive programming practical, Cajita omits various accident prone features of JavaScript, such as the magical `this` keyword.

The following diagram explains the current relationships between Cajita and JavaScript.

---

![http://google-caja.googlecode.com/svn/trunk/doc/images/cajita-vs-valija.png](http://google-caja.googlecode.com/svn/trunk/doc/images/cajita-vs-valija.png)

---

  * _ES3R_ is used in various EcmaScript discussions for "EcmaScript 3 plus reality", meaning, consensus JavaScript as implemented by current [A-grade browsers](http://developer.yahoo.com/yui/articles/gbs/).
  * _ES5_ is the [draft standard](http://wiki.ecmascript.org/doku.php?id=es3.1:es3.1_proposal_working_draft) for the next version of JavaScript.
  * _[ES5-strict](http://google-caja.googlecode.com/svn/trunk/doc/html/es5-talk/es5-talk.html)_ is a more principled sublanguage of ES5 supported by ES5's `"use strict"` directive.
  * Caja defines two variants of JavaScript:
    * _Cajita_ (Spanish for _small box_) is the Caja kernel language --- an object-capability language supporting defensive programming.
    * _Valija_ (Spanish for _baggage_) is the ES3 subset of ES5-strict. Old JavaScript code can easily be ported to Valija. Once ported, it will work on both current and future JavaScripts. Valija exists _only_ to accommodate old code. The additional elements of JavaScript it includes, such as `this`, make it unsuitable for writing defensive code. Instead, Valija is used only to isolate potentially offensive code.

Besides the normal object-capability constraints, Cajita was shaped by the need to simultaneously support all four of the depicted relationships, as well as is practical today.

  * **_T = TranslationTarget_**. Cajita is implemented by translation to JavaScript as run by current browsers.
  * **_N = NiceNeighbor_**. Cajita code coexists with some amount of untranslated JavaScript code. This coexistence must have a well defined semantics and useful safety properties. Today, the NiceNeighbor property supports safety only in one direction -- protecting untranslated JavaScript from potentially offensive Cajita code.
  * **_S = SimpleSubset_**. The first enemy of security is complexity. Due to JavaScript's history, it is needlessly complex. Cajita is approximately the simplest subset of JavaScript we could find with no loss of functionality for new code.
  * **_E = EmbeddedEmulation_**. JavaScript, and therefore Valija, supports mutable static state, which violates object-capability rules. To support multiple isolated Valija sandboxes on one web page, we translate Valija to Cajita so as to emulate this static state in terms of isolated Cajita object states.

Among secure JavaScript variants, [ADsafe](http://www.adsafe.org/), [Jacaranda](http://jacaranda.org/), and [dojox secure](http://www.sitepen.com/blog/2008/08/01/secure-mashups-with-dojoxsecure/) are all designed for a similar degree of support for NiceNeighbor and SimpleSubset. Rather than translate to ES3R, they instead take a verification-only approach, which has various engineering benefits such as zero runtime overhead. However, for ES3R, verification requires blacklisting rather than whitelisting, which is harder to secure.

Note that Jacaranda does define a set of static rules for permitting safe limited use of `this`. A future Cajita may adopt these rules.

Facebook's [FBJS2](http://developers.facebook.com/news.php?blog=1&story=189) and Microsoft's [WebSandbox](http://websandbox.livelabs.com/) have no intermediate object-capability language, but rather, emulate JavaScript directly in JavaScript, i.e., conventional sandboxing. This supports isolation of potentially offensive code, but provides no direct support for secure cooperation or the writing of defensive access abstractions.


# Caja Tomorrow #

Currently, each of the above relationships are fragile or imperfect in various ways documented on each of these respective pages. On browsers that implement the next version of JavaScript, ES5, these relationships will be strengthened, as explained in this section.


---

![http://google-caja.googlecode.com/svn/trunk/doc/images/ses-vs-es5.png](http://google-caja.googlecode.com/svn/trunk/doc/images/ses-vs-es5.png)

---


The [Secure EcmaScript](http://ses.json.org/) working group of the EcmaScript committee seeks to define a simple secure subset of future JavaScript, to be known as "Secure EcmaScript" or SES. Based on lessons from Cajita and these other systems, we propose that SES and future EcmaScript be co-designed to improve all four relationships between them. In this section, we explain first how Cajita will coexist better with ES5. We then explain some deficiencies in those relationships suggesting further improvements in future EcmaScripts and in Cajita as a proposal for Secure EcmaScript.

  * **_T = TranslationTarget_**. Cajita can be translated trivially to ES5-strict, and the resulting translated code should be able to run at close to full speed.
  * **_N = NiceNeighbor_**. Because ES5 enforces the primary security mechanisms Cajita depends on -- closure encapsulation and frozen objects -- untrusted untranslated ES5 objects can be given direct access to Cajita objects without being able to harm their integrity.
  * **_S = SimpleSubset_**. Once the Cajita's target language supports the ES5 meta operations, it becomes practical for Cajita too as well.
  * **_E = EmbeddedEmulation_**. Once Cajita supports the ES5 meta operations, it becomes practical to emulate these operations in the embedded language, bringing it up to full ES5-strict.