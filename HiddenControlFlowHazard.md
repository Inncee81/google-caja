Seemingly safe Caja data computations may result in a control-flow transfer to a potential adversary.

# Introduction #

Reported by Tyler Close.

When the Caja programmer writes, for example, "x + y", where x or y are supplied by another Caja module, this may cause a (surprising and thereby hazardous) control transfer to code determined by that other module. The reason is JavaScript's coercion rules that may cause implicit calls to valueOf() and/or toString() methods.

Our current plans, to fix other bugs, is to prohibit binding of valueOf(), and to allow binding of toString() only to functions as wrapped by asSimpleFunc() or asXo4a() or something. However, because we will still allow toString() to be bound to Caja-determined code, the hazard of the implicit control-flow transfer will remain.