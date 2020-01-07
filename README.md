# fpddd

## Part I. Putting the Domain Model to Work

### Communication and the Use of Language
#### UBIQUITOUS LANGUAGE
Ubiquitus Language means at the same time one world with an associated définition to avoid synonym or packing all specifities in the same word. 

Your language can bring a lot of technical details like technical keyword and so on. It is very important at the design time to choose a language with the less keyword. This is call Consiveness of the language. With XP and a consive lang, the communication through the code is very good. 

That way it is possible to create a discussion between the domain expert and the dev team with a code file as media by challenging functions and vocabulary to precise and add high domain fidelity.

The best of optimization (in term of performance) is not the technical one but always the one which starts from a consensus between dev and domain expert. Ex : temporal, domain compression by applying values on the same interval is really better than the naive solution consisting to send request as day basis.

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

Typing all the things helps to separate the implementation from the design. Reviewing/challenging it thanks to the compiler helps a lot to see problems and take a decision quickly on the domain.

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
The layered architecture separates in fact model representation from model domain (In the Weather app, the UI is culture/UI Framework dependent and adaptation has to be done to convert unit of measure) : https://github.com/cboudereau/fabulous-weather.

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

```fsharp
type Color = Blue | Green
```

Dependending the domain the identity can be relevant or not. For example in geo address identifier is important but not in the case of eCommerce site. For an autocomplete service, it is use full to transform the address from a value object to a selected address entity. That way it is really easy to index data with by address identity.

Value object are immutable which is by default in fsharp but not in csharp.

Another important part for VALUE OBJECTS, they can be used to send a message. This kind of architecture transform a continuous time to a discrete time which is better to have for resilience in long running process.

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

"It is difficult to guarantee the consistency of changes to objects in a model with complex associations. Invariants need to be maintained that apply to closely related groups of objects, not just discrete objects. Yet cautious locking schemes cause multiple users to interfere pointlessly with each other and make a system unusable."

Invariant are really important, thanks to property based testing, invariant/properties can be found quickly if needed. Thanks to pattern matching, finding invariant in depth is really helpull. Pattern matching is really mature in fsharp rather than in csharp.

How to track and manage lifecycle objects dependencies ? Model consistency and well known associtions live togethers and die togethers.

AGGREGATE Root : a seed/zero on the fold function. In accounting, it is the report of the previous closing.
AGGREGATE Boundary : the end of the accounting year, the end of day in finance for trading or expiry for deals, when the client comes in hotel for bookings, ... The boundray can be small (hour/day) or large (years). Larger one can impact performance, this is why CQS/CQRS help a lot to treat those case. FP fit well on aggregate implementation with Monoid where : mappend is a binary operation to combine seed/zero/mempty and the ENTITY to another VALUE OBJECT or ENTITY.


#### AGGREGATES

##### Model a Purchase Order System
Here is the fsharp implementation in DDD way : 
```fsharp
//Type is the Aggregate root
type PurchaseOrder = 
    { PurchaseOrderNumber:PurchaseOrderNumber
      ApprovedLimit:ApprovedLimit
      PurchaseOrderLineItems:PurchaseOrderLineItem list }
//and are underlying ENTITIES or VALUE OBJECTS
and PurchaseOrderNumber = PurchaseOrderNumber of int
and PurchaseOrderLineItem = { Quantity:int; Part:Part }
and Part = Part of Money
and ApprovedLimit = ApprovedLimit of Money
     
and Money = 
    | Money of decimal
    //Arithmetic operations inside the domain
    static member (+) (Money x, Money y) = Money (x + y)
    static member Zero = Money 0m
    static member (*) (Money x, qty:int) = Money (decimal qty * x)

//DDD Service
type AddItem = AddItem of (PurchaseOrderLineItem -> PurchaseOrder -> PurchaseOrder option)

//Implementation
module PurchaseOrder = 
    let zero number approvedLimit = { PurchaseOrderNumber=number; ApprovedLimit=approvedLimit; PurchaseOrderLineItems=[] } 
    
    module ApprovedLimit = 
        let toMoney (ApprovedLimit money) = money

    module PurchaseOrderLineItem = 
        let amount p = let (Part money) = p.Part in money * p.Quantity

    let tryAdd = 
        AddItem <| fun item order ->
            let items = item :: order.PurchaseOrderLineItems
            let total =  items |> List.sumBy PurchaseOrderLineItem.amount
            
            match order.ApprovedLimit with
            | (ApprovedLimit limit) when total > limit -> None
            | _ -> { order with PurchaseOrderLineItems = items } |> Some

//Composition root
let (AddItem addItem) = PurchaseOrder.tryAdd

//Sandbox
let order = PurchaseOrder.zero (PurchaseOrderNumber 123) (ApprovedLimit (Money 100m))
let part = { Quantity=1; Part=Part (Money 100m) }

//Assertions
order |> addItem part = Some ({order with PurchaseOrderLineItems=[part]})
order |> addItem { part with Quantity=2 } = None
```

