# cpp
Repository for notes and code examples for C++

# Pointers

## std::shared_ptr

https://en.cppreference.com/w/cpp/memory/shared_ptr

Smart pointer for sharing ownership through pointer. Several shared pointer objects, shared_ptr, may share same object. The object is destroyed and its memory deallocated when either of the following happens:

* The last remaining shared_ptr owning the object is destroyed
* The last remaining shared_ptr owning the object is assigned another pointer via operator= or reset()

The object can also just be destroyed.

The shared pointer stores a pointer. Also stores a use_count, which counts the usages of the pointer.

A wrapper around a normal pointer.

`std::make_shared()` to create a shared pointer, of type `std::shared_ptr<T>`.


It's almost always useful to create a shared pointer as opposed to a regular pointer, since a shared pointer automatically will deallocate the memory after usage is over, as opposed to a standard pointer, which required you to manually deallocate the memory. This is one reason why memory leaks can occur, since is a pointer is not deallocated, the object in the heap it is pointed to is never collected by the GC.

Inside of the `std::make_shared()` call is a call to `new` which instantiates the pointer. Sometimes, to create a pointer to a new object, new be used with an object. Example `std::make_shared(new MyObject())`


## std::unique_ptr

std::unique_ptr is a smart pointer that owns and manages another object through a pointer and disposes of that object when the unique_ptr goes out of scope. 

Removes the object whenever the pointer goes out of scope.

The object is removed when:

* the managing unique_ptr object is destroyed
* the managing unique_ptr object is assigned another pointer via operator= or reset()

Can also be deleted.

Ownership can be transferred from a non-const unique pointer to another.

`std::unique_ptr` is commonly used to manage the lifetime of objects. Example:

* Provide exception safety to classes and function by guaranteeing that an object is deleted on both normal exit and exception exit
* Passing ownership of uniquely owned objects with dynamic lifetime into functions
* Acquiring ownership of uniquely owned objects with dynamic lifetimes from functions
* ...

How to create, example:

`unique_ptr<int> uptr (new int(3));`

Constructor accepts raw pointer of object type T, i.e. T*. In the instance above new generates a pointer to an int.

Common to use "new" to ensure that the object is initialized inside the pointer, so that it cannot be referenced by other unique pointers elsewhere.

https://stackoverflow.com/questions/16894400/how-to-declare-stdunique-ptr-and-what-is-the-use-of-it


C++14, syntax::

`unique_ptr<int> p = make_unique<int>(42);`


After transferring ownership the previous owner will not return null, and nullptr for get(), since the unique_ptr no longer points to anything.

This is similar to creating a move with Rust. It does not seem that the C++ compiler is able to catch these errors.

Use of std::move() can also be used to move ownership of a pointer.

Factory example: https://stackoverflow.com/questions/26318506/transferring-the-ownership-of-object-from-one-unique-ptr-to-another-unique-ptr-i


## Smart pointers

Smart pointers are an addition to C++ to help with memory leak issues. As long as contracts are followed, the prevent memory leakage.

Prevents you from having to de-allocate the memory which was assigned with a regular pointer. The smart pointers have rules for when it happens.

Whenever the *new* keyword is used, memory is allocated on the heap memory. It will be possible to get a pointer to the memory in the heap.


# Find and Friends

std::find, std::find_if, std::find_if_not

The functions return an iterator to the first element in a range that satisfies criteria:

* std::find searches for an element equal to value
* std::find_if searches for an element for which predicate p returns true
* std::find_if_not searches for an element for which predicate q returns false

https://en.cppreference.com/w/cpp/algorithm/find

See example section.

`auto result = std::find(begin(vector), end(vector), value);`


# Remove and Friends

Remove elements satisfying criteria:

* Removes all elements that are equal to value, using operator== to compare them
* Removes all elements for which predicate p returns true

`std::remove` removed all elements where elements are equal to value.

`std::remove_if` remove all elements where unary operator returns true.


# Optional

`std::optional`, available since C++17.

Useful in cases where an operation can fail, a value can be present or not. An alternative to `std::pair<T, bool>`, which is a workaround. Optional is significantly more expressive, clear what is meant.

Optional object models an object not a pointer.

When converted to a bool, an optional with a value evaluates to true, and an optional without a value evaluates to false.

Several clever operations which can be called on an optional type.


Examples:

https://en.cppreference.com/w/cpp/utility/optional

Note that the bottom example uses std::reference_wrapper<T> to create a reference, and get is called() and set on the reference.



Use of *{}* to return empty optional.

Use of *std::nullopt* to create empty optional:

```
// std::nullopt can be used to create any (empty) std::optional
auto create2(bool b) {
    return b ? std::optional<std::string>{"Godzilla"} : std::nullopt;
}
```


# Bind

`std::bind`

The function template bind generates a forwarding call wrapper for f. Calling this wrapper is equivalent to invoking f with some of its arguments bound to args.

Use of placeholder arguments, `std::placeholders_#`

## Partial Function Application

std::bind is a partial function application, can be used to wrap a function call to simplify it by passing static parameters to the wrapped function.

https://stackoverflow.com/questions/6610046/stdfunction-and-stdbind-what-are-they-and-when-should-they-be-used

You generally use it when you need to pass a functor to some algorithm. You have a function or functor that almost does the job you want, but is more configurable (i.e. has more parameters) than the algorithm uses. So you bind arguments to some of the parameters, and leave the rest for the algorithm to fill in.

Used to simplify functions by wrapping them.


## Generalized Function Pointer

Can also be used to generate a pointer to a function.

Equivalent to creating a safe function pointer.

https://stackoverflow.com/a/40944576/18558004


Why do we need to pass `this` after the reference to the function? Because a non-static member function has an implicit first parameter of type Class*, i.e. a pointer to the object it is part of. Because of this, we need to also supply a pointer to the object which the member function belongs to when using std::bind as a function pointer.

The syntax is then:

`std::bind(<function_reference>, <object_reference>, <function_arguments>)`.

Where `<object_reference>` is often `this`, since we're commonly binding a function which is a member to the class which is executing the bind.


When binding a function that is not a member, we can simply provide a function reference and then the arguments to the function with placeholders.


# Lambda

A lambda is convenient way of defining an anonymous function object (a closure). Useful to avoid function bloat in a codebase.

https://learn.microsoft.com/en-us/cpp/cpp/lambda-expressions-in-cpp?view=msvc-170

```
[]() mutable throw() -> int {
    ...
}
```


## Capture

[] capture clause. C++14. Can access or *capture* variables from the surrounding scope.

Variables put inside the [] can be accessed inside. How variables are captures can be specified with & or =, & is by reference, = is by value.

Examples:

```
[&total, factor]
[factor, &total]
[&, factor]
[=, &total]
```

`[&, ...]`, sets the default capture method as if by reference.

This can also be captured.


**Note** Static values do not have to be captured to be used inside of the lambda.


## Generalized Capture C++14

Parameters can be initialized in capture clause. See source.


## Parameter List

Similar to function. Optional for lambda. Parameters for the function body.

Use of auto in parameter list is used as a template. Each instance of auto in parameter list is equivalent to distinct type.


## Mutable Specification

The mutable specification enables the body of a lambda expression to modify variables that are captured by value. They are not actually modified at the calling location, but they can be modified inside of the lambda body.


## Exception Specification

You can use the noexcept exception specification to indicate that the lambda expression doesn't throw any exceptions.


## Return Type

Automatically inferred form parameter list or must be specified with trailing-return-type, `->`.


## Lambda Body

Where the logic of the function itself is written.


