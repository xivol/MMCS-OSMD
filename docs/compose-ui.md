# Lecture Notes: Jetpack Compose Session 1 – Foundations

**Instructor Notes – 1.5‑hour session**

---

## Introduction (2 minutes)

Welcome participants. Briefly introduce yourself and the topic: "Today we're going to start our journey into Jetpack Compose, Android's modern UI toolkit. We'll cover the fundamental concepts that will allow you to build simple yet powerful UIs declaratively. By the end of this session, you'll understand composable functions, layouts, modifiers, basic state management, and theming."

Remind them that this is the first of two sessions; the next one will dive into interactivity, navigation, and lists.

---

## Slide 1: Title Slide
*Display the title slide. Wait for attendees to settle. Read the title and mention that the session is hands‑on; encourage them to follow along in Android Studio if they have it ready.*

---

## Slide 2: Agenda
Go through the agenda quickly. Emphasize that we'll build from ground up, starting with what Compose is, then writing our first composables, learning how to arrange them with layouts and modifiers, understanding how state drives UI updates, and finally giving our app a consistent look with MaterialTheme. Keep the timing in mind: Introduction (10 min), Core Concepts (15), Layouts and Modifiers (25), Basic State Management (20), Theming Essentials (10), Q&A (10). Point out that there will be time for questions at the end, but attendees can also ask along the way.

---

## Slide 3: What is Jetpack Compose?
**Key points to cover:**
- Compose is a modern, fully Kotlin‑based UI toolkit.
- It replaces the old XML‑based system with a **declarative** approach.
- In the old imperative way (Views), you had to manually update each UI element when data changed (`findViewById`, `setText`, etc.). In Compose, you simply describe what the UI should look like for a given state, and Compose takes care of the rest.
- Compose is interoperable: you can mix Compose and traditional Views in the same app, making adoption gradual.

**Example to illustrate:** In the old system, updating a TextView after a button click required finding the view and calling `setText()`. In Compose, you change a variable and the UI updates automatically.

**Transition:** "Now that we know what Compose is, let's see how it's different in practice."

---

## Slide 4: Declarative vs Imperative UI
**Elaborate on the comparison:**
- **Imperative** (like traditional Android with XML):
  - You manually construct the UI hierarchy.
  - You hold references to views.
  - You respond to events and update views explicitly.
  - Prone to bugs when state and UI get out of sync.
- **Declarative** (Compose):
  - You describe the UI as a function of state.
  - When state changes, Compose automatically re‑executes (recomposes) the affected parts.
  - No manual view lookups; the UI is always a reflection of the current state.

**Show the code snippet:**
```kotlin
@Composable
fun Greeting(name: String) {
    Text("Hello, $name")
}
```
Explain that this function takes a `name` parameter and displays it. If `name` changes (e.g., from user input), Compose will call this function again with the new value and update the screen. No need to find the TextView and call `setText()`.

**Emphasize:** This is the core mental model – "UI = f(state)".

---

## Slide 5: Setting Up a Compose Project
**Walk through the steps:**
- In Android Studio (Arctic Fox or newer), create a new project and select "Empty Compose Activity". That's the easiest way.
- For existing projects, you need to enable Compose in the module's `build.gradle.kts`:
  - Set `compose = true` in `buildFeatures`.
  - Specify the Kotlin compiler extension version.
  - Add the Compose dependencies. Show the BOM (Bill of Materials) approach – it simplifies version management.
- Mention the main dependencies: `ui`, `material3`, `ui-tooling-preview` (for previews), and `ui-tooling` (for debugging tools).
- Remind them that the BOM ensures all Compose libraries are compatible.

**Practical tip:** If attendees are following along, they can open Android Studio now and create a new Compose project. Pause for a minute to let them do that.

---

## Slide 6: Composable Functions
**Explain the anatomy of a composable:**
- Annotated with `@Composable`.
- They don't return a value; they emit UI into the composition tree.
- They can call other composables.
- They are conceptually similar to React components or SwiftUI views.

**Example:**
```kotlin
@Composable
fun WelcomeScreen() {
    Column {
        Text("Welcome")
        Button(onClick = { /* do something */ }) {
            Text("Click me")
        }
    }
}
```
Point out that `Column` and `Button` are themselves composable functions. This nesting creates the UI hierarchy.

**Important:** Composables can only be called from other composables or from a `setContent` block (like in `MainActivity`). The Compose runtime manages the composition.

---

