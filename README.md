# mysql2 serverless-plugin-optimize bug

This repo serves to show an issue with the npm modules serverless-plugin-optimize and mysql2.

It appears due to mysql2 _requiring_ it's internal dependencies dynamicly, serverless-plugin-optimize does not see that it needs the files, and removes them from the compiled bundle.
This was done with:
- serverless 1.12.1
- serverless-plugin-optimize 3.0.4-rc.1
- mysql2 1.5.1

You can reproduce it by cloning this repo and running:

```
npm install
sls invoke local -f test
```

You will get a stack trace that looks like the following:
```
andrew@7fbc75a98c89:~/repos/mysql2-serverlessoptimize-test$ sls invoke local -f test
Serverless: Optimize: starting engines
Serverless: Optimize: aws-nodejs-dev-test
{ Error: Cannot find module './auth_switch_request.js'
    at Function.Module._resolveFilename (module.js:469:15)
    at Function.Module._load (module.js:417:25)
    at Module.require (module.js:497:17)
    at require (internal/module.js:20:19)
    at s (/home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:1:543)
    at /home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:1:734
    at /home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:10345:16
    at Array.forEach (native)
    at Object.36../packet (/home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:10344:4)
    at s (/home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:1:683)
    at /home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:1:734
    at Object.27../commands/index.js (/home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:6087:15)
    at s (/home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:1:683)
    at /home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:1:734
    at Object.25../lib/connection.js (/home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:6005:18)
    at s (/home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:1:683)
    at /home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:1:734
    at Object.53.mysql2 (/home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:13564:15)
    at s (/home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:1:683)
    at e (/home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:1:854)
    at /home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:1:872
    at a (/home/andrew/repos/mysql2-serverlessoptimize-test/_optimize/aws-nodejs-dev-test/test.js:1:150) code: 'MODULE_NOT_FOUND' }
```





## References:

The issue in mysql2 https://github.com/sidorares/node-mysql2/blob/5f0fb8f1f5035e2c0207490aa2f0b838dc82fdc2/lib/packets/index.js

```
'auth_switch_request auth_switch_response auth_switch_request_more_data binlog_dump register_slave ssl_request handshake handshake_response query resultset_header column_definition text_row binary_row prepare_statement close_statement prepared_statement_header execute change_user'
  .split(' ')
  .forEach(function(name) {
    var ctor = require('./' + name + '.js');
    module.exports[ctor.name] = ctor;
    // monkey-patch it to include name if debug is on
    if (process.env.NODE_DEBUG) {
      if (ctor.prototype.toPacket) {
        var old = ctor.prototype.toPacket;
        ctor.prototype.toPacket = function() {
          var p = old.call(this);
          p._name = ctor.name;
          return p;
        };
      }
    }
  });
```

