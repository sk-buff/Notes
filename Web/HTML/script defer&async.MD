The difference between `defer` and `async` tag in the `script` tag is like the following [1]:

![script defer&async_1](script defer&async_1.png)

Here are some supplements:

For the `defer` attribute, the HTML5 specification indicates that scripts with it will be executed in the order in which they appear, so the first deferred script executes before the second deferred script, and both will execute before the `DOMContentLoaded` event. In reality, though, deferred scripts don’t always execute in order or before the `DOMContentLoaded` event, so it’s best to include just one when possible.

For the `async` attribute, unlike `defer`, scripts marked with it are not guaranteed to execute in the order in which they are specified. Asynchronous scripts are guaranteed to execute before the page’s `load` event and may execute before or after `DOMContentLoaded`.