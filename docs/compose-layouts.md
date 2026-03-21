# Lecture Notes: Jetpack Compose Session 2 – Standard Layouts & State Binding

**Instructor Notes – 1.5‑hour session**

---
We'll restructure Session 2 to re‑include state hoisting and ViewModel before moving to lists. We'll keep the layout deep dive, then cover state hoisting and ViewModel integration, then essential Material components, then LazyColumn, and finish with best practices. The total slides will be around 20–22, fitting the 1.5‑hour timeframe.

---

## Updated Outline: Session 2 – Standard Layouts, State Binding & Essential Material Components

**Total time: 1 hour 30 minutes**

### Part 1: Standard Layouts (30 min)
1. Recap & Agenda  
2. Column, Row, Box Revisited  
3. Arrangement & Alignment in Depth  
4. Modifier Chaining – Order Matters  
5. Weight & Align Scoped Modifiers  
6. ConstraintLayout for Complex Layouts  
7. Custom Layouts with `Layout` Composable  
8. Intrinsic Measurements – Why They Matter  
9. Layout Summary  

### Part 2: State Hoisting & ViewModel (20 min)
10. State Hoisting – Making Composables Reusable  
11. Unidirectional Data Flow (UDF)  
12. Integrating ViewModel with Compose  
13. `rememberSaveable` – Surviving Process Death  
14. Derived State with `derivedStateOf`  

### Part 3: Essential Material Components (15 min)
15. Scaffold – Basic App Structure  
16. Snackbar – Temporary Messages  
17. AlertDialog – Modal Dialogs  
18. Material Theming Recap & Component Customization  

### Part 4: Lists with LazyColumn (15 min)
19. Efficient Lists – LazyColumn / LazyRow  
20. Keys, Scroll State, Sticky Headers  
21. LazyColumn with ViewModel (Quick Demo)  

### Part 5: Best Practices & Wrap-up (10 min)
22. Layout & Component Best Practices  
23. Q&A and Resources  

---

## Lecture Notes – Session 2 (Final)

**Instructor Notes – 1.5 hours**

---

### Introduction (2 min)

Welcome back. Today we'll deepen our understanding of layouts – how to arrange UI precisely – then learn how to manage state effectively using state hoisting and ViewModel. After that, we'll cover essential Material components like `Scaffold`, `Snackbar`, and `AlertDialog`. Finally, we'll tackle efficient lists with `LazyColumn`. Let's get started.

---

## Part 1: Standard Layouts (30 min)

### Slide 1: Recap & Agenda (2 min)

Quick recap of composables, modifiers, basic state. Agenda: layouts deep dive, state hoisting & ViewModel, essential Material components, lists.

---

### Slide 2: Column, Row, Box Revisited (3 min)

Show simple examples of each. Emphasize these are the building blocks.  
**Live demo:** Create a `Column` with a `Text`, a `Row` with two `Text`s, and a `Box` with an `Image` and overlay `Text`. Run preview.

---

### Slide 3: Arrangement & Alignment in Depth (5 min)

- **Main axis vs cross axis** – diagram.
- `verticalArrangement` / `horizontalArrangement` options: `Top`, `Center`, `Bottom`, `SpaceBetween`, `SpaceEvenly`, `SpaceAround`.
- `horizontalAlignment` / `verticalAlignment`.
- **Live demo:** Modify `Column` with `verticalArrangement = Arrangement.SpaceEvenly` and `horizontalAlignment = Alignment.CenterHorizontally`.

---

### Slide 4: Modifier Chaining – Order Matters (5 min)

Modifiers applied in sequence. Show:
- `.padding(16.dp).background(Color.Blue)` – background inside padding.
- `.background(Color.Blue).padding(16.dp)` – background under padding.
**Live demo:** Two `Text`s with swapped order, using background colors.

---

### Slide 5: Weight & Align Scoped Modifiers (5 min)

- **`weight`** inside `Row`/`Column` – distributes leftover space.
  ```kotlin
  Row {
      Text("Fixed", Modifier.weight(0f))
      Text("Expands", Modifier.weight(1f))
  }
  ```
