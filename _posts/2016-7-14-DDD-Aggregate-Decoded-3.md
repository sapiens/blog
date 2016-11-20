---
layout: post
title: DDD Decoded - The Aggregate and Aggregate Root Explained (Part 3)
category: DDD Decoded
---

* [Part 1: Theory](http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-1) 
* [Part 2: Modelling Example](http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-2)

Coding time! I bet you've been waiting for this part for some time. Yes, once we have our model, it's time to write the code. Let's start with the Value Objects first.

```csharp
public class TransferNumber
    {
        public string Value { get; private set; }
        public Guid EntityId { get; private set; }

        public TransferNumber(string value,Guid entityId)
        {
            //validate format based on domain rules

            Value = value;
            EntityId = entityId;
        }
    }

    public class AccountNumber:IEquatable<AccountNumber>
    {
        public string Number { get; private set; }
     
        public AccountNumber(string number)
        {
            //validate format based on domain rules

            Number = number;
            
        }

        public bool Equals(AccountNumber other) => other != null && Number == other.Number;

    }

    public class Debit
    {
        public decimal Value { get; private set; }
        public AccountNumber Account { get; private set; }


        public Debit(decimal value,AccountNumber account)
        {
            Value = value;
            Account = account;
            value.Must(v=>v>=0);//business rule
            account.MustNotBeNull();//ensure VO is valid            
        }
    }
     public class Credit
    {
        public decimal Value { get; private set; }
        public AccountNumber Account { get; private set; }


        public Credit(decimal value,AccountNumber account)
        {
            value.Must(v=>v>=0);//business rule
            account.MustNotBeNull();//ensure VO is valid            
            Value = value;
            Account = account;
        }
    }

```

 Code should be self-explaining, although I bet you're wondering about 2 things:

 1. Why `TransferNumber` gets a Guid?
 2. Why there's no VO implementation for the creation date?

 Well, let me start with no. 2. The creation date doesn't need encapsulation as there are no actual business constraints (in this example). Why complicate the implementation with another class?

`TransferNumber` acts as a natural id for the `Transfer`, however, for technical purposes, I prefer to have a technical id too, hence the Guid. We'll be using it in situations where we need to specify the entity, but we don't need a full Value Object, for example in messages. 

And speaking of messages, here's our domain **change** , expressed as a Domain Event

```csharp
 public class TransferedRegistered
    {
        public Guid EntityId { get; set; }
        public string TransferNumber { get; set; }
        public decimal Amount { get; set; }
        public string DebitAccountNo { get; set; }
        public string CreditAccountNo { get; set; }
        public DateTimeOffset CreatedOn { get; set; }=DateTimeOffset.Now;
    }

```

A nice, flattened data structure, containing _valid_ data. Valid, because it's the Aggregate Root (AR) who's in charge of its generation. Now, I'm going to use a more "exotic" approach for the AR's implementation.

```csharp

 public static class Transfer
    {
        public static TransferedRegistered Create(TransferNumber number, Debit debit, Credit credit)
        {
            number.MustNotBeNull();
            debit.MustNotBeNull();
            credit.MustNotBeNull();

            debit.Account.MustNotBe(credit.Account);

            var ev=new TransferedRegistered();
            ev.EntityId = number.EntityId;
            ev.TransferNumber = number.Value;
            ev.Amount = debit.Value;
            ev.DebitAccountNo = debit.Account.Number;
            ev.CreditAccountNo = credit.Account.Number;
            return ev;
        }
    }

```
Wait... what?! Static class??? Static function???!!! Say what?! Yeah, it's called a functional approach. I prefer a hybrid OOP-FP style to code DDD models, especially ARs. Why? Because it's the simplest implementation. Remember that AR is a _role_ , and the function is the implementation. It makes little sense to make it a full class, when a function is sufficient.

Now, sure, I may have another command case involving `Transfer` but you see, each model is relevant to one case only and I want my code to reflect that. Basically,I want some boundaries between cases, so the class `Transfer` is the concept and each static method is an AR enforcing a specific model. Things being static it means they **shouldn't share** any state. They're grouped together only as a _convenience_.

Our AR makes sure the business rules are respected and then it communicates the change by generating and returning a domain event.

This is not the only way to implement our model, but I like it since it keeps things simple and our code is Event Sourcing friendly. In a future post, when we'll talk about Application Services, we're going to see how the AR is used by the service and what to do with the event.
 

  

