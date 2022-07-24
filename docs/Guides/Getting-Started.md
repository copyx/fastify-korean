<h1 align="center">Fastify</h1>

## 시작하기

안녕하세요! Fastify를 확인해주셔서 감사합니다!

이 페이지는 예제와 문서의 다른 부분들에 대한 링크가 포함된 아주 쉽고 간단한 서문으로,
프레임워크와 기능을 친절하게 소개하는 것에 중점을 두고 있습니다.

그럼 시작하시죠!

### 설치하기

<a id="install"></a>

npm으로 설치하기:

```
npm i fastify
```

yarn으로 설치하기:

```
yarn add fastify
```

### 당신의 첫 번째 서버

<a id="first-server"></a>

우리의 첫 번째 서버를 작성해봅시다:

```js
// 프레임워크를 불러오고 객체를 생성

// ESM
import Fastify from "fastify";
const fastify = Fastify({
  logger: true,
});
// CommonJs
const fastify = require("fastify")({
  logger: true,
});

// 라우트 정의
fastify.get("/", function (request, reply) {
  reply.send({ hello: "world" });
});

// 서버 실행!
fastify.listen({ port: 3000 }, function (err, address) {
  if (err) {
    fastify.log.error(err);
    process.exit(1);
  }
  // 이제 서버는 ${address}에서 연결 대기 중입니다.
});
```

`async/await` 사용을 선호하시나요? Fastify는 별다른 설정없이 지원합니다.

```js
// ESM
import Fastify from "fastify";
const fastify = Fastify({
  logger: true,
});
// CommonJs
const fastify = require("fastify")({
  logger: true,
});

fastify.get("/", async (request, reply) => {
  return { hello: "world" };
});

/**
 * 서버 실행!
 */
const start = async () => {
  try {
    await fastify.listen({ port: 3000 });
  } catch (err) {
    fastify.log.error(err);
    process.exit(1);
  }
};
start();
```

대박, 이건 쉬웠어요.

안타깝게도 복잡한 애플리케이션을 작성하려면 위 예제보다 상당히 많은 코드가 필요합니다. 여러개의 파일, 비동기적 부트스트랩, 코드의 구조를 어떻게 처리할지는 새 애플리케이션을 만들 때 고전적인 문제입니다.

Fastify는 위에서 언급한 문제를 포함한 더 많은 문제의 해결에 도움되는 쉬운 플랫폼을 제안합니다!

> ## Note
>
> The above examples, and subsequent examples in this document, default to
> listening _only_ on the localhost `127.0.0.1` interface. To listen on all
> available IPv4 interfaces the example should be modified to listen on
> `0.0.0.0` like so:
>
> ```js
> fastify.listen({ port: 3000, host: "0.0.0.0" }, function (err, address) {
>   if (err) {
>     fastify.log.error(err);
>     process.exit(1);
>   }
>   fastify.log.info(`server listening on ${address}`);
> });
> ```
>
> Similarly, specify `::1` to accept only local connections via IPv6. Or specify
> `::` to accept connections on all IPv6 addresses, and, if the operating system
> supports it, also on all IPv4 addresses.
>
> When deploying to a Docker (or another type of) container using `0.0.0.0` or
> `::` would be the easiest method for exposing the application.

### Your first plugin

<a id="first-plugin"></a>

As with JavaScript, where everything is an object, with Fastify everything is a
plugin.

Before digging into it, let's see how it works!

Let's declare our basic server, but instead of declaring the route inside the
entry point, we'll declare it in an external file (check out the [route
declaration](../Reference/Routes.md) docs).

```js
// ESM
import Fastify from "fastify";
import firstRoute from "./our-first-route";
/**
 * @type {import('fastify').FastifyInstance} Instance of Fastify
 */
const fastify = Fastify({
  logger: true,
});

fastify.register(firstRoute);

fastify.listen({ port: 3000 }, function (err, address) {
  if (err) {
    fastify.log.error(err);
    process.exit(1);
  }
  // Server is now listening on ${address}
});
```