## Slide 7: Declarative UI and Recomposition
**Deepen understanding:**
- Compose keeps track of which composables read which state.
- When a state value changes, Compose schedules recomposition of only those composables that depend on that state.
- This is highly optimized; you don't have to worry about performance if you follow best practices.

**Draw a simple diagram (or describe):** Imagine a screen with a header, a counter button, and a footer. Only the button's text reads the counter state. When the counter increments, only the button recomposes – the header and footer are skipped.

**Key term:** **Recomposition** – the process of re‑running composables to reflect state changes.

**Reassure:** You don't need to manually invalidate anything; Compose does it automatically.

---

## Slide 8: Your First Composable
**Live demo (or code walkthrough):**
- Open `MainActivity.kt` in the newly created project.
- Replace the default `Greeting` composable with:
```kotlin
@Composable
fun HelloWorld() {
    Text("Hello, Compose!")
}
```
- Call it from `setContent`: 
```kotlin
setContent {
    MyAppTheme {   // or just HelloWorld() if no theme wrapper
        HelloWorld()
    }
}
```
- Run the app. Show that it displays "Hello, Compose!".
- Point out that there is no XML, no `onCreate` inflation – it's all Kotlin.

**Ask:** "How many lines of code did we write? Just a few. That's the power of Compose."

---

## Slide 9: Using Preview
**Introduce the `@Preview` annotation:**
- Without running the app, you can see your composable in the IDE.
- Add `@Preview(showBackground = true)` above `HelloWorld`.
- The preview pane appears (usually on the right side) – you can interact with it (click, type) if the composable supports it.
- Customize previews: `widthDp`, `heightDp`, `locale`, etc.
- Previews are great for rapid UI iteration and testing different states.

**Demo:** Show the preview in Android Studio. Also mention that you can have multiple previews for the same composable with different parameters.

**Note:** Previews require the `ui-tooling-preview` dependency.

---

## Slide 10: Basic Composables
**List and briefly describe:**
- **`Text`**: Displays a string. Parameters like `color`, `fontSize`, `maxLines`.
- **`Button`**: A clickable surface with content (text, icon, etc.). Takes an `onClick` lambda.
- **`Image`**: Displays an image from resource, painter, or bitmap. Use `painterResource(R.drawable.my_image)`.
- **`Icon`**: For icons – typically from `Icons.Default` (e.g., `Icons.Default.Favorite`). Has a mandatory `contentDescription` for accessibility.

**Show a combined example:**
```kotlin
Button(onClick = { /* handle like */ }) {
    Icon(Icons.Default.Favorite, contentDescription = "Like")
    Text("Like")
}
```
Explain that the `Button`'s content is a trailing lambda that can contain multiple child composables. `Row` is implied inside `Button` by default.

**Tip:** Mention that these are Material Design components. They come styled according to the theme.

---

## Slide 11: Layout Containers
**Introduce the three fundamental layout composables:**
- **`Column`**: Places children vertically.
- **`Row`**: Places children horizontally.
- **`Box`**: Stacks children on top of each other (like `FrameLayout` in the old system).

**Show a simple example:**
```kotlin
Column {
    Text("Above")
    Row {
        Text("Left")
        Text("Right")
    }
    Box {
        Image(painter = painterResource(R.drawable.background), contentDescription = null)
        Text("Overlay", modifier = Modifier.align(Alignment.Center))
    }
}
```
Explain that in `Box`, children are positioned by default in the top‑start corner, but you can use `Modifier.align()` inside the `Box` scope to change alignment.

**Note:** All three are composable functions that accept a `content` lambda with a special receiver that provides layout‑specific modifiers (like `align` in `Box`).

---

## Slide 12: Modifiers: Chainable UI Properties
**Explain the concept of modifiers:**
- Modifiers are objects that decorate or add behavior to composables.
- They are applied in a chain, and **order matters**.
- For example, `padding` then `background` gives padding around the background; reversing would put background on the whole area including padding.
- Modifiers are like CSS properties but with a functional, chainable syntax.

**Example:**
```kotlin
Text(
    "Styled Text",
    modifier = Modifier
        .padding(16.dp)
        .background(Color.Yellow)
        .border(2.dp, Color.Red)
        .clickable { /* handle click */ }
)
```
Walk through each modifier: adds padding, then yellow background, then red border, then makes the whole area clickable.

**Emphasize:** Many modifiers are available – for size, layout, graphics, input, etc.

---

