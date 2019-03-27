# Protocol Buffers (Protobufs)

Interface definition languages, such as protobufs, are most commonly used to
store and transmit data between applications.

They define a way to *serialize* structures as text, as well as *de-serialize*
it again.

Here's an example of a protobuf message in *text format*:

```protobuf
KvPair {
  key: "TiKV"
  value: "Astronaut"
}
```

When a message is actually *sent* between two applications, a binary format is
used. You can learn more about the binary format in the [protobuf documentation](https://developers.google.com/protocol-buffers/docs/encoding).

The message above would be an instance of a structure predefined in a `.proto`
file:

```protobuf
message KvPair {
  string key = 1;
  string value = 2;
}
```

The fields are numbered to support backwards compatibility and field renaming.
This makes it possible to evolve your application's communication in a
compatible way.

## Why Probobufs

Protobufs are simply much faster and more efficient than things like JSON.
Additionally, protobufs can generate all the required code for your desired
language.

If you have used [`serde_json`](https://docs.serde.rs/serde_json/) or another
JSON library, you may have experienced the task of defining schemas for
structures. This becomes a maintenance burden as your infrastructure grows to
span many languages.

You need to do this with protocol buffers as well, but you only do it once, and
the protobuf compiler will generate bindings for any language it knows how to.

Protobuf generates this code in a backwards compatible manner. If an application
finds unfamiliar data is isn't familiar with, it just ignores them. This allows
for a safe evolution of an API.

## More than just data

Protobufs also enable the definition of *services*. This allows the
definition of RPC calls in the `*.proto` files.

This example demonstrates a service called `ScanService` which provides a remote
procedure call `Scan` that accepts two strings and returns a stream of `KvPair`s:

```protobuf
service ScanService {
  rpc Scan (string, string) returns (repeated KvPair) {};
}
```

This is particularly useful as it allows users to call remote functions almost
as if they were local thanks to code generation.

Next, we'll use gRPC to provide these services.