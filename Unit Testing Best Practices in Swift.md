# Unit Testing Best Practices in Swift

# Table Of Contents
1. [Follow the FIRST Principles](#first)
1. [Test only public methods](#test_public)
1. [Name the test properly](#naming)
1. [Follow the AAA structure](#unit_tests_structure)
1. [Avoid Logic in Tests](#logic_in_tests)
1. [Avoid Test Interdependence](#unit_tests_interdependence)
1. [Perform One Act Per Test Method](#one_act)
1. [Perform One Assertion Per Test Method](#one_assertion)
1. [Use Specialized Assertions](#unit_tests_assertions)
1. [Use Simple Values in Assertions](#unit_tests_best_practices_simple_values)
1. [Asserting Floating Point Numbers](#unit_tests_best_practices_floating)
1. [Avoid Magic Strings](#magic_strings)
1. [Automate Mocks Generation](#unit_tests_best_practices_sourcery)
1. [Use `setUp()` and `tearDown()` To Control Side-effects](#unit_tests_best_practices_side_effects)
    1. [Prepare and Tear Down State for a Test Class](#unit_tests_best_practices_side_effects1)
    1. [Prepare and Tear Down State for Each Test Method](#unit_tests_best_practices_side_effects2)
    1. [Tear Down State After a Specific Test Method](#unit_tests_best_practices_side_effects3)
1. [Use proper testing strategies for UI Testing](#unit_tests_best_practices_testing_ui)
1. [References](#unit_tests_best_practices_references)

# Follow the FIRST Principles <a name="first"></a>

- **F** - **Fast**: 
    - Unit tests should take little time to run. Milliseconds.
- **I** - **Isolated** (standalone): 
    - Unit tests should be run in isolation, and have no dependencies (e.g. a file system or database).
- **R** - **Repeatable** (deterministic): 
    - Unit tests should always return the same result if you don't change anything in between runs.
- **S** - **Self-validating**: 
    - Unit tests should be able to automatically detect if they passed or failed. There must be no manual check or human interaction.
- **T** - **Timely**: 
    - Unit tests shouldn't take a disproportionately long time to write compared to the code being tested. 
    - If you find testing the code taking a large amount of time compared to writing the code, consider a design that is more testable.

# Test only public methods <a name="test_public"></a>

- In most cases, there shouldn't be a need to test a private method. 
    - Private methods are **an implementation detail** and never exist in isolation.
- Instead, test **public methods**.
    - What you should care about is the end result of the public method that calls into the private one.
    - You can validate private methods by unit testing public methods.
        - At some point, there's going to be a public facing method that calls the private method as part of its implementation.


# Name the test properly <a name="naming"></a>

Name your tests properly. Use a consistent naming convention.

Example of a good template for unit tests naming convention:
```swift
func test_methodName_whenCondition_shouldExpectation {}

// OR

func test_behaviour_whenCondition_shouldExpectation {}
    
```

# Follow the Arrange-Act-Assert structure <a name="unit_tests_structure"></a>

Separate tested functionality from the setup and assertion by using *Arrange-Act-Assert* (or *Given-When-Then*):

1. **Arrange** / Given: 
- Initialize test data. 
- Initialize SUT (system under test).

2. **Act** / When: 
- Call tested function (usually on the SUT).
- Capture a result.

3. **Assert** / Then:
- Check that the result matches what is expected.

Example:
```swift
func testInvalidEmail() {
    // 1. Arrange
    let invalidEmail = "ab.com"
    let sut = EmailValidator()

    // 2. Act
    let result = sut.isValid(invalidEmail)

    // 3. Assert
    XCTAssertFalse(result) 
}
```

# Avoid Logic in Tests <a name="logic_in_tests"></a>

To minimize the risk of introducing bugs in your unit tests, it's advisable to avoid implementing complex logic within them:
- You can identify the presence of logic by assessing your use of constructs like `if`, `while`, `for`, `switch`.
- Also, be cautious with operators such as `do-catch` which can introduce branching logic.

# Avoid Test Interdependence <a name="unit_tests_interdependence"></a>

- Each test should handle its own setup and tear down. 
- Test dependency occurs when one unit test depends on the outcome of another.

# Perform One Act Per Test Method <a name="one_act"></a>

When writing your tests, try to only include one act per test. Multiple acts need to be individually Asserted.
- When the test fails, it is clear which act is failing.
- Ensures that the test is focused on just a single case.
- Gives you the entire picture as to why your tests are failing.

```swift
func test_addEmptyEntries_shouldBeTreatedAsZero() {
    // Arrange
    let sut = StringCalculator()

    // Act
    let actual1 = sut.add("")
    let actual2 = sut.add(",") // ❌

    // Assert
    XCTAssertEqual(0, actual1)
    XCTAssertEqual(0, actual2)
}

func test_addEmptyString_shouldBeTreatedAsZero() {
    // Arrange
    let sut = StringCalculator()

    // Act
    let actual = sut.add("") // ✅

    // Assert
    XCTAssertEqual(expected, actual)
}
```

# Perform One Assertion Per Test Method <a name="one_assertion"></a>

When writing your tests, try to only include one act per test.

Why?
- When the test fails, it is clear which act is failing.
- Ensures that the test is focused on just a single case.
- Gives you the entire picture as to why your tests are failing.

# Use Specialized Assertions <a name="unit_tests_assertions"></a>

Use the most specialized assertion available when there is such a choice. Example:

```swift
// Equality
XCTAssert(x == y) // ❌
XCTAssertEqual(x, y) // ✅

// Nil and Non-nil
XCTAssert(x != nil) // ❌
XCTAssertNotNil(x) // ✅
```

# Use Simple Values in Assertions <a name="unit_tests_best_practices_simple_values"></a>

For instance, when testing square root method, do not use value for which nobody knows the answer.

```swift
func testSquareRoot() {
    XCTAssertEqual(sqrt(3), 1.73205080757, accuracy: epsilon) // ❌
    
    // Better, we all know that sqrt(4) = 2
    XCTAssertEqual(sqrt(4), 2, accuracy: epsilon) // ✅
}
```

# Asserting Floating Point Numbers <a name="unit_tests_best_practices_floating"></a>

Floating point numbers should not be compared for equality. Instead, we should verify that they are almost equal by using some error bound:

```swift
let x: Float = 1.48329351
let y: Float = 1.48329351

XCTAssertEqual(x, y) // ❌

let epsilon = 0.0001
XCTAssertEqual(x, y, accuracy: epsilon) // ✅
```

# Avoid Magic Strings <a name="magic_strings"></a>

Unit tests shouldn't contain magic strings.

```swift
func test_addBigNumber_throwsException() throws {
    let sut = StringCalculator()
    
    let act = {
        try sut.add("1001") // Magic string ❌
    }
    
    XCTAssertThrowsError(try act(), "An big number error should be thrown") { error in
        XCTAssertEqual(error as? StringCalculator.Error, .bigNumber)
    }
}
```

```swift
func test_addBigNumber_throwsException() throws {
    let sut = StringCalculator()
    let MAXIMUM_RESULT = "1001"
    
    let act = {
        try sut.add(MAXIMUM_RESULT) // ✅
    }
    
    XCTAssertThrowsError(try act(), "An big number error should be thrown") { error in 
        XCTAssertEqual(error as? StringCalculator.Error, .bigNumber)
    }
}
```

# Automate Mocks Generation <a name="unit_tests_best_practices_sourcery"></a>

[Sourcery](https://github.com/krzysztofzablocki/Sourcery) is a great tool for mocking. 

For a given protocol conforming to `AutoMockable`, it generates mocks that are ready-to-use in your tests. You can check:
- Whether your functions were called.
- The number of times your functions were called.
- The parameters that were passed to the function calls.
- Invoke a closure that takes the passed function parameters.

```swift
extension MyProtocol: AutoMockable {}

class MyProtocolMock: MyProtocol {

    //MARK: - sayHelloWith
    var sayHelloWithNameCallsCount = 0
    var sayHelloWithNameCalled: Bool {
        return sayHelloWithNameCallsCount > 0
    }
    var sayHelloWithNameReceivedName: String?
    var sayHelloWithNameReceivedInvocations: [String] = []
    var sayHelloWithNameClosure: ((String) -> Void)?

    func sayHelloWith(name: String) {
        sayHelloWithNameCallsCount += 1
        sayHelloWithNameReceivedName = name
        sayHelloWithNameReceivedInvocations.append(name)
        sayHelloWithNameClosure?(name)
    }
}
```

There are other useful protocols in Sourcery, for example `AutoFixturable` which makes creating new instances of an object easier

## Use `setUp()` and `tearDown()` To Control Side-effects <a name="unit_tests_best_practices_side_effects"></a>

If a property of a `XCTestCase` class is shared between unit tests, and our test changes the property, this indirectly affects the whole test suite. To prevent this from happening we must cleanup before and after each test. In our `XCTestCase` class, override following methods:

### Prepare and Tear Down State for a Test Class <a name="unit_tests_best_practices_side_effects1"></a>
```swift
override class func setUp() {
    // This is the setUp() class method.
    // XCTest calls it before calling the first test method.
    // Set up any overall initial state here.
}
override class func tearDown() {
    // This is the tearDown() class method.
    // XCTest calls it after the last test method completes.
    // Perform any overall cleanup here.
}
```

### Prepare and Tear Down State for Each Test Method <a name="unit_tests_best_practices_side_effects2"></a>
```swift

// MARK: setUp
override func setUp() async throws {
    // This is the setUp() async instance method.
    // XCTest calls it before each test method.
    // Perform any asynchronous setup in this method.
}

override func setUpWithError() throws {
    // This is the setUpWithError() instance method.
    // XCTest calls it before each test method.
    // Set up any synchronous per-test state that might throw errors here.
}

override func setUp() {
    // This is the setUp() instance method.
    // XCTest calls it before each test method.
    // Set up any synchronous per-test state here.
}

// MARK: tearDown
override func tearDown() {
    // This is the tearDown() instance method.
    // XCTest calls it after each test method.
    // Perform any synchronous per-test cleanup here.
}

override func tearDownWithError() throws {
    // This is the tearDownWithError() instance method.
    // XCTest calls it after each test method.
    // Perform any synchronous per-test cleanup that might throw errors here.
}

override func tearDown() async throws {
    // This is the tearDown() async instance method.
    // XCTest calls it after each test method.
    // Perform any asynchronous per-test cleanup here.
}
```

### Tear Down State After a Specific Test Method <a name="unit_tests_best_practices_side_effects3"></a>
```swift
func testMethod1() throws {
    // This is the first test method.
    // Your testing code goes here.
    addTeardownBlock {
        // XCTest executes this when testMethod1() ends.
    }
}

func testMethod2() throws {
    // This is the second test method.
    // Your testing code goes here.
    addTeardownBlock {
        // XCTest executes this last when testMethod2() ends.
    }
    addTeardownBlock {
        // XCTest executes this first when testMethod2() ends.
    }
}
```

# Use proper testing strategies for UI Testing <a name="unit_tests_best_practices_testing_ui"></a>

The common strategies of testing UI:
- End-to-end tests (think `XCUI`-driven-tests)
- Snapshot tests
    - Suited for testing **visual layout**.
    - Not suited for testing *the content*.
        - Testing *the content* of our UI is straightforward and can be done on low-level with unit tests.

Both kinds of tests are expensive to maintain since the user interface changes quite often. The UI doesn’t have to be tested in an end-to-end fashion. 

Strategy for UI testing:
- Use snapshot tests for testing **visual layout**.
- Use low-level unit tests for testing *the content*

## References <a name="unit_tests_best_practices_references"></a>

- https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices
- https://www.vadimbulavin.com/unit-testing-best-practices-on-ios-with-swift/
- https://www.browserstack.com/guide/unit-testing-best-practices
- https://engineering.theblueground.com/how-we-saved-our-time-with-sourcery-in-ios/
- https://developer.apple.com/documentation/xctest/xctestcase/set_up_and_tear_down_state_in_your_tests
- https://github.com/andredesousa/javascript-unit-testing-best-practices
