# Lecture Notes: Jetpack Compose Session 2 – Standard Layouts & State Binding

**Instructor Notes – 1.5‑hour session**

---

## Introduction (3 minutes)

Welcome participants back to the second session. Briefly recap Session 1:
- We learned that Jetpack Compose is declarative; UI is built with `@Composable` functions.
- We used `Column`, `Row`, `Box` for basic layouts and `Modifier` for styling.
- We introduced state with `remember` and `mutableStateOf`, and used `MaterialTheme` for styling.

Now, we'll go deeper into **layouts** – how to use them effectively, and **state binding** – how to connect UI state with architecture components like `ViewModel`. We'll also cover efficient lists with `LazyColumn`. Navigation is postponed to a later session.

---

## Slide 2: Agenda (1 min)

Review the agenda: Standard Layouts (30 min), State Binding (40 min), Lists with LazyColumn (10 min), Q&A (5 min). Keep an eye on the clock; adjust if needed.

---

## Part 1: Standard Layouts (30 minutes)

### Slide 3: Layout Basics – Column, Row, Box (3 min)

Start by revisiting the three fundamental layout composables. Show code examples.

- **Column**: Arranges children vertically, one after another.
- **Row**: Arranges children horizontally.
- **Box**: Stacks children on top of each other (like FrameLayout).

**Live demo idea:** Open Android Studio and create a simple composable with a Column containing a Text, a Row with two Texts, and a Box with an Image and a Text overlay. Run the preview to show how they behave.

**Key point:** These three are the building blocks. Almost any layout can be built by nesting them.

---

### Slide 4: Arrangement & Alignment (5 min)

Explain the concepts of main axis and cross axis. Use a diagram if possible.

- **Main axis**: the direction in which children are placed.
  - Column: vertical (top to bottom)
  - Row: horizontal (left to right)
- **Cross axis**: perpendicular to main axis.

**Arrangement** controls spacing along the main axis. Show options:
- `Arrangement.Top`, `Bottom`, `Center`, `SpaceBetween`, `SpaceEvenly`, `SpaceAround`

**Alignment** controls placement along the cross axis:
- For Column: `Alignment.Start`, `CenterHorizontally`, `End`
- For Row: `Alignment.Top`, `CenterVertically`, `Bottom`

**Live demo:** Modify the previous example to add `verticalArrangement = Arrangement.SpaceEvenly` and `horizontalAlignment = Alignment.CenterHorizontally` to a Column. Show how children are spaced and centered.

**Common pitfall:** `Arrangement` is only for the main axis; `Alignment` for cross axis. Many beginners confuse them.

---

### Slide 5: Modifier Chaining & Order Matters (5 min)

Emphasize that modifiers are applied **in the order they appear**. Show a before/after example.

**Example 1: Padding then background**
```kotlin
Modifier
    .padding(16.dp)
    .background(Color.Blue)
```
Result: padding adds space around the composable, then the background is drawn inside that padded area.

**Example 2: Background then padding**
```kotlin
Modifier
    .background(Color.Blue)
    .padding(16.dp)
```
Result: background fills the whole area, then padding adds space, but the background remains behind the padding (so it may not be visible if the composable is small).

**Important for clickable:** If you want a larger click area, put `clickable` after padding. If you want only the content to be clickable, put it before padding.

**Live demo:** Create two Text composables with the same modifiers but in different order, and compare results. Use a background color to visualize the difference.

---

### Slide 6: Layout Scoping – `Modifier.align` and `Modifier.weight` (5 min)

Explain that some modifiers are only available inside certain layout scopes.

- **`align`** inside `Box`:
  ```kotlin
  Box {
      Text("TopStart", Modifier.align(Alignment.TopStart))
      Text("Center", Modifier.align(Alignment.Center))
  }
  ```
  This places the text relative to the Box's bounds.

- **`weight`** inside `Column` or `Row`:
  ```kotlin
  Row {
      Text("Fixed width", Modifier.weight(0f)) // takes only its content width
      Text("Expands", Modifier.weight(1f))     // takes all remaining space
  }
  ```
  Weight distributes leftover space proportionally. A weight of 0 means the child is not flexible; higher weight gets more space.

**Live demo:** Show a Row with two Texts, one with weight 1f and one with weight 2f, and see how the space is split. Then show how adding `fillMaxWidth()` interacts with weight.

---

### Slide 7: Common Modifiers for Layout (3 min)

