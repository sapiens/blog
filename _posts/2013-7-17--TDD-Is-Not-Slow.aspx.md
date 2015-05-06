---
layout: post
title:  TDD Is Not Slow
category: Best Practices
---

I think the 'slowness' of TDD is the most frequently used argument for not using TDD. But I don't agree with it and I see it more of a reason to continue the 'old ways', a resistance to change. Here is why.

 Writing code without checking if it works (does what it should do, not just compiles) is very poor practice. I understand that in some cases you just modify something small and because of the time constraints you just ship it but this is just a reason to have automated tests. The point is you should always make sure that the code works properly.

 In the old days, the most common method was to write the code and then manually test it (enter data, click, view results etc). Guess what, it takes time. And lots of it. Nevermind the fact that when you find a bug you have to spend TIME hunting it down. The fix it, then test again manually. I think this is the most time consuming and cumbersome way to do it. And the sad part is that for many (young) developers this is the only way to test.

 Because doing things manually takes time, someone thought about having automated tests. Click once, the tests run and after a few seconds (minutes if there are thousands of tests) you ca see the result. Of course, this means someone has to write those tests. Have you written unit tests after you wrote the code? I bet you did and so did I. It's boring as hell with the added 'benefit' that knowing how the code is implemented allows you to come up with some well 'optimized' tests exactly for that implementation. Oh, and hope the code is easily testable in the first place, or else you'd waste even more time to refactor it.

 See? Testing without TDD is already slow. Yes, the only way to be fast is to not test at all, but then we're talking about delivering a poor product. If I am the client, I want a good product. I'm not paying you to deliver shoddy work, that will break as soon as an user touch it or every time a feature needs to be added. Poor product means a lot of time wasted fixing bugs and very slow changes. Your 'fast' delivery turns out to be a very SLOW maintenance and progress. And angry stakeholders.

 Back to testing, automated testing doesn't mean TDD. You have to write tests first before coding to be come TDD. And here's an interesting fact about it: you have to actually think what you expect from a piece of code BEFORE you write it! Madness! Instead of just writing code, you have to think before you code! And if you can't formulate properly an expectation, HOW do you write the code in the first place?!

 If the automated tests are slow, that's because your code might have unwanted coupling to slow dependencies such as the db or a remote service. But they are still faster than testing manually.

 Writing tests first doesn't slow you down. It makes you think properly about what functionality you need to implement. It also ensures the code is working properly. When you finished coding, you KNOW the code can be trusted and you have the prove (tests) for that. And I can bet that the code developed with TDD looks quite different than the code developed without it.

 Writing tests before or after coding will take some time. But you'll have to do it anyway, if you want to deliver a proper product that is. Instead of assuming that it works or getting bored writing tests after, be efficient and **combine designing and testing into one action**. TDD SAVES time while ensuring quality. I don't know many people who love spending their time hunting down bugs with their boss (or client) heavy breathing over their necks, so TDD helps you keep the bug hunting at very low levels. It's a win-win for both the developer and the client.

 The problem with adopting TDD is that is a 'weird' way of doing things. And you have to change the way you're doing things. But you know, everybody looks for tools and methods that helps them do things faster and better. You're using a (micro)ORM because it makes it easier to work with a db. You can use the TDD tool as way to develop good code faster.

 As a client, I'm not paying you to deliver me thousands of lines of code, I couldn't care less about that. I care about YOU delivering me a product that works properly and it's not bug ridden and it's easier to implement changes. I will promptly forget that you delivered version one in 2 weeks if I have to waste another 2 weeks to fix (some of) the bugs, then you need more and more time to implement changes. I'm not paying you to fix bugs and I want my software to evolve fast so I can keep up with the competition.

 TDD makes you faster in developing and maintaining the product for its whole life span. In the realities of today, you have more and more software that needs to adapt and evolve in real time, as opposed to 'next version'. In the age of web apps, PaaS, SaaS etc next version means tomorrow.