- **`align`** inside `Box` – positions child.
**Live demo:** `Row` with weights; `Box` with two texts using `align`.

---

### Slide 6: ConstraintLayout for Complex Layouts (5 min)

- Add dependency: `implementation("androidx.constraintlayout:constraintlayout-compose:1.0.1")`
- Basic syntax: `ConstraintLayout { ... }` with `createRefs()` and `constrainAs`.
- Show a simple form layout. Explain when to use: flatten deep nesting.

---

### Slide 7: Custom Layouts with `Layout` Composable (5 min)

- Why custom layout: when standard containers aren't enough.
- Break down `Layout`:
  - `measurables` – children to measure.
  - `constraints` – min/max constraints.
  - Steps: measure → calculate container size → place.
- **Live demo:** Write a `VerticalStack` custom layout.

---

### Slide 8: Intrinsic Measurements – Why They Matter (3 min)

- Some layouts need a child's "natural" size before final measurement (e.g., `Column` with `fillMaxWidth` children).
- Built‑in composables provide intrinsic measurements; custom layouts can optionally implement them.

---

### Slide 9: Layout Summary (2 min)

Recap: standard layouts, weight, align, ConstraintLayout, custom `Layout` for special needs.

---

## Part 2: State Hoisting & ViewModel (20 min)

### Slide 10: State Hoisting – Making Composables Reusable (5 min)

- Problem: a composable that holds its own state is hard to reuse and test.
- **Hoisting** = moving state up to the caller, making the composable stateless.
- Pattern: composable accepts `state` and an `event` lambda.
  ```kotlin
  @Composable
  fun CounterDisplay(count: Int, onIncrement: () -> Unit) {
      Button(onClick = onIncrement) {
          Text("Count: $count")
      }
  }
  ```
- Benefits: reusability, testability, single source of truth.

---

### Slide 11: Unidirectional Data Flow (UDF) (3 min)

- **State flows down, events flow up.**
- Diagram: UI → Event → State Update → UI recompose.
- This creates a predictable cycle, making the app easier to reason about.

---

### Slide 12: Integrating ViewModel with Compose (7 min)

- `ViewModel` holds UI state, survives configuration changes.
- Obtain ViewModel in composable with `viewModel()`.
- Option 1: Using `mutableStateOf` inside ViewModel:
  ```kotlin
  class CounterViewModel : ViewModel() {
      private var _count = mutableStateOf(0)
      val count: State<Int> = _count
      fun increment() { _count.value++ }
  }
  // In composable:
  val viewModel: CounterViewModel = viewModel()
  CounterDisplay(
      count = viewModel.count.value,
      onIncrement = { viewModel.increment() }
  )
  ```
- Option 2 (recommended): Using `StateFlow`:
  ```kotlin
  class CounterViewModel : ViewModel() {
      private val _count = MutableStateFlow(0)
      val count: StateFlow<Int> = _count
      fun increment() { _count.value++ }
  }
  // In composable:
  val count by viewModel.count.collectAsState()
  ```
- **Live demo:** Build a simple counter with ViewModel and `collectAsState()`.

---

### Slide 13: `rememberSaveable` – Surviving Process Death (3 min)

- `remember` survives recomposition but not process death.
- `rememberSaveable` uses Bundle to save state across process death.
- Example:
  ```kotlin
  var text by rememberSaveable { mutableStateOf("") }
  TextField(value = text, onValueChange = { text = it })
  ```
- Useful for UI‑only transient state like text input, scroll position.

---

### Slide 14: Derived State with `derivedStateOf` (2 min)

- When a derived value depends on other state, use `derivedStateOf` to avoid unnecessary recomposition.
- Example:
  ```kotlin
  val count by remember { mutableStateOf(0) }
  val isEven by remember { derivedStateOf { count % 2 == 0 } }
  ```
- Now `isEven` only triggers recomposition when the parity changes.

---

## Part 3: Essential Material Components (15 min)

### Slide 15: Scaffold – Basic App Structure (3 min)

