Debugging Protractor Tests
==========================

End-to-end tests can be difficult to debug because they depend on an entire
system, may depend on prior actions (such as log-in), and may change the
state of the application they're testing. WebDriver tests in particular
can be difficult to debug because of long error messages and the separation
between the browser and the process running the test.

Types of Failure
----------------

Protractor comes with examples of failing tests ([failure_spec.js](https://github.com/angular/protractor/blob/master/debugging/failure_spec.js)).
To run, start up the test application and a Selenium Server, and run the command below. Then look at all the stack traces.

```
protractor debugging/failureConf.js
```

This test suite shows various types of failure:

-  WebDriver throws an error - When a command cannot be completed, for example
   an element is not found.
-  Protractor will fail when it cannot find the Angular library on a page.
   If your test needs to interact with a non-angular page, access the WebDriver
   instance directly with `browser.driver`.
-  Expectation Failure - Shows what a normal expectation failure looks
   like.

Pausing to Debug
----------------

Protractor allows you to pause your test at any point and interact with the
browser. To do this insert `browser.debugger();` into your test where you want
to break:

```javascript
it('should fail to find a non-existent element', function() {
  browser.get('app/index.html#/form');

  // Run this statement before the line which fails. If protractor is run
  // with the debugger (protractor debug <...>), the test
  // will pause after loading the webpage but before trying to find the
  // element.
  browser.debugger();

  // This element doesn't exist, so this fails.
  var nonExistant = element(by.binding('nopenopenope'));
});
```

Then run the test in debug mode:

```
protractor debug debugging/failureConf.js
```

This example uses the [node debugger](http://nodejs.org/api/debugger.html). Enter
`c` to start execution and continue after the breakpoint.

We use `browser.debugger();` instead of node's `debugger;` statement so that
the test pauses after the get command has been executed. Using `debugger;`
pauses the test after the get command is scheduled but has not yet
been sent to the browser.

Protractor's `debugger` method works by scheduling a node debug breakpoint
on the control flow.

When `debugger()` is called, it also inserts all the client side scripts
from Protractor into the browser as `window.clientSideScripts`. They can be
used from the browser's console.

```javascript
// In the browser console (e.g. from Chrome Dev Tools)
> window.clientSideScripts.findInputs('username');
// Should return the input element with model 'username'.

// You can also limit the scope of the locator
> window.clientSideScripts.findInputs('username', document.getElementById('#myEl'));
```


Setting Up WebStorm for Debugging
---------------------------------

To set up WebStorm for Protractor, do the following:

1. Open the Run/Debug Configurations dialog
2. Add new Node.js configuration.
3. On the Configuration tab set:
 - **Node Interpreter**: path to node executable
 - **Working directory**: your project base path
 - **JavaScript file**: path to Protractor cli.js file (e.g. *node_modules\protractor\lib\cli.js*)
 - **Application parameters**: path to your Protractor configuration file (e.g.
 *protractorConfig.js*)
4. Click OK, place some breakpoints, and start debugging.


Testing Out Protractor Interactively
------------------------------------

When debugging or first writing test suites, you may find it helpful to
try out Protractor commands without starting up the entire test suite. You can
do this with the element explorer.

Currently, the explorer runs only with chrome and expects a standalone Selenium
Server to be running at http://localhost:4444 (see [Setting Up the Selenium Server](/docs/server-setup.md)).

From the Protractor directory, run with:

    node ./bin/elementexplorer.js <urL>

This will load up the URL on WebDriver and put the terminal into a REPL loop.
You will see a > prompt. The `browser`, `element` and `protractor` variables will
be available. Enter a command such as:

    > element(by.id('foobar')).getText()

or

    > browser.get('http://www.angularjs.org')

To get a list of functions you can call, try:

    > browser

Typing tab at a blank prompt will fill in a suggestion for finding
elements.

Taking Screenshots
------------------

WebDriver can snap a screenshot with `browser.takeScreenshot()`.
This returns a promise which will resolve to the screenshot as a base-64
encoded PNG.

Sample usage:
``` javascript
// at the top of the test spec:
var fs = require('fs');

// ... other code

// abstract writing screen shot to a file
function writeScreenShot(data, filename) {
    var stream = fs.createWriteStream(filename);

    stream.write(new Buffer(data, 'base64'));
    stream.end();
}

// ...

// within a test:
browser.takeScreenshot().then(function (png) {
    writeScreenShot(png, 'exception.png');
});
```

Timeouts
--------

There are several ways that Protractor can time out. See the [Timeouts](/docs/timeouts.md)
reference for full documentation.
