# Protocol Buffers

Interface definition languages, such as protobufs, are most commonly used to store and transmit data between applications.

They define a way to *serialize* structures as text, as well as *de-serialize* it again.

Here's an example of a protobuf message in *text format*:

```protobuf
KvPair {
  key: "TiKV"
  value: "Astronaut"
}
```

When a message is actually *sent* between two applications, a binary format is used. You can learn more about the binary format in the [documentation of protobufs](https://developers.google.com/protocol-buffers/docs/encoding).

## Why not use JSON?

Protobufs are simply much faster and cheaper than JSON. Additionally, protobufs can generate all the required code for your desired language.

If you have used [`serde_json`](https://docs.serde.rs/serde_json/) or another JSON library you may have experienced the task of defining schemas for structures. This becomes a maintenance burden as your infrastructure grows to span many languages.

You need to do this with protobufs as well, but you only do it once, and protobuf will generate bindings for any language it knows how to.

The way protobuf generates this code is backwards compatible, if an application finds data is isn't familiar with, it just ignores them. This allows for a safe evolution of an API.

## More that just data

Protobufs also enable the definition of *services*. This allows for the definition of RPC calls.

```protobuf
service ScanService {
  rpc Scan (string, string) returns (repeated KvPair) {};
}
```

This is particularly useful as it allows users to call remote functions as if they were local thanks to the code generation.

Next, we'll use gRPC to provide these services.