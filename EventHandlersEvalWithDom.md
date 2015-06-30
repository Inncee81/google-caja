Reported by Gareth

# Event handlers execute in a scope that binds globals to DOM Properties #

## Effect ##
A DOM node may be manipulated by global assignment in a way that could execute unsanitized javascript.


## Background ##
DOM Level 2 defines [event handlers](http://www.w3.org/TR/DOM-Level-3-Events/events.html#Events-eventgroupings) to allow scripts to fire when an OS event occurs in the context of a node.

These handlers are typically snippets of javascript that execute in an environment that provides access to the `event` being handled, and bind `this` to the DOM node.

Some browsers also provide global bindings for DOM node fields.


## Assumptions ##
DOM event handlers are allowed and execute in the normal scope and global assignment is allowed.


## Versions ##
IE


## Example ##
```
<a onclick="URL=name" name="javascript:alert('hello')">clicky</a>
```