## Slide 13: Common Modifiers (Cheat Sheet)
**Present as a quick reference, not exhaustive.**
- **Size**: `.size(48.dp)` sets both width and height; `.width()`, `.height()`; `.fillMaxWidth()` fills the available width.
- **Padding**: `.padding(16.dp)` for all sides; `.padding(horizontal = 8.dp, vertical = 4.dp)`.
- **Background**: `.background(Color.Blue, shape = RoundedCornerShape(8.dp))` – can also use `Brush` for gradients.
- **Border**: `.border(2.dp, Color.Gray, RectangleShape)`.
- **Clipping**: `.clip(RoundedCornerShape(8.dp))` – clips content to shape.
- **Clickable**: `.clickable { }` adds tap handling.

**Tip:** Modifiers can also be conditional – e.g., apply `.clickable` only if enabled.

---

## Slide 14: Arrangement and Alignment
**Clarify the difference:**
- **Arrangement** controls spacing between children (along the main axis).
- **Alignment** controls how children are placed crosswise.

**For `Column`:**
- `verticalArrangement` – `Arrangement.Top`, `Bottom`, `Center`, `SpaceBetween`, `SpaceEvenly`, `SpaceAround`.
- `horizontalAlignment` – `Alignment.Start`, `CenterHorizontally`, `End`.

**For `Row`:**
- `horizontalArrangement` – similar options.
- `verticalAlignment` – `Alignment.Top`, `CenterVertically`, `Bottom`.

**Example:**
```kotlin
Row(
    horizontalArrangement = Arrangement.SpaceEvenly,
    verticalAlignment = Alignment.CenterVertically
) {
    Text("Left")
    Text("Center")
    Text("Right")
}
```
This spreads the three texts evenly with equal space around them and centers them vertically.

**Visual:** If possible, show a diagram illustrating these parameters.

---

## Slide 15: Example: Simple Profile Card
**Walk through the code, explaining each part:**
```kotlin
@Composable
fun ProfileCard(name: String, age: Int) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
            .background(Color.LightGray, shape = RoundedCornerShape(8.dp))
            .padding(16.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Image(
            painter = painterResource(R.drawable.profile),
            contentDescription = null,
            modifier = Modifier.size(64.dp).clip(CircleShape)
        )
        Spacer(modifier = Modifier.width(16.dp))
        Column {
            Text(text = name, fontSize = 20.sp)
            Text(text = "$age years old")
        }
    }
}
```
**Explain:**
- The outer `Row` takes full width, has outer padding, then a light gray background with rounded corners, then inner padding.
- `verticalAlignment = Alignment.CenterVertically` aligns the image and text column centrally.
- Image is sized 64dp and clipped to a circle.
- `Spacer` creates a gap.
- The `Column` contains two `Text`s.

**Emphasize:** This is a real‑world example; you can reuse this pattern for many list items.

---

## Slide 16: What is State?
**Introduce the concept of state:**
- State is any data that can change over time (e.g., counter value, text input content, checkbox status).
- In Compose, UI must reflect the current state.
- When state changes, Compose automatically recomposes the parts of the UI that depend on it.
- To make state observable, Compose provides `mutableStateOf`.

**Simple analogy:** Think of a spreadsheet: cell A1 contains a value; cell B1 shows a formula based on A1. When A1 changes, B1 updates automatically. Compose works the same way.

---

## Slide 17: `mutableStateOf` and `remember`
**Explain the two pieces:**
- `mutableStateOf(initialValue)` creates an observable state holder. Reading its `.value` property registers a dependency; writing it triggers recomposition.
- However, if you simply write `val count = mutableStateOf(0)` inside a composable, a new state object is created on every recomposition, losing the value. So you need to **remember** it across recompositions.
- `remember { mutableStateOf(0) }` stores the state object in the composition, so it survives recompositions.
- Kotlin property delegation simplifies: `var count by remember { mutableStateOf(0) }` – then you can use `count` directly (get/set).

**Example:**
```kotlin
@Composable
fun Counter() {
    val count = remember { mutableStateOf(0) }
    Button(onClick = { count.value++ }) {
        Text("Clicked ${count.value} times")
    }
}
```
And with delegation:
```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) {
        Text("Clicked $count times")
    }
}
```
**Note:** The `by` delegation requires `import androidx.compose.runtime.getValue` and `import androidx.compose.runtime.setValue` (usually automatically suggested).

---

