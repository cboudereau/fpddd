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
and ShoppingCart = ShoppingCart of Product list
and Order = 
  { Products : Product list
    CreditCard : CreditCard }
and CreditCard = CreditCard of string
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

## Part II. Building Blocks of a Model-Drive n Design

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
#### Associations
Associations are function signature like the Buy associate a SchoppingCart and a CreditCard number. The function signature brings also cardinality in the domain.

```fsharp
type Buy = Buy of (ShoppingCart -> CreditCard -> Order)
```

In this version, it is easy to identify cardinality and relationship between objects.

```fsharp
type Buy = Buy of (ShoppingCart -> CreditCard option -> Order option)
```

A higher order function/ function composition can help quickly to transform the first one version to the second one by having for example a full Buy option:

```fsharp
let buy = Buy <| fun (ShoppingCart products) creditCard -> { Products=products; CreditCard=creditCard }

let optionalBuy (Buy buy) creditCard shoppingCart = creditCard |> Option.map (buy shoppingCart)

let products = 
    ShoppingCart <| 
        [ { Price=100m
            Qty = 1
            Reference = Reference "ABC123"} ]

optionalBuy buy None products = None
optionalBuy buy (Some (CreditCard "40000000000000")) products = Some { Products = [{ Reference = Reference "ABC123"; Qty = 1; Price = 100M }]; CreditCard = CreditCard "40000000000000" }
```

In this version, the order is made optional by composing the function with the CreditCard option in the optionalBuy higher order function. You bring that way dependency injection by adding the Buy function as parameter.

Encoding association is made easy in FP.

The main idea is that association are directly infered from the usage of the domain so that it avoid extra association that could come from a discussion of the model with the domain expert. This is a representation of association USED in the software.

#### ENTITIES (AKA REFERENCE OBJECTS)
In FP, Entities are mapped directly to Record and Sum types.
Identity and Equality in FP and DDD are important concept and the heart of the AGGREGATES. Sometimes identity and equality serves the same purpose but sometimes not. In FP referential transparency is a good tool to enhance the identiy and equality problems. 

"An object defined primarily by its identity is called an "ENTITY""

Having an identity or equality helps a lot to have idempotency (enhance software resilience)

```fsharp
type Customer = 
    | Phone of Phone
    | Email of Email
    | PhoneAndEmail of (Phone * Email)
and Phone = Phone of string
and Email = Email of string
```

#### VALUE OBJECTS
Very interesting point there : puting identity everywhere can hurt the system performance (IE : dependency injection framework where everything is an instance of a class). In most cases value objects can be a struct but when struct is a graph, list or a big datastructure, it is difficult to copy it all time. Thanks to record types, struct and ref readonly on dotnet core it is pretty easy to get value objects even if they are bigs.

For example, a banking transaction is an ENTITY but the amouunt in Money type is a value type represented as a record type or struct in FP.

In OOP value objects are mostly represented with primitives or enum which is a bad practice. Instead, a sum type is used in FP to represent the value object of the domain. For example, if the color is static :

Dependending the domain the identity can be relevant or not. For example in geo address identifier is important but not in the case of eCommerce site. For an autocomplete service, it is use full to transform the address from a value object to a selected address entity. That way it is really easy to index data with by address identity.

Value object are immutable which is by default in fsharp but not in csharp.

Another important part for VALUE OBJECTS, they can be used to send a message. This kind of architecture transform a continuous time to a discrete time which is better to have for resilience in long running process.

```fsharp
type Color = Blue | Green
```

But if the color is Dynamic and can be represented is different format :
```fsharp
type Color = 
  | Hexa of int
  | HSL of (uint * uint * uint)
```

##### Special Cases: When to Allow Mutability

This operation is simple as folding a list.

In FP, mutation can be switched easily in a Monad to take the benefits of the performance and boundary (unit of work). This can be a line in a Http request or a char if your are building a json parser. That way when you fold the monoid (a zero and a binary operation) it is easy to keep track of the "pseudo mutation" aka context switching over a simple function.  

For performance, it is nice to use a structure which mutes but can be use only for write or read operation and separating the process of writing and reading. That is the case with read readonly struct in dotnet, buffer management (circular buffer in a proxy, ...)

Actor model helps a lot to mute the state inside an actor instance and send notification when the state changed. If a value mutes then it must not be shared. If the value is shared, deadlock or awaiting process are hard to optimize. A message passing approach can bring real performance and parallelization for free and avoid bottlenecks. 

A good example : channel vs multithreading

#### SERVICES

"Manager" anti-pattern, in FP services can combine services together of simply be a function.
Services are implemented as interface in OOP but applied to SOLID principles, those interfaces can become anemic, this is why in FP the unit of work for a service is a function defined in a type like this one (always a "verb" rather than a "noun") : 

In FP, this definition can be in one file which contains the full domain model of the application separated with modules.

```fsharp
//DDD Services
type TryBook = TryBook of (Room -> Rate -> BookingReference -> Planning -> Planning * Booking option)
type DecrementPlanning = DecrementPlanning of (RoomAvailability -> Rate -> Planning -> Planning option) 
```

Stateless operation : in FP states are managed through unit of work and traversable types likes Monads. That way a type is responsible to manage the state and map a function to transform each values. In/Out monads are very usefull to work with bounded process (like http request/response life cycle, database operations, ...)

Service are partitionated in : Application, Domain, Infrastructure. Thoses layers can be defined as internal inside a module and a service can exposed and hide some specific domain details which is not core domain but important to structure

Application : messaging handling message and notification
Domain : specific types (fine-grained domain objects) for the Service domain (core service functions)
Infrastructure : send emails, edge of the Hexagon (in Haxagonal architecture)

Services map well with the Top-Down Agile patterns

