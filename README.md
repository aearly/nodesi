[![Build Status](https://travis-ci.org/Schibsted-Tech-Polska/nodesi.svg?branch=master)](https://travis-ci.org/Schibsted-Tech-Polska/nodesi)
[![Coverage Status](https://coveralls.io/repos/Schibsted-Tech-Polska/nodesi/badge.svg)](https://coveralls.io/r/Schibsted-Tech-Polska/nodesi)
[![Dependency status](https://david-dm.org/Schibsted-Tech-Polska/nodesi.svg)](https://david-dm.org/Schibsted-Tech-Polska/nodesi)

## What is this?

It's a subset of [Edge Side Include](http://www.akamai.com/html/support/esi.html) standard implemented with [promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)-based interface.

## Features

* Support for <esi:include> tags
* Out of the box Express support
* Custom logging
* Lots of good stuff like caching, retires, request collapsing and such provided by [Good Guy HTTP](https://github.com/Schibsted-Tech-Polska/good-guy-http)

...and more, take a look at test cases for complete list.

## Installation

```npm install nodesi```
   
## Usage

#### Basic:
```javascript
    var ESI = require('nodesi');

    var esi = new ESI();
    esi.process('<esi:include src="http://full-resource-path/stuff.html" />').then(function(result) {
        // result is a fetched html
    });
```

#### As Express middleware:
```javascript
    var esiMiddleware = require('nodesi').middleware;
    var app = require('express')();

    // inject the middleware before your route handlers
    app.use(esiMiddleware());
```

All the ESI constructor options described below are also applicable for middleware function.
Just pass them like that: `esiMiddleWare({baseUrl: ...});`

If you'd like to pass options like headers to ESI middleware, use `req.esiOptions` object:
```javascript
...
    app.use(esiMiddleware());

    app.get('/example', function(req, res) {
        req.esiOptions = {
            headers: {
                'Authorization': 'Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ=='
            }
        };
        res.render('example');
    });
```

#### With base URL for relative paths:
```javascript
    var ESI = require('nodesi');

    var esi = new ESI({
        baseUrl: 'http://full-resource-path'
    });
    esi.process('<esi:include src="/stuff.html" />').then(function(result) {
        // result is a fetched html
    });
```

#### With headers:
```javascript
    var ESI = require('nodesi');

    var esi = new ESI({
        baseUrl: 'http://full-resource-path'
    });
    esi.process('<esi:include src="/stuff.html" />', {
        headers: {
            'Authorization': 'Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ=='
        }
    }).then(function(result) {
        // result is a fetched html
    });
```

## Error handling

You can provide onError callback to a ESI constructor. It will recieve two arguments: source URL and error object.

It should return a string that will be put in place of errorous content.

#### Example
```javascript
    var esi = new ESI({
        onError: function(src, error) {
            if(error.statusCode === 404) {
                return 'Not found';
            }
            return '';
        }
    });
```

## Logging

It's a common anti-pattern that libraries write to stdout w/o users permission. 

We want to be nice so you can provide your own logging output with ```logTo``` configuration option. 

It's expected to be an object with "write" method on it that accepts a single string.


#### Examples

Logging to a custom object
```javascript
    var esi = new ESI({
        logTo: {
            write: function(log) {
                // do some stuff with log string here
            }
        }
    });
```

Logging to a standard output (same as console.log):
```javascript
    var esi = new ESI({
        logTo: process.stdout
    });
```

Logging to a file (possible, [but please don't do that](http://12factor.net/logs)):
```javascript
    var logFile = require('fs').createWriteStream('./log.txt');
    var esi = new ESI({
        logTo: logFile
    });
```

## Performance testing

You can run performance tests with ```npm run perf [args]```

This tool assumes you have [Siege](http://www.joedog.org/siege-home/) installed and added to your Path variable.

[args] are list of arguments that will be passed to Siege.


