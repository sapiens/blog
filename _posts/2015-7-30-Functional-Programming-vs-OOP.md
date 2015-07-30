---
layout: post
title: Functional Programming vs OOP
category: Programming
---

I have little experience with Functional Programing (FP) and even less with a FP language like F#. The FP I've done was mainly Linq stuff in C# and my attempt to write a build script for FAKE. I must say that I wanted to see how I fare with F# considering the number of its very vocal fans. Well, it was a painful experience and not only because of the strange syntax. The main problem is the mindset. Really, a FP mindset is required when using a FP language, which is pretty logical. But the big question is: Why would I want to use the FP mindset?

I follow devs on twitter who are very passionate about FP in general and F# in particular. For them, FP is the greatest thing ever, the ultimate golden hammer. Everything would be better if FP would be used (with F# of course). FP makes your code simpler, cleaner, less buggier etc. All hail FP the greatest hammer of them all. Thor is green of envy now.

Based on my experience as a developer, I beg to differ. While I've used the FP approach, I did it _only_ where it makes sense. I'd say that the majority of problems/use cases aren't naturally fit for FP. But I know that the FP crowd says the opposite. And my opinion is they found their perfect solution and now they force every problem into it.

Let's see some examples. There's a pretty famous [site](http://fsharpforfunandprofit.com) full of articles why FP and F# rulz and OOP and C# sux (not said directly but clearly implied). I've read it in order to try and do something with F#, but after reading some _very_ biased articles I decided it's more of a propaganda center than a useful resource (I'm sure others disagree but that's my opinion). It seems that OOP is such a sin for the FP cult (some devs are really behaving like zealots) that they need to come up with articles like [this](http://fsharpforfunandprofit.com/posts/fsharp-decompiled/) . In the article, F# source code is compared to the same code but decompiled to C# and surprise, surprise the C# code is ugly.

But let me show you other code decompiled to C#

```csharp
private void ExecuteTasks(TaskDictionary list)
    {
      foreach (KeyValuePair<int, List<Func<string>>> keyValuePair in (IEnumerable<KeyValuePair<int, List<Func<string>>>>) Enumerable.OrderBy<KeyValuePair<int, List<Func<string>>>, int>((IEnumerable<KeyValuePair<int, List<Func<string>>>>) list.Items, (Func<KeyValuePair<int, List<Func<string>>>, int>) (d => d.Key)))
      {
        string str1 = keyValuePair.Key.ToString();
        if (keyValuePair.Key == int.MaxValue)
          str1 = "no order";
        foreach (Func<string> func in keyValuePair.Value)
        {
          string str2 = func();
          string message = "Executed task ({0}) '{1}' ";
          object[] objArray = new object[2];
          int index1 = 0;
          string str3 = str1;
          objArray[index1] = (object) str3;
          int index2 = 1;
          string str4 = str2;
          objArray[index2] = (object) str4;
          LogManager.LogInfo<AStrapbootWithContainer<T>>(this, message, objArray);
        }
      }
    }
```

Yes, it's ugly. And here is the original source of the above code

```csharp
private void ExecuteTasks(TaskDictionary list)
       {
           foreach (var tasks in list.Items.OrderBy(d=>d.Key))
           {
               var pos = tasks.Key.ToString();
               if (tasks.Key == int.MaxValue)
               {
                   pos = "no order";
               }
               foreach (var task in tasks.Value)
               {
                   var name = task();
                   this.LogInfo("Executed task ({0}) '{1}' ", pos,name);
               }

           }
       }
```

A bit better right? Based on the article's logic then hand written C# is better than decompiled code to C#, therefor C# is better than C#.

But wait, F# has immutable types and the C# equivalent is more verbose and uglier (FP people really care about source code beauty). Yes, it might be but... WHY would I want that _everything_ should be immutable and value compared (*cough* C# structs *cough*)?.  Well, FP crowd will tell you how immutability means fewer bugs etc. I get that immutability is great but should it be used everywhere? Even F# has mutable support. The point is, if it exists and it's great in F# or any FP language, it doesn't mean I'll need it in an OOP language.

And this brings me to the real issue. The FP mindset **requires** that every operation should be expressed as a mathematical function. But we, normal people don't think like that. Unless you're a mathematician, but not everyone is. In the 'human' language we think imperative, very similar with an OOP language. Doing object oriented design is like being a manager in the 'real' world: you decide who does what, which are the teams and their projects. And unlike the real world, you can create the perfect professionals/employees which are specialists at their job.

OOP is quite natural. When I design something or try to come up with an algorithm, I think in a mix of Romanian and English. Only after I have a solution I expressed it in a programming language. But my thought process is unrelated to a programing language. I'm using a 'natural' mindset which then can be easily expressed in an OOP way. To change my mindset to functional just because someone says it's "much better", would be stupid. Most of the problems can be solved and expressed easily in OOP. **Some** of the problems are naturally functional and then can be easily expressed using a FP language.

It's simply a matter of how many use cases can be solved using a 'normal' mindset and how many use cases fit the functional paradigm better. My experience says the functional use cases are more limited  and I can use C# for those cases, even if in F# I could have done them easier. There are simply too few to matter.

But some of the FP crowd doesn't think that. For them everything is better if it's expressed in a functional way.  Ok, let's see how it goes with the famous FizzBuzz test. The code below is taken from [F# for fun and profit](http://fsharpforfunandprofit.com/posts/railway-oriented-programming-carbonated/).

Let's start with the imperative version but expressed in F#.

```fsharp
module FizzBuzz_IfPrime =

    let fizzBuzz i =
        let mutable printed = false

        if i % 3 = 0 then
            printed <- true
            printf "Fizz"

        if i % 5 = 0 then
            printed <- true
            printf "Buzz"

        if not printed then
            printf "%i" i

        printf "; "

    // do the fizzbuzz
    [1..100] |> List.iter fizzBuzz
```

Very similar to a C# version. But wait, we know that FP is the best paradigm _ever_ so let's solve the problem in a functional way, cause everyone knows FP means less code, fewer bugs, beauty etc.

```csharp
module FizzBuzz_Pipeline_WithTuple =

    // type Data = int * string option

    let carbonate factor label data =
        let (i,labelSoFar) = data
        if i % factor = 0 then
            // pass on a new data record
            let newLabel =
                labelSoFar
                |> Option.map (fun s -> s + label)
                |> defaultArg <| label
            (i,Some newLabel)
        else
            // pass on the unchanged data
            data

    let labelOrDefault data =
        let (i,labelSoFar) = data
        labelSoFar
        |> defaultArg <| sprintf "%i" i

    let fizzBuzz i =
        (i,None)   // use tuple instead of record
        |> carbonate 3 "Fizz"
        |> carbonate 5 "Buzz"
        |> labelOrDefault     // convert to string
        |> printf "%s; "      // print

    [1..100] |> List.iter fizzBuzz
```

Hmm, so this is how less, concise code looks like. For whatever reason I prefer the bloated and uglier (and of course hard to maintain) imperative version. But wait, there's more!!

OOP has design patterns because it's such an imperfect mindset and we need workarounds. FP is so great it doesn't need patterns. Until it needs them. Enter the [Railway pattern](http://fsharpforfunandprofit.com/posts/recipe-part2/) . And here's FizzBuzz using this pattern.

```csharp
module FizzBuzz_RailwayOriented_UsingCustomChoice =

    open RailwayCombinatorModule

    let (|Uncarbonated|Carbonated|) =
        function
        | Choice1Of2 u -> Uncarbonated u
        | Choice2Of2 c -> Carbonated c

    /// convert a single value into a two-track result
    let uncarbonated x = Choice1Of2 x
    let carbonated x = Choice2Of2 x

    // carbonate a value
    let carbonate factor label i =
        if i % factor = 0 then
            carbonated label
        else
            uncarbonated i

    let connect f =
        function
        | Uncarbonated i -> f i
        | Carbonated x -> carbonated x

    let connect' f =
        either f carbonated

    let fizzBuzz =
        carbonate 15 "FizzBuzz"
        >> connect (carbonate 3 "Fizz")
        >> connect (carbonate 5 "Buzz")
        >> either (printf "%i; ") (printf "%s; ")

    // test
    [1..100] |> List.iter fizzBuzz

```

Is it me or this version is even longer than the previous one? And to me, being a n00b when it comes to F#, it looks quite complicated and ugly. At least compared to the imperative version.

The point of this is to show you that you can force anything to work with the FP golden hammer but the result isn't the expected one. Clearly in this case the imperative approach is the most simpler one. But does this mean that only simple problems are fit for the imperative (OOP) mindset? Not really, remember that normally we think in an imperative mindset. The Domain expert will communicate according to that mindset. It takes explicit effort to 'convert' everything to a functional approach. And as seen with FizzBuzz, you end up with abominations.

I'm really annoyed by FP zealots who claim that FP is the answer to everything. As the saying goes "A real programmer writes assembler in any language", and in the end it's the developer who's in charge of the code quality. Your codebase isn't more bug free just because you're using F# , it's because you wrote good code. You may like FP and your brain can easily convert every problem to a FP representation. Ok, but mine and other's don't work that way. This doesn't mean our code is crap and the codebase is huge.

For me OOP is natural and I have no problem using it. For the little FP I do when it's the case, C# is enough. Maybe if I have enough use cases that can fit the FP mindset, I could write them in F#. But that's it. OOP is not automatically inferior for everything, FP is not superior for everything. And the fact that very few devs are using a functional language is a sign that it's a niche mindset for niche problems. And zealotry won't change things.

P.S: I'm one of the devs who say "always use DDD" regardless of the app. But I'm talking about the strategic aspect. Most of the time you're building an app that provides some business value. It's natural for the app to be designed according to the business needs, hence domain driven design. But that doesn't mean you should always use the DDD tactical tools. And obviously for experiments, toys, learning some new tool, there's no need for anything DDD.
