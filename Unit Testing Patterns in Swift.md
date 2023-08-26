# Unit Testing Patterns in Swift

## Table Of Contents
1. [How To Test Functions That `throws`?](#throws)
1. [How To Test Optional Values?](#optional_values)

## How To Test Functions That `throws`? <a name="throws"></a>

Mark `XCTestCase` methods as throwable to avoid `do-catch` and force `try!` in your test code.

```swift
func test_addBigNumber_throwsException() {
    let sut = StringCalculator()
    let MAXIMUM_RESULT = "1001"
    
    let act = {
        try sut.add(MAXIMUM_RESULT)
    }
    
    XCTAssertThrowsError(try act(), "An big number error should be thrown") { error in // ✅
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

## References
- https://www.avanderlee.com/swift/unit-tests-best-practices/
