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


# gTest Testing

Use of macros such as EXCEPT and ASSERT to make assertions.

Prefer EXCEPT over ASSERT when it makes sense. EXCEPT records failure but continues, ASSERT does not continue.

If ASSERT is placed over clean-up it can cause memory issues.

Custom failure messages can be provided by streaming into the the assertion macros, `EXCEPT_TRUE(false) << "Statement should always be true"`.


Syntax:

`TEST(<test_suite>, <test_name>)`

Tests are grouped by test suite.

Names should be CamelCased. Should not contain underscore (_).


## Test Fixtures

Configuration which makes it possible to reuse objects for several tests.

* Derive a class from `::testing::Test`
* Declare members inside class to be used in test
* Start body with `protected:`, we want to access fixture methods from sub-classes
* If necessary, create `SetUp()` method to prepare objects for each test, use override to ensure override
* If necessary, create `TearDown()` method to release any allocated resources
* Define subroutines shared by tests

`TEST_F(<test_fixture>, <test_name>)`

Fixture must be defined before it can be used.

For each test defined with `TEST_F()`, googletest will create a fresh test fixture at runtime, immediately initialize it via `SetUp()`, run the test, clean up by calling `TearDown()`, and then delete the test fixture.

Note that different tests in the same test suite have different test fixture objects, and googletest always deletes a test fixture before it creates the next one. googletest does not reuse the same test fixture for multiple tests. Any changes one test makes to the fixture do not affect other tests.

Test fixtures are **not** shared between tests in the fixture.


Naming convention for testing a class is `<ClassNameTest>`.


## Invoking tests

TEST and TEST_F implicitly register their tests with googletest. No need to relist test to have them run.

Can be run with RUN_ALL_TESTS(). 0 all tests are successful, 1 otherwise.


When run, googletest in essence saves the state of all googletest flags, and then starts running tests.

Simplified cycle:

* Create fixture
* Run test
* Remove fixture
* Restore flags

Until all tests have been run.

https://google.github.io/googletest/primer.html#invoking-the-tests


Output must not be ignored, should be returned, output it not returned in stdou/stderr only the return. Cannot be run more than once for advanced reasons.


## Writing the main() Function

It should not be necessary to write own main function, link with gtest_main instead, to have a main function automatically included.


## Template

Template for writing tests with main().

https://google.github.io/googletest/primer.html#invoking-the-tests


# gMock Mocking

Not always feasible to rely on real objects.

A mock object implements the same interface as a real one, so it can be used as a real object, but lets you specify at run time how it will be used wand what it should do.

Mock != Fake

* Fake objects have working implementations but take shortcuts and are not production suitable, example in-memory db
* Mock objects are pre-preprogrammed with expectations, which form specification of calls they are expected to receive

Mocks makes it possible to check interaction between user and an object.

gMock is library to create mocks for C++.


Inspired by frameworks in Python an Java.

It's an alternative to hand-writing mocks, gMock does a lot of the heavy lifting using macros.


## Creating a Mock

The destructor of interface must be virtual, as is the case for all classes you intend to inherit from - otherwise the destructor of the derived class will not be called when you delete an object through a base pointer, and you’ll get corrupted program states like memory leaks.

Note that it's significantly easier to mock virtual methods, because of this, it's nice to inherit from an abstract interface, i.e. one with all pure virtual methods.

How to Mock a method from an interface:

* Derive a class from the interface
* In `public:` section of the child class write `MOCK_METHOD();`
* Add the function signature and add commas between return type, name, and argument list
  * `MOCK_METHOD(void, function_name, ())`
* If mocking a const method, add `(const)` as 4. parameter
* If mocking a virtual method, add `(override)` as 4. parameter
  * `(const, override)`

Repeat this for all virtual function which should be mocked. All virtual methods in abstract class must be mocked or overridden.

This should be done in a mock class. Naming suggests `ClassNameMock`.

It's possible to both to mock and create implementations in the mock class.

Include required: `#include "gmock/gmock.h"`


Example mock:

```
class MockTurtle : public Turtle {
 public:
  ...
  MOCK_METHOD(void, PenUp, (), (override));
  MOCK_METHOD(void, PenDown, (), (override));
  MOCK_METHOD(void, Forward, (int distance), (override));
  MOCK_METHOD(void, Turn, (int degrees), (override));
  MOCK_METHOD(void, GoTo, (int x, int y), (override));
  MOCK_METHOD(int, GetX, (), (const, override));
  MOCK_METHOD(int, GetY, (), (const, override));
};
```

The mock is simply a collection of functions marked for mocking.


## Where to Put Mocks?

Generally mocked should be owned by the team owning the interface that is being mocked.

Common to see them in the `_test.cc` file, but they can also be their own file.

There should only be one single mock for an interface.

When defining a mock class, it should be placed in `.h\.hpp` file, and be added to a `cc_library` target with `testonly=True`.


## Using Mocks

Typical workflow:

* Import gMock names from testing namespace so they can be used unqualified
  * Example: `using ::testing::AtLeast;` to get AtLeast
* Create mock objects
* Specify expectation on them, how many times will they be called? With what arguments? What should it do?
* Exercise code that uses mocks, optionally check results
* When a mock is destructed, gMock will automatically check whether all expectations on it have been satisfied

If mock objects are never deleted the verification will never happen.

