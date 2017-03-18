# Webtask Compilers

A collection of useful [webtask compilers](https://webtask.io/docs/webtask-compilers) for use with [Auth0 Webtasks](https://webtask.io). 

All you need to use the features below is [the webtask CLI](https://webtask.io/cli). 

## Static compiler

Host static content (HTML, JS files, CSS) on webtasks and serve over HTTP GET along with a configurable set of HTTP response headers (e.g. Content-Type). 

**NOTE** Serving static content using webtasks is not a good idea in production systems, there are far more efficient and performant ways of doing it. But it is a convenient way to serve frequenty changing static content during development.

Webtask script: 

```
cat > page.txt <<EOF
This is text, but it could also be 
HTML, JSON, CSS, or JavaScript.
EOF
```

Create webtask using static compiler, specify custom HTTP response headers using webtask secrets:

```
wt create page.txt --name page \
  --meta wt-compiler=https://raw.githubusercontent.com/tjanczuk/wtc/master/static.js \
  -s content-type=text/plain
```

## Webtask context extension

This compiler demonstrates how the webtask context can be enhanced with additional data or properties before the user-defined webtask function is called. In this example, the compiler will fetch the content from a URL specified via the `DATA_URL` secret and add it to the context object where the webtask function can immediately use it. Similar mechanism can be used to add data from an external database, or add any other utility functions or properties to the webtask context. 

Webtask script: 

```
cat > webtask.js <<EOF
module.exports = function (ctx, cb) {
  // The `ctx.externalData` property will be added by the compiler
  cb(null, { length: ctx.externalData.length });
};
EOF
```

Create webtask using [extend_context.js](https://github.com/tjanczuk/wtc/blob/master/extend_context.js) compiler and specify the `DATA_URL` secret to indicate where the compiler should download external content from: 

```
wt create webtask.js \
  --meta wt-compiler=https://raw.githubusercontent.com/tjanczuk/wtc/master/extend_context.js \
  -s DATA_URL=https://google.com
```

## ES6 classes as webtasks

This compiler demonstrates how a JavaScript class can be used as a programming model for webtasks with simple built-in dispatch mechanism: 

* constructor of the class is called once during initialization call and provided with webtask secrets and metadata, 
* requests are dispatched to instance methods based on the HTTP verb of the request (you can easily modify dispatch logic to determine method to call using other criteria).

Webtask script: 

```
cat > webtask.js <<EOF
'use strict';

module.exports = class MyWebtask {

  constructor(secrets, meta) {
    this.secrets = secrets;
    this.meta = meta;
  }
  
  get(ctx, cb) {
    cb(null, { hello: 'from get' });
  }  
  
  post(ctx, cb) {
    cb(null, { hello: 'from post' });
  }
  
  patch(ctx, cb) {
    cb(null, { hello: 'from patch' });
  }

  // if an HTTP verb is not defined, compiler responds with HTTP 405
};
EOF
```

Create webtask using [class_compiler.js](https://github.com/tjanczuk/wtc/blob/master/class_compiler.js) compiler: 

```
wt create webtask.js \
  --meta wt-compiler=https://raw.githubusercontent.com/tjanczuk/wtc/master/class_compiler.js
```
