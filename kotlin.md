# Kotlin Basics

only list concepts I'm not familiar with

## expression

consists of variables, operators, methods calls etc that produce a single value. 

## primary constructor

define paramters at class level. 

automatically define the paramters as properties of the class __if you use 'var/val' in the constructor__.

ex:

```
class MyClass(var name: String, address: String)
```

- name: a property of this class
- address: just paramter of the primary constructor (not property of this class)


## secondary constructor

constructor function inside a class.

you cannot define a property which is not defined at primary constructor

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

## object keyword

### object declaratioin

```
object MySingletonObject { .. }
```

### object expression

mainly used when you need to create anonymous implementation of a class esp abstract.

```
abstract class MyAbstractClass {
    abstract fun doSomething()
    abstract fun sleep()
}

fun main() {
    val myObject = object : MyAbstractClass() {
        override fun doSomething() {
            // ...
        }

        override fun sleep() { // ...
        }
    }
    myObject.doSomething()
}

```

mostly used when you want to create a singlton object.

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

it can be invoked by __only another suspend function or inside the coroutine__.

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

### suspending function

a function with _suspend_ keyword. you can use this function inside a coroutine.

### dispatcher

decide which thread or threads the corrsponding coroutine uses to execute its execution. for example, it can confine a given execution to be run on a specific thread, send it to thread pool, or run on the main thread.

#### default (Dispatchers.Default)

limited to the number of CPU cores (with a minimum of 2) so only N (where N == cpu cores) tasks can run in parallel in this dispatcher.

is intended for CPU intensive tasks, where there is little or no sleep.

#### IO (Dispatcher.IO)

by default 64 threads, so there could be up to 64 parallel tasks running on that dispatcher.

spends a lot of time waiting.

### context switching

switching execution contex based on its task

1. update LiveData and the UI on __Dispatchers.Main__.
2. do database and network operations on __Dispatchers.IO__.
3. anything else on __Dispatchers.Default__.

how to switch to a specific context when we I have a specific task?

use _withContext(<DISPATCHER>)_.

### coroutineScope

a coroutine builder that can be used inside any suspending function to peform multiple concurrent operations. 

```
// Sequentially executes doWorld followed by "Done"
fun main() = runBlocking {
    doWorld()
    println("Done") // 4
}

// Concurrently executes both sections
suspend fun doWorld() = coroutineScope { // this: CoroutineScope and allow you to run multiple coroutine concurrently
// but main thread is blocked until this is done.
    launch { // run a different thread
        delay(2000L)
        println("World 2") // 3
    }
    launch { // run another different thread
        delay(1000L)
        println("World 1") // 2
    }
    println("Hello") // 1
}
```

### runBlocking

a coroutine builder that bridges the non-coroutine world of a regular function, so should not be called from a coroutine. 

this block the code inside until the execution inside that is completed.

you usually dont use this in production. 

### Job object

a handle to the launched coroutine and you can explicitly wait for its completion (e.g., 'join()')

```
val job = launch { // launch a new coroutine and keep a reference to its Job
    delay(1000L)
    println("World!")
}
println("Hello")
job.join() // wait until child coroutine completes
println("Done") 
```

also, you can cancel a coroutine with _cancel()_ of the Job object.

but if a coroutine is working in a computation and does not check for cancellation, it cannot be cancelled.

## coordinate multiple coroutines

use _async_ keyword to coordinate multiple coroutine sequentially.

```
val time = measureTimeMillis {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    println("The answer is ${one.await() + two.await()}")
}
println("Completed in $time ms")
```
### Testing With Coroutines
    
use _runBlockingTest_ to test coroutines since this will block the main thread (testing) until the coroutine is completed. Also, you cannot call suspending functions from regular context. it is only called inside coroutines or another suspending function.

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

every time you assign a new value to the property which is observed, the handler is executed. you can set an initial value for it.

```
class User {
    var name: String by Delegates.observable("<no name>") { // initial value
        prop, old, new -> // prop: property name, old: previous value, new: new value
        println("$old -> $new")
    }
}
```

this name variable is observed and the handler is called every time the variable is updated.

if you want to intercept such as give an condition if the value is updated or not, you can use _vetoable()_ instead of _observable()_.

ref: https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/vetoable.html

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

## extension functions

you can append your own funcion to existing object such as String without inheritance.

useful when you want to modify a class from 3rd party libraries but you cannot modify.

