---
layout: post
title: The essence of Domain Driven Design
category: Domain Driven Design
---

"Domain-driven design is not a technology or a methodology. It is a way of thinking and a set of priorities, aimed at accelerating software projects that have to deal with complicated domains."  
So sayeth domaindrivendesign.org. Do you see anywhere Aggregate Roots, Entities, Ubiquitous Language etc? No, because in essence DDD is a mindset and that's what's important.  
  
Let me ask you something: What is the purpose of your application? What real life problems does it solve? These are the real reasons an application is developed and the application should be developed around them. But we learn somehow to start with the database, we design the database schema, what to save and how and then build the application around that. It's called a database centric approach and everybody does it or did it at one point.  However, if everything is centered around the database, doesn't this mean that the purpose of the application is the database?!  
  
Some people argue that the database is the most important part, but tell me, what's the point of staying on top of a mountain filled with gold if there are no means to mine it? Yes, you can think of the wealth, but until you can get the gold out, it is useless. The database is important only because of its content, however the means to extract and process meaningful information from the database it's more valuable and for most applications the real value comes from combining the stored data with the actual usage of it.  
  
Think of Twitter, do you use it (if you're using it) because it saves yours or other people's tweets? No, you're using it to keep in touch with people and subjects which matter to you (and to spam of course). The value of twitter is the service of fast and easy keeping up to date with what people are saying (thinking).  
  
DDD starts with asking what is the value of  the application and not where and  how we will store data. It's the natural, organic way to think about an application. With DDD, you design an application based on its domain , its value for the stakeholders and not based on a database schema. But does this approach work for every project? You probably learned that DDD is suitable only for big complex applications, but I'm in the camp of the people saying that DDD is just OOP done right.  
  
You don't start a project thinking: "this project will be quite complex, we'll use DDD ftw!" do you?  If you do, you shouldn't since the value of DDD is in the mindset and not in dogmatic rules. You don't have to use Aggregate Roots, Entities, Value Objects, Ubiquitous Language, Repositories and so on. There is no hard rule about that, but if you design the application according to its value and benefits for the stakeholders and technology constraints, then you'll be using all these 'tools' because they are a natural fit. Of course that it's natural to use the same language that the stakeholders are using, to uniquely identify a  post or a comment (_entities_), to consider that a blog has posts and a post has comments (_aggregates and aggregate root_) no matter where and how the data is persisted (_repository_).Yes, even the humble blog engine benefits from DDD - the domain is about publishing information that other people can read, discuss  and share.  What is an online newspaper (sic) but a glorifed blog with multiple authors? You can read how the [Guardian.co.uk used DDD](http://www.infoq.com/codesentations/rebuild-guardian-ddd-wills) for their new website.  
  
  Sure, some situations are very simplistic and you don't need more than an interface to send data to the database, but how do you know that beforehand? When you start with the database, you are biased to design everything to match the database structure. In a simplistic CRUD application, it's just happens that you can work directly with the database model. But if you need a bit more complexity and flexibile, maintainable code then the more changes you make, the more features are added and the more time passes, you and your company will wrestle with an increasing [Big Ball of Mud](http://en.wikipedia.org/wiki/Big_ball_of_mud) that in the end translates into more people needed, more time to implement changes, more bugs and consequently higher costs.  
  
If it is a small application, home assignment, prototype etc then yeah, you don't need DDD, TDD design patterns etc but for everything else there's Domain Driven Design.


