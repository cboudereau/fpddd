# fpddd

## Part I. Putting the Domain Model to Work

### Communication and the Use of Language
#### UBIQUITOUS LANGUAGE
Ubiquitus Language mean at the same time one world with an associated définition to avoid synonym or packing all specifities in the same word. 

Your language can bring a lot of technical details like technical keyword and so on. It is very important at the design time to choose a language with the less keyword. This is call Consiveness of the language. With XP and a consive lang, the communication through the code is very good. 

That way it is possible to create a discussion between the domain expert and the dev team with a code file as media by challenging functions and vocabulary to precise and add high domain fidelity.

The best of optimization (in term of performance) is not the technical one but always the one which starts from a consensus betwaeen dev and domain expert. Ex : temporal, domain compression by applying values on the same interval is really better than the naive solution consisting to send request as day basis.

The model is very important, that is why in FP, we have a file which contains the full domain of the app. That way, it is simpler to discover/navigate and discuss.

```fsharp
type Product = 
  { Reference : Reference
    Qty : int
    Price : decimal }
and ShoppingCart = 
  { Products : Product list }
and Order = 
  { Products : Product list
    CreditCard : CreditCart }
and CreditCart = CreditCard of string
and Reference = Reference of string
```

#### Documents and Diagrams

Sketing object can be made very easily with function signature aka Type Driven Development.
By defining arrows and transformation to link one object of the domain to another one, the use of function feat really well with that practice

fsi files in fsharp is a good place to do this activity.

```fsharp
type Buy = ShoppingCart -> CreditCard -> Order
```

#### Binding Model and Implementation
##### MODEL-DRIVEN DESIGN
FP is better than OOP on that field which not hide state inside object. The state is represented outside functions. That way functions becomes simple transition from one state to another one and follow the same rules of the Documents and Diagrams part.

The most important idea is having an explicit domain in the code helps developer and new comers understanding as quick as possible and with the maximum domain fidelity.

Typing all the things help to separate the implementation from the design. Reviewing/challenging it thanks to the compiler helps a lot to see problems and take a decision quickly on the domain.

## Part II. Building Bloc ks of a Model-Drive n Design

Package -> Module
Services -> Functions
Entities -> Record type
Value objects -> Type (with deep equality by default in fsharp and for free)

Aggregates : an aggregate is helpfull in a distributed system AKA CQS/CQRS when SPOF are centralized system and consensus accross different state with eventual consistency / CRDT / Compensation/ Consensus. In FP, an aggregates is basically the result of a fold function.

### Isolating the Domain
#### LAYERED ARCHITECTURE
Decoupling domain objects and other function is called function composition in FP. The aim is to split the function in smaller one and construct the result by composing them with higher order functions (combinators, monads, ...). This is a more intuitive usage of Onion or Hexagonal Architecture

#### The Domain Layer is Where the Model Lives
The layered architecture separates in fact model representation from model domain (In the Weather app, the UI is culture/UI Framework dependent and adptation has to be done to convert unit of measure) : https://github.com/cboudereau/fabulous-weather.

By adding conversion modules and model to views functions, the domain model is projected to the culture and interface of the user.

#### SMART UI ANTI-PATTERN
Interesting Point : Becarefull with SMART UI and frameworks that try to bring at the same time technical abstraction with inheritance and an existing layer. This is considered an anti pattern because the domain is dependent of the technical details. In FP, function does not impact dependencies, functions are just arrows to transform domain object to framework UI/Api objects. That way the domain is not dependent of the ui/web framework.

In SMART UI, new comers must start by understanding the technical framework BEFORE understanding the domain. This is why it is considered as an antipattern from the DDD point of view.

### A Model Expressed in Software




