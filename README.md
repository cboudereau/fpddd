# fpddd

## Putting the Domain Model to Work

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