```
fun String.spaceToCamelCase() { ... }

"Convert this to camelcase".spaceToCamelCase()
```

extension is not modifying the existing class so be careful and it is done statically, means that an extention defined for the type is called not its subclass.

ex:
```
open class Shape
class Rectangle: Shape()

fun Shape.getName() = "Shape"
fun Rectangle.getName() = "Rectangle"

fun printClassName(s: Shape) {
    println(s.getName())
}

printClassName(Rectangle()) // "Shape" not "Rectangle"
```

if extension function already exist in the target class, the member always wins (take a precedence of the target class method over your exteinsion).

but you can use overload (e.g., different return type and paramters but the same name) so that your extension is called.

also, you can define a extesion property, but you cannot use backing field since the target class is not modified. so you need to initialize the value without using the backing field:

```
val isEmpty: Boolean
    get() = this.size == 0
```

## if-not-null shorthand

syntax:

```
variable?.prop // return value if the variable is not null
```

execute if not null

```
value?.let { ... } // execute if the value is not null.
```

## if-not-null-else shorthand

syntax:

```
variables?.prop ?: "empty" // if the variable is null, this return "empty"
```
```
... = value["email"] ?: throw Eception() // if the value is null, throw the excepiton
```

default value 

```
val mapped = value?.let { transformValue(it) } ?: defaultValue
```

## when statement

flexible version of switch statement.

paramter is optional and you can put conditional statement or value.

## TODO function

when you have not implemented yet some code, you can use this function. it throws NotImplementedError. Also, return Nothing as a type 

## Coding Conventions

### directory structure

follow the package structure with the common root package omitted.

### source file names

single class => use the class name as file name
multiple classes/top-level declaration => use descriptive name 

use Pascal case (e.g., MyMultipleClasses.kt)

### property names

use underscore _ for internal private property

## types

### Bitwise operations

shl: signed shift left (keep the sign while shifting. if 1 is sign, keep the 1 and shift the rest of bit. if 0 is sign keep the 0 and shift the rest of bit)

```
    11011011 
 >> 11101101  

    01010010
 >> 00101001 
```

ushl: unsigned shift left (ignore the sign while shifting. always fill zero)

```
    11011011 
>>> 01101101  

    01010010
>>> 00101001
```

## inheritance

no multiple inheritance (like Python)

use interface or composition.

classes are final - the class can not be inherited. use the open keyword if make this inheritable.

this also apply for behaviors of the class. to override the behavior, the original bahevior must have the open keyword and the subclass method need to have the override keyword. Otherwies, the compiler complains.

### override keyword for methods and properties

behave same for both methods and properties

override keyword itself include meaning of the open keyword which means the the method is also overridden. 

if you prohibit the override, use the final keyword.

```
open class Rectangle() : Shape() {
    final override fun draw() { /*...*/ }
}
```
### override val and var property

can override val property with var property, but not vice versa.

this is because val in parent class just need to define the getter and override the val in subclass with var is just that you need to add a setter in the subclass. so it is possible. but the other way around is impossible since the parent class with var property already define the setter and getter and you cannot not modify the fact from the subclass.

### overrideing rules

if a class (subclass) inherit multiple implementations and those have the same members/functions, you need to override the members/functions in the subclass.

```
open class Rectangle {
    open fun draw() { /* ... */ }
}

interface Polygon {
    fun draw() { /* ... */ } // interface members are 'open' by default
}

class Square() : Rectangle(), Polygon {
    // The compiler requires draw() to be overridden:
    override fun draw() {
        super<Rectangle>.draw() // call to Rectangle.draw()
        super<Polygon>.draw() // call to Polygon.draw()
    }
}
```

## interfaces

you can implement the body of its members/functions (optional)

you can make an interface inherit from another interface.

be careful when you have the same members/functions in your interface, you need to override them in your subclass.

## functional (SAM) interfaces

def) an interface with only one abstract method is called _functional interface_ or _single abstract method (SAM) interface_.

can have multiple non-abstract members and one abstract member.

can extend other interfaces.

```
fun interface KRunnable {
   fun invoke()
}

val myRunnable = KRunnable { ... } 

myRunnable() // call the implementation 
```
### diff btw SAM and type aliases

type aliases: create a name of existing type

SAM: create a new type

## visibility modifier

modifier \ scope | packages | classes and interfaces
-- | -- | -- |
public (default) | everywhere | everywhere |
protected | N/A | include its subclass |
internal | the module | the module |
private | the file | the class/interface

### module

