---
id: rpc
description: An example of an RPC communication in r4bbit
---

# RPC Communication

This guide illustrates the RPC communication process between a server and a client using the r4bbit API. It provides an enhanced version of the initial setup, detailing all available API options.

<div class="alert alert--primary" role="alert">
  All the example code presented in this section can be accessed over <a href="https://github.com/r4bbit/r4bbit/blob/dev/examples/rpc/index.ts" target="_blank">r4bbit/examples/rpc</a>.
</div>
<br />

## Create a server and a client instances

We create a server and a client instances with a RabbitMQ connection URL and no options - default values will be used.

```ts
const server = await getServer("amqp://guest:guest@localhost:5672/");
const client = await getClient("amqp://guest:guest@localhost:5672/");
```

## Register a route

First, we need a function that would handle the incoming message and reply to it. It's like a controller method in HTTP servers.

```ts
const handler =
  (reply: ServerTypes.Reply) => (msg: Record<string, unknown> | string) => {
    if (!msg) {
      return;
    }
    reply((msg as { content: string }).content);
  };
```

Then we register a route with the handler function.

```ts
// create a server with one route
await server.registerRPCRoute(
  {
    queueName: "my-queue",
    exchangeName: "my-exchange",
    routingKey: "my.*",
  },
  handler,
  {
    replySignature: "rpc-server-signature",
    consumeOptions: {
      // optional
      noAck: false, // default is true
    },
    loggerOptions: {
      // optional
      isConsumeDataHidden: true, // default is false
      isSendDataHidden: true, // default is false
    },
    responseContains: {
      content: true, // default is true
      headers: true, // default is false
    },
  }
);
```

<div class="alert alert--secondary" role="alert">

`Logger` options are optional, if you don't provide them, the default values will be used. If you want to hide the data that is being sent or received, you can set the `isConsumeDataHidden` or `isSendDataHidden` to `true`.

`Response contains` options are optional, if you don't provide them, the default values will be used. If you would like to get the response headers or signature, you can set the `headers` or `signature` to `true`.

</div>

## Send the message as a client

Client sends the message

```ts
await client.publishRPCMessage(
  { message: "OurMessage", nested: { value: 15 } },
  {
    exchangeName: "my-exchange",
    routingKey: "my.routing-key",
    replyQueueName: "simple-rpc-reply",
    timeout: 5_000,
    responseContains: {
      content: true,
      headers: true,
      signature: true,
    },
  }
);
```

## Close the connection

Gracefully close the connection

```ts
await client.close();
await server.close();
```

## Logs

Note that this whole process is clearly visible in logs:

1. Publishing the message by the client

   ![An example that displays log of publishing the message by the client](./assets/rpc/1_client-publish.png)

2. Receiving it by the server

   ![An example that displays log of receiving the message by the server](./assets/rpc/2_server-receive.png)

3. Processing and publishing the response from the server to the client

   ![An example that displays log of processing and publishing the response from the server to the client](./assets/rpc/3_server-publish.png)

4. Receiving the response by the client

   ![An example that displays log of receiving the response by the client](./assets/rpc/4_client-receive.png)
