# Unit Testing View Controllers in Swift

# TODO
- Provide examples for unit testing view controllers.

# What To Unit Test in View Controllers? <a name="vc_unit_test"></a>

Appropriate aspects to test in View Controllers unit tests:
- Is `UILabel` text correct?
- Is the number of `UITableView` rows correct?
- Is `UIButton` enabled?

Inappropriate aspects to test in View Controllers unit tests:
- `UILabel` text color, font, and size.
- `UITableView` background color.
- `UIButton` autolayout constraints.

# How to Write Testable View Controllers? <a name="vc_testable"></a>

View controllers are considered testable if they are:
- **Passive**:
  - They delegate actions to appropriate dependencies, such as a presenter or a view model.
  - They focus solely on rendering the UI.
- **Isolated** from their dependencies:
  - They don't directly retrieve data from the model.
  - They aren't responsible for updating themselves based on the model.

MVVM and MVP are patterns that facilitate achieving these characteristics.

# References <a name="vc_testing_references"></a>

- <a href="https://www.vadimbulavin.com/unit-testing-view-controller-uiviewcontroller-and-uiview-in-swift/">Unit Testing View Controllers and Views in Swift</a>
