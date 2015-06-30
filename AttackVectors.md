# Properties of Interpreters or the Browser Environment that allow Privilege Escalation #

Below is a list of known attack vectors.  We discuss the EcmaScript 3 language, quirks of existing interpreters, and browser specific extensions that could allow privilege escalation  so that we can come up with tests for a safe JavaScript rewriter or verifier.

## Attack Vectors at the EcmaScript/JavaScript level ##
  * GlobalObjectPoisoning -- Global object poisoning
  * EvalArbitraryCodeExecution -- eval and the Function constructor allow arbitrary code execution
  * ArgumentsMaskedByVar -- function arguments array masked by `var arguments` on Opera
  * CrossScopeParameterModification -- arguments array allows modification of parameters
  * ArgumentsExposesCaller -- arguments Array and function object expose caller
  * FunctionMemberCrossScopeParameterAccess -- function object's arguments array expose arguments while call in progress
  * TypeofInconsistent -- typeof inconsistent for regular expressions
  * InaccessibleLocalVariables -- Inaccessible local variables
  * CatchBlocksScopeBleed -- catch blocks may cause global assignment, or local scope creep
  * GlobalScopeViaThis -- Global scope reachable via `this` from functions not invoked as methods
  * DeleteUnmasksGlobals -- Delete can unmask globals
  * FunctionConstructor -- Function constructor accessible via the 'constructor' property
  * ObjectEvalArbitraryCodeExecution -- Object.eval allows execution of unsanitized code on Firefox.
  * ObjectWatch -- Object.watch allows stealing and poisoning of otherwise restricted data
  * ObjectToSourceLeaksPrivates -- Object.toSource and uneval allow access to private fields
  * FunctionMethodsLeakGlobalScope -- Function.call or Function.apply can leak window with certain this-values.
  * ConditionalCompilationComments -- Conditional compilation may allow disabling of runtime checks.
  * StringObfuscationIsEasy -- Approaches that rely on detecting code for other languages in string literals is easy to defeat
  * ParentCircumventsScoping -- The javascript1.2 feature `__parent__` circumvents normal scoping.
  * JsControlFormatChars -- `[:Cf:]` can be used hide code in string or comments.
  * InconsistentlyReservedKeywords -- Different reserved keyword set can cause parser ambiguity
  * ErrorExposesParameterValues -- The stack property of Error includes parameter values.
  * HiddenControlFlowHazard -- Seemingly safe Caja data computations may result in a control-flow transfer to a potential adversary.
  * RegexpsLeakMatchGlobally -- Any regular expression can match against the last string passed to any other
  * EvalBreaksClosureEncapsulation -- Eval extensions allow reaching into the scope chain of closures
  * PostIncrementAndDecrementCanReturnNonNumber -- Incorrect implementations of postincrement and postdecrement can cause confusion as to which property is being accessed
  * MisOptimizations -- Some interpreters try to optimize javascript before execution subtly changing the semantics of builtin operators (PostIncrementAndDecrementCanReturnNonNumber is a specific example)
  * CompoundAssignmentsCanReturnNonNumber -- The type of assignment expressions may not be correct.
  * FinallySkipped -- An exception that is thrown not inside a `try/catch` caught skips `finally` blocks.

## Attack Vectors at the Browser Environment, DOM, HTML, or CSS levels ##
  * ScriptInHtml -- HTML Tags in Javascript Strings can allow Unsanitized Script Execution
  * SetTimeoutArbitraryCodeExecution -- setTimeout & setInterval allow arbitrary code execution
  * DomNodeAllowArbitraryCodeExecution -- ActiveXObject, document.createElement, document allow arbitrary code execution
  * InnerHtmlYieldsCdata -- script, style, xmp and listing elements' innerHTML cannot be safely inserted into another element's innerHTML
  * DomAllowsXsrf -- document object allows arbitrary XSRF with the user's credentials
  * DomAllowsKeylogging -- DOM access allows keylogging
  * XsrfViaXxe -- XMLHttpRequest and DOMParser parsing allow arbitrary XSRF via XXE
  * CssAllowsArbitraryCodeExecution -- Some CSS properties allows execution of unsanitized javascript?
  * CssImportsAllowUnsanitizedCodeExecution -- `@import` can import unsanitized CSS which can execute unsanitized javascript
  * NullCharEscapes -- Null characters in URL can disguise protocols such as `javascript:`
  * ConfusedHtmlParsers -- Differences in the way HTML parsers parse malformed HTML can hide unsanitized scripts
  * EventHandlersEvalWithDom -- The scope that event handlers are executed in may expose DOM properties as globals
  * DocTypesCanInjectUnsanitizedContent -- DOCTYPEs can define entities which can inject unsanitized script or markup.
  * EventChecksCircumventableByInfLoops -- Invariants preserved by event handlers can be circumvented by causing the browser to turn off javascript.
  * IdAndNameMasking -- Members of `HtmlCollection`, `HTMLFormElement`, etc. masked by ids&names
  * UrlFetchingSideChannel -- Side-channels from unproxied connections leak information across closed networks
  * HistoryMining -- CSS can be used to determine whether a user has visited a URL.
  * RedirectWithoutUserAction -- JS and HTML both allow redirection with user interaction.
  * PhishingViaCrossSiteHttpAuth -- An attacker can display an HTTP authorization dialog that looks like it may have come from another site.