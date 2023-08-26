# Unit Testing Terminology

# Table Of Contents

1. [Different Types of Test Doubles](#test_doubles)
    1. [Dummy](#dummy)
    1. [Stub](#stub)
    1. [Fake](#fake)
    1. [Spy](#spy)
    1. [Mock](#mock)
1. [References](#references)

# Different Types of Test Doubles <a name="test_doubles"></a>

- The classification of Mocks, Fakes, and Stubs can be inconsistent across various sources.
- Adding to the complexity, the term 'Mock' is often used colloquially for various types of test doubles.
- Moreover, it's not uncommon for some doubles to fulfill multiple roles, acting as both Mocks and Stubs simultaneously.

We can categorize the five types of doubles as follows:
- Those that don’t simulate behavior or observe interactions: Dummies.
- Those that simulate behavior: Stubs and Fakes.
- Those that observe interactions: Mocks and Spies.
- Those that don't have a functional implementation under the hood: Dummies, Stubs, and Mocks.
- Those that have a functional implementation under the hoods: Fakes and Spies.

## Dummy <a name="dummy"></a>

- Its purpose is to "satisfy a signature".
- Usually, this as a placeholder for an input parameter of the system under test.
    - It doesn't do anything. No interactions are performed on it.
    - It doesn't affect the behaviour you are testing.
- Dummy can be something as simple as passing `nil` or a void implementation with exceptions to ensure it's never leveraged.

```swift
protocol Repository {
    func save(data: SomeData)
}

struct DummyRepository: Repository {
    func save(data: SomeData) {
        // Does nothing
    }
}
```
<img src="images/dummy_usage.jpg" width="400" />

## Stub <a name="stub"></a>

- **Stubs** are objects that **return predefined values**:
    - Functions return hard-coded values. It always returns the same output regardless of the input.
    - Void functions are empty.
- They don't have working implementations.
- Examples: 
    - A Stub that always return the same value when called with any arguments.
    - A Stub that always throw same error when the function is called.

<img src="images/stub_usage.jpg" width="400" />

## Fake <a name="fake"></a>

- **Fakes** are objects with **simplified, lightweight and concrete implementation**.
- It eliminates heavyweight dependencies.
- It should be used sparingly and only in cases where a Stub or Mock is not usable.
- Example: using in-memory storage as a replacement for database storage.

<img src="images/fake_usage.jpg" width="400" />

## Spy <a name="spy"></a>

- **Spies** are objects that **replace or extend the concrete implementation of a dependency**.
    - It can be done by overriding some methods to record relevant information for the test verification.
- Lets us verify what functions were called, with what arguments, when, and how often.
    - Spies have some with some recording capability.
- We should use Spies where we don't care about the return values of functions.

<img src="images/spy_usage.jpg" width="400" />


## Mock <a name="mock"></a>

- **Mocks** are objects that **have predefined behavior and expectations.**.
    - Can act as Stubs, but the behavior of a Mock can be changed dynamically (can return dynamic value).
    - Can also act a Spy as it, for example, allows us to verify that a method was called.
- Mocks are generally used to test the behavior of our code rather than its output.
    -  We can use mocks to verify that our code is calling the dependencies in an expected way.
- Unlike Fakes, mocks don’t have working implementations.
- Example:
    - A mock object might be programmed to return a specific value when it is called with certain arguments. 

<img src="images/mock_usage.jpg" width="400" />


# References <a name="references"></a>

- https://www.youtube.com/watch?v=BNXPRIscfQ8
- https://www.ankursheel.com/blog/difference-stubs-mocks-fakes-spies
- https://mokacoding.com/blog/swift-test-doubles/
- https://proandroiddev.com/the-definitive-guide-to-test-doubles-on-android-part-1-theory-5aa2bffb568c
- https://www.educative.io/answers/what-is-faking-vs-mocking-vs-stubbing
- http://xunitpatterns.com/Test%20Double.html