#### MODULES (AKA PACKAGES)
Modules are concepts and used for Low coupling and high cohesion and brind consistency in an atomic concept.
MODULES are a communications mechanism.
In fsharp, module and all kind of declaration is dependent of their order. That mean putting at the end technical modules to not referenced them into domain modules. Only the composition root of the program bind technical/infra with domain

#### Personal Notes
Here is a full version of the ENTITIES, SERVICE and VALUE OBJECT usage
```fsharp
//// Domain Model
//Booking is an Entity because it is important to treat it only once to managmeent hotel availabilities
type Booking = 
    { Reference : BookingReference
      Room : Room
      Rate : Rate }
// Value objects
and Room = Double | Single
and Rate = Rate of decimal
and BookingReference = BookingReference of int
and Availability = Availability of int
//Here RoomAvailability is a Value object because only quantity is important.
and RoomAvailability = 
    { Room : Room
      Quantity : RoomQuantity }
// DDD ASSOCIATIONS
and Planning = Planning of Map<(Room * Rate), RoomQuantity>
//Ubiquituous Language : avoid primitive obsession in favour of domain word.
and RoomQuantity = 
    | RoomQuantity of intùm!
    with 
        //Arithmetic operations inside the domain
        static member (-) (RoomQuantity x, RoomQuantity y) = RoomQuantity (x - y)  
        static member (+) (RoomQuantity x, RoomQuantity y) = RoomQuantity (x + y)  
        static member Zero = RoomQuantity 0
        static member One = RoomQuantity 1

//DDD Services
type TryBook = TryBook of (Room -> Rate -> BookingReference -> Planning -> Planning * Booking option)
type DecrementPlanning = DecrementPlanning of (RoomAvailability -> Rate -> Planning -> Planning option) 

//Implementation : separate Domain model from Implementation
//DDD Package/Module
module Planning = 
    let empty = Planning Map.empty
    let tryDecrement = 
        DecrementPlanning <| fun roomAvailbility rate (Planning planning) -> 
            let identity = roomAvailbility.Room, rate
            planning 
            |> Map.tryFind identity
            |> Option.bind (fun roomQuantity ->
                match roomQuantity - roomAvailbility.Quantity with
                | RoomQuantity 0 -> planning |> Map.remove identity |> Planning |> Some
                | update when update > RoomQuantity.Zero -> planning |> Map.add identity update |> Planning |> Some
                | _ -> None
                | roomQuantity when roomQuantity - roomAvailbility.Quantity >= RoomQuantity.Zero -> None)

// DDD Package/Module
module Booking = 
    //Function to compose Services together with higer order function composition
    let tryBook (DecrementPlanning planningTransaction) = 
        TryBook <| fun bookingRoom bookingRate bookingReference planning ->
            let bookingRoomAvail = { Room=bookingRoom; Quantity = RoomQuantity.One}
            let booking  = { Reference=bookingReference; Room=bookingRoom; Rate=bookingRate }
            planning
            |> planningTransaction bookingRoomAvail bookingRate
            |> Option.map (fun updatedPlanning -> updatedPlanning, Some booking)
            |> Option.defaultValue (planning, None) 

//Composition root (root of the app)
let (TryBook tryBookWithDependencies) = Booking.tryBook Planning.tryDecrement

// Sandbox and Assertions
let planning = [ (Room.Double, Rate 100m), RoomQuantity 1 ] |> Map.ofList |> Planning
//Bad price
planning |> tryBookWithDependencies Room.Double (Rate 120m) (BookingReference 123) = (planning, None)
//Bad room
planning |> tryBookWithDependencies Room.Single (Rate 100m) (BookingReference 123) = (planning, None)
//Sounds good!
planning |> tryBookWithDependencies Room.Double (Rate 100m) (BookingReference 123) = (Planning.empty, Some { Reference = BookingReference 123; Room = Double; Rate = Rate 100M })
```

### Modeling Paradigms
Here, it is well explained that the choice of OOP is purely related to old topics like performance, memory optimizations which are not true at the time of read. FP compilers/ gc like fsharp and Rust are better than the better OOP one. 
For FP go for Domain Modelling Made Functional

#### Non-objects in an Object World 
Eric Evans sites Prolog which is a Logic programming language. Functional programming and Logic programming are closed to formal/ mathematic legacy. What about OOP ? It is purely created as part as an old state mangement optimization which does not really make senses in DDD.

### The Lifecycle of a Domain Object

AGGREGATES, FACTORIES, REPOSITORIES

"..., AGGREGATES tighten up the model itself by defining clear ownership and boundaries, avoiding a chaotic tangled web of objects. This is crucial to maintaining integrity in all phases of the lifecycle. 
Then, we focus on the beginning of the lifecycle, using FACTORIES to create and reconstitute complex objects and AGGREGATES, keeping their internal structure encapsulated. Finally, REPOSITORIES address the middle and end of the lifecycle, providing the means of finding and retrieving persistent objects while encapsulating the immense infrastructure involved. 
Although REPOSITORIES and FACTORIES do not themselves come from the domain, they have meaningful roles in the domain design. These constructs complete the MODEL-DRIVEN DESIGN by giving us accessible handles on the model objects. 
Modeling AGGREGATES and adding FACTORIES and REPOSITORIES to the design gives us the ability to manipulate the model objects systematically and in meaningful units throughout their lifecycle. AGGREGATES mark off the scope within which invariants have to be maintained at every stage of the lifecycle. FACTORIES and REPOSITORIES, operate on AGGREGATES, encapsulating the complexity of specific lifecycle transitions."

In finance, a closed is by the domain when the market is closed, in hospitality industry a booking ends when the client come at hotel, .... This kind of expiration are very important in DDD