```js
// CommonJs
/**
 * @type {import('fastify').FastifyInstance} Instance of Fastify
 */
const fastify = require("fastify")({
  logger: true,
});

fastify.register(require("./our-first-route"));

fastify.listen({ port: 3000 }, function (err, address) {
  if (err) {
    fastify.log.error(err);
    process.exit(1);
  }
  // Server is now listening on ${address}
});
```

```js
// our-first-route.js

/**
 * Encapsulates the routes
 * @param {FastifyInstance} fastify  Encapsulated Fastify Instance
 * @param {Object} options plugin options, refer to https://www.fastify.io/docs/latest/Reference/Plugins/#plugin-options
 */
async function routes(fastify, options) {
  fastify.get("/", async (request, reply) => {
    return { hello: "world" };
  });
}

module.exports = routes;
```

In this example, we used the `register` API, which is the core of the Fastify
framework. It is the only way to add routes, plugins, et cetera.

At the beginning of this guide, we noted that Fastify provides a foundation that
assists with asynchronous bootstrapping of your application. Why is this
important?

Consider the scenario where a database connection is needed to handle data
storage. The database connection needs to be available before the server is
accepting connections. How do we address this problem?

A typical solution is to use a complex callback, or promises - a system that
will mix the framework API with other libraries and the application code.

Fastify handles this internally, with minimum effort!

Let's rewrite the above example with a database connection.

First, install `fastify-plugin` and `@fastify/mongodb`:

```
npm i fastify-plugin @fastify/mongodb
```

**server.js**

```js
// ESM
import Fastify from "fastify";
import dbConnector from "./our-db-connector";
import firstRoute from "./our-first-route";

/**
 * @type {import('fastify').FastifyInstance} Instance of Fastify
 */
const fastify = Fastify({
  logger: true,
});
fastify.register(dbConnector);
fastify.register(firstRoute);

fastify.listen({ port: 3000 }, function (err, address) {
  if (err) {
    fastify.log.error(err);
    process.exit(1);
  }
  // Server is now listening on ${address}
});
```

```js
// CommonJs
/**
 * @type {import('fastify').FastifyInstance} Instance of Fastify
 */
const fastify = require("fastify")({
  logger: true,
});

fastify.register(require("./our-db-connector"));
fastify.register(require("./our-first-route"));

fastify.listen({ port: 3000 }, function (err, address) {
  if (err) {
    fastify.log.error(err);
    process.exit(1);
  }
  // Server is now listening on ${address}
});
```

**our-db-connector.js**

```js
// ESM
import fastifyPlugin from "fastify-plugin";
import fastifyMongo from "@fastify/mongodb";

/**
 * @param {FastifyInstance} fastify
 * @param {Object} options
 */
async function dbConnector(fastify, options) {
  fastify.register(fastifyMongo, {
    url: "mongodb://localhost:27017/test_database",
  });
}

// Wrapping a plugin function with fastify-plugin exposes the decorators
// and hooks, declared inside the plugin to the parent scope.
module.exports = fastifyPlugin(dbConnector);
```

```js
// CommonJs
/**
 * @type {import('fastify-plugin').FastifyPlugin}
 */
const fastifyPlugin = require("fastify-plugin");

/**
 * Connects to a MongoDB database
 * @param {FastifyInstance} fastify Encapsulated Fastify Instance
 * @param {Object} options plugin options, refer to https://www.fastify.io/docs/latest/Reference/Plugins/#plugin-options
 */
async function dbConnector(fastify, options) {
  fastify.register(require("@fastify/mongodb"), {
    url: "mongodb://localhost:27017/test_database",
  });
}

// Wrapping a plugin function with fastify-plugin exposes the decorators
// and hooks, declared inside the plugin to the parent scope.
module.exports = fastifyPlugin(dbConnector);
```

