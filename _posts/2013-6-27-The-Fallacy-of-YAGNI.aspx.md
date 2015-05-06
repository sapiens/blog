---
layout: post
title: The Fallacy of YAGNI
category: Best Practices
---

I'm seeing YAGNI (You Ain't Gonna Need It) thrown around as much as DRY (Don't Repeat Yourself). If you don't know what YAGNI is, it's an extreme programming principle which says to not waste time writing code for things that you don't need now but you might need them later, because you 'ain't gonna need it'.

 But as with any tool there is a time and place where it makes sense to use it. You can find the extreme YAGNI users, who are using it without even knowing. These are the 'pragmatists' that churn out code rapidly without wasting a second to think about long term maintenance. They are the authors and silent advocates of the Big Ball of Mud (BBM) and Spaghetti coding style. They apply YAGNI anywhere and anytime.

 At the opposite spectrum we have the people who fear YAGNI. They take time to think about any scenario that might appear and include features that might be needed someday.  Lots of time is wasted and probably their applications are over engineered. The resulting code is only marginally better than of the BBM crowd.

 YAGNI is a sane principle which requires to think a bit though. It's stupid not to take into account the long run or to write dozens of cases that won't ever happen. You can't codedict the future but you can't live only in the codesent, codetending the things will not change. Simply put, you have to decide when to apply YAGNI.

 Sadly, I see that this principle is invoked as a reason to not do something.  I wholly agree that it's a waste of time to code features or scenarios that you think that MIGHT happen, without any actual prove that it will happen but it's unwise to simply invoke YAGNI because it saves a whole 20s.

 Fact is, many applications change over time. And the changes come much faster than before as most companies try to be agile. Your code needs to be codepared to change often. You don't know what the change is, but you do know that it will come. The least you can do is to design your app so that it can handle change easily. Saying YAGNI will just make your life harder here.

 Fact is you already are codeparing your app for future change even if you don't know it. The layers you split your app in are exactly for that purpose. You might do it automatically because you are used to it or you consider it good practice but the benefits are about handling change. If you'd apply YAGNI here, you'd write the first iterations of your app simply as a monolith where UI, business and persistence are mingled together. At that point you don't need separate layers, so YAGNI means mix everything together. The next change means you modify only that parts you need and I can bet it will take a LOT of changes before you will REALLY need a proper layering.

 Using a DI Container might be an overkill. Do you REALLY need to inject abstractions when you know that you have only one concrete type and it's very unlikely to add another? YAGNI says to not bother with it, when you'll need it you'll refactor then, right? Easier said than done, that's for sure.

 What about the Repository? Well, yo have an ORM or a document db so a repository is useless because who changes the storage anyway? You could google a bit about mysql/postgres to mongodb or scalability issues and see the thousands of results. Yes, the persistence could be changed, not very often and certainly not after you just launched the app. But, if you understand the pattern you know that the Repository's purpose is to keep the business layer decoupled from the persistence and not to hide a database. Even simpler: it allows you to change things in either layer without worrying that the change broke something in the other layer.

 You see, YAGNI is not an excuse to have a poorly designed app that will become harder and harder to change over time. It's a reminder not to add useless features and to avoid divining the future. But with the exception of a throw away prototype or application that you are absolutely sure it won't change over time, it's a bad reason to not do proper app design and to avoid designs patterns or best practices.

 There's no recipe when YAGNI is good and when it's just an excuse, that's your decision as a developer. However, it's much easier to use a DI Container from the beginning, you're already using MVC or 3 tiers by default and a properly applied Repository allows you to develop and refactor your business layer without even knowing what database you'll use. Unit tests are there for your sanity while TDD provides the benefit of helping you design an object better AND write tests at the same time.

 Some things are just clutter while others need a 5 minute investment now and allow you to rip big benefits later. YAGNI just tells you to decide if the code you're going to write is just a cost or an investment.


