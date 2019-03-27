# HTTP/2

HTTP/2 is the 'Hyper Text Transfer Protocol Version 2'. HTTP/2 and its predecessor, HTTP/1, are the de-facto network protocols in use today.

They define things such as:

* [URIs]
* [Sessions]
* [Status Codes] (eg. 404 Not Found)
* [Methods] (eg. GET, POST, PUT)
* [Headers] (eg. Authorization, User-Agent)
* [Pipelining]

HTTP/2 evolved out of Google's experimental SPDY protocol as a successor to
HTTP/1.1. Since it maintains high-level compatibility with HTTP/1.1, let's first
take a look at HTTP/1.x.

## HTTP/1.x

HTTP/1.0 was developed at CERN in 1989. [IETF RFC 1945] in 1996 was the first
officially recognized HTTP/1.0 version. Just a few years later, HTTP/1.1 was
specified in [IETF RFC 2068]. The standard was improved in [IETF RFC 2616] in 1999.

Both of these specifications eventually were obseleted in 2007 in the following RFCs:

* [RFC 7230, HTTP/1.1: Message Syntax and Routing](https://tools.ietf.org/html/rfc7230)
* [RFC 7231, HTTP/1.1: Semantics and Content](https://tools.ietf.org/html/rfc7231)
* [RFC 7232, HTTP/1.1: Conditional Requests](https://tools.ietf.org/html/rfc7232)
* [RFC 7233, HTTP/1.1: Range Requests](https://tools.ietf.org/html/rfc7233)
* [RFC 7234, HTTP/1.1: Caching](https://tools.ietf.org/html/rfc7234)
* [RFC 7235, HTTP/1.1: Authentication](https://tools.ietf.org/html/rfc7235)

Thankfully it's not necessary to become familiar with every detail of these
RFCs. HTTP/1.0 is a request-reponse protocol, for now, we'll just focus on this.

### The Request

When a node is connected to another node via TCP/IP, it can make a request by
sending ASCII text like as below (the empty newline is required!):

```HTTP
POST / HTTP/1.1
Host: tikv.org

{ data: "Test" }
```

Common headers are things like `Authorization` (for access control), and
`Cache-Control` (for caching).

### The Response

The recipient of a message responds in a similar format:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8
Connection: close

{ data: "bar" }
```

## HTTP/2

The major differences in HTTP/2 lie not in topics like methods, status codes, or
URIs, but in data frames and transportation.

It builds on HTTP/1.1 by adding features like:

* Server Push, allowing the server to respond with more data than requested.
* Multiplexing, to avoid 'head-of-line' blocking problem in HTTP/1.1
* Pipelining, reducing network wait times.
* Compression of headers, to reduce overall network costs.

Compared to HTTP/1.1, HTTP/2.0 offers applications like TiKV many opportunities
for performance gains.

[IETF RFC 1945]: https://tools.ietf.org/html/rfc1945
[IETF RFC 2068]: https://tools.ietf.org/html/rfc2068
[IETF RFC 2616]: https://tools.ietf.org/html/rfc2616
[IETF RFC 7230]: https://tools.ietf.org/html/rfc7230
[IETF RFC 7231]: https://tools.ietf.org/html/rfc7231
[IETF RFC 7232]: https://tools.ietf.org/html/rfc7232
[IETF RFC 7233]: https://tools.ietf.org/html/rfc7233
[IETF RFC 7234]: https://tools.ietf.org/html/rfc7234
[IETF RFC 7235]: https://tools.ietf.org/html/rfc7235
[URIs]: https://en.wikipedia.org/wiki/Uniform_Resource_Identifier
[Sessions]: https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#HTTP_session
[Status Codes]: https://en.wikipedia.org/wiki/List_of_HTTP_status_codes
[Methods]: https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods
[Headers]: https://en.wikipedia.org/wiki/List_of_HTTP_header_fields
[Pipelining]: https://en.wikipedia.org/wiki/HTTP_pipelining