Quickly run through the most common layout modifiers:

- **Size**: `size(48.dp)`, `width(100.dp)`, `height(50.dp)`, `fillMaxSize()`, `fillMaxWidth()`, `fillMaxHeight()`
- **Padding**: `padding(16.dp)`, `padding(horizontal = 8.dp)`, `padding(start = 4.dp)`
- **Offset**: `offset(x = 8.dp, y = 8.dp)` – shifts without affecting layout constraints.
- **Constraints**: `requiredSize()`, `constrainAs()` (for ConstraintLayout).

**Tip:** `fillMaxSize()` will make the composable take all available space from the parent, but if the parent has no constraints (e.g., in a Box), it may not work as expected. Always ensure the parent provides constraints.

---

### Slide 8: ConstraintLayout (3 min)

Introduce ConstraintLayout for complex layouts with many relationships.

- Add dependency: `androidx.constraintlayout:constraintlayout-compose:1.0.1`
- Use `ConstraintLayout` composable, create references with `createRefs()`, and constrain using `Modifier.constrainAs`.

**Example:**
```kotlin
ConstraintLayout {
    val (button, text) = createRefs()
    Button(
        onClick = { },
        modifier = Modifier.constrainAs(button) {
            top.linkTo(parent.top)
            start.linkTo(parent.start)
        }
    ) { Text("Button") }
    Text(
        "Text",
        modifier = Modifier.constrainAs(text) {
            top.linkTo(button.bottom, margin = 8.dp)
            start.linkTo(button.start)
        }
    )
}
```

Explain when to use ConstraintLayout: when you have a complex UI that would require many nested layouts otherwise. It can improve performance because it flattens the hierarchy.

---

### Slide 9: Custom Layouts with the `Layout` Composable (5 min)

Now we go beyond standard layouts. Sometimes you need full control over measurement and placement.

**Concept:** The `Layout` composable gives you access to the list of children (`measurables`) and the constraints from the parent. You then:
1. Measure each child.
2. Decide the container size.
3. Place each child at specific coordinates.

**Show the basic structure:**
```kotlin
Layout(
    content = { /* children */ },
    modifier = modifier
) { measurables, constraints ->
    // Measure
    val placeables = measurables.map { measurable ->
        measurable.measure(constraints)
    }
    // Decide size
    val width = placeables.maxOfOrNull { it.width } ?: 0
    val height = placeables.sumOf { it.height }
    // Layout
    layout(width, height) {
        var y = 0
        placeables.forEach { placeable ->
            placeable.place(0, y)
            y += placeable.height
        }
    }
}
```

**Explain the steps:**
- `measurable.measure(constraints)` returns a `Placeable` that knows its measured size.
- `layout(width, height)` sets the final size of the custom container.
- Inside the layout lambda, you call `placeable.place(x, y)` for each child.

**Live demo idea:** Create a simple `VerticalFlow` that arranges items vertically, but if they exceed a certain width, they wrap to a new row. This demonstrates measuring children and placing them conditionally. You can start with a fixed-width container and show how children wrap.

---

### Slide 10: Custom Layout Step-by-Step (3 min)

Reiterate the four steps with a concrete example: a simple stack that places children in a diagonal.

**Code snippet:**
```kotlin
Layout(
    content = content,
    modifier = modifier
) { measurables, constraints ->
    val placeables = measurables.map { it.measure(constraints) }
    var x = 0
    var y = 0
    layout(constraints.maxWidth, constraints.maxHeight) {
        placeables.forEach { placeable ->
            placeable.place(x, y)
            x += placeable.width
            y += placeable.height
        }
    }
}
```

This places children in a diagonal line. You can use this to illustrate how to control coordinates.

**Emphasize:** Custom layouts are powerful but should be used only when standard layouts are insufficient. They require careful handling of constraints to avoid breaking the layout system.

---

### Slide 11: Intrinsic Measurements – When and Why (3 min)

Explain the concept of intrinsic measurements: sometimes a parent needs to know the "natural" size of a child before it can decide its own size.

- **Intrinsic width/height** are the dimensions the child would like to have given no constraints.
- Compose uses intrinsic measurements to handle cases like `Column` with `Modifier.fillMaxWidth()` children – it first asks each child for its intrinsic width to determine the column's width.
- For custom layouts, you can provide intrinsic measurements by implementing `IntrinsicMeasurable`. This is rarely needed but can improve performance.