Locking an object is now an antipattern causing deadlock and performance problems. Turn the lock into an intention : a command or a notification and handle the message to manage the aggregate by queuing messages without using lock systems. If the message is splitable, separates aggregates like the booking count and room stock for a hotel booking engine app.

Invariant can be encoded to function returning option or result value which indicates the reason.

```fsharp
//Domain
type PurchaseOrder = { OrderNumber : OrderNumber; PurchaseOrderLineItems : PurchaseOrderLineItem list; ApprovedLimit : ApprovedLimit }
and PurchaseOrderLineItem = { LineItem : LineItem; Quantity : Quantity; Part : Part }
and Part = { Price : Money; Name : PartName }
and Quantity = Quantity of int
and Money = | Money of decimal
            static member Zero = Money 0m
            static member (*) (Money x, Quantity y) = Money (decimal y * x)
            static member (+) (Money x, Money y) = Money (x + y)
and PartName = PartName of string
and OrderNumber = OrderNumber of int
and ApprovedLimit = ApprovedLimit of Money
and LineItem = | LineItem of int
               static member (+) (LineItem x, LineItem y) = LineItem (x + y)
module LineItem = let next = List.fold (+) (LineItem 1)
module PurchaseOrderLineItem = 
    let total x = x.Part.Price * x.Quantity
    let create quantity part lineItem = { LineItem = lineItem; Quantity = quantity; Part = part }
// DD Services with stateless purchaseorder (context is always the last parameters : PurchaseOrder)
type AddItem = Part -> Quantity -> PurchaseOrder -> (LineItem * PurchaseOrder) option
type DeleteItem = LineItem -> PurchaseOrder -> PurchaseOrder option
type UpdateItem = Quantity -> LineItem -> PurchaseOrder -> PurchaseOrder option
type PurchaseOrderCommand = AddItem of (Part * Quantity) | DeleteItem of LineItem | UpdateItem of (Quantity * LineItem)
type PurchaseOrderTransaction = PurchaseOrderCommand -> (LineItem * PurchaseOrder) option
//Implementation
let addItem : AddItem = fun part quantity order ->
    let newLineItem = order.PurchaseOrderLineItems |> List.map (fun x -> x.LineItem) |> LineItem.next
    let items = PurchaseOrderLineItem.create quantity part newLineItem :: order.PurchaseOrderLineItems
    let total = items |> List.sumBy PurchaseOrderLineItem.total
    match order.ApprovedLimit with
    | ApprovedLimit limit when limit > total -> None
    | _ -> Some (newLineItem, { order with PurchaseOrderLineItems = items })
let deleteItem : DeleteItem = fun lineItem order ->
    let (found, items) = List.foldBack (fun x (found, l) -> if x.LineItem = lineItem then true, l else found, x :: l) order.PurchaseOrderLineItems (false, [])
    if found then Some { order with PurchaseOrderLineItems = items }
    else None
let updateItem : UpdateItem = fun quantity lineItem order ->
    let (found, items) = List.foldBack (fun x (found, l) -> if x.LineItem = lineItem then true, { x with Quantity = quantity } :: l else found, x :: l) order.PurchaseOrderLineItems (false, [])
    if found then Some { order with PurchaseOrderLineItems = items }
    else None
//Full DD Service implementation which take a command and update order if possible
let update order = 
    function
    | AddItem (part, quantity) -> addItem part quantity order 
    | DeleteItem lineItem -> deleteItem lineItem order |> Option.map (fun x -> lineItem, x)
    | UpdateItem (quantity, lineItem) -> updateItem quantity lineItem order |> Option.map (fun x -> lineItem, x)
// Statefull lockfree actor: updating the state by queuing messages to avoid lock on concurrent commands
let statefullUpdate initialOrder : PurchaseOrderTransaction = 
    let actor order = MailboxProcessor.Start <| fun channel ->
        let rec read order = 
            async {
                let! (reply, command) =  channel.Receive()
                let state = update order command
                reply state
                return! state |> Option.map snd |> Option.defaultValue order |> read
            }
        read order
    let queue = actor initialOrder
    fun m -> queue.PostAndReply(fun channel -> channel.Reply, m)
//Sandbox
let guitars = { Name=PartName "Guitars"; Price=Money 100m }
let trombones = { Name=PartName "Trombones"; Price=Money 200m }
let initialOrder = { OrderNumber = OrderNumber 12946; ApprovedLimit = ApprovedLimit (Money 1000m); PurchaseOrderLineItems = [ { LineItem=LineItem 1; Quantity=Quantity 3; Part=guitars }; { LineItem=LineItem 2; Quantity=Quantity 2; Part=trombones }] }

let purchaseOrderTransaction = statefullUpdate initialOrder

[ DeleteItem (LineItem 1)
  DeleteItem (LineItem 10) 
  DeleteItem (LineItem 2)
  AddItem ( { Name=PartName "Piano"; Price=Money 1000m }, Quantity 1 ) ]
|> List.map (fun x -> async { return purchaseOrderTransaction x })
|> Async.Parallel
|> Async.RunSynchronously
```

