# webworkify-webpack

launch a web worker that can require() in the browser with webpack

inspired by [webworkify](https://github.com/substack/webworkify)

# example

First, a `main.js` file will launch the `worker.js` and print its output:

``` js
var work = require('webworkify-webpack');

var w = work(require('./worker.js'));
w.addEventListener('message', function (ev) {
    console.log(ev.data);
});

w.postMessage(4); // send the worker a message
```

then `worker.js` can `require()` modules of its own. The worker function lives
inside of the `module.exports`:

``` js
var gamma = require('gamma');

module.exports = function (self) {
    self.addEventListener('message',function (ev){
        var startNum = parseInt(ev.data); // ev.data=4 from main.js
        
        setInterval(function () {
            var r = startNum / Math.random() - 1;
            self.postMessage([ startNum, r, gamma(r) ]);
        }, 500);
    });
};
```

Now after [webpackifying](https://webpack.github.io) this example, the console will
contain output from the worker:

```
[ 4, 0.09162078520553618, 10.421030346237066 ]
[ 4, 2.026562457360466, 1.011522336481017 ]
[ 4, 3.1853125018703716, 2.3887589540750214 ]
[ 4, 5.6989969260510005, 72.40768854476167 ]
[ 4, 8.679491643020487, 20427.19357947782 ]
[ 4, 0.8528139834191428, 1.1098187157762498 ]
[ 4, 8.068322137547542, 5785.928308309402 ]
...
```

# methods

``` js
var work = require('webworkify-webpack')
```

## var w = work(require(modulePath))

Return a new
[web worker](https://developer.mozilla.org/en-US/docs/Web/API/Worker)
from the module at `modulePath`.

The file at `modulePath` should export its worker code in `module.exports` as a
function that will be run with no arguments.

Note that all the code outside of the `module.exports` function will be run in
the main thread too so don't put any computationally intensive code in that
part. It is necessary for the main code to `require()` the worker code to fetch
the module reference and load `modulePath`'s dependency graph into the bundle
output.

# install

With [npm](https://npmjs.org) do:

```
npm install webworkify-webpack
```

# caveats

* While developing make sure not to set webpack's devtool option to be with some eval configuration because from 1.1.0 version it won't work.
* When passing anonymous functions to webworkify-webpack, naming them can prevent potential issues - [#9](https://github.com/borisirota/webworkify-webpack/issues/9), [#10](https://github.com/borisirota/webworkify-webpack/issues/10)

# license

MIT
