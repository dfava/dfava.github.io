---
layout: post
title:  "Go, Scala and case classes"
date:   2023-03-14 13:00:00 +0100
categories: programming-languages
author: Daniel Fava, co-authored by ChatGPT
---
In this blog post, we will take a look at Go and Scala, and specifically, at their approach to case classes.

One of Scala's key features is its support for case classes. Case classes are meant for holding immutable data. They are similar to regular classes, but they come with a number of useful features out-of-the-box, such as the ability to generate a toString method, a copy method; they also come with matching support.

Go, on the other hand, does not have built-in support for case classes.  We will look at three different approaches that can be used instead.

<!--more-->

Before we dive into Go, here's an example of two case classes, `Dollar` and `Euro`, that extend a common `Currency` class in Scala:

```scala
abstract class Currency(val name: String, val alpha: String, val symbol: String)

case class Dollar() extends Currency("US Dollar", "USD, "$")
case class Euro() extends Currency("Euro", "EUR", "€")

val dollar = Dollar()
val euro = Euro()

println(dollar.name)   // Output: US Dollar
println(dollar.alpha)  // Output: USD
println(dollar.symbol) // Output: $
println(euro.name)     // Output: Euro
println(euro.alpha)    // Output: EUR
println(euro.symbol)   // Output: €
```

Next, we'll try three different approaches to model case classes in Go:

1. structs
2. enums
3. interface and "empty" types definitions


## Structs

We use structs to hold data.  Going back to the currency example, we can define a `Currency` as such:

```go
package main

import (
	"fmt"
)

type Currency struct {
	Name   string
	Alpha  string
	Symbol string
}

type dollar struct {
	Currency
}

type euro struct {
	Currency
}

var (
	Dollar = dollar{Currency{"US Dollar", "USD", "$"}}
	Euro   = euro{Currency{"Euro", "EUR", "€"}}
)

func main() {
	fmt.Println(Dollar.Name)   // Output: US Dollar
	fmt.Println(Dollar.Alpha)  // Output: USD
	fmt.Println(Dollar.Symbol) // Output: $
	fmt.Println(Euro.Name)     // Output: Euro
	fmt.Println(Euro.Alpha)    // Output: EUR
	fmt.Println(Euro.Symbol)   // Output: €
}
```

## Enums

Enums, short for enumerations, are a type in programming that allows you to define a set of named values. They are used to represent a fixed set of possible values for a variable, parameter, or property.

Here is an alternative implementation to our "currency" example.  In this implementation, we are removing information from the structs and putting it in methods.  If you squint, you can see the interplay between [data and code]():  Functions can be implemented as table look-ups, where the function's behavior is precomputed and stored in a data structure, rather than being computed on the fly.  Conversely, data can sometimes be computed or generated on-the-fly by code, rather than being stored.

The struct here would only have to hold one integer for each currency... so there is no point of a struct at all!  Instead of a struct holding an integer, we just have the integer.  And we have one integer value for each currency.  In other words, the struct collapses into an enum.

```go
package main

import (
    "fmt"
)

type Currency int

const (
    Dollar Currency = iota
    Euro
)

func (c Currency) String() string {
    switch c {
    case Dollar:
        return "US Dollar"
    case Euro:
        return "Euro"
    default:
        return "Unknown"
    }
}

func (c Currency) Alpha() string {
    switch c {
    case Dollar:
        return "USD"
    case Euro:
        return "EUR"
    default:
        return "Unknown"
    }
}

func (c Currency) Symbol() string {
    switch c {
    case Dollar:
        return "$"
    case Euro:
        return "€"
    default:
        return "?"
    }
}

func main() {
    dollar := Dollar
    euro := Euro

    fmt.Println(dollar.String()) // Output: US Dollar
    fmt.Println(Symbol(dollar))  // Output: $
    fmt.Println(euro.String())   // Output: Euro
    fmt.Println(Symbol(euro))    // Output: €
}
```

## Empty struct

We can push this interplay between data and code further.  We don't need an integer for each currency, we can have that information in the type itself.  With this approach, the struct is empty.

We define the `Currency` type as an interface, and each currency then implements this interface accordingly:

