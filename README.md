# Welcome to Auth0 Web Task

These instructions show you the basic usage of Auth0 Web Task. For an overview of the technology please have a look at the [JSConf.AR 2014 slides](http://tjanczuk.github.io/about/sandbox.html#/). 

## What's in the box

You will receive two pieces of information when Auth0 Web Task is provisioned for you:

1. **The Web Task URL**. This is an HTTPS endpoint you use for all communication with the system. This quickstart uses https://webtsk.it.auth0.com as an example (not a real endpoint). 
2. **The key**. This is a secret key you must provide as the `key` URL query parameter to authenticate your calls to the Web Task URL. Examples below assume the key is stored in the `KEY` environment variable. 

## Hello, world

Run a web task that returns a string on behalf of *tenant1*: 

```
curl https://webtsk.it.auth0.com/tenant1?key=$KEY --data-binary '
return function (cb) { 
  cb(null, "Hello, world!"); 
}
'
```

## Programming model

### Web Task endpoint

The Auth0 Web Task exposes a single HTTPS POST endpoint that is used to submit code for execution:

```
POST /{tenant}?key={webtask_key}
```

For example: https://webtsk.it.auth0.com/tenant17?key=abc123

The {tenant} URL path segment is an arbitrary string that defines the isolation boundary for code execution. Calls made with the same {tenant} value MAY share the same execution environment, possibly including residual effects of prior or concurrent calls *from the same tenant*. Calls made with different {tenant} values WILL NEVER share the same execution environment, and are isolated from each other in terms of network, memory, and CPU utilization. 

The choice of {tenant} values and their mapping to higher level tenancy concepts is application specific and defined at the layer above the Auth0 Web Task. 

### Simple closure

Submit an HTTP POST request to the Web Task URL with JavaScript (Node.js) code which returns a JavaScript function that accepts **one** parameter: a *callback*. Within the function you must invoke the callback when you are done. The callback accepts two parameters: an error (if any), and a single return value which can be serizalied to JSON. For example:

```javascript
return function (cb) {
    cb(null, { a: 'Foo', b: 12 });
}
```

### Parameterized closure

Similar to the simple closure mechanism above, except the JavaScript function now accepts **two** parameters: a *context* and a *callback*. The context will contain URL query parameters specified in the call to the Web Task URL, represented as a JavaScript object. For example:

```javascript
return function (context, cb) {
    cb(null, "Hello, " + context.data.name);
}
```

When the code above is submitted to https://webtsk.it.auth0.com/tenant1?key=abc123&name=world, the response will be *Hello, world*. 

**NOTE** the context will not contain the *key* URL query parameter. 

### What about modules

The Auth0 Web Task provides a uniform execution environment for all calls. The environment contains several pre-installed Node.js modules which can be accessed with `require`. This is the list: 

```
async: ~0.9.0
bcrypt: ~0.7.5
couchbase: ~1.2.1
ip: ~0.0.1
knex: ~0.6.3
mongodb: ~1.3.15
mysql: ~2.0.0-alpha8
easy-pbkdf2: 0.0.2
jsonwebtoken: ~0.4.1
q: ~1.0.1
request: ~2.27.0
tedious: ~0.1.4
xml2js: ~0.2.8
xmldom: ~0.1.13
xpath: 0.0.5
xtend: ~1.0.3
edge: ~0.9.3
winston: ~0.8.1
azure-storage: ~0.4.1
range_check: 0.0.1
pg: ^4.1.1
node-cassandra-cql: ^0.4.4
lodash: ~2.4.1
pubnub: ^3.7.0
oauth: ~0.9.12
aws-sdk: ~2.1.13
```

One module that is worth singling out on the list is [Edge.js](http://tjanczuk.github.io/edge/#/). Edge.js enables running C# code in addition to Node.js in the Auth0 Web Task. Please refer to the documentation at http://tjanczuk.github.io/edge for more details. Here is a *Hello, world* in C# via Edge.js: 

```javascript
return function (context, cb) {
    require('edge').func(function () {/*
        async (dynamic context) => {
            return "Hello, " + context.data.name + "!";
        }
    */})(context, cb);
}
```

## Logging

Auth0 Web Task allows streaming access to real-time logging information over HTTPS. Logs are available as a stream of JSON [bunyan](https://github.com/trentm/node-bunyan) records in a long running HTTPS response. There are two kinds of logs: system-wide logs, and tenant-specific logs. 

### System wide logs

You can access streaming, system-wide logs with an HTTPS GET call to sandbox URL. Take note of the path and the secret key:

```
curl -N -s https://webtsk.it.auth0.com/logs/system?key=$KEY
```

If you want to make the output easy on the eye, you can filter through the `bunyan` client: 

```
npm install bunyan -g
curl -N -s https://webtsk.it.auth0.com/logs/system?key=$KEY | bunyan
```

### Tenant logs

Tenant logs contain only the output generated to stdout and stderr by the custom code running on behalf of that tenant. Logs generated across several calls made on behalf of a single tenant are consolidated into a single stream.

```
curl -N -s https://webtsk.it.auth0.com/logs/tenant/tenant23?key=$KEY | bunyan
```

The last URL path segment denotes the tenant identifier of the tenant to get logs for. 

### Customizing logging

The following URL query parameters can be used to customize the behavior of logging endpoint:

`max` (optional). Maximum number of messages to return. If not provided, there is no limit. 

`timeout` (optional). Inactivity timeout in milliseconds. If the timeout is exceeded without a message being sent, the server will close the response. If not provided, the server never closes the response. 

`offset` (optional). A Kafka offset starting from which the messages will be replayed. This allows obtaining historical data. If not provided, streaming will start from the moment the connection request is made. 

`f.msg` (optional). A regular expression to match the text of the log entry against. Only matching messages will be sent. If the value starts with `!` only messages that *do not* match the regular expression are returned. Use `!!` to escape the starting exclamation mark. 

`f.{any_property}` (optonal). Only bunyan records containing {any_property} with the specified value will be returned.

For example: 

```
curl -N -s https://webtsk.it.auth0.com/logs/tenant/tenant4?key=$KEY\&f.msg=Hello\&max=100\&timeout=1000
```

Will return up to 100 messages for tenant `tenant4` that contain the string `Hello`. If no messages are generated within any 1s interval, the server will close the response. 

## Feedback and support

File issues at https://github.com/auth0/auth0-sandbox-quickstart/issues

## Issue Reporting

If you have found a bug or if you have a feature request, please report them at this repository issues section. Please do not report security vulnerabilities on the public GitHub issue tracker. The [Responsible Disclosure Program](https://auth0.com/whitehat) details the procedure for disclosing security issues.

## Author

[Auth0](auth0.com)

## License

This project is licensed under the MIT license. See the [LICENSE](LICENSE) file for more info.
