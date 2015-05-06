---
layout: post
title:  What is the best way to do this? What is the best design pattern for this problem?
category: Best Practices
---

Or "Can you show me a full project using DDD / CQRS"? These questions appear very often on StackOverflow (SO) and it shows that people want to learn things properly (and that some people are really lazy). Unfortunately, knowingly or not , they're asking for a recipe for their problem.

 A lot of questions are from people who think that someone more experienced has all the best answers. And they want to learn directly the best solution, skipping all the pain of using less than optimum solutions. But here's the catch: if I'm telling you directly the "best" solution (which might not really be the best in your case) it means I'm thinking in your place and doing your work.

 How would you learn if you don't spend the time battling the problem? How can you understand why a solution is better than other when you don't have even a duct-tape solution that shows you tried to solve the problem in the first place? The worst I can do for you is to give you directly a solution (even if I explain it to you), you'll think of it as some recipe and try to use it again in a different context where it's inappropriate.

 I've noticed this mindset "give me directly the best solution, so I won't be wasting time to find it". As developers we're always learning (regardless of our experience) and we learn by doing things. Also, it's our job to come up with solutions. If there were recipes suitable for all cases, there wouldn't have been a need for developers. There would be a software writing programs.

 It's our job to think and if you're a student/junior you have to think even more than a senior. Because your brain needs to learn how to think. Receiving all the solutions on a silver plate doesn't help, it just makes you lazy or worse, a help vampire. Unlike school, in the real world we first have a problem then we care for a solution. School gives you solutions to problems you never had or have. You'll know a lot of answers but you don't know the questions.

 The best way to solve your problem is for you to think  and come up with ANY solution that works. If you can't find one google for it, chances are someone else had that problem too. If you're not satisfied or you don't have a solution then go and ask more experienced developers about a solution. But not the best solution. It's up to you to come up with the best solution because it's your problem. At most you can ask people to help you improve your solution, but you should have a good reason asking for it.

 While you can think of design patterns as optimum solutions to common problems, there isn't a design pattern for every problem you might have. I cringe every time I see on SO questions like: "this is my problem what design pattern should I use?" . Someone told them about design patterns and clearly we need a design patten to solve the problem.

 We don't. Personally, I don't care about design patterns. I use them a lot, but when I have a problem I just one to solve it, design pattern or not. Sure, I'll discover later that my solution is a design pattern, great! Now I know how it's called. This doesn't mean I'll try to use next time.

 Of course, as you get more experienced you'll be using some design patterns by default and you'll be very aware of them. But that's it. I don't know all design patterns and I'm using only a handful. I've seen that students and juniors are a lot about counting design patterns or their names.  Yes, it's great to know more, but if you don't have the problem you won't understand the solution. Throwing buzzwords around when you don't understand them will just tell everyone that you don't really know much.

 And there are some things that can be learned only by doing them. DDD is on of them. CQRS as well, although CQRS is just a principle. Learning DDD has little to do with naming your classes Entites, Value Objects, Repository etc and a lot to do with business modeling. A lot of people ask for an example of DDD. Here's the fact: it's useless to show you code done with DDD, because you'll see only a result but not the design process, the most important thing in DDD.

 The only way for you to learn something is for me (or others) to [explain the code](http://www.sapiensworks.com/blog/post/2014/04/29/-Modelling-DDD-Behaviour-And-Use-Cases-By-Example.aspx) bit by bit in order for you to understand why the code a written that way. And there are books for that. But this is the most someone can do for you to teach you DDD. The rest you'll learn from experience. You just start to do it. And you'll do it improperly but you won't be aware of it. But in time you'll hit some walls and you'll start thinking and realize that your DDD isn't quite the 'real' DDD. And you think some more and after some time and problems solved you'll finally understand how things are and why they are. Identifying Aggregates or Aggregate Roots will become much easier.

 About CQRS, I don't really understand why so many people need a code example (or worse a full project). What's so hard in using 2 models instead of one? I think this issue is very common with people who heard about CQRS but again, never had the problem CQRS solves. When you do a lot of CRUD, CQRS doesn't make sense. But try modelling properly a rich model to be suitable for querying and CQRS becomes obvious.

 See? A lot of things make sense after you tried to solve the specific problems. Learning solutions before  problems is a waste of time. Asking people to give you the best solution on a silver plate shows you're just lazy and that you won't learn anything. And in many cases, there are a lot of factors that need to be taken into consideration in order to come up with the best solution which you'll realize later that wasn't really the best solution.

 In conclusion, start solving problems. Is your job as a developer, that's why you're paid and that's how you'll progress. In software development there aren't magic recipes (or tools) or ancient secrets that the more experienced (and selfish) developers keep for themselves. There are only principles, guidelines and best practices which can be [bent](http://www.sapiensworks.com/blog/post/2014/04/22/Bending-The-Principles-Separation-of-Concerns.aspx) if the context demands it. 


