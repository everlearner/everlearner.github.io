---
title: "Favourite Links"
date: 2022-05-01T12:49:31+05:30
tags: [Links, websites, best-websites]
---
A collection of `some very neatly` designed websites, can help in daily use. Updated Regularly.

### Design and Images
- [Profile Pic Maker](https://pfpmaker.com)
- [Spline](https://spline.design/)
- [Blush](https://blush.design)

### PDFs and documents
- [Sejda](https://sejda.com)
- [TinyWow](https://tinywow.com/)
- [Wordtune](https://www.wordtune.com/)

### Misc
- [Slap Chris](https://slapchris.com)
- [Price History](https://pricehistoryapp.com/)
- [Scribe How](https://www.scribehow.com)
- [Versus util](https://versusutil.com/)

<!-- We found a couple of recovered panics in the logs. These panics were happening pretty frequently. This is what the panic message said:

> panic: interface conversion: interface {} is `nil`, not []\*typeName

From the panic message, it is pretty obvious that somewhere in the code we are trying to convert an `interface{}` to a specific array type when the value is actually `nil`. And the panic message also has a trace to the exact line where this conversion is taking place. This is a very common error and is easy to fix. But while debugging this, I encountered a smell in the code that led to this in the first place. This is what the code kind of looked like:

There is an interface with a single method which returns `interface{}`
```go
type I interface {
  fun() interface{}
}
```

There are two structs (`impl1` and `impl2`) that implement the above interface

```go
// impl1 implements I
type impl1 struct{}

func (i impl1) fun() interface{} {
  return nil
}
```

```go
// impl2 implements I
type impl2 struct{}

func (i impl2) fun() interface{} {
  return i.helper()
}

func (i impl2) helper() []*int {
  return nil
}
```

What do you think will be the output of the below code?

```go
func main() {
  var A, B I

  A = impl1{}
  B = impl2{}

  fmt.Println(A.fun().([]*int)) // statement 1
  fmt.Println(B.fun().([]*int)) // statement 2
}
```

As you can probably guess, `statement 1` panics and `statement 2` never gets executed. But what if `statement 1` wasn't there. What do you think would happen then?

```go
func main() {
  var B I

  B = impl2{}

  fmt.Println(B.fun().([]*int))
}
```

Would this code panic?

It wouldn't. This code actually prints `[]` (empty slice). This can be made more clear by printing the `type` of the return values.

```go
func main() {
  var A, B I

  A = impl1{}
  B = impl2{}

  fmt.Println(reflect.TypeOf(A.fun())) // prints "<nil>"
  fmt.Println(reflect.TypeOf(B.fun())) // prints "[]*int"
}
```

This is because when you assign `nil` to a slice in go, it releases the underlying array to the garbage collector and resets the slice to its "zero value". Empty slices are treated the same as `nil`.

```go
var arr []int
arr == nil // evaluates to true
```

Since the `helper()` method of the struct `impl2` has the return type as `[]*int`, the `nil` returned aquires the type `[]*int` with "zero value"

### The Fix

There are a couple of obvious fixes for this. The quick way is, The `fun()` method of `impl1` should not return `nil`. Instead, it should return an empty slice.

```go
type impl1 struct{}

func (i impl1) fun() interface{} {
  var arr []*int
  return arr
}
```

Alternatively, you could also check if the value is nil before trying to typecast it. 

```go
func main() {
  var A I

  A = impl1{}

  result := A.fun()
  if result == nil {
    // return an error or handle it any other way
  }

  fmt.Println(result.([]*int))
}
```

The **right way**, is to consider re-writing this part of the code. The interface should be given another thought.

### Lessons learnt

It is very convenient to use `interface{}` where we want to **allow any type**. But it can cause a lot of unpredictable behavior. The empty interface is not a feature, it's a side effect of another feature which is the *implicit implementation of interfaces*. Since `interface{}` has no methods in it, every object in golang can conform to it.

Sure, it can be useful when you want to **accept any type** to be passed as an argument to a function (like in `fmt.Println` and many others). It works out in this case because the treatment of the value passed is in the hands of the author of the function. If you are writing such a function, you would know exactly what kind of types to expect, and how to handle unexpected types. But when the function returns an empty `interface{}`, the user of the function has no idea of all the possible types that can come out. They will only know what type they expect and only handle that (leading to situations like the above). 

The **need** to even return an empty interface exposes a design flaw in the code. In fact, returning **any interface** is probably not a good idea. 

> interface{} says nothing

See what [Rob Pike](https://en.wikipedia.org/wiki/Rob_Pike) has to say about it (the video will start from a specific time, but watch the whole video when you get a chance):

{{< youtube "PAAkCSZUG1c?start=455" >}}
<br>

So, return a concrete type unless it's absolutely necessary to return an interface. In the above case, what could possibly be the reason to use an empty `interface{}`? `impl1` and `impl2` might return different types? Then, in that case, they shouldn't really conform to the same interface. There is almost always a way to **not** use an empty `interface{}`

Hope this post was interesting. If you have any suggestions/corrections, consider [making a PR](https://github.com/arjunmahishi/arjunmahishi.github.io/edit/master/content/posts/returning-interface-golang.md) :) or you can also slide into my [Twitter DM](https://twitter.com/messages/131552332-131552332?text=hi) -->
