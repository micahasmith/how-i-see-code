# Software Architecture in 20 Minutes

One of the things I typically hear from people trying to switch from lightweight, web-style (think jQuery) programming to server-side, large scale application programming is "I understand the language, I just don't know how to piece it all together." What follows are my rules for "piecing it all together" that I've built from my experience.

Rules should always follow from principles. So principally, what we're looking for is code/applications that lend themselves towards:
- code reuse, NOT code duplication (otherwise known as DRY)
- maintainability
- readability
- test-ability
- grow-ability
- scale-ability

Now that we have our principles/goals, what follows is how I see the rules.

*Disclaimer: I re-purpose generally accepted terms in this doc. For instance, my definition of a "service" doesn't match many others' definitions. This is not meant to confuse the reader.*

## A Program Is Either a "Script" Or It Is A "Domain-Driven" Program

*(Otherwise known as Martin Fowler's distinction between a transaction script and a domain model)*

So while there are units of code that we refer to as **objects** (think classes), a program is either a **group of objects (domain model)** or it is a **script**. 

**Scripts** are single use, one-off programs with either one or very few objects. Scripts can break all the rules of good software design in exchange for speed of delivery. Sometimes you just need to get something done real quick, for something that only needs done now. A script is good for that. Sometimes you need something done that really only takes 10 lines of code to do-- a script is also good for that.

**Domain models** are groups of objects that are built for reuse, maintainability, and all the other principles we outlined above. If you're building something you want to build upon and use for a long time, you should go after a domain model. A program that is based off of a domain model is a domain driven program.

The rest of our discussion will primarily revolve around domain models, and more specifically, objects in domain models.

## An Object Is Either "Stuff" Or It Is "Something That Does Something With Stuff"

*(Otherwise known as the distinction between model and service)*

You should never have an object that is both.

If an object "is the stuff" or "is stuff," then its a **model**.

If an object "does something with stuff," then its a **service**.

Again, an object *should* never be both-- never, ever, ever. You will notice that most programs/frameworks break this rule.

Examples:
- an object that contains a customer's first name, last name, and address are pieces of data -> they are "stuff" -> model
- an object that gets data from a database -> does something with stuff (a database connection in this instance) -> service
- an object that takes a web request and returns a web page -> does something with stuff -> service
- an object that reads a file from a hard drive -> does something with stuff -> service
- an object that contains file data -> stuff -> model
- an object that holds a customer's first name and last name and when either of those changes auto-saves that change to the db -> is the stuff AND does something with the stuff -> RULE BREAKER = WRONG

What should we do if we want to make something that's a rule breaker? Well, we should separate that into models and services until we're no longer breaking rules!

## There Are Types of Models

*(Otherwise known as mutability vs immutability)*

The term "model" is really just indicative of a category of objects-- there are a few different types of models.

That's right! A model can be **mutable**, **immutable**, or **partially immutable**. A mutable model has properties/sub-objects that you can change. An immutable model has properties/sub-objects that you can't change, and is sometimes referred to as a DTO (data transfer object-- you'll find out why later). A partially immutable object is one where some parts of it you can change, but some parts you can't. Pretty simple stuff-- but why/when do you need to use each? Let's explore that via some examples:

Examples:
- an object containing user-entered data (think form data) will need to be changeable (since the user will be changing it), thus it should be **mutable**
- an object containing the result of a calculation will be **immutable** since that result will never change
- an object containing customer data from a db could be **partially immutable**, while its OK to change some parts of the data like the customer's name and such its not OK to change things like IDs/keys-- that could break the referential integrity of the database

The rule of thumb here is that *wherever possible, make things immutable*. In other words, don't allow something to be changeable if you 1) don't need it to be changeable or 2) don't want to deal with the possible side effects from someone changing it. Believe me, you don't want to deal with those side effects, and there could be more than you think. So put your models on "lock down" every chance you can get-- you can always make parts mutable in the future if you notice you need to.

## There Are Types of Services

*(Otherwise known as design patterns, architectural patterns, and other things)*

The term "service" is really just indicative of a category of objects-- there are many, many, many types of services. Let's explore a few via examples:

Examples:
- fetches data from a database -> repository service
- converts one model into another model -> adapter service
- builds an object using business rules -> builder service
- builds an object by supplying its required dependencies -> factory service
- a centralized point to allow publishing and subscribing of event data -> mediator service
- tracks changes to mutable models so that they can be handled atomically -> unit of work service
- evaluates whether or not a model does or does not obey a rule -> predicate service
- evaluates a group of rules/predicates against a model -> validator service 

And there are still many more. I would recommend reading up on design patterns (google it) in order to increase your design pattern vocabulary.

[TODO show some examples of the above mentioned patterns]

## An Object Should Do One Thing

*(Otherwise known as the single responsibility principle)*

And it should do it well. But it should never do 2, 3, or any other number of things-- just one.

What do models do? A model should hold the stuff relating to a single thing/context. What do services do? A service should perform a single type of action on "stuff." Simple!

By the way, if your code for any single object is longer than can fit in your screen, there is a good chance you're doing too many things in it. Not always, but there's a good chance. You should probably break it up into things that only do one thing. 

### That's Outrageous. Couldn't You Break Each Line of Code Down Into an Object?

A great point-- you probably could, or at least get close to it!

So it begs the question, how far is "too" broken down? The answer is simple-- after you're within a single "type" of service or model, don't break down farther than you need to reuse. 

At the same time though, if you see multiple things going on in an object, do break it down.

But DEFINITELY do break down if you will reuse any of the broken down parts. 

### "Do One Thing" Does Not Mean "Have One Member Function"

So you might be thinking,  "if an object only does one thing, that means all objects should have at most one function, right? One function to do that one thing?"

This is hard to put into words exactly.

When I say "an object should do one thing" I really mean "do one type of thing." I could also rephrase that as "do one category of thing" or "do things related to ___ ." It's more contextual than anything else.

A great example of this is the repository service type of object. A repository not only projects a single item from a data store (such as a db) to a model, it also (typically) allows a user to specify a condition (think "where" clause) to match items in the data store to project into models. So one could say that the repository service type of object "does things related to grabbing data from a data store and projecting it to a model."

[TODO Repository Example]

## Objects have constructors. What Is A Constructor For?

In a sense, a constructor is the "beginning" of an object. Therefore, in a sense, it follows that it should be the starting point of our deeper discussion of objects.

All objects have constructors. A **constructor** is a function that sets up an object so that it can perform its (single) duty. The constructor should never "perform the duty" or "start to perform the duty" or anything else-- it should only make the object ready to "perform the duty." The details of the role of a constructor is different for models and services.

### What A Model's Constructor Should Do

A model's constructor could provide the model's data, especially if the model is immutable. So within an immutable model's constructor you will see the model's properties getting the actual data they will hold.

Since the code within a model should only ever have to do with "holding the stuff" you should never see a model's constructor "loading the stuff" or anything else.

### What A Service's Constructor Should Do

A service's constructor should provide it the other services it needs, if any, to perform its single duty. A service's constructor gets passed "objects it needs to perform its duty." It should never, ever get passed "object[s] it will perform a duty on." Once the other services are passed in to the object, it should make them private members of itself so that its functions can have access to them.

This also means that services' constructor or member functions should never instantiate instances of other services-- all interaction with other services should take place using one passed in to the object's constructor.

So again, placing this within our definition of a service (which is "something that does something with stuff") :

- the constructor of a service gets passed objects that enable it to do "something with stuff"
- in contrast to the constructor function, the member function[s] of a service get passed "the stuff" to perform actions on
- and, the function[s] of a service use the services passed in to the constructor when needed-- they don't instantiate them

So in other words, constructors should never, ever get "the stuff." If you're passing a model into a service constructor, there's a good chance you're doing something wrong. 


## Services Should Communicate With Other Services via DTOs (Models)

Whoa that was a big one. 

## Entity is a Facade is a Compound Object is a LIE

## Projection is Everywhere and is Everything (Programs are Layers of Services that Communicate via Models)

## A Function is a Query or a Command

## Stateful vs Not Stateful

## Notice I Never Mention Object Inheritance

## Breaking The Rules-- Patterns for Working With OPC (Other People's Code)

## Project File/Directory Structure




