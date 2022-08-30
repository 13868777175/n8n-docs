# Item linking errors

You can access data items from earlier in a workflow from the Function node, or from the expressions editor in any node, using the built-in [node output](/code-examples/methods#node-output) methods.

If you encounter the following error:

> ERROR: Invalid expression under 'Could not find parameter "undefined"'
> The expression uses data in the node '<node-name>' but there is no path back to it. Please check this node is connected to it (there can be other nodes in between).

Check that:

* `<node-name>` spelling is correct.
* The node you're referring to exists in the workflow.
* The node you're referring to is part of the same path as the current node.
