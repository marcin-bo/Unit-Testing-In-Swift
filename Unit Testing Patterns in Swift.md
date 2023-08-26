# Unit Testing Patterns in Swift

## Table Of Contents
1. [How To Test Functions That `throws`?](#throws)
1. [How To Test Optional Values?](#optional_values)
1. [How To Check That a Callback is Not Called?](#not_called)
1. [How To Test Asynchronous Callbacks](#asynchronous)
1. [References](#references)

## How To Test Functions That `throws`? <a name="throws"></a>

Mark `XCTestCase` methods as throwable to avoid `do-catch` and force `try!` in your test code.

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

## References <a name="references"></a>
- https://www.avanderlee.com/swift/unit-tests-best-practices/
