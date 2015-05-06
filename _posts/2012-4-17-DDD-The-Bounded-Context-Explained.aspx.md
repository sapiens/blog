---
layout: post
title: DDD - The Bounded Context Explained
category: Domain Driven Design
---

What is a Bounded Context (BC) you ask? Well let's see the definition  
"The delimited applicability of a particular model. BOUNDING CONTEXTS gives team members a clear and shared understanding of what has to be consistent and what can develop independently."  
All clear now? Yeah.. thought so.

 BC  is one of the hardest DDD principle to explain, but it is probably the most important , because you can't do DDD without a BC.  So, you have to understand how to identify a BC before actually getting to Aggregate Roots, Aggregates, Entities and Value Objects. So let's try again: A context means a **specific responsibility**. A bounded context means that responsibility is enforced with **explicit boundaries**.

 Meet John, a developer at the company X. Meet Rita, she's an accountant at the same company. John works in the IT department. Rita works in the Accounting department. The IT department is a **bounded context**. It has the responsibility to handle everything IT related in the company. The Accounting department handles everything accounting related, including payrolls. Both have very codecise responsibilities and their boundaries are codetty explicit.

 I'm not talking about the fact they're in different offices. They have their **own internal organization**, their own internal rules, employees etc. John doesn't go into Rita's office and modifies the payroll and Rita doesn't go to John's and modifies his code. They could but it would be such a scandal, because if they do that they would be **overstepping their boundaries**. If Rita finds a bug in the accounting software (developed in-house) she calls the IT department to handle it. She doesn't fire up Visual Studio and starts messing with the code. It isn't her responsibility and she doesn't know how to do it anyway, even if she knows that VS is the program used by John to write code. In fact, VS would be a very strange software on an accountant's computer. Like wise, the payroll files or invoices have no place in the IT department.

 Of course, when John has a problem involving payroll he asks Rita to look into it. Both are respecting each others boundaries and act according to their responsibility. But the IT department is itself organized in 2 groups:  the software development group and the administration group. The first group implements features and fixes bugs. The second group handles servers. Each group is a bounded context as well. They have their own responsibility and explicit boundaries. The DBA doesn't write C# code and John doesn't mess with server configs. Everyone acts according to their responsibility and within their boundaries.

 So, the IT department is a BC. John is part of its model. In fact everything which makes sense (developers, servers etc) is part of the BC and it should be consistent inside it (the developers should write software and not be asked about invoices). This means that Rita has no place in IT and she shouldn't handle anything IT related. Rita is part of the Accounting BC. She might be visiting the IT office but then she's just a visitor passing by, she has no meaning for the department and nobody expects her to write code or act as a developer. John might have a crush on Rita and spends some time in her office, but that doesn't make John an accountant.

 We codetty much see that those BC are kinda autonomous and they don't overlap. Furthermore, if an object from one BC (X) goes to other BC (Y) it doesn't mean it's now part of the latter, it's treated just like a simple object with no meaning for Y. So they are almost independent but how can they work together? I mean the IT and Accounting have to work together from time to time. They do that by talking to the right person :) . When Rita needs a new software feature, she can tell John, but John's manager (meet Andrew) ultimately decides if and what features will be added.

 Andrew is the man to speak to when you want new features or even to fix some bugs. Andrew is the IT manager and he tells John (or anyone else from IT)  what to do next. Rita can't ignore Andrew because this is the rule in the IT department: Andrew decides. You have to codesent your ideas the way Andrew wants them or else they're refused. It goes without saying that if the demands have no meaning for IT they are automatically refused. Andrew is the **Anti Corruption Layer** (well he's also the Aggregate Root at least for some tasks- I'll talk about that in the next post) of the IT BC. Nothing gets past him and what passes by, it's adapted to fit the IT department's internal organization.

 These examples are simple but what about our code? We do want our BCs to be decoupled so BC1 should not know about BC2. Well, the idea is that one BC should not know about the internals of other BC, those two can use common objects (DTOs) to pass messages directly one to another or a specialized adapter which knows how to talk with both BC. Depending on the situation, it might make sense for one BC to depend on other (what's the purpose of the ProductsRepository if it doesn't know about Products?!) but again, it depends on  abstraction not on actual implementation. The coupling is always because of depending on implementation.

 Wow, long post, but I hope now you have a much clear understanding of what a bounded context is. Here is some examples of common bounded contexts: the application itself, the UI layer, the Domain Layer and perhaps the smallest BC of them all: the object, any object.