#### FACTORIES

Bring invariant into a struct/object creation. In FP this is called making illegal states unrepresentable. Builder in OOP is really hard to implement as the number of invariant increase. Instead in FP, the railway oriented programming helps a lot to implement factory without builder problems.

Operations should be atomic even if it is eventual consistent

This part does not explain how to implement invariant checks in constructor or factories. In constructor, when invariant is not meet, an exception is used. Exception is a lie on a method signature and it is harder to catch deep one (ie: Java exception runtime handling...). Another way is to use sum type like option or result one in a function as a factory. In OOP, builders are used for that but there the code can be messy as the aggregate structure is complex.

#### REPOSITORIES
Document store instead of full database model, this is the purpose of CQS, put a document for read operation updated by applying updates by queuing updates in an eventual consistency way. Write operations are centralized in a DB for example. Read store can be build from scratch from the write model. 2 aggregates one for read operation and another for write operations. The read aggregates can be rebuild from the write one but not the opposite.

#### Designing Objects for Relational Databases
METADATA MAPPING LAYERS : optics/lens is FP is the pattern for that.

In that parts, ORM are not the one to treat all kind of problems like aggregation. The best way is to use simple tools like type providers whithout the need of a global technical solution for all repositories. Performance and Space are a good reasons to take care of that kind of problems. Why using full structures if only the count is used ? This is is kind of question that the dev should focus first on the domain before the solution.

In that part, caching technique is discussed, CQS is a kind of caching by taking care of the events to update the read model (cache)

###  Using the Language in an Example: A Cargo Shipping System

Organize code by domain/boundary not by pattern like MVC...

## Part III. Refactoring Toward Deeper Insight

"To create a design really fitted to the problem at hand, you must first have a model that captures the central relevant concepts of the domain" : Type Driven Development intent is to create first types and interactions/transformations between types like DDD does.

#### A Decent Model, and Yet… 
Instead of using inheritance which is hard to maintain and refactor, function composition is used to combine function together. That way the composition offers the same behavior like inheritance by composing function with much more simplicity and no refactoring problems. This is why it is important to type functions.

```fsharp
type Loan = Loan
type LoanInvestment = LoanInvestment
//Here a function is used to explain the relationship between Loan and LoanInvestment which is lost when using inheritance..
type Contribution = Loan -> LoanInvestment
```

### Making Implicit Concepts Explicit

Continuous time to discrete time : the batch process can be translated to a scheduled event

SPECIFICATION : avoid uses of boolean to represent cases, in FP a dedicated types exists : sum types

"Developers working in the logic-programming paradigm would handle this differently. Such rules would be expressed as "predicates". Predicates are functions that evaluate to “true” or “false” and can be combined using operators like "and" and "or" to express more complex rules. With predicates, we could declare rules explicitly and use them with the Invoice. If only we were in the logic paradigm. " In FP, function combinators are used to at the same time branch and compute result. This way the cyclomatic complexity is always 1 because the branching operations are encoded inside combinators operators which are basically functions.

When SPECIFICATION impacts performance, type provider helps a lots because the generated code is not SQL one (fsharp -> SQL) this is the opposite from SQL -> fsharp. That way it is easy to implement rules directly in sql server side. This is the difference between a conventional ORM that transform objects to relational database. Type provider + ANTICORRUPTION LAYER are BUILDING BLOCKS to bring SPECIFICATIONS at the right technical level (sometimes hard to implement with hexagonal architectures)


