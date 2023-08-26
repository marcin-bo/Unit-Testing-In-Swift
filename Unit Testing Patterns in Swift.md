# Unit Testing Patterns in Swift

## Table Of Contents
1. [How To Test Functions That `throws`?](#throws)
1. [How To Test Optional Values?](#optional_values)
1. [How To Check That a Callback is Not Called?](#not_called)

## How To Test Functions That `throws`? <a name="throws"></a>

Mark `XCTestCase` methods as throwable to avoid `do-catch` and force `try!` in your test code.

```swift
func test_addBigNumber_throwsException() {
    let sut = StringCalculator()
    let MAXIMUM_RESULT = "1001"
    
    let act = {
        try sut.add(MAXIMUM_RESULT)
    }
    
    XCTAssertThrowsError(try act(), "A big number error should be thrown") { error in // ✅
        XCTAssertEqual(error as? StringCalculator.Error, .bigNumber)
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

## How To Check That a Callback is Not Called? <a name="not_called"></a>

Use `expectation.isInverted` for to check that a callback is not called:

```swift
func test_observeQueueBecomingEmpty_whenDequeuedCalledAndQueueIsStillNotEmpty_shouldNotCallObservingHandler() {
    let sut = QueueService(queue: ["George", "Sam", "Steven"])

    let expectation = self.expectation(description: "Handler for the queue becoming empty")
    expectation.isInverted = true  // ✅

    queueService.observeQueueBecomingEmpty {
        XCTFail("The observation handler for the queue becoming empty is not triggered")
        expectation.fulfill()
    }
    
    sut.dequeue()

    wait(for: [expectation], timeout: 0.05)
}
```
- https://www.avanderlee.com/swift/unit-tests-best-practices/
