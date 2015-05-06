---
layout: post
title: When Not To Use Design Patterns
category: Best Practices
---

I do like the fact that lots of devs want to write good code and that the school caught up and actually teaches design patterns. However, it does so poorly, but this post isn't about the education system. It's about when and how to use deign patterns, regardless where you learn them.

 I can recognize a design patterns novice in a few seconds. They heard that design patterns are good and should be used everywhere all the time. And they come up with questions like:"I'm starting this app, what design pattern should I use?" (?!?!?!?!?) or "In this case what design pattern is the best: X or Y?"

 Well, you see, I've been there and I know that when learning about patterns you feel you've found out that magic recipes exists. Sadly they don't and I'm telling you, very seriously to NOT use design patterns. Unless you really understand what they are.

 Yes, I'm aware you know the 'definition'. But the important thing is design patterns are not silver bullets nor recipes. You don't decide what design pattern to use. You come up with a solution to a problem. In time you find out that the solution is quite used and it's considered a design pattern and has a name.

 This comes with experience and not by learning design patterns by heart. And contrary to what you might believe, there isn't a design pattern for every problem you have. I see this worrying trend where the first thing a junior does when he has a problem is to go ask someone (on stack overflow): what pattern should I use here? **None!**

 Even if there were a well known pattern, at this point you don't understand them, you haven't spent the time to come up with a solution in order to apcodeciate the pattern, in order to understand WHY the pattern works that way. And judging by the number of people misusing the Repository pattern, chances are you will think that you've understood the pattern when in fact you're applying an anti-pattern.

 When coding, don't think about design patterns, think about solving problems. It doesn't matter that your solution isn't the perfect one, few are and from the first try, none is. Good solutions appear in time and until you haven't dealt with a bad solution, you won't understand the good one.

 Read about design patterns and use cases. But that's it. Don't try to fit them in your app. Chances are you're using at least one pattern anyway. Some you don't know about, others you are fully aware of. It doesn't really matter, in time you'll get to recognize the use cases and the patterns and you'll know their names.

 Don't think that you can find a shortcut, you can recite all the design patterns in existence (I don't know if there's someone knowing all the patterns) that doesn't mean you know when to apply them and how to apply them. In some cases, you need to alter the pattern in order to fit your problem and you can do that safely only if you understood both the problem and the pattern properly.

 Design patterns are great to use, but it takes time as it's an evolutionary process. And even they can change. The Singleton is not what it used to be, many people are considering it an anti-pattern now. Personally, I think it still has some uses but I favour a DI Container if available.

 Once again, don't think about design patterns, think about solving problems. There's no benefit in trumpeting "I'm using design patterns" when you don't understand what you're doing and you're actually using anti-patterns. Good code comes with experience, there is no magic elixir or hidden secret that will make you write good code from the beginning.


