# connectanum-dart

This is a wamp client implementation for dart or flutter projects. The projects aims to 
provide a simple an extensible structure.

## Supported WAMP features

### Advanced RPC features

- [x] Progressive Call Results
- [x] Progressive Calls
- [ ] Call Timeouts
- [ ] Call Canceling
- [x] Caller Identification
- [ ] Call Trust Levels
- [x] Shared Registration
- [ ] Sharded Registration

### Advanced PUB/SUB features

- [x] Subscriber Black- and Whitelisting
- [x] Publisher Exclusion
- [x] Publisher Identification
- [ ] Publication Trust Levels
- [x] Pattern-based Subscriptions
- [ ] Sharded Subscriptions
- [ ] Subscription Revocation

## code structure

```
+----------------------------------------------------------------+
|                                                                |
|                     +----------------------------------------+ |
|                     |                                        | |
|       Client        |           Authentication               | |
|                     |                                        | |
|                     +----------------------------------------+ |
|                                                                |
|  +---------------+  +----------------------------------------+ |
|  |               |  |                                        | |
|  |               |  |   +---+                                | |
|  |               |  |   |   | Session                        | |
|  |               |  |   +-+-+                                | |
|  |               |  |     |                                  | |
|  |               |  |     |   outgoing messages              | |
|  |               |  |     |                                  | |
|  |               |  |     |       incomming messages         | |
|  |   Transport   |  |     |     +---------------------+      | |
|  |               |  |     |     |                     |      | |
|  |               |  |  +--v-----+--+ +----------------v---+  | |
|  |               |  |  |           | |                    |  | |
|  |               |  |  | Transport | | Protocol-Processor |  | |
|  |               |  |  |           | |                    |  | |
|  |               |  |  +-----------+ +--------------------+  | |
|  |               |  |                                        | |
|  +---------------+  +----------------------------------------+ |
|                                                                |
+----------------------------------------------------------------+
```

The client wraps all classes to process the wamp protocol. The transport can by any type of class inheriting
the `AbstractTransport` class. The transport also cares about serialization and deserialization of incoming
messages. 

The authentication is processed by the any class inheriting the `AbstractAuthentication` class. It is used by
the client to negotiate a session id with the router instance.

After the authentication process is successful, the client will build a session object. The
session will carry the transport as well as a protocol processor that handles the incoming deserialized
messages. The protocol processor also triggers all behaviour subjects located in the session to trigger
invocations and events. Outgoing messages are created by the session object and are passed to the transport.

The basic message logic is build to use serialized messages. The message objects are also used to handle RPC and 
PUB/SUB events. For example a `SUBSCRIBED` will also hold the incoming message stream for event messages. This way
the protocol itself is used to structure the code. Therefore it is necessary that a router sends a subscribed even 
though it is not mandatory. 


## Start the client

To start a client you need to choose a transport module and connect it to the desired endpoint.
When the connection has been established you can start to negotiate a client session by calling
the `client.connect()` method from the client instance. On success the client will return a
session object.

If your transport disconnects the session will invalidate. If a reconnect is configured, the session
will try to authenticate an revalidate the session again. All subscriptions and registrations will
be recovered if possible.

```dart
final transport = new WebSocketTransport("wss://localhost:8443");
final client = new Client(
    realm: "my.realm",
    transport: transport
);
final session await client.connect();
```

## RPC

to work with RPCs you need to have an established session. 

```dart
final client = new Client(realm: "my.realm",new WebSocketTransport("wss://localhost:8443"));
final session = await client.connect();

// Register a procedure
final registered = await session.register("my.procedure");
registered.invocationStream.listen(/*Your endpoint goes here*/)

// Call a procedure
final result = await session.call("my.procedure");
```
