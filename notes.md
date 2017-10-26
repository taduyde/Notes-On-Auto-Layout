# Auto Layout 
## Understanding Auto Layout
- - - -
* 3 approaches to laying out a UI:
	1. Programmatically - Total control, but very complex
	2. Autoresizing masks - Assists with programmatic layout
	3. Auto Layout - New paradigm focused on constraints
* **Auto Layout** - system which calculates the size and location of views based on provided constraints
*  **External changes** occur when the size or shape of your superview changes.
* **Internal changes** occur when the size of the views or controls in your user interface change.
* Auto Layout dynamically responds to both internal and external changes
* 2 steps to mastering Auto Layout:
	1.  Understand the logic behind constraint-based layouts
	2. Learn the API


## Auto Layout Without Constraints 
- - - -
* Stack views provide an easy way to leverage the power of Auto Layout without introducing the complexity of constraints
* A single stack view defines a row or column of user interface elements.
* If an object has an intrinsic content size, it appears in the stack at that size.
* If it does not have an intrinsic content size, you can modify the layout by adding constraints directly to the arranged views
* As a general rule of thumb, if a view’s size defaults back to its intrinsic content size for a given dimension, you can safely add a constraint for that dimension.
* The stack view also bases its layout on the arranged views’ content-hugging and compression-resistance priorities.
* Additionally, you can nest stack views inside other stack views to build more complex layouts.
* In general, use stack views to manage as much of your layout as possible. 


## Anatomy of a Constraint
- - - -
* The layout of your view hierarchy is defined as a series of linear equations.
* Your goal is to declare a series of equations that has one and only one possible solution.

* The form of an Auto Layout equation:
	* (Item.Attribute) (Relationship) (Multiplier) x ((Optional) Item.Attribute) + (Constant)
	* Examples:
  
	`view1.leading = 1.0 x view2.trailing + 10.0`
  
	`view3.height = 0.0 x NotAnAttribute + 50.0`
  
	`view5.width = 2.0 x view5.height + 0.0`
* An **item** must be a view or layout guide
* An **attribute** is the thing being constrained
* A **relationship** can be =, >=, or <=
* A **constant** is a floating-point number representing some offset 
* When setting an attribute to a constant:
	* The multiplier on the rhs should be 0.0
	* The item on the rhs should be omitted
	* The attribute on the rhs should be set to **NotAnAttribute**

- - - -
### Auto Layout Attributes

* In Auto Layout, the attributes define a feature that can be constrained
* There are two basic types of attributes:
	1. Size attributes (Height, Width, …)
	2. Location attributes (Leading, Left, Top, …)
* The following rules apply:
	* You cannot constrain a size attribute to a location attribute.
  
	`view1.height = 1.0 x view2.trailing + 0.0`
	* You cannot assign constant values to location attributes.
  
	`view.leading = 0.0 x NotAnAttribute  + 20`
	* You cannot use a nonidentity multiplier (a value other than 1.0) with location attributes.
  
	`view1.leading = 2.0 x view2.trailing + 0.0`
	* For location attributes, you cannot constrain vertical attributes to horizontal attributes.
	
  `view1.top = 1.0 x view2.leading + 0.0`
	* For location attributes, you cannot constrain Leading or Trailing attributes to Left or Right attributes.
	
  `view1.leading = 1.0 x view2.left + 0.0`

> Avoid using Left and Right attributes. Use Leading and Trailing instead. This allows the layout to adapt to the view’s reading direction. You can always adjust how the view interprets its leading and trailing edges using its `semanticContentAttribute` property  

- - - -
### Choosing a Solution
> Auto Layout frequently provides multiple ways to solve the same problem. Ideally, you should choose the solution that most clearly describes your **intent**. However, different developers will undoubtedly disagree about which solution is best. Here, _being consistent is better than being right_.   

Some good rules of thumb:
* Whole number multipliers are favored over fractional multipliers.
* Positive constants are favored over negative constants.
* Wherever possible, views should appear in layout order: leading to trailing, top to bottom.

- - - -
### Creating Non-ambiguous, Satisfiable Layouts
* **Ambiguous** constraints have more than one possible solution.
* **Unsatisfiable** constraints don’t have valid solutions.