a set of Kotlin files compiled together such as Maven project, Gradle source set, and so on

## backing field

a field that will be generated for a property in a class only if it uses the default implementation of at least one of the accessors, or if a custom accessor references it through the field identifier.

```
class User{
    var firstName : String  //backing field generated 
        get() = firstName
        set(value) {firstName = value}
} 
```

Kotlin throws StackOverFlow error. this is because when you access getter/setter, it call default accessor will be called. so this ends up that a getter calls the getter again and gain so wind up calling stackoverflow error.

the solution is to use backing fields (e.g., field keyword) 

```
class User{
    var firstName : String  
        get() = field // <- backing field
        set(value) {field = value} // <- backing field                            
}
```

an initializer assign the backing field directly

```
var counter = 0 // you need to have the backing field to initialize this variable
```

define property without backing field

```
val isEmpty: Boolean
    get() = this.size == 0
```

## data class

used for POJO such as DTO. only hold data with accessors and other essential utility members (such as equals, toString, copy, componentN)

only properties defined with the primary constructor are used for this data class and its utility members. so if you define a property in the body, it is not included in the utility members.

## sealed class 

to restrict inheritance hierarchies. you can specify a set of classes which is only allowed to inherit and prohibit inheriting from other class.

sealed class is abstract class so you need a subclass to inherit.

useful when you need more flexibility over Enum. for example, you want each member of an enum to have state, you can use Sealed class.

still don't get it. read this: https://blog.mindorks.com/learn-kotlin-sealed-classes

## generics 

### out modifier (covariant)

```
interface Source<out T> {
    fun nextT(): T
}
```
T is only used for return type (produced) and not for input (consuming)

### in modifier (contravariant)

```
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}
```
it can only be consumed and never produced. 

### type project 

read when you have time

## nested and inner classes

### inner keyword

make the inner calss access the members of the parent class.

## inline class

primitives for class version. called _value-based class_. i guess this is treated like primitive (store in stack not heap) but it is an object.

does not have id and only hold a value.

use _value_ modifier:

```
value class Password(private val s: String)
```

must have a single property initialized in the primary constcutor. 

you can also declare properties, functions, and init block.

no backing fields, lateinit, delegated

must be final and can implement interfaces.

## delegation (delegation pattern)

an alternative (could be better than) of inheritance.

use when you want to override existing method or add new method. 

```
interface Producer {
    fun produce(): String
}

class ProducerImpl : Producer {
    override fun produce() = "ProducerImpl"
}

class EnhancedProducer(private val delegate: Producer) : Producer by delegate {
    override fun produce() = "${delegate.produce()} and EnhancedProducer"
}
```

## functions

### infix notation

allow you to call a function without dot and the parentheses.

```
infix fun Int.shl(x: Int): Int { ... }

// calling the function using the infix notation
1 shl 2

// is the same as
1.shl(2)
```

### local functions 

you can declare a function inside another function.

it has a closure, so an inner function can access to variables of the outer function.

### tail recursion

convert a recursive method to a loop for performance gain. (esp function call is expesive than a loop) but there are conditions to make this works.

1. use _tailrec_ keyword
2. call the recursive method as the last operation.
3. cannot use _try/catch/finally_ block.

## Higher-order functions

a function that takes functions as parameters, or returns a function.
    
used to generalize application logic such as loading spinner. you can think it as the same as the HOC in React. extract application logic and reuse for any component.

## inline functions

lambda cause the extra memory allocations and extra virtual method call. to mitigate the performance degration, you can use inline functions.

instead of compiling to the function declaretion as another place, the compiler write the body inline where the functtion is called. that's why you don't need to waste memory and call above. 

use _inline_ keyword to make your function inline

```
inline fun <T> Collection<T>.each(block: (T) -> Unit) {
    for (e in this) block(e)
}
```

by default, the lambda passed as paremeter is inlined too. if you want to change this behavior you can use _noinline_ keyword. 

```
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) { ... } // notInlined function is not inlined.
```

usually type paramters are erased at runtime, but for inline functions, we can avoid this limitation with _reified_ keyword.

some inline function with type paramter would not be compiled if you don't use _reified_.

```
inline fun <reified T> Any.isA(): Boolean = this is T
```

__we can inline functions with lambda parameters only if the lambda is either called directly or passed to another inline function. Otherwise, the compiler prevents inlining with a compiler error__.

__only use inline when the lambda is short__. if the body is long, increase the code a lot.
