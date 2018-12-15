# docker-tester 

[![](https://img.shields.io/npm/v/docker-tester.svg?style=for-the-badge)](https://www.npmjs.com/package/docker-tester)
[![](https://img.shields.io/david/mintzo/docker-tester.svg?style=for-the-badge)](https://www.npmjs.com/package/docker-tester)
![](https://img.shields.io/npm/l/docker-tester.svg?style=for-the-badge)

[![NPM](https://nodei.co/npm/docker-tester.png)](https://nodei.co/npm/docker-tester/)

Set up a testing environment with a docker-compose file and verify its up before running tests

-----------
## Install

```sh
npm i docker-tester --save-dev
```
*  _docker and docker-compose are required to be installed and acsecible from the terminal, you can [get it here](https://docs.docker.com/compose/install/)_

## Example
### running tests in mocha
```js
/// mocha_test_file.spec.js

const TestingEnvironment = require('docker-tester');

const testingEnvironment = new TestingEnvironment({
  dockerComposeFileLocation: __dirname,
  dockerFileName: 'test.docker-compose.yml',
  verifications: { 
    httpServer: { 
      promise: async (service) => {
        // check that service is up (usually http request), reject if not ready
      },  promiseRetryOptions: { retries: 4 } }
  } });

(
  async () => {
  await testingEnvironment.start();
  run();  //starts mocha tests
})();

// stopping environment after tests are done
after(async function () {
  this.timeout(0);
  await testingEnvironment.stop();
});

describe('Simple Usage', () => {
  it('some tests', () => {
    // test code only runs after environment is ready
  });
});
```
### docker-compose file
```yaml
# test.docker-compose.yml
version: '3.1'
services:
  example-node-server:
    image: node
    ports:
      - 7000:80
    environment:
      verificationType: httpServer # verification type for service 
  example-mongo:
    image: mongo
    ports:
      - 7001:80
    environment:
      verificationType: mongodb
```

## Usage
create a new TestingEnvironment instance, ```.start()``` and ```.stop()``` async function, use ```docker-compose up``` and ```docker-compose down```

```.stop()``` resolves when all containers have stopped.

```.start()``` resolves when all containers are up and ready.

in the docker-compose file, services requiring verification that they are ready will be verified according to there defined verification type, found under environment -> verificationType

TestingEnvironment instance will match verifications key to ```verificationType``` in the docker-compose file.


## Documentation
### TestingEnvironment() Constructor
the testing environment can be configured by passing in an object with the fallowing properties

required parameters:
* ```dockerComposeFileLocation``` - the folder path where the docker-compose file is found
* ```dockerFileName``` - the docker-compose full file name

optional:

* ```verifications``` - verifications by type that check when services are ready
  * ```promise``` - *required* , the promise function to verify the service, receives the service information when called
  * ```promiseRetryOptions``` - _(optional)_ - promise retry settings, same as [promise-retry](https://www.npmjs.com/package/promise-retry)
    * ```retries``` - number of retries , _default 5_
* ```enableLogs``` - logs to console docker-tester actions, when set to ```true```

example options object:
```js
new TestingEnvironment({
  dockerComposeFileLocation: __dirname,
  dockerFileName: 'test.docker-compose.yml',
  verifications = {
  verificationType: { // the verification to run matching the verificationType in the docker-compose file
    promise,
    promiseRetryOptions
  }
}
```

### .start({ stopIfUp, verifyUp })
starts all services found in the docker-compose file _(``docker-compose up -d``)_, verifies they are ready and then resolves, rejects if there was a problem or if verify promises are rejected

optional settings:

* ```stopIfUp``` - _(default: true)_  - runs ```.stop()``` before starting services
* ```verifyUp``` - _(default: true)_ - runs ```.verifyAllServices()``` after starting services

example code:
```js
const testingEnvironment = new TestingEnvironment({
  // required options...
});

await testingEnvironment.start();
```

### .stop()
stops all services running services _(``docker-compose down``)_ then resolves,rejects if there was a problem or if verify promises are rejected.

example code:
```js
const testingEnvironment = new TestingEnvironment({
  // required options...
});

await testingEnvironment.start();
await testingEnvironment.stop();
```

### .verifyAllServices()
verifies all services are ready using the service ```verificationType``` then resolves,rejects if there was a problem or if verify promises are rejected.

example code:
```js
const testingEnvironment = new TestingEnvironment({
  // required options...
});

await testingEnvironment.start({ verifyUp: false });
await testingEnvironment.verifyAllServices();
```
