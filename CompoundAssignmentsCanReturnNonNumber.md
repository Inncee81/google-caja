## Effect ##

Properties that should not be readable may be read (and possibly written). It may be possible to combine this with other attack vectors such as EvalArbitraryCodeExecution to run arbitrary code.


## Assumptions ##

  * In a JavaScript subset, properties named by stringified numbers are implicitly readable (and possibly writable).
  * The subset implementation assumes that a compound assignment expression other than += always evaluates to a number, or that += evaluates to a (number or string), or that a simple assignment evaluates to its right-hand-side.
  * The JavaScript interpreter incorrectly implements assignments, so that they may return a value that has been coerced to a different type.
  * An object that performs such a coercion when its properties are assigned is directly accessible to code written in the subset language (either untrusted code, or a tamed API implementation that can be exploited as a confused deputy).
  * The subset allows LeftHandSideExpressions that might refer to a property of such an object.


## Background ##

The compound assignment operators (+=, `*`=, /=, %=, -=, <<=, >>=, >>>=, &=, ^=, |=) are specified to return the value computed by the corresponding operator. This should always be a number, or a (number or string) in the case of +=. Caja, ADsafe, and Jacaranda unconditionally allow reading of properties named by "stringified numbers", that is, strings that can be the result of ToString(ToNumber(x)) for some x.

However, in some versions of SpiderMonkey, if `obj.prop `_op_`= rhs` is a compound assignment then for certain properties `obj.prop`, the assigned value `obj.prop `_op_` rhs` can be coerced to a value of a different type, which is then incorrectly used as the result of the assignment operator. (The problem is not the coercion, but the fact that the assignment evaluates to the coerced value.) See [Mozilla bug 312354](https://bugzilla.mozilla.org/show_bug.cgi?id=312354).

For example, `typeof(window.location += '')` or `typeof(window.location -= 0)` will return `'object'`, when they should return `'string'`. This happens because the `window` object coerces its `location` property to type object.

This means that a property access of the form `a[x.prop -= 0]`, for example, might be allowed under the assumption that it is accessing a stringified number property, but actually access a different property that should not have been readable (or writable when the access is a LeftHandSideExpression).

The same problem occurs for a simple assignment, if the subset implementation assumes that simple assignments evaluate to their right-hand-side.

The potential for an exploit is limited by the fact that most secure subsets attempt not to grant direct access to host objects. However, even some non-host objects, such as RegExp and Array objects, have coercing properties in SpiderMonkey.

The same result can be achieved with a [watcher](https://developer.mozilla.org/en/Core_JavaScript_1.5_Reference/Global_Objects/Object/watch) or setter on non-host objects in SpiderMonkey (see examples below); however, most subsets block the `'watch'` and `'__defineSetter__'` methods.

In Jacaranda, this issue is not exploitable (despite the fact that RegExp objects are accessible) because only local variables and properties of `this` can be used as LeftHandSideExpressions, and `this` must refer to an object created by an object literal.

In Caja and ADsafe, it is not exploitable because currently there is no reliance on compound assignments evaluating to a number or simple assignments evaluating to their right-hand-side. (CHECK for Caja) (CHECK for ADsafe)


## Versions ##
  * SpiderMonkey in Firefox 3.0.4 and earlier.
  * Apparently fixed in SpiderMonkey trunk.


## Examples ##

```
  foo[window.location -= 0];     // accesses a property of foo named by the current URL
```
```
  (<xml/>.x = 0) + 0;            // returns "00", not 0 (depends on E4X)
```
```
  typeof (/a/.lastIndex = "7");  // returns "number", not "string"
```
```
  var obj = {};
  obj.watch(
      'foo',
      function (prop, oldval, newval) {
        return typeof newval === 'number' && isNaN(newval) ? oldval : newval;
      });
  obj.foo = 'foo';
  (obj.foo -= 1);  // returns 'foo', not NaN
```

The following test script is from the Mozilla bug report:
```
var gprop = [];
var p = "length";

function test0()
{
  function inner()
  {
    var r;
    return typeof (r = this[p] = "5");
  }

  var thisp = [];
  return inner.call(thisp);
}

function test1(argvar)
{
  var r;
  return typeof (r = argvar = "5");
}

function test2()
{
  var r;
  return typeof (r = gprop.length = "5");
}

this.__defineSetter__("topSet", function(v) { return 17; });

function test3()
{
  var r;
  return typeof (r = topSet = "5");
}

function test4()
{
  var r, c = [];
  return typeof (r = c[p] = "5");
}

function test5()
{
  var r, local = [];
  return typeof (r = local.length = "5");
}

function test6()
{
  function inner()
  {
    var r;
    return typeof (r = this.length = "5");
  }

  var thisp = [];
  return inner.call(thisp);
}

function test7()
{
  if (typeof (<foo/>.x = 5) == "number")
    return "string";
  return "number got converted!";
}

function test8()
{
  if (typeof (<foo/>.x = true) == "boolean")
    return "string";
  return "boolean got converted!";
}

var allTests = [test0, test1, test2, test3, test4, test5, test6, test7, test8];

for (var i = 0; i < allTests.length; i++)
{         
  if ("string" !== allTests[i]())
    throw "fail test" + i + "()";
}
```