---
layout: post
title: "Channels: Go and Rust"
date: 2020-04-25 08:19
categories: [channels, rust, go]
---

Channels are a means of communication between multiple threads (or channels in case of Go). Channels usually provide an uni-directional flow of data, that is the data flows from the sender to the receiver. These channels play a vital role building concurrent applications. This blog post discusses the use of channels in both Go and Rust.

Go: The language that was developed with concurrency in mind. Consider channel as a data pipe where data flows from one coroutine to another, enabling a means of data sharing between coroutines. The channels in go could be single-producer-single-consumer (spsc), multiple-producer-multiple-consumer channels. This means that a channel can be owned by more than one coroutine at the same time. 

For example, let us write a piece of code reads from a huge text file and prints its output. Let's first implement this in Go with the reading happening in one coroutine and the printing it in the main thread.

```go
package main

import (
	"bufio"
	"fmt"
	"log"
	"os"
)

func main() {
	c := make(chan string)
	file, err := os.Open("./largefile.txt")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	go readFile(file, c)

	fmt.Println("Printing contents of file")
	for str := range c {
		fmt.Printf("%s\n", str)
	}
}

func readFile(file *os.File, c chan string) {
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		c <- scanner.Text()
	}

	if err := scanner.Err(); err != nil {
		log.Fatal(err)
	}
	close(c)
}
```

This is a simple example of a producer consumer communication happening through channels. You can also see that the channels, like any other references, can be passed around to other coroutines. The push to the channel `c <- scanner.Text()` is a non blocking call, where as receiving the data from the channel `for str := range c` is a blocking call.

Similar to Go, Rust enables concurrency through channels. But unlike Go, Rust channels are multiple producer and single consumer. This means that at any given point of time, only one thread can be the owner of the channel. A channel is declared as below

```rust
let (tx, rx) = std::sync::mpsc::channel();
```

This creates a `tx`, i.e. a sender and a `rx`, a receiver. The datatype of the data passed through the channel is determined dynamically when the data is pushed (or we can explicitly declare the datatype for the channel). The sender can be passed around functions and can be at least (half) owned by the threads by `cloning` it. The receiver, on the other hand must have a single owner. This makes the Rust channels multiple-producer-single-consumer.

Implementing the same example in rust would look like this:

```rust
use std::fs::File;
use std::io::{self, BufRead};
use std::path::Path;
use std::sync::mpsc::{Sender, Receiver};
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx): (Sender<String>, Receiver<String>) = mpsc::channel();
    if let Ok(lines) = read_lines("./largefile.txt") {
        thread::spawn(move || {
            for line in lines {
                if let Ok(ip) = line {
                    match tx.send(ip.to_string()) {
                        Ok(_) => {},
                        Err(err) => println!("Cannot send data on channel: {}", err.to_string())
                    }
                }
            }
        });
    }

    loop {
        match rx.recv() {
            Ok(res) => println!("Receiving: {}", res),
            Err(_) => { break }
        }
    }
}

fn read_lines<P>(filename: P) -> io::Result<io::Lines<io::BufReader<File>>>
where P: AsRef<Path>, {
    let file = File::open(filename)?;
    Ok(io::BufReader::new(file).lines())
}
```

Similar to Go, the `tx.send()` method is a non-blocking call where as the `rx.recv()` is a blocking call. 

One other thing that needs to be observed here is that Go provides co-routines which are runtime level threads mapped onto OS level threads. The channels and coroutines are a builtin feature in Go. A coroutine is spawned using the `go` keyword. Rust does not have the concept of runtime threads or have built in channels. It allows the developer to spawn OS level threads and channels are provided as a part of the standard library. 

Channels are quite useful in Rust while we implement a gRPC service that streams data from one end to the other. This implementation will be discussed in the next post. 