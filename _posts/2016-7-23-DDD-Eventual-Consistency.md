---
layout: post
title: DDD Decoded - Don't Fear Eventual Consistency
category: Domain driven design
---

Inevitably when talking DDD, someone asks: "It's all great with those aggregates, events and stuff, but how do I query things?". Inevitably, the answer is:"Use CQRS" and inevitably, a new issue appears: "You have to account for Eventual Consistency (EC)", a necessary evil.

But I'm going to say this: The only people worried about EC are CRUD programmers. 

# Let's go back in time 100 years

No computers, no CQRS, no EC. Or maybe.... Neah! John and Mary have a shared account at a local bank and it just happens that they decided to withdraw 100$ the same day. Unknowingly, they interact with different tellers, just a couple of minutes apart. Being an epoch without computers, for every operation needing to check an account's balance the tellers need to talk to another clerk who's in charge of maintaining the balances. At moment the _T_, the first teller goes and gets the account balance (it's 100$). Then he starts to fill in the required paperwork, then handles John 100$. In the mean time, at _T+2_,the second teller (serving Mary) goes to the back-office clerk and gets the account's balance. While he's on the way back, teller #1 goes to tell the clerk that a transaction has occurred and the  account balance needs to be updated. Teller #2 returns to Mary, handles the paperwork and gives her 100$. Then he goes to the clerk and tells him to register the new operation, thus updating the account balance.

John and Mary withdraw together 200$, from an account which only has 100$. When the clerk calculates the latest account balance, it gets a negative number.
 Dang! We have a problem! Does the bank blame it on CQRS and Eventual Consistency, heads will roll because nobody used a Unit of Work? Nope! The bank says: "Meh... yet another opportunity to make money! Start charging interest on the negative balance and notify the client they owe us 100$(+interest)". Business as usual.

 As we can see, Eventual Consistency exists without computers or CQRS. Like rain, it's a normal phenomenon in the real world. And businesses exist in the real world, so for them EC is just normality, even if a domain expert doesn't know what the words "Eventual Consistency" mean. And being normality, the business has a solution for this problem.

 As programmers, we are _trained_ , almost indoctrinated to pray to the ACID god. We feel fragile if 2 changes aren't wrapped in a transaction and EC is like the boogie man, we fear it and try to avoid it. But this is just the CRUD mindset, a one trick pony that doesn't reflect a moderately complex domain.

 DDD is about changing our mindset from CRUD, data driven to business driven. And the funny thing is, not only that EC is pretty normal but it's the default,too. Remember that we try to _identify aggregates_ i.e we're actively looking for areas, islands that need to be **immediately consistent** in an ocean of eventual consistency. Those are not in-your-face visible, we have to carefully 'hunt' them, in a way proving that only in those cases we need a unit of work.

 And maybe that's one of the biggest things with DDD, the revelation that a model doesn't really need to always be immediately consistent, like in the CRUD approach; some parts need it, others don't. Our objective is to identify which needs what.

 So, don't fear Eventual Consistency and try to go beyond the limited CRUD mindset. CQRS works great and usually the delay between sync-ing our event store to a read model is measured in milliseconds. In a world without computers, the delay is way bigger and if the business can survive that, it won't care about a couple of milliseconds.  
    






