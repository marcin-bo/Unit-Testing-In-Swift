# Unit Testing Patterns in Swift

## Table Of Contents
1. [How To Handle `XCTAssert*` in Helper Methods?](#file_line)
1. [How To Detect Memory Leaks in Unit Tests?](#memory_leaks)
1. [How To Test That Function Throws An Error?](#throws)
    1. [Using `do-catch`](#throws1)
    1. [Using `XCTAssertThrowsError`](#throws3)
1. [How To Test That Function Does Not Throw An Error?](#no_throw)
    1. [Using `do-catch`](#no_throw1)
    1. [Using `throws` in the test method](#no_throw2)
    1. [Using `XCTAssertNoThrowError`](#no_throw3)
1. [How To Test Optional Values?](#optional_values)
1. [How To Check That a Callback is Not Called?](#not_called)
1. [How To Test Asynchronous Callbacks?](#asynchronous)
1. [How To Test `viewDidLoad` method from `UIViewController`?](#viewDidLoad)
1. [References](#references)

## How To Handle `XCTAssert*` in Helper Methods? <a name="file_line"></a>

When calling the `XCTAssert*` functions outside the test method (e.g., in helper methods), it's crucial to provide the `#filePath` and `#line` parameters. 
This allows Xcode to precisely identify the specific test that failed and its location (the actual tested method, not a helper method).

```swift
func test_method() {
    // ... Arrange & Act
    
    // Assert: helper method
    helperExpect(param) // It should fail here ✅
}

private func helperExpect(param: Bool, file: StaticString = #filePath, line: UInt = #line) {
    //...
    XCTAssertTrue(param: Bool, file: file, line: line) // Not here ❌
}
```

## How To Detect Memory Leaks in Unit Tests? <a name="memory_leaks"></a>

If you're testing the collaboration between two classes in a parent-child relationship (for example, where the parent is the SUT and the child is a Spy), there's a risk of having a retain cycle, especially when asynchronous functions with closures are involved. To verify the presence of a retain cycle and potential memory leaks, you can follow these steps:

```swift
// A factory helper for SUT creation
private func makeSUT(file: StaticString = #filePath, line: UInt = #line) -> SomeSutType {
    // Instantiate the Spy
    let spy = ...
    
    // Instantiate the SUT 
    let sut = ...
    
    assertForMemoryLeakOnTeardown(spy, file: file, line: line) // ✅  Check the Spy instance for memory leaks
    assertForMemoryLeakOnTeardown(sut, file: file, line: line) // ✅  Check the SUT instance for memory leaks
        
    return sut
}

private func assertForMemoryLeakOnTeardown(_ object: AnyObject, file: StaticString = #filePath, line: UInt = #line) {
    // ✅  This block runs assertion when a test is finished
    addTeardownBlock { [weak object] in
        XCAssertNil(object, "The object instance has not been deallocated.", file: file, line: line)
    }
}
```

## How To Test That Function Throws An Error? <a name="throws"></a>

#### Using `do-catch`

This method adds branching logic (`do-catch`) to the test which is not desired.

```swift
func test_validate_whenPhoneNumberIsInvalid_shouldThrowException() {
    let sut = PhoneNumberValidator()
    let INVALID_NUMBER = "g122345j"
    
    do {
        try sut.validate(INVALID_NUMBER)
        
        XCTFail("The validate() was supposed to throw an error.")
    } catch PhoneNumberValidator.Error {
        // Successfully passing
        return
    } catch {
        XCTFail("The validate() was supposed to throw `PhoneNumberValidator.Error` when phone number is invalid. A different error was thrown.")
    }
}
```

#### Using `XCTAssertThrowsError`

You can also avoid `do-catch` by using `XCTAssertThrowsError`.

```swift
func test_validate_whenPhoneNumberIsInvalid_shouldThrowException() {
    let sut = PhoneNumberValidator()
    let INVALID_NUMBER = "g122345j"
    
    let act = {
        try sut.validate(INVALID_NUMBER)
    }
    
    XCTAssertThrowsError(try act(), "Invalid number error should be thrown") { error in // ✅
        XCTAssertEqual(error as? PhoneNumberValidator.Error, .invalidNumber)
    }
}
```

## How To Test That Function Does Not Throw An Error? <a name="no_throw"></a>

#### Using `do-catch`  <a name="no_throw1"></a>

```swift
func test_validate_whenPhoneNumberIsValid_shouldNotThrowException() {
    let sut = PhoneNumberValidator()
    let VALID_NUMBER = "777-777-777"
    
    do {
        let result = try sut.validate(VALID_NUMBER)
        XCTAssertTrue(result)
    } catch {
        XCTFail("The validate() was not supposed to throw an error.")
    }
}
```

### Using `throws` in the test method <a name="no_throw2"></a>

Mark `XCTestCase` methods as throwable to avoid `do-catch` in your test code.

```swift
func test_validate_whenPhoneNumberIsValid_shouldNothrowException() {
    let sut = PhoneNumberValidator()
    let VALID_NUMBER = "777-777-777"
    let result = try sut.validate(VALID_NUMBER)
    XCTAssertTrue(result)
}
```

#### Using `XCTAssertNoThrow` <a name="no_throw3"></a>

```swift
func test_validate_whenPhoneNumberIsValid_shouldNotThrowException() {
    let sut = PhoneNumberValidator()
    let VALID_NUMBER = "777-777-777"
    
    XCTAssertNoThrow(try sut.validate(VALID_NUMBER), "The validate() should not throw an error when the phone number is valid")
}
```

## How To Test Optional Values? <a name="optional_values"></a>

Use `try XCTUnwrap()` to test optional values

```swift
func testOptional() throws {
    let optionalValue: Int? = 11
    XCTAssertNotNil(unwrappedValue, optionalValue!) // ❌
}

func testOptional() throws {
    let optionalValue: Int? = 11
    let unwrappedValue = try XCTUnwrap(optionalValue) // ✅
    XCTAssertEqual(unwrappedValue, 11)
}

func test_viewDidLoad_loadsConsentScript() throws {
    // Given `SomeViewController()` which might return `nil`
    let sut = try XCTUnwrap(SomeViewController())

    // When
    sut.loadViewIfNeeded()

    /* ... */
}
```

## How To Check That a Callback is Not Called? <a name="not_called"></a>

Use `expectation.isInverted` for to check that a callback is not called:

```swift
func test_observeQueueBecomingEmpty_whenDequeuedCalledAndQueueIsStillNotEmpty_shouldNotCallObservingHandler() {
    let sut = QueueService(queue: ["George", "Sam", "Steven"])

    let expectation = expectation(description: "Handler for the queue becoming empty")
    expectation.isInverted = true // ✅

    sut.observeQueueBecomingEmpty {
        XCTFail("The observation handler for the queue becoming empty should not be triggered")
        expectation.fulfill()
    }
    
    sut.dequeue()

    waitForExpectations(timeout: 0.1)
}
```

## How To Test Asynchronous Callbacks? <a name="asynchronous"></a>

```swift
func test_fetch_shouldGetBooks() {
    let sut = BookRepository()
    
    // Create an expectation ✅
    let expectation = expectation(description: "Loading books") 

    sut.fetch {
        // Mark the expectation as fulfilled ✅
        expectation.fulfill()
    }

    // Wait for all expectations to be fulfilled ✅
    waitForExpectations(timeout: 1)

    XCTAssertFalse(sut.books.isEmpty)
}
```

## How To Test `viewDidLoad` method from `UIViewController`? <a name="viewDidLoad"></a>

Do not call the `viewDidLoad` method directly, as it might be called multiple times. Instead, call `loadViewIfNeeded` to invoke `viewDidLoad` indirectly.

```swift
func test_viewDidLoad() {
    // Given
    let sut = SomeViewController()

    // When
    sut.loadViewIfNeeded()

    // Then
    /* ... */
}
```

## References <a name="references"></a>
- https://www.avanderlee.com/swift/unit-tests-best-practices/