**our-first-route.js**

```js
/**
 * A plugin that provide encapsulated routes
 * @param {FastifyInstance} fastify encapsulated fastify instance
 * @param {Object} options plugin options, refer to https://www.fastify.io/docs/latest/Reference/Plugins/#plugin-options
 */
async function routes(fastify, options) {
  const collection = fastify.mongo.db.collection("test_collection");

  fastify.get("/", async (request, reply) => {
    return { hello: "world" };
  });

  fastify.get("/animals", async (request, reply) => {
    const result = await collection.find().toArray();
    if (result.length === 0) {
      throw new Error("No documents found");
    }
    return result;
  });

  fastify.get("/animals/:animal", async (request, reply) => {
    const result = await collection.findOne({ animal: request.params.animal });
    if (!result) {
      throw new Error("Invalid value");
    }
    return result;
  });

  const animalBodyJsonSchema = {
    type: "object",
    required: ["animal"],
    properties: {
      animal: { type: "string" },
    },
  };

  const schema = {
    body: animalBodyJsonSchema,
  };

  fastify.post("/animals", { schema }, async (request, reply) => {
    // we can use the `request.body` object to get the data sent by the client
    const result = await collection.insertOne({ animal: request.body.animal });
    return result;
  });
}

module.exports = routes;
```

Wow, that was fast!

Let's recap what we have done here since we've introduced some new concepts.

As you can see, we used `register` for both the database connector and the
registration of the routes.