```go
package currency

type Currency interface {
	String() string
	Alpha() string
	Symbol() string
	IsValid() bool
}

type dollar struct{}
type euro struct{}
type empty struct{}

var (
	Dollar = dollar{}
	Euro   = euro{}
	Empty  = empty{}
)

// Dollars
func (dollar) String() string {
	return "US Dollar"
}

func (dollar) Alpha() string {
	return "USD"
}

func (dollar) Symbol() string {
	return "$"
}

func (dollar) Valid() bool {
	return true
}

// Euros
func (euro) String() string {
	return "Euro"
}

func (euro) Alpha() string {
	return "EUR"
}

func (euro) Symbol() string {
	return "€"
}

func (euro) Valid() bool {
	return true
}

// A zero value for the class
func (empty) String() string {
	panic("attempting to access an invalid currency")
}

func (empty) Alpha() string {
	panic("attempting to access an invalid currency")
}

func (empty) Symbol() string {
	panic("attempting to access an invalid currency")
}

func (empty) Valid() bool {
	return false
}
```


## Comparison

The three approaches above have their pros and cons.

1. struct
  - *pros:* the variable `structs.currency.Dollar` cannot be redefined; the variable is of type `dollar` and these types are unexported.
  - *cons:* we expose the internal structure to library clients