**Example use case:** A custom `FlowLayout` that wants to know the maximum intrinsic width of all children to decide whether to wrap.

**Tip:** For most custom layouts, you can ignore intrinsic measurements and just measure with the constraints you have. The system will work, but it might be less efficient.

---

### Slide 12: Summary of Standard Layouts (3 min)

Briefly recap the layout section:
- Use `Column`, `Row`, `Box` for most cases.
- `Arrangement` and `Alignment` control spacing and positioning.
- Modifier order matters – layout modifiers first, then drawing modifiers.
- `weight` in Row/Column distributes remaining space.
- `ConstraintLayout` for complex constraints.
- Custom `Layout` for full control.
- Intrinsic measurements are advanced but important for performance.

**Transition:** Now that we can arrange UI, let's see how to connect it to data and state effectively.

---

## Part 2: State Binding (40 minutes)

### Slide 13: State Hoisting – What and Why (5 min)

Start with a problem: a composable that manages its own state is hard to reuse and test. Example: a counter button that holds its own count.

**Solution: State hoisting** – move the state up to the caller and pass it down as a parameter.

**Pattern:**
```kotlin
@Composable
fun CounterDisplay(count: Int, onIncrement: () -> Unit) {
    Button(onClick = onIncrement) {
        Text("Count: $count")
    }
}
```
Now `CounterDisplay` is stateless and can be used with any count source.

**Benefits:**
- **Reusability** – the same UI can show different counts.
- **Testability** – you can test it by passing fake state and verifying the event is called.
- **Single source of truth** – state lives in one place (e.g., ViewModel), not scattered.

**Live demo:** Show a simple app with a `CounterScreen` that manages the state and passes it to `CounterDisplay`. Then modify `CounterDisplay` to show how it can be reused.

---

### Slide 14: Unidirectional Data Flow (UDF) (3 min)

Explain the concept:
- **State flows down** from the ViewModel (or parent composable) to UI.
- **Events flow up** from UI to the ViewModel (or parent) to update state.
- This creates a predictable cycle: UI → Event → State Update → UI recompose.

**Diagram:** Draw on whiteboard or show a slide with arrows.

**Why it matters:** UDF makes the app easier to reason about. You always know where state comes from and how it changes.

---

### Slide 15: Integrating ViewModel with Compose (8 min)

Now show how to use Jetpack ViewModel to hold and expose state.

**Option 1: Using `mutableStateOf` inside ViewModel**
```kotlin
class CounterViewModel : ViewModel() {
    private var _count = mutableStateOf(0)
    val count: State<Int> = _count
    fun increment() { _count.value++ }
}

@Composable
fun CounterScreen(viewModel: CounterViewModel = viewModel()) {
    CounterDisplay(
        count = viewModel.count.value,
        onIncrement = { viewModel.increment() }
    )
}
```
- `viewModel()` obtains the ViewModel (scoped to the lifecycle of the screen).
- The UI reads `viewModel.count.value` and triggers recomposition when it changes.

**Option 2: Using StateFlow**
```kotlin
class CounterViewModel : ViewModel() {
    private val _count = MutableStateFlow(0)
    val count: StateFlow<Int> = _count
    fun increment() { _count.value++ }
}

@Composable
fun CounterScreen(viewModel: CounterViewModel = viewModel()) {
    val count by viewModel.count.collectAsState()
    CounterDisplay(count, viewModel::increment)
}
```
- `collectAsState()` converts the Flow to Compose State.
- This is the recommended way for production apps.

**Option 3: Using LiveData** (brief mention)
- `observeAsState()` works similarly.

**Live demo:** Build a simple counter using ViewModel with StateFlow. Show that the counter survives configuration changes (rotate screen). Emphasize that this is the main advantage of ViewModel.

---

### Slide 16: `rememberSaveable` – Surviving Process Death (5 min)

`remember` survives recomposition but not process death (when the system kills the app to reclaim memory). `rememberSaveable` uses the Bundle mechanism to save state across process death.

**Example:**
```kotlin
var text by rememberSaveable { mutableStateOf("") }
TextField(value = text, onValueChange = { text = it })
```
When the system kills the app and restores it, the text will still be there.

