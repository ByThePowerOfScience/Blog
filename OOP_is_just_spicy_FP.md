# Preliminary Info

The core of OOP is the idea of solely interacting with data through an "interface": a set of behaviors that "constitute" that object's operations. (not the keyword)

Basically, instead of manually doing things to the data in the box, you:
1. Seal the box,
2. Give it ways to be interacted with, and
3. Just use the ways instead of opening the box.


I like to think of it as the buttons on a cash register: rather than manually manipulating wires and modifying the internal state of the cash register directly, it exposes certain things it can do through its buttons, and pressing each one does a different thing.  You can just view it from the outside as "a cash register", not "a box of wires that takes the shape of a cash register".

One point of solely interacting through an object's "interface" is that if someone puts the exact same buttons on something else, and they did the exact same thing, then it doesn't matter if it's a bunch of squirrels on the inside opening the drawer instead of wires, that thing's still a cash register.  That's where *polymorphism*, usually regarded as the primary principle of OOP, comes into play.

&nbsp;

Something interesting is that a lot of languages have classes implicitly have an interface corresponding to a class's type: the set of all of its public members.  I wouldn't say that's a bad thing, but it masks what's really going on.

The first two examples I show below are just explicitly extracting that true model of an object: its "interface" (again, not "interface class", I mean "the way you interact with it"), its state (the data that interface acts on), and the implementation of the interface to do the things the interface describes.

&nbsp;

You also have a ton of design patterns laid on top of that, like the "actor model", where each object "does something" and you solely hand off tasks between those "actors".  That's something I originally conflated with OOP as a whole, but after debating my theories with someone else, I found that it's largely built _on top of_ OOP and doesn't actually _constitute_ it.

&nbsp;

# The Realization

Ok, so follow me here.  Let's look at a standard "class" in OOP:

```kotlin
class MyDataType {
	private var storedData: Int = 0
	
	fun addNumber(number: Int) {
		storedData += number
	}
}

fun foo(myData: MyDataType) {
	myData.addNumber(2)
}
```

Pretty simple, right? We define a class (`MyDataType`), the data it stores (`storedData`), and a way you interact with it (`addNumber`).  This is your basic object: a data type that has behavior.

<br>

Let's abstract this a bit.  Setting aside all *implementation details* - optimizations, vtables, etc. - and just focusing on how you **use it**, is the above fundamentally different from this:

```kotlin
interface IMyDataInterface {
	fun addNumber(number: Int)
}

class MyDataType : IMyDataInterface {
	private var storedData: Int = 0
	
	override fun addNumber(number: Int) {
		storedData += number
	}
}

fun foo(myData: IMyDataInterface) {
	myData.addNumber(2)
}
```

It isn't, right? As mentioned above, this is just extracting the implicit "interface" from the object, turning the ways you're supposed to interact with it into a type all on its own.

---

Now, what if we took it a step further?  Is *that* <u>fundamentally</u> different from *this*?

```kotlin
interface IMyDataInterface {
	fun addNumber(myData: MyDataType, number: Int)
}

class IMyDataInterfaceImpl : IMyDataInterface {
	override fun addNumber(myData: MyDataType, number: Int) {
		myData.storedData += number
	}
}

class MyDataType {
	@AccessibleTo(IMyDataInterface)
	private var storedData: Int = 0
}

fun foo(itf: IMyDataInterface, myData: MyDataType) {
	itf.addNumber(myData, 2)
}
```

This one might seem a little weird, but just think about it for a second.  The core abstraction of OOP is "what it *does* vs what it *is*"; you see this a lot with composition, traits, and all non-classical forms of inheritance.  

In this case, this is a basic trait: we have an interface that fully governs how we interact with the data, it just doesn't *determine* the "type" of the data itself.  

These are, fundamentally, the same thing as virtual methods, just with a different implementation.


<br>

For further clarity, let's look at where the pattern comes from:

```c++
template<K, V, Hasher = std::hash<K>>
void HashMap::put(K* key, V* value) {
	this->put_by_hashcode(Hasher::hash(key), value)
}
```
This is a compile-time version of the same thing: the function is *given* an interface for getting the hashcode of some type - `std::hash<T>` - and it's *applied to* some data type - `K`.  This interface can be implemented on multiple types, it's just that instead of a type having to *be* an std::hash to have a hashcode, `std::hash` is *defined for* a type and applied to it later.

---

