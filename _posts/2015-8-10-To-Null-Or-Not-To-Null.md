---
layout: post
title: To Null Or Not TO Null aka Is Null Really Evil?
category: Programming
---

It's not a secret that some devs are considering null to be 'evil' and even the 'father' of null himself said that introducing null was his billion dollars mistake. But, is null that evil? Some 'purists' (in lack of a better word) think so and they advocate that we should stay away of nulls no matter what!

Personally, I don't care much about null. For me null means 'lack of value' or 'nothing' and I treat it as such. 'Nothing' is a valid result in many scenarios and the alternative is to have a `Maybe<T>` type which will be used exactly as null. For me, it would be just a string replacement and my codeflow would be the same. Thing is, from a developer point of view, null is just a tool. The code isn't more buggy because C# has nulls. And the null reference exceptions are great signals that you have a unhandled use case (read: bug)  there.

I think that adding an option type to replace null is more of a cosmetic change. We'll still be checking if a result exists instead of null and the NullObjects pattern will still be used. Personally, I don't see this pattern as a 'necessary evil' (and some people are calling it an anti-pattern altogether) because it really makes sense in cases where you always expect to have a value. External services like logging, caching and even persistence are use cases where we expect some instance, but there is _none_ configured (and in this case 'none' is a valid - and the default - configuration option). So we simply need a value which looks and acts like the expected service but does _nothing_ , hence the null object.

I really think that most devs 'screaming' that nulls are evil are nitpicking. While null might not be the _best_ technical solution, it's not really as bad as some people claim to be. Yes, null is the easiest way, yes bad programmers write crappy code and knives can kill people. In the end the tool is less important, the tool wielder makes or breaks something.

Null is just a simple tool and I would welcome a better tool if it would become available in the future. But better should translate into tangible (yeah...) benefits like less code required, not replacing a word with another word.
