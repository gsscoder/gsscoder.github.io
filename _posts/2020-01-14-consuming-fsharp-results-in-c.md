---
layout: post
title: "Consuming F# results in C#"
tags: [Interoperability, Software Design]
---

We know that **C#** is the **.NET** language we should use to build _vanilla_ class libraries. A library built with that language will be easly consumed in other **.NET** dialects. After all this is why the **BCL** is written in this language.

However there are excellent libraries written in **F#** that are commonly user by **C#** developers. The first that come to my mind is [FsCheck](https://fscheck.github.io/FsCheck/), designed for properties testing.

Another example is [quote.fs](https://github.com/gsscoder/quote.fs). It's project of mine, still in embryonic state. Essentially an experiment aimed to design an **F#** library usable from **C#** (and eventually other **.NET** languages).

I've included static class to map the **functional** surface API and convert every `AsyncResult<T>` to `Task<T>` (more **C#** frendly). Anyway functional languages don't like `null` values and you will need to face some kind of result type in **C#**.

**F#** built-in result type is defined as follow:
```fsharp
type Result<'T,'TError> =
    | Ok of ResultValue:'T
    | Error of ErrorValue:'TError
```
It contains a value or an error in a very simal way to **Haskell** either type ([Data.Either](http://hackage.haskell.org/package/base-4.12.0.0/docs/Data-Either.html)). You can find a **C#** implementation in **CSharpx** [Either.cs](https://github.com/gsscoder/CSharpx/blob/master/src/CSharpx/Either.cs).

##### Let's code

A result type is (as you can verify with _Reflector_ or _Intellisense_) has a compiled name of `FSharpResult<T, Error>`. Hit `dotnet fsi` from a terminal and see how result value is built from the **F#** side:
```fsharp
> let useless x =
-     match x with 
-     | true -> Ok 3.14
-     | _ -> Error "bad input";;
val useless : x:bool -> Result<float,string>
```

The `useless` function will succeed if value of `x` is `true`. In this case it will a return a _PI number_ otherwise an error message. This is why **F#** REPL states that `useless` function return type is `Result<float,string>`. In **F#** consuming that value is elegant and simple like creating it:
```fsharp
let result = useless true
match result with
| Ok value -> Numbers.Add value
| Error message -> printfn "Trouble: %s" message
```

From an **C#** standpoint, supposing that you wrap this function in a static method of a static class, you will end up consuming it the in following way:
```csharp
var result = CSharpFriendly.Useless(true);
if (result.IsOk) {
    // success, do something with the value
    Numbers.Add(result.ResultValue);
} else {
    // fail, report an error message
    Console.WriteLine(result.ErrorValue);
}
```

I don't think this can stand comparison with **F#** pattern matching. But if you add [CSharpx](https://github.com/gsscoder/csharpx) to your project a better result (forgive the pun) can be achived with easy. Just use the `FSharpResult<T, Error>` extension method that fits your needs. In such case `Match` is the perfect choice, since allows you to _mimic_ **F#** pattern matching using lambda functions:
```csharp
// refactoring
var result = CSharpFriendly.Useless(true);
result.Match(
    value => Numbers.Add(value),
    message => Console.WriteLine($"Trouble: {message}"));
```

_Voilà!_ Game is done, ugly code is gone away. You can learn all available extension methods directly from [FSharpResultExtensions](https://github.com/gsscoder/CSharpx/blob/master/src/CSharpx/FSharpResultExtensions.cs) source and some usage example from [Unit Tests](https://github.com/gsscoder/CSharpx/blob/master/tests/CSharpx.Tests/Unit/FSharpResultExtensionsTests.cs).

##### Conclusion

I think that, with very little effort from both sides, **F#** libraries can be comfortably used from **C#** code. In this way a developer that knows both languages is free to decide which fit better for a particular task.

In few words, freedom of design!