---
layout: post
title: "gRPC: the good, the bad and the ugly"
date: 2020-04-16 20:26
comments: true
categories: [grpc, rust, programming]
---

Even today, many companies use [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) as the go-to mechanism for service to service communication. This architecture is based on text-based messaging mechanisms like JSON or XML. 

This is good in many aspects such as, it is easily understood by developers. Since it is built on top of HTTP and all languages have a good HTTP support, client libraries can be written for any language. Additionally, the server and client are loosely coupled. This makes it great for external facing or public facing APIs.

But REST has a fair share of drawbacks which actually does not make it ideal for internal service-to-service communications. 

- To start with, there is no formal machine readable API contract. This would mean that for most of the supported languages, the client libraries have to be written by developers, which requires a lot of time and effort. 

- Second, since it is based on a text based protocol, say JSON, there is an overhead of parsing the request or the response using libraries like Gson or Jackson. These textual representations are not network friendly.

- Streaming data is extremely difficult, impossible in many cases.

- In a lot of cases, REST APIs are just HTTP endpoints. Most of us would have seen/written an end point resembling `/sendMail` or `/restoreBackup` etc, which do not necessarily follow the REST pattern.


Here is where gRPC saves the day. By the definition of [grpc.io](https://grpc.io) 
> gRPC is a modern open source high performance RPC framework that can run in any environment. 

This is a system built on top of HTTP/2 protocol. This was initially developed by Google and now it is widely adopted by a variety of companies such as Google, Netflix, Square, GoJek etc. This uses [protocol buffers aka protobuf](https://developers.google.com/protocol-buffers) for its interface description.

To start writing a service, we start with writing a `.proto` file. For example, if I (being a Pokemon fan) want to create a Pokedex service, the proto shall look like this:

```proto
syntax = "proto3";

package pokedex;

enum PokemonType {
    FIRE = 0;
    GROUND = 1;
    WATER = 2;
    GRASS = 3;
}

message Query {
    string value = 1;
}

message PokemonResponse {
    int32 id = 1;
    string name = 2;
    repeated PokemonType pokemonType = 3;
}

service PokeDex {
    rpc GetPokemonByName(Query) returns (PokemonResponse);
}
```

This proto files acts as a specification or a contract for the service to be implemented. This service contract is shared between the server and the client. This is pretty concise, easy for the developer to read and understand. All the developer does here is define the methods that are exposed on the server for the client to call remotely. 

Most importantly, these proto files are machine readable which means that when run through a compiler ([protoc](http://google.github.io/proto-lens/installing-protoc.html)), this is going to generate the necessary files needed for writing the server skeleton code and the client libraries. 

For example, let's take Go as an example:

```bash
$ protoc --go_out=plugins-grpc:. ./pokedex.proto
```
would generate a bunch of code (pokedex.pb.go in this case) that you need to start writing the server and client with a few lines of code.

The advantages of gRPC:

- One important advantage with gRPC is that the client opens a long-living connection with the server. Every request is only a logical stream on top of that physical connection 
- This would mean that the connection shall enable streaming on data __bi-directionally__ between server and client.
- The request and the responses are encoded using protocol buffers and sent as binary data which is basically unmarshalled at the other end into an object 
- gRPC is polyglot friendly. The server and client implementation can be on different languages.
- protoc has the support to generate client libraries in many languages. 


Despite these advantages, gRPC also comes with its share is caveats. 

- There is no OOTB Load balancing solution for gRPC services. You need to work around it (for example: using client side load balancing)
- Error handling 
- Managing breaking changes on the API

I believe that the advantages of gRPC outweigh the disadvantages, thus making it a logical and a suitable choice for internal service-to-service communication. I shall also blog about implementing the aforesaid Pokedex service in Rust in [my next post](/grpc-and-rust).

#### Disclaimer
Pokémon, Pokêdex names and information (c) 1995-2014 Nintendo/Game freak. Those are being referenced here entirely for education purposes only.