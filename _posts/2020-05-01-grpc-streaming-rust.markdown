---
layout: post
title: "gRPC: Streaming data in Rust"
date: 2020-05-01 21:56
comments: true
categories: [grpc, rust, programming]
---

One of the important advantages of gRPC is that we can stream data between the server and the client. This post explores how we can achieve the same. 

Let us take the Pokédex server that we discussed in the previous [post](/grpc-and-rust) as an example. Now let us add another service to the server which takes a Pokémon type as an input and streams a list of Pokémons matching those types. The proto would look similar to this:

```proto
message GetPokemonsByTypeRequest {
    PokemonType pokemonType = 1;
}

service PokeDex {
    rpc GetPokemonsByType(GetPokemonsByTypeRequest) returns (stream PokemonResponse);
}
```

The return type of the rpc is declared explicitly to stream PokemonResponse(s). When the project is built, it shall generate the correponding server types. This also adds these lines to the server mod in the generated file:

```rust
 type GetPokemonsByTypeStream: Stream<Item = Result<super::PokemonResponse, tonic::Status>>
            + Send
            + Sync
            + 'static;

async fn get_pokemons_by_type(
    &self,
    request: tonic::Request<super::GetPokemonsByTypeRequest>,
) -> Result<tonic::Response<Self::GetPokemonsByTypeStream>, tonic::Status>;
```

Let's dissect this code. The GetPokemonsByTypeStream is declared as a stream type which streams items of a monadic type containing the PokemonResponse. The `Send` trait added to the item type make it safe to send it to another thread. The `Sync` trait added to the item type makes it share the data between threads. The `'static` is the longest possible lifetime for a variable and it last till the lifetime of the program. 

Now let's implement this in the PokedexServer `server.rs`.

```rust
type GetPokemonsByTypeStream: Stream<Item=Result<super::PokemonResponse, tonic::Status>>
+ Send
+ Sync
+ 'static = mpsc::Receiver<Result<PokemonResponse, Status>>;
```

To start with we need to declare the GetPokemonByTypeStream as a [channel](/concurrency-go-rust) receiver of the data type. We shall return this receiver type from the method. Here is one use of channels when streaming data in gRPC in Rust.

Let's now start implementing the method. We shall start by declaring a channel with a desired channel capacity. The channel capacity denotes the number of items that can be stored in the channel pending receipt at any given point of time. We have declared a channel of capacity 4. 

```rust
let (mut tx, rx) = mpsc::channel(4);
```

The `tx`, i.e. the sender is declared mutable because this will be borrowed by another thread to push the data. After the channel is declared, spawn a new thread to fetch the details from the database and push it to the sender. Finally we need to return the receiver as the response of this method. 

```rust
Ok(Response::new(rx))
```

Stitched together, the method shall look like this:

```rust
use tokio::sync::mpsc;

async fn get_pokemons_by_type(&self, request: Request<GetPokemonsByTypeRequest>) -> Result<Response<Self::GetPokemonsByTypeStream>, Status> {
    let requested_type = get_pokemon_type_string(request.into_inner().pokemon_type);
    let (mut tx, rx) = mpsc::channel(4);


    tokio::spawn(async move {
        match db::Pokemon::find_by_type(requested_type) {
            Ok(pokemons) => {
                for p in pokemons {
                    let pokemon = p.clone();
                    tx.send(Ok(PokemonResponse {
                        id: pokemon.id,
                        name: pokemon.name,
                        pokemon_type: to_pokemon_types(p.types),
                    })).await.unwrap();
                }
            }
            Err(_) => {}
        }
    });

    Ok(Response::new(rx))
}
```

That's it! The server is ready to stream PokemonResponse to the clients. The client implementation doesn't change much. The client keeps receiving the response until it the end of stream.

```rust
let mut stream = client.get_pokemons_by_type(get_pokemons_by_type).await?.into_inner();

while let Some(pokemon_response) = stream.message().await? {
    println!("Name: {}", pokemon_response.name);
}
```

Thus with a few lines of code, we can now stream data from server to client. The complete implementation can be found [here](https://github.com/kumaranvram/pokedex-rust-grpc-sample/tree/streaming).

#### Disclaimer
Pokémon, Pokêdex names and information (c) 1995-2014 Nintendo/Game freak. Those are being referenced here entirely for education purposes only.