# Project rest-in-contract

## Project Page

- Project rest-in-contract's Homepage: [http://blog.airic-yu.com/2062/project-rest-in-contract](http://blog.airic-yu.com/2062/project-rest-in-contract)
- Github: [https://github.com/airicyu/rest-in-contract-project](https://github.com/airicyu/rest-in-contract-project)

### Related Projects

- rest-in-contract (Contract Server nodejs module)
  - Module Homepage: [http://blog.airic-yu.com/2064/rest-in-contract-nodejs-module-for-rest-api-contract-server](http://blog.airic-yu.com/2064/rest-in-contract-nodejs-module-for-rest-api-contract-server)
  - Github: [https://github.com/airicyu/rest-in-contract](https://github.com/airicyu/rest-in-contract)
  - NPM: [https://www.npmjs.com/package/rest-in-contract](https://www.npmjs.com/package/rest-in-contract)

# What is rest-in-contract

## Consumer-driven contracts

`rest-in-contract` is a product to let you embrace **[Consumer-driven contracts](https://martinfowler.com/articles/consumerDrivenContracts.html)**. It is REST in nature so that it fits for integrating with all kind of programming languages.

### General Story for REST API providers/consumers in Consumer-driven contracts

#### For REST API providers:
REST API providers can write API contracts to describe their REST API request/response formats. They can then use the contracts to do contract testing against their API implementations.

#### For REST API consumers:
REST API consumers can use the API contracts to setup stubs for local testing or drafting of API contracts.

## How does rest-in-contract different from other Consumer-driven contracts solution?

### REST in nature, cross language, easy integration

There are many Consumer-driven contracts solution existing but many of them are SDK libraries or embedded solution for stubbing or testing which may fixed in a certain language. rest-in-contract is designed in a prespective that we do not want a language fix-in solution.

rest-in-contract is a node modeule to setup a lightweight agent server which API providers/consumers can kick to start in their environment easily. No matter doing contract testing(For provider) or API stubbing(For consumer), you can always do them by calling rest-constract agent server's REST API. That's why rest-in-contract is a cross language solution for Consumer-driven contracts.

Thanks for REST in nature, it is very easy to do integration with DevOps. No need maven or gradle build. You just need node.js(v7+) installed in your environment to kick the server. All later interactions are REST API call which you can call with curl or any other HTTP client tools.

### Contract as file

Some Consumer-driven contracts solutions may let you wiring stub servers by SDK methods. Hence, the contract is writen as embedded code. Such way has difficulties for supporting different kind of programming languages.

Instead, we think that contract should be defined in a less coupling way that can be separated from your business logic codes.

The contract in rest-in-contract is described in JS script format which exporting an Contract object. We supporting some middleware function call in the contract. It also support regular expression, jsonpath etc.

An example of contract file would be like this:
```javascript
module.exports = 
{
	"name": "testing for hello contract",
    "request": {
        "method": "POST",
        "urlPath": value({stub: regex("/hello/[a-z]*"), test: "/hello/apple"}),
        "queryParameters": [{
            "name": "a", "value": "b"
        }],
        "body": {
            "test": {
                "a": value({stub: regex("[0-9]*"), test: "13579"}),
                "b": value({stub: integer({gt:0, lt:60000}), test: 24680}),
                "c": ["apple", "orange", "banana"]
            }
        },
        "headers": {
        }
    },
    "response": {
        "status": 200,
        "headers": {
            "test-header": "dummy",
            "test-regex-header": regex("\"/hello/[a-z]{3,5}\"")
        },
        "body": {
            "num" : value({stub: 56789, test: integer({gt:0, lt:60000})}),
        }
    }
}
```

Although the contract file is written in javascript in syntax, but you can just treat them as general files and stored in your projects.
It is because that your code/application would never necessary to directly interact with the contracts. You can always pass them to the Contract Server to let it do its job.

## What are the possible architecture configurations of rest-in-contract?

### Architecture Components

***Contract Server*** is a server instance which supporting Contract testing and stubbing by REST API. It is typically storing and reading contracts in local storage.

***Contract Agent Server*** is actually a Contract Server. The different is that it read contracts from remote contract repository instead of local storage.

### Architecture Configurations

We imagined that that there may be two architecture configurations of using rest-in-contract.

1) Centralized Contract Server architecture

In this mode, there would be a centralized Contract Server which serving all API provider and consumers. It is the centralized storage server for persisting all contracts in database.

API providers & consumers would setup their own Contract Agent Server in their own environment which get the contracts remotely from centralized Contract Server through REST API. Then they would use their local Contract Agent Server to do contract testing or stubbing.

2) Decentralized Contract Server architecture

In this mode, API providers would keep contracts in their own way. For examples, if their application is on Github, they may put the contracts under a folder in their source codes.
API providers can kick Contract Server in local environment to do contract testing.

On the other side, API consumers can checkout the project from Github in order to get the API contracts. Then API consumers can kick Contract Server in local environment to do stubbing.


------------------------

## Story walkthrough

#### Beginning of the Story: John(API Provider) wrote an application *Foo* which provide REST API.

John wrote an application "Foo" which the base application URL is "http://example.com/foo". It has a version path "/v1.0". And the API endpoint url is "/hello".

The API has such format:

Request:
```
POST http://example.com/foo/v1.0/hello

{
    "name": "John"
}
```

Response:
```
Hello John
```
The request body has an attribute `name` with a string value.

#### Next Story: Mary(API Consumer) is writing an application *bar* which want to consume API from John's API.

Mary want John to enhance his API to include a new integer attribute "age" in request and output it in response like this:

Request
```
POST http://example.com/foo/v2.0/hello

{
    "name": "John",
    "age": 20
}
```

Response:
```
Hello John! You are 20 years old.
```

Hence, Mary and John have a discussion and drafted an API contract like this:

```javascript
module.exports = 
{
	"name": "Support age attribute",
    "request": {
        "method": "POST",
        "urlPath": "/hello",
        "body": {
            "name": value({stub: regex("[a-zA-Z ]*"), test: "John"}),
            "age": value({stub: integer({gt:0}), test: 20})
        }
    },
    "response": {
        "status": 200,
        "body": value({
            stub: `Hello ${jsonpath("$.req.body.name")}, you are ${jsonpath("$.req.body.age")} years old.`,
            test: "Hello John! you are 20 years old."
        })
    }
}
```

This single contract would be used by both John and Mary. For John, he would use this contract to do contract testing against its implementation. For Mary, she would use this contract to generate stub for local testing.

To support both use case of contract testing and stubbing, they need to define `value(stub(...), test(...))` in the contract. The meaning of `value(stub(...), test(...))` in contract is that, the certain values which would be used for generating stub and contract testing are different.

Why different values for stubbing and testing?

For Mary, she wants stub. The stub would accept any "name" attribute which is in `[a-zA-Z ]*` pattern. It means that Mary can send a request to the stub with "name" set as "Susan" or "Sam" or any other valid names...... And the response should correctly showing the same name in the request.
The "name" attribute should support a flexible value so that Mary can test more dynamically instead of a always hard coded value. Hence, it is represented by a regular expression pattern `regex("[a-zA-Z ]*")`.

For John, he wants API test. The API test just need to define a test value which is used for testing. (Actually he can use regular expression pattern as well, but here is just for demo)
Hence, a hardcoded test value "John" is used. And the response value is also hardcoded test value.

#### Next Story: John use the contract to generate testing endpoint to test against the API contract

Firstly, John start a local Contract Server with port 8000.

And then he create(register) an App in Contract Server by this REST API call:
```
POST http://localhost:8000/api/v1/apps
Content-type: application/json

{
    "name": "Foo",
    "servers":["http://example.com:8001"],
    "basePath": "/foo"
}
```
Assume the app ID is "80a69a44-3f3b-48c1-a7d1-b34b89117e75".

For this app, John create(register) an App version "2.0" in Contract Server by this REST API call:

```
POST http://localhost:8000/api/v1/apps/80a69a44-3f3b-48c1-a7d1-b34b89117e75/versions
Content-type: application/json

{
	"v": "2.0",
	"path": "/foo/v2.0",
	"contracts": []
}
```

And then John create(register) the API contract in Contract Server by this REST API call:
```
POST http://localhost:8000/api/v1/contracts/
Content-type: application/vnd.js.contract

module.exports = 
{
	"name": "Support age attribute",
    "request": {
        "method": "POST",
        "urlPath": "/hello",
        "body": {
            "name": value({stub: regex("[a-zA-Z ]*"), test: "John"}),
            "age": value({stub: integer({gt:0}), test: 20})
        }
    },
    "response": {
        "status": 200,
        "body": value({
            stub: `Hello ${jsonpath("$.req.body.name")}, you are ${jsonpath("$.req.body.age")} years old.`,
            test: "Hello John, you are 20 years old."
        })
    }
}
```
(Assume the contract ID is "94923fbd-9092-4a46-ad65-0d8a2e2f551e")

And then John can update "foo" 's API version 1 to include this API contract. He can do it by calling this REST API:
```
PUT http://localhost:8000/api/v1/apps/80a69a44-3f3b-48c1-a7d1-b34b89117e75/versions
Content-type: application/json

{
	"v": "2.0",
	"path": "/foo/v2.0",
	"contracts": [
        "94923fbd-9092-4a46-ad65-0d8a2e2f551e"
    ]
}
```

And then John can use Contract Server's REST API to trigger contract testing against app "Foo" in his local environment (Port 8001). Once triggered, Contract Server would build mock requests according to the API contract and then send to the target API endpoint. The testing result and the whole Request/Response context information would be returned.

The Contract Test REST API would be like this:

Request:
```
POST http://localhost:8000/api/v1/apps/80a69a44-3f3b-48c1-a7d1-b34b89117e75/wiretests
Content-type: application/json

{
    "server": "http://localhost:8001"
}
```

The Response may look like this:
```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "2.0": {
    "94923fbd-9092-4a46-ad65-0d8a2e2f551e": {
      "testInfo": {
        "timeMS": 27.645099997520447,
        "success": true,
        "errors": [],
        "appId": "80a69a44-3f3b-48c1-a7d1-b34b89117e75",
        "version": "2.0",
        "contract": {
          "id": "94923fbd-9092-4a46-ad65-0d8a2e2f551e",
          "name": "Support age attribute"
        }
      },
      "request": {
        "method": "POST",
        "urlPath": "http://localhost:8001/foo/v2.0/hello",
        "queryParams": {},
        "headers": {
          "Content-type": "application/json"
        },
        "body": {
          "name" : "John",
          "age" : 20
        }
      },
      "expectedResponseScript": "......",
      "response": {
        "status": 200,
        "headers": {
          "x-powered-by": "Express",
          "content-type": "application/json; charset=utf-8",
          "content-length": "13",
          "etag": "W/\"d-X7zzcBORmHVzBfO4n/4UzEkpfkY\"",
          "date": "Mon, 01 May 2017 19:16:27 GMT",
          "connection": "close"
        },
        "body": "Hello John! you are 20 years old."
      }
    }
  }
}
```


#### Next Story: Mary use the contract to kick start stub server

Thie time, Mary start a local Contract Server with port 8000.
And then she create the App(John's app "Foo"), Version and Contracts just like the last story. Again, it is done by REST API calls.

But at the last step, thie time she would not call the API contract testing operation ( `/api/v1/apps/{{appId}}/wiretests` ).
Instead, she want to wire a stub server. Hence, she would call the API wire stub operation ( `/api/v1/apps/{{appId}}/wirestubs` ).

```
POST http://localhost:8000/api/v1/apps/{{appId}}/wirestubs
Content-type: application/json

{
    "port": 8001
}
```

After that, a stub server which responses according to the API contracts is started at local port 8001.

And then, Mary can test the `hello` API with the stub server. She can send a request like this:

```
POST http://localhost:8001/foo/v2.0/hello

{
    "name": "Mary",
    "age": 18
}
```

And she would get this response:
```
Hello Mary! You are 18 years old.
```

After Mary's testings, Mary can shutdown the stub server by this API:
```
DELETE http://localhost:8000/api/v1/apps/{{appId}}/wirestubs
```

Or, she can shutdown the whole local Contract Server instead.

------------------------

## Credit to Spring-Cloud-Contract & WireMock

We have to give credit to [Spring-Cloud-Contract](https://github.com/spring-cloud/spring-cloud-contract) & [Wiremock](http://wiremock.org/) because this project is inspired by them.
We like Spring-Cloud-Contract's design and usage of `value(stub(...), test(...))` so we bring it in rest-in-contract. We also appreciate Wiremock which showing us an well-made product for case study/feature analysis to help us make our own new product.