Expectations must be set before the mock functions are called. Do not alternate between EXPECT_CALL() and mock calls.

This means that EXPECT_CALL() expects future calls, not past ones.

Example:

```
#include "path/to/mock-turtle.h"
#include "gmock/gmock.h"
#include "gtest/gtest.h"

using ::testing::AtLeast;                         // #1

TEST(PainterTest, CanDrawSomething) {
  MockTurtle turtle;                              // #2
  EXPECT_CALL(turtle, PenDown())                  // #3
      .Times(AtLeast(1));

  Painter painter(&turtle);                       // #4

  EXPECT_TRUE(painter.DrawCircle(0, 0, 10));      // #5
}

```


## Setting Expectations

Overall syntax:

```
EXPECT_CALL(mock_object, method(matchers))
    .Times(cardinality)
    .WillOnce(action)
    .WillRepeatedly(action);
```

matchers can be excluded for non-overloaded methods.

Example:

```
using ::testing::Return;
...
EXPECT_CALL(turtle, GetX())
    .Times(5)
    .WillOnce(Return(100))
    .WillOnce(Return(150))
    .WillRepeatedly(Return(200));
```

The syntax is sometimes called DSL (Domain-Specific Language). It's specific to the macro language used by gMock. Some technical reasons also, easier for debugging.


## Matchers: What Arguments Do We Expect?

The matchers can specify the arguments we expect the function to be called with.

An underscore can be used to match anything inside a matcher statement. Examples:

```
// Expects the turtle to move forward by 100 units.
EXPECT_CALL(turtle, Forward(100));
```


```
using ::testing::_;
...
// Expects that the turtle jumps to somewhere on the x=50 line.
EXPECT_CALL(turtle, GoTo(50, _));
```


Matches are excluded, arguments do not matter.

```
// Expects the turtle to move forward.
EXPECT_CALL(turtle, Forward);
// Expects the turtle to jump somewhere.
EXPECT_CALL(turtle, GoTo);
```


## Cardinalities: How Many Times Will It Be Called?

First clause specified after EXPECT_CALL() is Times(), cardinality, specifies how many times the call should occur.

Some fuzzy rules, i.e. where requirements are not set completely in stone. AtLeast() as an example.

If Times() is omitted, it is inferred by gMock based on rules. https://google.github.io/googletest/gmock_for_dummies.html#cardinalities-how-many-times-will-it-be-called

Times(AtLeast(n)).


## Actions: What Should It Do?

Defining what the mock does when it is called.

For built-in types, there is a default return, void pointer, false, 0.

For non built-in's or user-defined types, you can specify the action to be taken each time using `WillOnce()` and `WillRepeatedly()`.


Example:

```
using ::testing::Return;
...
EXPECT_CALL(turtle, GetY())
     .WillOnce(Return(100))
     .WillOnce(Return(200))
     .WillRepeatedly(Return(300));
```

Times() is not explicit, this function will be called at *least twice*.

If a function is called more than is specified with WillOnce(), and no WillRepeatedly() is specified, the mock will return the default value.

Possible to return reference among other things.

**Note**, the EXPECT_CALL() statement evaluates the action clause only once, even though the action may be performed many times. Example of returning n++ which does not work as intended.


## Multiple Expectations

By default, when a mock method is invoked, gMock will search the expectations in the reverse order they are defined, and stop when an active expectation that matches the arguments is found (you can think of it as “newer rules override older ones.”).

If an input matches an expectation it will be "caught" by that expectation even though it could match expectations which are defined earlier. It starts from the bottom.

https://google.github.io/googletest/gmock_for_dummies.html#MultiExpectations

This encourages us to write general expectations earlier, and specific expectations later (general -> specific).


## Ordered vs Unordered Calls

By default, an expectation can match a call even though an earlier expectation hasn’t been satisfied. In other words, the calls don’t have to occur in the order the expectations are specified.

If we require strict sequencing we can get it, example:

```
using ::testing::InSequence;
...
TEST(FooTest, DrawsLineSegment) {
  ...
  {
    InSequence seq;

    EXPECT_CALL(turtle, PenDown());
    EXPECT_CALL(turtle, Forward(100));
    EXPECT_CALL(turtle, PenUp());
  }
  Foo();
}
```


## All Expectations Are Sticky (Unless Said Otherwise)

Catchall's can be used to ignore numbers. Example:

```
using ::testing::_;
using ::testing::AnyNumber;
...
EXPECT_CALL(turtle, GoTo(_, _))  // #1
     .Times(AnyNumber());
EXPECT_CALL(turtle, GoTo(0, 0))  // #2
     .Times(2);
```

Note use of underscores at matches and Times(AnyNumber()) to essentially ignore any input that is not (0,0).

However, this function will report an error if (0,0) is passed more than twice, since we're only expecting that two times, and it will match with an expectation that is defined further down.

https://google.github.io/googletest/gmock_for_dummies.html#StickyExpectations

?

It's possible to specify that this should not happen.


## Uninteresting Calls

https://google.github.io/googletest/gmock_cook_book.html#uninteresting-vs-unexpected

A call `x.Y(...)` is uninteresting if there’s not even a single `EXPECT_CALL(x, Y(...))` set. In other words, the test isn’t interested in the `x.Y()` method at all, as evident in that the test doesn’t care to say anything about it.


## Unexpected Calls

.


