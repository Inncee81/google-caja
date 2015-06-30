## Effect ##
Malicious code that can deny service can circumvent checks that rely on code being executed.


## Background ##
Many browsers (including IE and FF) will throttle frames that execute too much javascript without pausing.  They often ask the user a question like:

> ### Unresponsive script ###

> A script on this page may be busy or it may have stopped responding.  ...

> `[Stop Script] [Continue]`



## Assumptions ##
Excessive computation (such as by executing an infinite loop) is not preempted before the browser can intercede.

Security constraints are enforced via event handlers.

Security can be breached by unscripted browser behavior such as form submission.



## Versions ##
Many including IE and FF.



## Example ##
Consider a tamed DOM that allows forms to be used as collections of inputs but prevents them from being submitted:
> `<form onsubmit="foo()">...</form>`
> → `<form onsubmit="try { foo(); } finally { return false; }">...</form>`

If `foo` is `function foo() { while (1); `} then the browser will eventually prompt the user to turn off scripting at which point the form will either submit, or will become submittable should the user click a button.