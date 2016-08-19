---
layout: post
title: DDD Decoded - Application Services Explained
category: DDD Decoded
---

We have all those domain models, [aggregates](http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-1) and [domain services](http://blog.sapiensworks.com/post/2016/08/16/DDD-Domain-Services-Explained) but they are just parts of business cases, floating around. We need something to put things together, to orchestrate the fulfillment of a specific business case. We need a manager of sorts and in DDD that role is performed by an **Application Service**.

# What does an Application Service (AS) actually do?

The simplest way to look at it is as the 'hosting environment' of a business case. It takes care of invoking the correct domain models and services, it mediates between Domain and Infrastructure and it shields any Domain model from the outside. Basically, in a DDD app, only the Application Service interacts directly with the Domain model. The rest of the application only sees the Application Service (interesting choice of name).

Business functionality is expressed as business cases and the AS acts as a shell, encapsulating one business case (it should anyway, in practice we can host a simple process i.e more than one business case, but that's more of a pragmatic compromise). This means the UI/API or some higher layer talks to the AS responsible for that requested functionality and the AS gets to work. At most, the calling layer receives some result which shouldn't be nor contain a domain model i.e entities or value objects.

It's important to note that an AS doesn't have any business behaviour. It knows only **what** model to use but not **how** that model works. So, the AS knows which Aggregate to invoke, it knows about some Domain Services that might be required, but that's it.

# An example

Let's continue the [money transfer example](http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-2). But now, we have the business rule that says the transfer can be done only if the account balance has enough money. So we're introducing 2 Domain Services: `Calculate account balance` and `Can account be debitted` both implemented as functions of a `IDomainServices` abstraction. But we also need an `AccountBalance` Value Object. 

For obvious reasons, the code is heavily simplified. Also, this is just one of the possible implementations. It's a matter of style in the end, as long as the code is clear and respects the domain model you can go wild with the implementation.

```csharp

  public class AccountBalance
    {
        
    }

    /* Domain Services implementation */
    public interface IDomainServices
    {
        AccountBalance CalculateAccountBalance(AccountNumber acc);
        bool CanAccountBeDebitted(AccountBalance balance,Debit debit);
    }

    /* abstraction representing the query model, implementation resides in Infrastructure */
    public interface IDomainQueries
    {
        AccountNumber GetAccountNumber(string number);
    }

    /* Event store abstraction, basically our 'repository' */
    public interface IEventStore
    {
        
    }

    //command object, data structure
    public class TransferMoney
    {
        public Guid Id { get; set; }
        public string AccountFrom { get; set; }
        public string AccountTo { get; set; }
        public decimal Amount { get; set; }

    }

    /* Application Service!!! */

    public class TransferManager
    {
        private readonly IEventStore _store;
        private readonly IDomainServices _svc;
        private readonly IDomainQueries _query;
        private readonly ICommandResultMediator _result;

        public TransferManager(IEventStore store, IDomainServices svc,IDomainQueries query,ICommandResultMediator result)
        {
            _store = store;
            _svc = svc;
            _query = query;
            _result = result;
        }

        public void Execute(TransferMoney cmd)
        {
            //interacting with the Infrastructure
            var accFrom = _query.GetAccountNumber(cmd.AccountFrom);

            //Setup value objects
            var debit=new Debit(cmd.Amount,accFrom);

            //invoking Domain Services
            var balance = _svc.CalculateAccountBalance(accFrom);
            if (!_svc.CanAccountBeDebitted(balance, debit))
            {
                //return some error message using a mediator
                //this approach works well inside monoliths where everything happens in the same process 
                _result.AddResult(cmd.Id, new CommandResult());
                return;
            }

            //using the Aggregate and getting the business state change expressed as an event
            var evnt = Transfer.Create(/* args */);

            //storing the event
            _store.Append(evnt);
            
            //publish event if you want
        }
    }

```
 
This particular approach uses a message driven architecture as well as Event Sourcing. We see how the AS orchestrates the business case; the domain model isn't aware of the command object, the event store or the result mediator. One thing that might be confusing is the branching, it almost looks like the AS needs to know the domain rules of what happens when an account balance is too low, however it doesn't :) . That's one of the "tell me if I can continue with the business case" type of situations and the AS uses a Domain Service to get an answer for that. Again, it's a matter of knowing **who** is responsible for something, not **how** something needs to be done. The AS _never_ micro-manages.

If you want to keep things even simpler, you can call this service directly from the UI/API layer, and make `Execute()` return a `CommandResult`. We can implement things as complex or as simple we want, as always, it depends on many factors.

# Conclusion

The Application Service is the host of a business case and acts as a mediator between the app and the domain model. As long as you keep the domain behaviour inside the dedicated patterns and none leaks into the AS, you're good to go. Implementations vary and there's not a recipe of how a "good" AS looks like. Just remember its role and respect the domain model, the actual code is up to you.     

