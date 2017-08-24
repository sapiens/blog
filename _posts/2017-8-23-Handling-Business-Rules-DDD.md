---
layout: post
title: How To Handle Business Rules in Domain Driven Design
category: DDD Decoded
---

The main point of DDD is to identify relevant domain behaviour, which means getting to know plenty of domain rules. Now, the tricky part is we have domain rules at different levels and all of them are important. Let's start with the simplest ones.

## Value objects (VO)

We're using VOs mainly for their business semantics but a lot of them encapsulate simple business constraints. A VO implementation **guarantees** that value is valid from a business point of view. And there can be many rules involving that value, however, at this level it's more of basic rules like: "quantity > 0". In some cases you might more complex validation rules and in that case, a Factory even a Domain Service are better suited to encapsulate those rules.

For example, I might have a `TransactionDate`that has 2 rules: it can't be a weekend day and it can't be an official holiday. It's easy to enforce the first rule, `DateTime` has properties for that, but hte second rule clearly is out of the VO scope. And since calculating holidays is not domain behaviour, the best thing is to have a Factory that will always create that VO.

If you're thinking: "But what about restoring it from the database?", I'd argue that using a CQRS approach means VO are always created and stored, while a Read Model (of that VO) is restored. At least this is what I consider to be the most maintainable approach.

To view things in a simple manner a VO = specific data + very close related business constraints.

## Aggregates (Agg)

An aggregate by its definition is a group of 'lower level' concepts that need to always consistent. Technically speaking, an Agg is literally a group of value objects + Agg level domain rules. Note that these are rules that are **required** to maintain the aggregate consistency i.e rules that keep things valid and prevent the Agg entering an invalid state.

I will digress a bit here, because from the domain rules point of view the Agg contains validity constraints, while from a consistency point of view the Agg defines a consistent area in our business model, which technically speaking is a Unit of Work. From another point of view, the Agg works only with VOs (these are the Agg's 'data') which themselves are a mix of data and business rules.

So, which rules belong to the Agg? The ones which involve the role and the place of the VOs _inside_ the Agg. Stuff like: "all Agg components are required". Never rules that involve the Agg itself as part of a bigger operation.

Actually, the hardest and the most important part of identifying an Agg is to identify the relevant VOs, the Agg rules themselves are quite easy to spot. And if your business rule target the Agg as a whole then it's an 'outside of the Agg' rule that should be part of a Domain Service.

## Domain Services

Most of the obvious business rules which don't validate simple values will fit here. You know that "username must be unique" rule? That's the one! Or "the user can post at most 15 videos" etc. A lot of developers mistakenly put this as part of the Agg which itself is improperly modelled, resulting in a long term mess waiting to happen.

These should be part of a Domain Service and depending on the rule, it can be technically trivial or not, to implement it.

### Business Impact

Yeah, we have to talk about that 'cuz the rules can have an... impact... on how simple or complex or maintainable your code will be. And that translates into how quickly you will deliver functionality aka productivity, how much technical debt you'll going to acquire and how expensive (in money or work-hours) enforcing a business rule will be.

Here it's something you have to talk to the Domain Expert and the project owner. In the end everything boils down to what's cheaper for the business: to enforce a rule 100% or to accept occasional violations.

Let's take the "user can post max 15 videos" rule. The simple implementation is to read your business state (read model for CQRS) and find out how many videos that user has. The only problem is concurrency (and not eventual consistency). What happens if the user presses the button twice fast or something.

It's a fact of life that everything is (perceived) delayed. Even with the fastest hardware and there's still going to be a delay from the moment you sent that query until you get the results. 1 microsecond before or after someone changes things. It's inevitable.

And that matters when you're dealing with high concurrency. Because at one point the business rule will be violated. The question is what happens next. Sure, you can come up with some complicated solution to enforce it, but is the Client willing to pay the price? Maybe is cheaper to ignore those edge cases with very little probability to happen and possible 0$ business impact. Or that rule needs to be enforced at all costs, no matter what.

What if the rule is very important but it's a line of business app used by at most 5 people with close to 0 chances to become a high concurrency app? We can implement a very simple solution which takes advantage of that and we're very aware that won't be scalable past a specific number (that we hope it won't be achieved). As always, it depends, but you have to be aware of both the technical and business context. And there's no single best solution that works for every situation.

Luckily, very few applications need 100% percent enforced rules and are highly concurrent. And for those cases, you need _very specific_ solutions.

In implementing rules, we look for the simplest solution and try to evaluate the probability and impact of the edge cases.

## Conclusion

 Business rules can be found everywhere in the domain and it's our objective as domain modellers to assign those rules to the relevant patterns. And probably you'll be surprised to find out that most rules are part of VOs and Domain Services. That's why properly understanding all the patterns is the key to success: all are equally important and we should let the Domain tell us what belongs to where, instead of playing favourites (I'm looking at you 'Aggregate' classes full of primitive type properties).