- `Scaffold` provides standard app layout with slots.
- Minimal usage:
  ```kotlin
  Scaffold(
      topBar = { TopAppBar(title = { Text("My App") }) }
  ) { innerPadding ->
      Box(modifier = Modifier.padding(innerPadding)) {
          // content
      }
  }
  ```
- Explain `innerPadding` – automatic padding to avoid system bars.

---

### Slide 16: Snackbar – Temporary Messages (5 min)

- `Snackbar` is used for brief messages, often with an action.
- Use with `ScaffoldState`:
  1. `val scaffoldState = rememberScaffoldState()`
  2. Pass to `Scaffold`
  3. Show snackbar via `scaffoldState.snackbarHostState.showSnackbar()`
- Example:
  ```kotlin
  Scaffold(
      scaffoldState = scaffoldState,
      // ...
  ) { innerPadding ->
      Button(onClick = {
          scaffoldState.snackbarHostState.showSnackbar("Message")
      }) { Text("Show") }
  }
  ```
- Parameters: `message`, `actionLabel`, `duration`.

---

### Slide 17: AlertDialog – Modal Dialogs (5 min)

- Controlled by a boolean state.
- Structure:
  ```kotlin
  var showDialog by remember { mutableStateOf(false) }
  if (showDialog) {
      AlertDialog(
          onDismissRequest = { showDialog = false },
          title = { Text("Confirm") },
          text = { Text("Are you sure?") },
          confirmButton = { TextButton(onClick = { /* action */ }) { Text("Yes") } },
          dismissButton = { TextButton(onClick = { showDialog = false }) { Text("No") } }
      )
  }
  ```
- Explain `onDismissRequest`, button slots.

---

### Slide 18: Material Theming Recap & Component Customization (2 min)

- Quick recap: `MaterialTheme` provides colors, typography, shapes.
- Customizing components:
  ```kotlin
  Button(
      onClick = { },
      colors = ButtonDefaults.buttonColors(
          containerColor = MaterialTheme.colorScheme.secondary
      )
  ) { Text("Custom") }
  ```

---

## Part 4: Lists with LazyColumn (15 min)

### Slide 19: Efficient Lists – LazyColumn / LazyRow (5 min)

- `LazyColumn` composes only visible items.
- Basic usage:
  ```kotlin
  LazyColumn {
      items(100) { index -> Text("Item #$index") }
  }
  ```
- For custom data: `items(myList) { item -> ... }`

---

### Slide 20: Keys, Scroll State, Sticky Headers (5 min)

- **Keys** for stable identity: `items(items, key = { it.id })`.
- **Scroll state**: `val listState = rememberLazyListState()`
  ```kotlin
  LazyColumn(state = listState) { ... }
  Button(onClick = { listState.scrollToItem(0) }) { Text("Top") }
  ```
- **Sticky headers**: `stickyHeader { ... }` inside `LazyColumn`.

---

### Slide 21: LazyColumn with ViewModel (5 min)

- ViewModel holds a list of items.
- Observe with `collectAsState()`.
- Display in `LazyColumn` with keys.
- **Live demo:** Show a simple todo list.

---

## Part 5: Best Practices & Wrap-up (10 min)

### Slide 22: Layout & Component Best Practices (5 min)

- Use standard layouts; custom only when necessary.
- Modifier order: layout → appearance → interaction.
- Hoist state; use ViewModel for persistent state.
- Use `Scaffold` for consistent structure.
- For lists, always use `LazyColumn` for large data sets.

---

### Slide 23: Q&A and Resources (5 min)

- Open questions.
- Resources:
  - [Compose Layout](https://developer.android.com/jetpack/compose/layout)
  - [State in Compose](https://developer.android.com/jetpack/compose/state)
  - [ViewModel](https://developer.android.com/jetpack/compose/libraries#viewmodel)
  - [Scaffold](https://developer.android.com/jetpack/compose/components/app-bars)
  - [Snackbar & AlertDialog](https://developer.android.com/jetpack/compose/components/dialogs)
  - [Lazy lists](https://developer.android.com/jetpack/compose/lists)