* **Not all** non-ambiguous, satisfiable layouts are equally useful.
* Design layouts that dynamically adapt to their environment
* You should avoid assigning constant sizes to views.
* Sometimes it is virtually impossible to objectively prove that one approach is strictly superior to another.

- - - -
### Constraint Inequalities
* Constraints can represent inequalities as well
	* example setting minimum and maximum width on a view
  
	`View.width >= 0.0 * NotAnAttribute + 40.0`
  
	`View.width <= 0.0 * NotAnAttribute + 280.0`

- - - -
### Constraint Priorities
* By default, all constraints are required (priority of 1000)
* All other constraints (priority < 1000) are optional
* Auto Layout attempts to satisfy all the constraints in priority order from highest to lowest.
	* If it cannot satisfy an optional constraint, that constraint is skipped

> Even if an optional constraint cannot be satisfied, it can still influence the layout. If there is any ambiguity in the layout after skipping the constraint, the system selects the solution that comes closest to the constraint. In this way, unsatisfied optional constraints act as a force pulling views towards them.  

Optional constraints and inequalities often work hand-in-hand:

*  Example

`Blue.leading >= 1.0 * Red.trailing + 8.0`

`Blue.leading <= 1.0 * Red.trailing + 8.0`
> The greater-than-or-equal relationship could be required , and the less-than-or-equal relationship could be a lower priority. This means that the blue view cannot be closer than 8.0 points from the red. However, other constraints could pull it farther away. Still, the optional constraint pulls the blue view towards the red view, ensuring that it is as close as possible to the 8.0-point spacing, given the other constraints in the layout.  

- - - -
### Intrinsic Content Size
* Some views have a natural size given their current content
* `IntrinsicHeight` and `IntrinsicWidth` constants represent the height and width values from the view’s intrinsic content size
* **Content hugging** pulls the view inward so that it fits snugly around the content.
* **Compression resistance** pushes the view outward so that it does not clip the content.

```
// Compression Resistance
View.height >= 0.0 * NotAnAttribute + IntrinsicHeight
View.width >= 0.0 * NotAnAttribute + IntrinsicWidth

// Content Hugging
View.height <= 0.0 * NotAnAttribute + IntrinsicHeight
View.width <= 0.0 * NotAnAttribute + IntrinsicWidth
```

> Each of these constraints can have its own priority. By default, views use a 250 priority for their content hugging, and a 750 priority for their compression resistance. Therefore, it’s easier to stretch a view than it is to shrink it.  

* Whenever possible, use the view’s intrinsic content size in your layout.
* Avoid giving views required CHCR priorities.


## Programmatically Creating Constraints
- - - -
* You have three choices when it comes to programmatically creating constraints: 
	1. Layout anchors
	2. NSLayoutConstraint
	3. Visual Format Language

- - - -
### Layout Anchors
* Let you create constraints in an easy-to-read, compact format

`myView.leadingAnchor.constraint(equalTo: margins.leadingAnchor).isActive = true`
`myView.heightAnchor.constraint(equalTo: myView.widthAnchor, multiplier: 2.0).isActive = true`

* Provides additional type safety
* Helps prevent the accidental creation of invalid constraints

- - - -
### NSLayoutConstraint
* Explicitly converts the constraint equation into code

`NSLayoutConstraint(item: myView, attribute: .leading, relatedBy: .equal, toItem: view, attribute: .leadingMargin, multiplier: 1.0, constant: 0.0).isActive = true`

> As seen in this example, you must specify a value for each parameter, even if it doesn’t affect the layout. This results in a fair amount of boilerplate code and makes the constraint harder to read than the layout anchor API. Overall, this approach is inferior to using anchors.   

- - - -
### Visual Format Language
* Lets you use ASCII-art like strings to define your constraints

```
let views = ["myView" : myView]
let formatString = "|-[myView]-|"

let constraints = NSLayoutConstraint.constraints(withVisualFormat: formatString, options: .alignAllTop, metrics: nil, views: views)

NSLayoutConstraint.activate(constraints)
```

* Although it helps to visualize the constraints, the visual format language can’t represent all constraints and doesn’t provide compile time validation like anchors do
