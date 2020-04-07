---
layout: post
title: "99 Bottles of beer"
date: 2019-11-17 19:42
categories: [functional, oo, elixir, haskell, ruby, programming]
---

I was recently introduced me to this problem of `99 Bottles of Beer on the wall` by one of my good friends at work. The problem statement is to print a [folk song with the same name](https://en.wikipedia.org/wiki/99_Bottles_of_Beer). That is to print:


> 99 bottles of beer on the wall, 99 bottles of beer.
>
> Take one down, pass it around, 98 bottles of beer on the wall...
>
>
> 98 bottles of beer on the wall, 98 bottles of beer.
>
> Take one down, pass it around, 97 bottles of beer on the wall...
>
> ...
>
> 1 bottle of beer on the wall, 1 bottle of beer.
>
> Take one down and pass it around, no more bottles of beer on the wall.
>
>
> No more bottles of beer on the wall, no more bottles of beer.
>
> Go to the store and buy some more, 99 bottles of beer on the wall.

He cited this problem as a good example to talk about clean code. One of the most revered author in the Ruby World, Sandi Metz, had used this example to demonstrate the concepts of clean code, TDD, etc. 

I wanted to implement this simple problem in two of my favorite languages Elixir and Haskell.

```elixir
defmodule BeerSong do
    def bottles(n) do
        case n do
            0 -> "no more bottles"
            1 -> "1 bottle"
            _ -> "#{n} bottles"
        end
    end

    def verse(n) do
        case n do
            0 -> "No more bottles of beer on the wall, no more bottles of beer. " <> 
                 "Go to the store and buy some more, 99 bottles of beer on the wall."
            _ -> "#{bottles(n)} of beer on the wall, #{bottles(n)} of beer. " <> 
                 "Take one down and pass it around, #{bottles(n-1)} of beer on the wall.\n" <> 
                 verse(n-1)
        end
    end
end

99 
|> BeerSong.verse 
|> IO.puts
```

and the Haskell Implementation:

```haskell
bottles n
    | n == 0 = "no more bottles"
    | n == 1 = "1 bottle"
    | n > 1 = show n ++ " bottles"

verse n
    | n == 0 = "No more bottles of beer on the wall, no more bottles of beer.\n" ++ 
               "Go to the store and buy some more, 99 bottles of beer on the wall."
    | n > 0 = bottles n ++ " of beer on the wall, " ++ bottles n ++ " of beer.\n" ++ 
            "Take one down and pass it around, " ++ bottles (n-1) ++ " of beer on the wall.\n" ++
            verse(n-1)

main =  putStrLn $ (verse 99)
```

Though this example is simple, this demonstrate the power of recursion and pattern matching, the most common traits of functional programming. 