## Slide 18: Counter Example Step by Step
**Reinforce with a live coding demo:**
1. Create a new composable `Counter`.
2. Add state: `var count by remember { mutableStateOf(0) }`.
3. Add a `Button` with `onClick = { count++ }`.
4. Inside the button, add `Text("Clicked $count times")`.
5. Show the result in the app/preview.

**Point out:** Each time you click, the text updates. You can add a `Log.d` inside the composable to see how often it recomposes – you'll see it recomposes only the button's content, not the whole screen.

**Discuss:** This is the foundation of interactive UIs in Compose.

---

## Slide 19: Recomposition in Action
**Explain the optimization:**
- Compose tracks which state is read by which composable.
- When `count` changes, only composables that read `count` are invalidated.
- In the `Counter` example, only the `Text` inside the `Button` reads `count`. So when you click, only that `Text` recomposes. The `Button` itself does not recompose (unless its internal state changes).
- This is a huge performance advantage over the old system where you might redraw entire layouts.

**Mention:** You can use tools like Layout Inspector to visualize recomposition counts.

---

## Slide 20: MaterialTheme
**Introduce theming:**
- Compose provides a `MaterialTheme` composable that wraps your app content.
- It provides three main pillars:
  - **`colorScheme`**: Colors for primary, secondary, background, surface, error, etc.
  - **`typography`**: Text styles (headline, body, caption, etc.)
  - **`shapes`**: Corner shapes for components (small, medium, large).
- You can access these values anywhere inside the theme using `MaterialTheme.colorScheme.primary`, `MaterialTheme.typography.bodyLarge`, etc.
- This ensures consistency across your app.

**Example:**
```kotlin
Text(
    "Hello",
    color = MaterialTheme.colorScheme.primary,
    style = MaterialTheme.typography.headlineMedium
)
```
Explain that by using theme values, you can easily change the entire app's look by modifying the theme.

---

## Slide 21: Customizing Theme
**Show how to override defaults:**
- Typically you define your own `MyTheme` composable that calls `MaterialTheme` with custom parameters.
- Example:
```kotlin
val MyColors = lightColorScheme(
    primary = Color(0xFF6200EE),
    secondary = Color(0xFF03DAC5),
    background = Color.White
)

@Composable
fun MyTheme(content: @Composable () -> Unit) {
    MaterialTheme(
        colorScheme = MyColors,
        typography = MyTypography, // you can define custom typography
        shapes = MyShapes,
        content = content
    )
}
```
- Then use `MyTheme { ... }` in your app instead of the default.
- Mention that `lightColorScheme` and `darkColorScheme` are helper functions.

**Note:** You can also copy the default theme and tweak only a few values.

---

## Slide 22: Dark Mode Support
**Explain automatic dark mode:**
- If you provide both light and dark color schemes, Compose can switch automatically based on system setting.
- Use `isSystemInDarkTheme()` to decide:
```kotlin
@Composable
fun MyTheme(content: @Composable () -> Unit) {
    val colors = if (isSystemInDarkTheme()) {
        darkColorScheme(primary = Color(0xFFBB86FC))
    } else {
        lightColorScheme(primary = Color(0xFF6200EE))
    }
    MaterialTheme(colorScheme = colors, content = content)
}
```
- Compose will recompose when the system night mode changes.
- You can also use `dynamicColor` for Material You on Android 12+.

**Tip:** Test dark mode in the preview with `@Preview(uiMode = Configuration.UI_MODE_NIGHT_YES)`.

---

## Slide 23: Key Takeaways
**Summarize the session:**
- Jetpack Compose is declarative – you describe UI based on state.
- `@Composable` functions are the building blocks.
- Modifiers let you style and configure composables.
- Layouts (`Column`, `Row`, `Box`) arrange children.
- State with `remember` + `mutableStateOf` drives updates.
- `MaterialTheme` provides consistent styling.
- You can now build simple static UIs and add basic interactivity.

**Encourage:** Practice by building a small profile screen or a todo item.

---

## Slide 24: Q&A / Resources
**Open floor for questions.** If time permits, address any doubts.

**Point to resources:**
- Official documentation: [developer.android.com/jetpack/compose](https://developer.android.com/jetpack/compose)
- Compose pathway: [developer.android.com/courses/pathways/compose](https://developer.android.com/courses/pathways/compose)
- Sample apps: [github.com/android/compose-samples](https://github.com/android/compose-samples)

**Remind about next session:** We'll dive deeper into state hoisting, ViewModel integration, navigation, and lists. If attendees have questions about those, they can note them for next time.


---

*End of Lecture Notes*
