# Kotlin Basics

only list concepts I'm not familiar with

## expression

consists of variables, operators, methods calls etc that produce a single value. 

## primary constructor
## secondary constructor
## init 
## val vs var
## scope function (let, apply, with, run, and also)

temporary access to the inside of a specific object with this scope function.

### Context Object

return the object itself so you can use it with chain functions (e.g., do().then().handle()....)

### Lambda 

return the your lambda result so the caller should prepare like (var result = YourObject.let { ... }

### it and this

same thing which point to the context object 

'run' does nto have its argument, so you cannot use 'it'/'this'

## lateinit

delay the initialization of a variable later time (esp, not in the contructor)

```
lateinit var Num: String
```

## Companion Objects

an alternative of static members/behaviors in Kotlin 

Kotlin does not have a concept of static, and you always have to create an instance.

companion object is an singleton instance of a class, you can declare the static members/behaviors inside the companion object.

## Double Bang (!!) Operator

not null, and throw an exception (esp, KotlinNullPointerException) when the associated variable is null

## Array 

fixed size

mutable

## List

dynamic size

immutable

## const

compile-time variable

a value must be assigned at compile-time

only String and primiive values are asigned to const

## val 

runtime immutable variable

## open

the opposite of 'final' in Java. in Kotlin, it means open to inherit

## public

default class scope, any members/behaviors are visible to the outside.

## suspend function

a function which can be paused and resumed later time (async).

it can be invoked by only another suspend function or inside the coroutine.

useful when you want to abstract an async logic inside the function from 'runBlock' 

```
fun main() = runBlocking { // this: CoroutineScope
    launch { doWorld() }
    println("Hello")
}

// this is your first suspending function
suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```

## coroutine

an instance of suspendable computation.

take a block of code and run concurrently. 

might run on different thread. a coroutine does not bound to a spcific thread.

a light-weight version of threads

## runBlocking

a coroutine builder that bridges the non-coroutine world of a regular function

## package-level functions

functions declared outside class

you need to declare the following to the file:
```
package <name> 

fun myStaticFunc() = {}
```

this function is considered as static function (all clients receive the same instance)

## Unit Type

same as void in java

## Nothing Type

it has no values, so Nothing? is null.

## Void Type

we don't have void type in kotlin

## delegate property

syntax:
```
val/var <property name>: <Type> by <expression>
```

the expression is called delegate. when assigning the value of this variable, this delegates the assigning logic to the expression (e.g., function, lambda)

## observable property



## lazy property

run the lambda first and remember the result and return the result any consecutivie access.

```
val aVar by lazy {
    println("I am computing this value")
    "Hola" // this result is remembered for any consecutive access 
}

// 1st access e.g., println(aVar):
"I am computing this value"
"Hola"
// 2nd access
"Hola"
```

## lambda 

when you want to return a value, write like this;

```
nts.filter {
    val shouldFilter = it > 0
    shouldFilter // same
}

ints.filter {
    val shouldFilter = it > 0
    return@filter shouldFilter // same
}

```

don't need to write return keyword
