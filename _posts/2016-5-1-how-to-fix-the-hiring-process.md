---
layout: post
title: Let's Fix The Broken Hiring Process 
category: Business
---

Posts about the stupidity of the hiring process aren't news in the developers community. Most readers agree with the author that the process is broken, often humiliating and more important, not _that_ effective. Very few people are actually defending the process but for whatever reason this is the norm in the industry. Like many others, I've read and agreed with [Sahat Yalkabov](https://medium.com/@evnowandforever/f-you-i-quit-hiring-is-broken-bb8f3a48d324#.xkw5f6r14). We have a problem and seemingly very few solutions. Some say the process, like democracy, might not be the best, but it's the best we have now; after all, the big, successful companies use it and they're doing well. However, in my opinion the process is not even remotely decent and if this is "the best we have now", it's sad considering it's a field populated by many smart people. 

## What are you looking for?

But let's start with the beginning. What's the purpose of hiring a developer? Note the word _developer_, I didn't say 'programmer' and the distinction is important. A developer builds products that represent value for a business, a programmer writes code. Any business wants to lease, rent or buy assets that provide functionality needed by it. From this point of view, developers are just rented assets. And you need them to deliver business value. No manager (or stakeholder) actually cares about the code itself, they care about the functionality the company gets from that code. The code is just a production detail.

When your hiring process targets programmers, you want people who can write code. When you target developers, you want people who deliver business value (using code). Do you think that after you get through a byzantine process and you've played your part in the interview theater and finally got the job, your boss (and their boss) will care about how efficient or cool or even maintainable your code is? No, they care about you implementing feature X. You fixing bug Y. You don't get a bonus if you are the author of that great algorithm. Actually, they couldn't care less if you've just copy pasted it from Stack Overflow. They just want you to deliver business value.

Sure, it's important to come up with your _own_ solutions, but the most important thing is to come up with solutions. Maintainable, reliable solutions. Your boss doesn't care who spent 3 nights at home to think about the solution, they care about who implemented the solution, regardless of who did the hard work. So, yes if someone on SO will design an architecture for you, from your team/boss/company point of view **YOU** are the problem solver. Now, if you fail to understand the solution, the value you'll bring is not going to be enough, but the point here is that stakeholders want a valid result and are not concerned about **how** that result came to be.

The current hiring process looks for people who can theoretically solve problems on their own. The issue here is that many problems have been already solved and reinventing the wheel is just wasting time (read: money). And this is exactly the kind of problems a candidate is asked to solve in an interview. And if you deliver a working solution found on SO, it's bad because it wasn't you who thought of it. As if the original source of the solution is the most important thing, not the solution itself.

## Is it about showing off (ego) or delivering value?

You're not tested if you can bring value, you're tested if you can think yourself of a solution from scratch regardless if it's needed or not. The reasoning here is "I want to see how you think, how you solve problems". But wait, the solution needs to come from you directly, you can't use already existing and available solutions that can deliver value fast. No, you have to prove that "YOU're the (wo)man!". After all, in your daily job you'll be inventing the wheel everytime, 'cuz using an already invented wheel is not cool. And if you can prove you can build a wooden wheel, it's so obvious that you can build a car, too!

Most interviews are about showing how smart or knowledgeable you are, instead of showing your worth of how quickly you can deliver business value TODAY. And the logical fallacy is that if you know how to build a hammer, clearly you know how to be a good carpenter and you can also build a house. An interviewer tries to guess today, by the means of artificial scenarios or worse, puzzles, how 'good' you'll be in the future, in an _unknown_ scenario. A good developer would know about the principle of [YAGNI](http://blog.sapiensworks.com/post/2013/06/27/The-Fallacy-of-YAGNI.aspx). It seems that people conducting technical interviews don't know that it can be applied outside the code as well. Hint: The point of YAGNI is that you shouldn't waste time trying to guess the future functionality.  

An interviewer should care about the ability of a candidate to deliver solutions. If you ask someone to reverse a string and they invoke a function from a library, that's a valid solution. Because it does what it is supposed to do, in a maintainable manner. Writing a new function is a waste of time. You can build a better string reversal, but the question is: do you really need it or the current solution is good enough? Doing 'what if' katas doesn't bring value in this context. 

You can ask the candidate to solve a simple, easy to explain and to understand domain related problem, a realistic scenario where there aren't libraries or the problem is to specific to google an answer, and the candidate must be able to deliver a working, maintainable solution. 

A business doesn't have time to waste, it wants value as fast as possible. And it's the job of a developer to know when they can use an existing solution or when they need to customize or to build one from scratch. If the interviewer asks about problems that have been solved 1000 times already, expect the candidate to NOT reinvent the wheel and to use a library or a tool like google to solve it.    

## "He who doesn't have wise men should buy them" - Romanian Saying

Another problem is that it seems very trendy to disregard the practical experience of a candidate. Not only this is insulting, but it's a sign of the process' stupidity. In any field (except IT?!) experience is a highly priced asset. You know why? Because it can't be taught and it can't be faked. I've seen this reasoning from some interviewers: "people lie all the time on their CV" or "I don't have 2 hours to spend to read your blog, your StackOverflow profile or your OSS work [therefore I will ignore it]".

Well, if YOU, the technical interviewer, a senior developer, can't tell if someone lies when they're talking about their experience, it means that YOU don't have experience and you shouldn't conduct technical interviews. Any developer worth their salt will detect in 2-5 minutes if a candidate tries to bullshit their way out of a question. Only people who don't have a clue think you can fake experience. They're detected so fast it's not even funny.

And you don't need hours to assess someone's technical level. A quick read of their CV or better, their code, technical blog (just look at the titles) or SO profile(number of answers/questions ratio for the relevant tags) and in 10 minutes you have a vague idea (junior, middle, senior). Then you can just talk to the candidate about their work and you can (gasp!) solve together a real problem. Using a computer, writing code, delivering value. 

Things like "every engineer should know this [trivia bit that shows how smart I am]" are also pointless, _unless_ your daily work requires everyone to use that. Don't ask about things that "anyone who took a 101 CS class should know" if they aren't needed. Remember that TV show "Are You Smarter Than a 5th Grader"? Well, are you? After all, they are things every 5th grader knows. How come YOU don't them?  
 
 ## A simple solution
 
 If we apply a bit of common sense (I've heard Walmart sells it, limited offer, get it while it lasts), a technical interview can be a very efficient and cost effective affair. A candidate who seems good enough to be invited to an on-site interview should be treated for 1 day like a new member of the team. They will have to work with the team to solve some problems that doesn't require a deep understanding of the existing code base. For example: ask them to fix a simple bug, implement a minor functionality, refactor some technical debt etc. But the point here is to put them into a context where they can deliver business value.
 
 As an interviewer you get these benefits:

 * Simple and natural to implement, you don't need special training or reading what questions to ask 
 * See how the candidate communicates 
 * See how the candidate works in a team
 * See the _actual_ worth of the candidate. Potential is good, delivering value on the spot is better. Maybe they will tell you about a new library you didn't know about. Maybe they show you a different way to approach a problem. Maybe they spot (and fix) some bugs for you. If the candidate is good, it would be very beneficial for the company even if you don't make an offer (but you should). A weak candidate will be weed out in a couple of hours at most and in that case don't waste time, just send them home.
  
 As a candidate, you won't be the subject of an idiotic, artificial process designed by people who either don't care or they're simply incompetent. No 4-5 interviews, no 2-3 weeks/months process, no repeating the same answers to the same (basic) questions asked by different people. No wasted time and you get to show what you know and what value you can bring. In return, you get an idea about the quality of the code base, the management process, the tools, the working environment etc. It's WIN-WIN!
  
 Any developer worth their salt will LOVE this. And I know there are some (very few...) companies  who are using a similar approach and they are very satisfied with the results. Others favour a homework style approach (sometimes paying the candidate for their work) which I can't say I prefer but it's still way better than the standard whiteboarding crap we have today.
 

## Conclusion

Want to hire good developers, with minimum effort and cost?

* Test the ability to deliver business value, instead of raw knowledge
* Don't discard experience, try to use it to get more insight from the candidate
* When a candidate shows you their Open Source code or StackOverflow profile or their technical blog posts, they're making your life EASIER. They give you fuel for your questions
* See how the candidate performs in your day-to-day working environment, solving real business problems.

To quote a redditor
>If you want to hire a truck driver to take your goods to New York, you put it on a truck and see how fast and safe he can reach New Your, you don't test its truck-drving-to-new-york skills by asking him to make stunts on a bicycle.


