---
layout: post
section-type: post
title: Understanding references and pointers and Go
category: Go
tags: [ 'Go' ]
image:  /img/posts/2017-06-05-understanding-references-and-pointers-in-go/go.jpg
---

Lately I have been making more of attempt to get involved with Go and I wanted to make a post to cover the basic concept of functions.

If you come from more of an Operations *(or are simply new to Go)* background and are looking to learn and understand basic Go concepts of referencing and functions then this is the post for you.


Contents
=================

* [The Basics](#the-basics)
* [Passing by Value and Reference](#passing-by-value-and-reference)
* [Variadic Function](#variadic-function)

### The Basics

Even in Go's most basic form we've already worked with functions, by that I of course mean the `main` function. However, let's dig deeper into this concept with a few of our own functions and really demonstrate what Go has to offer when it comes to building functions in order make your code readable and maintainable. In regards to the `main` function, there is really nothing special here, it is simply responsible for being the entry and exit point for our application.

Lets make our own function along side the main function to demonstrate this basic concept.

```Go
package main

func main() {
    printAbc()
}

func printAbc() {
    println("Abc")
}
```

If you save this and run it now, it works fine. Simple eh?

This is great and all but we're a bit limited in what our function can do sice we cannot pass any information along with the invocation yet.
In order to do that Go allows us to pass in parameters that will be passed onto the function to alter its behavior.

```Go
package main

func main() {

    message := "Abc"

    printAbc(message)
}

func printAbc(message string) {
    println("Abc")
}
```

If we run this now we can see that we've been able to make our function much more reusable and far easier to read.

### Passing by Value and Reference

So what do you think would happen if we wanted to alter the value of the incoming message variable? Lets find out!

```Go
package main

func main() {

    message := "Abc"

    printAbc(message)
    println(message)
}

func printAbc(message string) {
    println("Abc")

    message = "Xyz"
}
```

Well, now we simply get `Abc` displayed back to the console twice. But didn't we reassign the message variable? What happened here?

This is due to the fact that Go has copied the value of the variable into the function's parameter. That means that changing the message variable in the function only changes the copy and **not** the original.

If want want to allow the function the change that value then we need to change the code a little bit. Instead of passing the value of the variable, lets pass in the memory location of the variable. By providing the function a reference of how to get to the variable it will then be able to manipulate the value directly.

Let's do that by changing our code as follows.

```Go
package main

func main() {

    message := "Abc"

    printAbc(&message) // Step 2 - adding in ampersand in front of our parameter to pass the memory address not the value
    println(message)
}

func printAbc(message *string) { // Step 1 - change the expected datatype to be a pointer.
    println(*message) // Step 4 - If we left this asterisk off here we would be printing out the passed in parameter, which is a memory address. We don't want that so lets add the astrix here also.

    *message = "Xyz" // Step 3 - change the message variable to assign its value to the memory location of the pointer.
}
```

Let's break down everything we've changed here.

**Step 1** In our *printAbc* function we've added an asterisk to the expected type for our function's parameter.
This will tell the function that it should receive a memory address rather that is pointing to a string, or in other words a **pointer**.

**Step 2** After we've done that we need to ensure we're now passing in that memory address rather than the string itself. We can do that by adding in an ampersand to the left side of our parameter that we're passing into our function.

**Step 3** The next change we'll need to make is to add an asterisk to message variable inside the printAbc function.
This time we're asking Go to put the string into the memory location which was referenced by the pointer.

**Step 4** The last step is to our `println` function inside *printAbc*, we want this to print out the value of the variable and not the memory location.
Again, in order to do that we simply add an asterisk to the front of the variable name. This concept in Go is called "dereferencing a pointer".

Now if we run the application we can see that we have succeeded in changing the message variable in the main function from within our printAbc function.

In order to fully understand this concept follow the path of execution through the two functions.
If you can understand each of the steps outlined in the comments of the snippits and map it to the logical execution of the program then you are in the clear.


### Variadic Function

In Go we have the concept of something called a "Variadic Function" the idea here is simple.
We want to be able to pass in any amount of parameters to our function. In order to show this, let's change our application a little bit.

```Go
package main

func main() {

    printNames("Flynn", "Dave", "Sanchez") // Step 3 - We're calling our function and passing in strings for our variadic function to take.

}

func printNames(names ...string) { // Step 1 - We've changed our function here to acceptance variadic function, indicated by the '...'
    for _, name := range names {  // Step 2 - We're iterating over the slice of names using the inbuilt range function and priting the name to the console
        println(name)
    }
}
```
When called, Go is going to take all of the names we have specified and pass them into the function as a slice of strings that are assigned to the names variable.
From there, we can work with the values like a normal slice.

The Steps in the above comments indicate the changes we've made and how this works exactly.