**Custom Savers:** For complex objects, you can define a `Saver`. Show a simple example:
```kotlin
val mySaver = run {
    val key = "myKey"
    Saver<MyData, Bundle>(
        save = { bundleOf(key to it.someValue) },
        restore = { bundle -> MyData(bundle.getInt(key)) }
    )
}
var data by rememberSaveable(stateSaver = mySaver) { mutableStateOf(MyData(0)) }
```
But usually, you'll put such data in ViewModel and use `rememberSaveable` only for UI‑only transient state.

---

### Slide 17: `derivedStateOf` – Reducing Unnecessary Recomposition (5 min)

Sometimes you have a derived value that depends on other states. If you compute it directly, every change to the source state will cause recomposition of any composable that reads the derived value – even if the derived value didn't change.

**Example:**
```kotlin
val count by remember { mutableStateOf(0) }
val isEven = count % 2 == 0  // recomputes on every count change
```
If `isEven` is used in a composable, that composable will recompose on every count change, even when `isEven` stays the same (e.g., from 1 to 2 changes parity, but from 2 to 3 changes again – it changes half the time). But that's not the worst; `derivedStateOf` is more about optimizing when the derived value doesn't change often.

**Better:**
```kotlin
val isEven by remember {
    derivedStateOf { count % 2 == 0 }
}
```
Now `isEven` only changes when the parity changes. Any composable that reads `isEven` will only recompose when parity flips, not on every count increment.

**Live demo:** Add a `Text` that shows "Even" or "Odd" using `derivedStateOf`. Log recompositions to show the difference.

---

### Slide 18: Side Effects – Brief Mention (Optional, 2 min)

If time permits, introduce side effects briefly. State updates trigger recomposition, but sometimes you need to run one‑time actions (e.g., showing a toast, navigating). For that, we have side‑effect APIs:

- **`LaunchedEffect`**: runs a coroutine when the key changes. Useful for one‑time tasks like loading data.
- **`DisposableEffect`**: runs a cleanup when the composable leaves the composition.
- **`SideEffect`**: runs on every successful recomposition.

**Example:**
```kotlin
LaunchedEffect(Unit) {
    // runs once when composable enters composition
    loadData()
}
```
Mention that side effects are important for handling events that shouldn't be part of the UI state.

---

## Part 3: Lists with LazyColumn (10 minutes)

### Slide 19: Efficient Lists – `LazyColumn` and `LazyRow` (4 min)

Explain the problem: using `Column` for many items is inefficient because it composes all items at once. `LazyColumn` composes only the visible items, like RecyclerView.

**Basic usage:**
```kotlin
LazyColumn {
    items(100) { index ->
        Text("Item #$index")
    }
}
```
- `items(count)` is a convenience function for simple lists.
- For a list of data: `items(myList) { item -> ... }`.

**Keys:** Provide a stable key to help Compose animate and preserve state:
```kotlin
items(items = myList, key = { it.id }) { item -> ... }
```

---

### Slide 20: Advanced LazyList Features (4 min)

- **Sticky headers:** Use `stickyHeader { }` inside `LazyColumn`.
- **Different item types:** Use `items` with `contentType` parameter to help Compose reuse layouts.
- **Scroll state:** `val listState = rememberLazyListState()` – use it to read or control scroll position.
  ```kotlin
  LazyColumn(state = listState) { ... }
  // Programmatic scroll:
  LaunchedEffect(Unit) {
      listState.scrollToItem(0)
  }
  ```
- **Pagination:** You can observe scroll position and load more when near the end.

**Live demo:** Show a `LazyColumn` with 100 items. Add a button to scroll to top using `listState.scrollToItem(0)`.

---

### Slide 21: Best Practices for State Binding (2 min)

Summarize key points:
- Hoist state to make composables reusable and testable.
- Follow unidirectional data flow.
- Use ViewModel for state that survives configuration changes.
- Use `rememberSaveable` for UI state that must survive process death.
- Use `derivedStateOf` to minimize recompositions.
- Side effects go in `LaunchedEffect`, `DisposableEffect`, etc.

---

## Part 4: Q&A & Wrap-up (5 minutes)

### Slide 22: Q&A & Resources

Open the floor for questions. If time is short, ask participants to write down questions.

**Resources:**
- [Compose Layout Documentation](https://developer.android.com/jetpack/compose/layout)
- [State in Compose](https://developer.android.com/jetpack/compose/state)
- [ViewModel in Compose](https://developer.android.com/jetpack/compose/libraries#viewmodel)
- [Lazy lists](https://developer.android.com/jetpack/compose/lists)
