# Table of Contents

<!--ts-->

- [Setup tests in npm project using jest](#setup-tests-in-npm-project-using-jest)
- [Init nodejs project using npm](#init-nodejs-project-using-npm)
- [Install jest](#install-jest)
- [Install other dependencies](#install-other-dependencies)
- [Write functions](#write-functions)
- [Write tests for functions](#write-tests-for-functions)
- [Run Test](#run-test)

<!--te-->

# Setup tests in npm project using jest

In this article, I will demonstrate how to set up `Jest`, a popular testing framework, to run tests in a Node.js project. I will guide you through the process of installing Jest, configuring it for your project, and running sample tests to ensure everything is set up correctly. By the end of this article, you will have a fully functional Jest setup for testing your Node.js code.

# Init nodejs project using npm

First, let's set up a Node.js project to organize the code. We will use npm to initialize the project by running the command `npm init` in your project's root directory.

```bash
mkdir -p npm-test; cd npm-test; npm init --yes
```

The `--yes` flag is used to automatically accept the default values for all prompts. This will generate a package.json file with default values for the package name, version, description, entry point, test command, repository, author, and license.

Output:

```
Wrote to /Users/naonao/tmp/kkkk/npm-test/package.json:

{
  "name": "npm-test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

# Install jest

[Jest](https://jestjs.io/) is a popular JavaScript testing framework developed by Facebook.

> Jest is a delightful JavaScript Testing Framework with a focus on simplicity.

It provides a complete solution for running tests, making assertions, mocking dependencies, and generating code coverage reports. Jest's key features include a descriptive syntax, a wide range of built-in matchers, support for snapshot testing, simplified testing of asynchronous code, and extensive configuration options. It is widely used and highly regarded for its simplicity, comprehensive documentation, and powerful testing capabilities

```bash
npm install --save-dev jest
```

# Install other dependencies

We can install any dependencies using `npm install <library>`. Below is an example of adding `dayjs` as dependency.

```bash
npm install dayjs
```

# Write functions

Now, we'll write a function to convert different relative time formats (`now-15m`, `now-1h`, `now-24h`, `now-7h`) to their respective absolute time representations. The function looks like this:

```javascript
const dayjs = require("dayjs");

const getAbsoluteTime = (relativeTime) => {
  if (relativeTime.includes("now")) {
    let parsedDate = dayjs();

    if (relativeTime.includes("m")) {
      const relativeValue = parseInt(
        relativeTime.split("now-")[1].split("m")[0]
      );
      parsedDate = parsedDate.subtract(relativeValue, "minute");
    } else if (relativeTime.includes("h")) {
      const relativeValue = parseInt(
        relativeTime.split("now-")[1].split("h")[0]
      );
      parsedDate = parsedDate.subtract(relativeValue, "hour");
    } else if (relativeTime.includes("d")) {
      const relativeValue = parseInt(
        relativeTime.split("now-")[1].split("d")[0]
      );
      parsedDate = parsedDate.subtract(relativeValue, "day");
    }

    return parsedDate.format("YYYY-MM-DDTHH:mm");
  }

  return null;
};

module.exports = getAbsoluteTime;
```

We'll save it in `getAbsoluteTime.js` file in the project's root directory.

# Write tests for functions

Now, let's write tests for this simple function.

```javascript
const dayjs = require("dayjs");
const getAbsoluteTime = require("../getAbsoluteTime"); // Assuming the function is exported from a separate file

describe("getAbsoluteTime", () => {
  it('should return the correct absolute time for relative time "now-15m"', () => {
    const relativeTime = "now-15m";
    const absoluteTime = getAbsoluteTime(relativeTime);
    const currentTime = dayjs(); // Get the current time using dayjs
    const expectedTime = currentTime
      .subtract(15, "minute")
      .format("YYYY-MM-DDTHH:mm");
    expect(absoluteTime).toBe(expectedTime);
  });

  it('should return the correct absolute time for relative time "now-1h"', () => {
    const relativeTime = "now-1h";
    const absoluteTime = getAbsoluteTime(relativeTime);
    const currentTime = dayjs(); // Get the current time using dayjs
    const expectedTime = currentTime
      .subtract(1, "hour")
      .format("YYYY-MM-DDTHH:mm");
    expect(absoluteTime).toBe(expectedTime);
  });

  it('should return the correct absolute time for relative time "now-24h"', () => {
    const relativeTime = "now-24h";
    const absoluteTime = getAbsoluteTime(relativeTime);
    const currentTime = dayjs(); // Get the current time using dayjs
    const expectedTime = currentTime
      .subtract(24, "hour")
      .format("YYYY-MM-DDTHH:mm");
    expect(absoluteTime).toBe(expectedTime);
  });

  it('should return the correct absolute time for relative time "now-7h"', () => {
    const relativeTime = "now-7h";
    const absoluteTime = getAbsoluteTime(relativeTime);
    const currentTime = dayjs(); // Get the current time using dayjs
    const expectedTime = currentTime
      .subtract(7, "hour")
      .format("YYYY-MM-DDTHH:mm");
    expect(absoluteTime).toBe(expectedTime);
  });
});
```

Save this file in `tests/getAbsoluteTime.test.js` and we are ready to run test using `jest`.

# Run Test

Before running any tests, you have to add `"scripts" : {"test": "jest"}` to `package.json` as follows:

```json
{
  // ...
  "scripts": {
    "test": "jest"
  }
  // ...
}
```

Now, you can run tests using `npm run test` command.

Output:

```bash
> relative-time-to-absolute-time@1.0.0 test
> jest

 PASS  tests/getAbsoluteTime.test.js
  getAbsoluteTime
    ✓ should return the correct absolute time for relative time "now-15m" (2 ms)
    ✓ should return the correct absolute time for relative time "now-1h"
    ✓ should return the correct absolute time for relative time "now-24h" (1 ms)
    ✓ should return the correct absolute time for relative time "now-7h"

Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
Snapshots:   0 total
Time:        0.149 s, estimated 1 s
Ran all test suites.
```