2. enums
  - *pros:* currencies like `enum.currency.Dollar` are defined as `const`, so they cannot be re-assigned
  - *cons:* we need to implement accessor methods (those methods don't come for free)

3. empty struct; information in the type
  - *pros:* the variable `emptystruct.currency.Dollar` cannot be redefined; the variable is of type `dollar` and these types are unexported.  We have well defined interfaces.
  - *cons:* we need to implement accessor methods (those methods don't come for free in Go, like they do in Scala)

To me, option (3) is what comes the closest to case classes.  Unfortunately, options (3) is not very idiomatic Go.  That option also relies more heavily on the capabilities of the type system.  And that's where things start to go wrong for Go.  To illustrate, let's look at marshaling.


## Marshaling and Unmarshaling

Marshaling is the process of converting an object or data structure from its in-memory representation to a format that can be stored or transmitted. In other words, marshaling takes an object in a program's memory and converts it to a format that can be written to disk, sent over a network, or otherwise persisted.

It is trivial to marshal and unmarshal structs as in example (1) and enums, like in example (2).  When marshaling a struct, you do structural decomposition of the struct's elements until you get to elementary data types.  Thankfully, the `json` package does that for us, so we don't even think about this decomposition.

Marshaling for (3) is also easy:

```go
jsonData, err := json.Marshal(currency.Dollar)
if err != nil {
    println(err.Error())
}
```

How about unmarshal?

```go
var d currency.Currency
err = json.Unmarshal(jsonData, &d)
if err != nil {
    fmt.Println("Error unmarshaling JSON:", err)
}
```

The code above gives a runtime error: `Error unmarshaling JSON: json: cannot unmarshal object into Go value of type currency.Currency`.  If we trace through the json package, we see the following stack trace:

```
json.Unmarshal
  json.(*decodeState).unmarshal
    json.(*decodeState).value
      json.(*decodeState).object
```

The `object()` function is trying to figure out what we are unmarshaling into.  It can unmarshal to the empty interface:

```go
if v.Kind() == reflect.Interface && v.NumMethod() == 0 {
    oi := d.objectInterface()
    v.Set(reflect.ValueOf(oi))
    return nil
}
```

Otherwise, the `object()` function checks if the kind of the target is a map or a struct.  If it is neither, it returns the error we've seen:

```go
default:
    d.saveError(&UnmarshalTypeError{Value: "object", Type: t, Offset: int64(d.off)})
    d.skip()
    return nil
```

Maybe you are thinking... "wait a minute, how can the unmarshaler be able to tell what object it's dealing with?"  Let's make the example simpler.  Say we write a marshaler that marshals our currency into strings like "Dollar" and "Euro".  Then what we want the unmarshaler to do is simple:

1. Parse the JSON, unmarshal it as a string, then check:
2. If the string is "Euro", return `euro{}`.  If the string is "Dollar" return `dollar{}`.  Otherwise, return `empty{}`.

Unfortunately, if we try to force the default unmarshaler down this path, we again get a `json: cannot unmarshal string into Go value of type currency.Currency` error.  The call stack is a bit different:

```
json.Unmarshal
  json.(*decodeState).unmarshal
    json.(*decodeState).value
      json.(*decodeState).object
        json.(*decodeState).literalStore
```

We to get a bit deeper into the decoder.  The error message is now coming from `literalStore()`.  At this point, the unmarshaler has determined that it's unmarshaling a string and it is trying to put it into an interface.  Inside `literalStore()` we see this:

```go
case reflect.Interface:
    if v.NumMethod() == 0 {
        v.Set(reflect.ValueOf(string(s)))
    } else {
        d.saveError(&UnmarshalTypeError{Value: "string", Type: v.Type(), Offset: int64(d.readIndex())})
    }
```

Again, when trying to unmarshal into an interface, the unmarshaler checks the number of methods declared by the interface.  If there is one or more methods declared, the unmarshaler errors out.  So it's possible to unmarshal to an empty interface, but we can't unmarshal to `Currency` because `Currency` declares several methods.

We want to somehow tell the unmarshaler to use custom logic when dealing with the `Currency` interface.  We could then implement this logic in the currency package alongside the interface.  The algorithm would work for all known currencies (USD, EUR, etc).  By "known" I mean currencies known at compile time.  The method would check if the string is `Dollar` and create the dollar type, similar for `Euro` and etc.  We want something like this:

```go
func UnmarshalJSON(data []byte) (Currency, error) {
    var s string
    if err := json.Unmarshal(data, &s); err != nil {
        return Empty, err
    }
    switch (s) {
      case "USD":
        return Dollar, nil
      case "EUR":
        return Euro, nil
      default:
        return Empty, "Invalid currency"
    }
}
```

Even if this were possible in Go and the json package, what happens if someone creates a new currency by implementing the interface methods?


### Sealed interfaces

Say we managed to ship the currency package out.  Then someone comes along and implements another currency; the Brazilian Real, for example. What should our custom unmarshaler do when it encounters this new currency?  Since the unmarshaler was implemented before this new currency, it will not recognize that currency and will return the empty one instead along with an error.  What can the currency package managers do about this?  

Scala faces a similar issue, and it offers a couple of solutions:

- we can prevent classes from being extended by declaring them as `final`, or
- we can define a `sealed` trait. A `sealed` trait can only be extended in the same source file it is declared.

In Go, we can get a similar effect as sealed traits by adding an unexported function signature to the interface.  For example:

```go
type Currency interface {
	Name() string
	Alpha() string
	Symbol() string
    IsValid() bool
    sealed()
}
```

Because of `sealed()`, only the currency package will be able to implement currencies.  (Note that there is nothing special about the name "`sealed`".  We could have used any lowercase function name.) 

I learned about sealed interfaces in Go from [Chewxy](https://blog.chewxy.com/2018/03/18/golang-interfaces/#sealed-interfaces).  And by using sealed interfaces, [BurntSushi](https://github.com/BurntSushi/go-sumtype) has built a tool for checking exhaustive patter matching in Go.  Neat :-)

Sealed interfaces solves one of our problems: that of safely extending currencies inside the `currency` package.  But we are still left with one problem: _how to plumb the implementation of our unmarshaling function into the `encoding/json` package._  Concretely, we want to use `func UnmarshalJSON(data []byte) (Currency, error)` inside `literalStore()` in the `encoding/json` package.

The problem boils down to associating a "static" function to an interface.  Looking at the method signature for our unmarshaler; it doesn't have a receiving object:

```go
func UnmarshalJSON(data []byte) (Currency, error) {
```

The custom unmarshal logic doesn't operate on an instance; it "operates on the class."  For better or worse, there is no way in Go to express a "static" function of an interface. At least not as far as I know.

## Conclusion

There are may ways to implement something like case classes in Go.  None of them seem to do justice, in my opinion.  As a Go programmer, I miss algebraic data types.  If we had them, error handling in Go would look nicer.  But I am getting ahead of myself!  Before you think that I'm advocating for changes to Go or the json package, you should watch [this talk by Rob Pike](https://www.youtube.com/watch?v=rFejpH_tAHM&t=3s).  Languages are different.  I too get annoyed with the lack of this or that.  But I'm also glad that Go isn't like Scala!  Scala is great in many ways, I am glad it exists, and I love functional programming in general.  But Go's goals are different from Scala's, and I welcome their difference.