This is one of the best features of Fastify, it will load your plugins in the
same order you declare them, and it will load the next plugin only once the
current one has been loaded. In this way, we can register the database connector
in the first plugin and use it in the second _(read
[here](../Reference/Plugins.md#handle-the-scope) to understand how to handle the
scope of a plugin)_.

Plugin loading starts when you call `fastify.listen()`, `fastify.inject()` or
`fastify.ready()`

The MongoDB plugin uses the `decorate` API to add custom objects to the Fastify
instance, making them available for use everywhere. Use of this API is
encouraged to facilitate easy code reuse and to decrease code or logic
duplication.

To dig deeper into how Fastify plugins work, how to develop new plugins, and for
details on how to use the whole Fastify API to deal with the complexity of
asynchronously bootstrapping an application, read [the hitchhiker's guide to
plugins](./Plugins-Guide.md).

### Loading order of your plugins

<a id="plugin-loading-order"></a>

To guarantee consistent and predictable behavior of your application, we highly
recommend to always load your code as shown below:

```
└── plugins (from the Fastify ecosystem)
└── your plugins (your custom plugins)
└── decorators
└── hooks
└── your services
```

In this way, you will always have access to all of the properties declared in
the current scope.

As discussed previously, Fastify offers a solid encapsulation model, to help you
build your application as single and independent services. If you want to
register a plugin only for a subset of routes, you just have to replicate the
above structure.

```
└── plugins (from the Fastify ecosystem)
└── your plugins (your custom plugins)
└── decorators
└── hooks
└── your services
    │
    └──  service A
    │     └── plugins (from the Fastify ecosystem)
    │     └── your plugins (your custom plugins)
    │     └── decorators
    │     └── hooks
    │     └── your services
    │
    └──  service B
          └── plugins (from the Fastify ecosystem)
          └── your plugins (your custom plugins)
          └── decorators
          └── hooks
          └── your services
```

### Validate your data

<a id="validate-data"></a>

Data validation is extremely important and a core concept of the framework.

To validate incoming requests, Fastify uses [JSON
Schema](https://json-schema.org/).

(JTD schemas are loosely supported, but `jsonShorthand` must be disabled first)

Let's look at an example demonstrating validation for routes:

```js
/**
 * @type {import('fastify').RouteShorthandOptions}
 * @const
 */
const opts = {
  schema: {
    body: {
      type: "object",
      properties: {
        someKey: { type: "string" },
        someOtherKey: { type: "number" },
      },
    },
  },
};

fastify.post("/", opts, async (request, reply) => {
  return { hello: "world" };
});
```

This example shows how to pass an options object to the route, which accepts a
`schema` key that contains all of the schemas for route, `body`, `querystring`,
`params`, and `headers`.

Read [Validation and
Serialization](../Reference/Validation-and-Serialization.md) to learn more.

### Serialize your data

<a id="serialize-data"></a>

Fastify has first-class support for JSON. It is extremely optimized to parse
JSON bodies and serialize JSON output.

To speed up JSON serialization (yes, it is slow!) use the `response` key of the
schema option as shown in the following example:

```js
/**
 * @type {import('fastify').RouteShorthandOptions}
 * @const
 */
const opts = {
  schema: {
    response: {
      200: {
        type: "object",
        properties: {
          hello: { type: "string" },
        },
      },
    },
  },
};

fastify.get("/", opts, async (request, reply) => {
  return { hello: "world" };
});
```

By specifying a schema as shown, you can speed up serialization by a factor of
2-3. This also helps to protect against leakage of potentially sensitive data,
since Fastify will serialize only the data present in the response schema. Read
[Validation and Serialization](../Reference/Validation-and-Serialization.md) to
learn more.

### Parsing request payloads

<a id="request-payload"></a>

Fastify parses `'application/json'` and `'text/plain'` request payloads
natively, with the result accessible from the [Fastify
request](../Reference/Request.md) object at `request.body`.

The following example returns the parsed body of a request back to the client:

```js
/**
 * @type {import('fastify').RouteShorthandOptions}
 */
const opts = {};
fastify.post("/", opts, async (request, reply) => {
  return request.body;
});
```

Read [Content-Type Parser](../Reference/ContentTypeParser.md) to learn more
about Fastify's default parsing functionality and how to support other content
types.

### Extend your server

<a id="extend-server"></a>

Fastify is built to be extremely extensible and minimal, we believe that a
bare-bones framework is all that is necessary to make great applications
possible.

In other words, Fastify is not a "batteries included" framework, and relies on
an amazing [ecosystem](./Ecosystem.md)!

### Test your server

<a id="test-server"></a>

Fastify does not offer a testing framework, but we do recommend a way to write
your tests that uses the features and architecture of Fastify.

Read the [testing](./Testing.md) documentation to learn more!

### Run your server from CLI

<a id="cli"></a>

Fastify also has CLI integration thanks to
[fastify-cli](https://github.com/fastify/fastify-cli).

First, install `fastify-cli`:

```
npm i fastify-cli
```

You can also install it globally with `-g`.

Then, add the following lines to `package.json`:

```json
{
  "scripts": {
    "start": "fastify start server.js"
  }
}
```

And create your server file(s):

```js
// server.js
"use strict";

module.exports = async function (fastify, opts) {
  fastify.get("/", async (request, reply) => {
    return { hello: "world" };
  });
};
```

Then run your server with:

```bash
npm start
```

### Slides and Videos

<a id="slides"></a>

- Slides

  - [Take your HTTP server to ludicrous
    speed](https://mcollina.github.io/take-your-http-server-to-ludicrous-speed)
    by [@mcollina](https://github.com/mcollina)
  - [What if I told you that HTTP can be
    fast](https://delvedor.github.io/What-if-I-told-you-that-HTTP-can-be-fast)
    by [@delvedor](https://github.com/delvedor)

- Videos
  - [Take your HTTP server to ludicrous
    speed](https://www.youtube.com/watch?v=5z46jJZNe8k) by
    [@mcollina](https://github.com/mcollina)
  - [What if I told you that HTTP can be
    fast](https://www.webexpo.net/prague2017/talk/what-if-i-told-you-that-http-can-be-fast/)
    by [@delvedor](https://github.com/delvedor)