Now, take a sec and get comfortable with this, because I'm about to blow your mind.

Is this:
```kotlin
interface IMyDataInterface {
	fun addNumber(myData: MyDataType, number: Int)

	fun getNumber(myData: MyDataType): Int
}

fun foo(itf: IMyDataInterface, myData: MyDataType) {
	itf.addNumber(myData, 2)
	val x = itf.getNumber(myData)
}
```
*fundamentally* different from this?

```kotlin
fun foo(addNumber: (MyDataType, Int) -> Unit, getNumber: (MyDataType) -> Int, myData: MyDataType) {
	addNumber(myData, 2)
	val x = getNumber(myData)
}
```


<br>

<br>

<br>

*OK, SO.*

This obviously doesn't seem right, but let's think about it:

- What we're doing when we make an "interface" is we're *collecting a set of functions under a name*.   The basic assertion is that if a type has *one* of those functions, it has *all* of them, so we can treat anything with that interface as being fully interchangeable with any other thing with that interface.
- When we use a trait, we separate the interface from the data itself, giving the raw set of functions as an argument to whatever is doing something with it.  
	- This still fulfills the assertion of an "interface": the scope has access to all functions that act on the data, and the data cannot have one of those functions without having all the rest.  Anything with a trait will have all of the capabilities of any other with that trait, and you interact with it through the trait.
- What we're doing when we use a trait then, is we're *passing a set of functions **under a name***, which is just a *named version of functional programming*.  And yet it's <u>undeniably</u> object-oriented!!!

<br>

Even when you think of OOP as more of a *design philosophy* than a "set of mechanics", it *still* doesn't hold up:

> I want to define a List as some object such that I can add and remove elements and be able to sub out the behind-the-scenes part with anything else that has those methods-
>
> wait no what are you doin-

*\*metal pipe.wav\**

```kotlin
fun <LIST, EL> usesList(
	listData: LIST, 
	add: LIST.(EL) -> Unit,
	remove: LIST.(EL) -> EL?,
	contains: LIST.(EL) -> Boolean,
	...
)
```

<br>

"Ok but isn't this really clunky and dumb?"  *YES.*  

"Isn't this just shoddy structuring?" ***ABSOLUTELY.***

The point isn't to encourage this style of programming *at all*.  In fact, I think it's really bad design.  
You *should* be organizing your code, giving names to commonly-used things, and designing your program such that it's *understandable* rather than "adherent to some strict design philosophy".

<br>

But part of that is *understanding the tools you're working with*.  If we adhere to OOP or FP like it's "the one true form of programming", we lose the ability to *adapt*, to *stretch our definitions*, to recognize how each form can *seamlessly* blend into the other to create something greater than the sum of its parts.

When you first learn, it's helpful to have these strict definitions of what something "is" or "isn't".  But if you truly want to be a master, you have to understand that at the end of the day, all of our "hard classifications" are just a line in the sand we drew to try to sort the ever-changing world into neat little boxes.  To be a master, you need to take notes from *everything*, even those you think are the opposite side of the spectrum.


<br>

In the end, there's more-"object oriented" styles and more-"functional" styles, but those are just the restrictions we place on ourselves.  They're ultimately two sides of the same coin leading to the same conclusion, like derivatives and integrals, and only focusing on one means you limit what you can do.

It's all fundamentally the same toolbox.  It's only once we embrace it, once we learn to grow outside our little shells, that we become strong enough to face our true enemy: 

JavaScript.


<br><br><br><br><br><br>


<hr>



PS:

Now obviously, I'm not saying they're "the same thing". There are different styles of programming that make something "more functional" or "more object-oriented". In fact, the "pure" versions of both OOP and FP are extremely easy to identify: pure OOP being the endpoint of leaning into the naming, and pure FP being the endpoint of leaning into the free functions, and neither really being that much fun to work with.

But my point is that the fundamentals of both are the same, just like how derivatives and integrals form two sides of Calculus. If you try to defend one and chastise the other, you can't use either effectively, and just dig yourself deeper into a hole of design patterns to make up for how inflexible you've left yourself. In truth, once we dissect how OOP really works, we can deconstruct it to replace most design patterns with something that borrows from each of them - Factories with constructor references, listeners with function references, list mapping with transducers - all without violating any OOP principles.

It's only once we intuitively understand their foundations that we can use OOP less rigidly and FP more structured, creating something that's far greater than the sum of its parts.

