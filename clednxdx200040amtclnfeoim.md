# Jetpack Recomposition

*This post is a summary of the official document at* [*developers android*](https://developer.android.com/jetpack/compose/mental-model#recomposition)*. Please read it for a more detailed explanation.*

### What is Recomposition?

Recomposition is the (re)drawing of views in Compose. Recomposition occurs when you want the app to draw the initial UI or want to update the UI with new data. Since views in Compose are immutable, when recomposition occurs, the view is redrawn from scratch.

To be more performant, the Compose framework executes intelligent recomposition to recompose only the components that have changed.

**Intelligent Recomposition**

Intelligent Recomposition is the Compose framework only updating things that may have changed. For instance, have a look at the sample code from the [developers android](https://developer.android.com/jetpack/compose/mental-model#skips):

```kotlin
@Composable
fun NamePicker(
    header: String,
    names: List<String>,
    onNameClicked: (String) -> Unit
) {
    Column {
// this will recompose when [header] changes, but not when [names] changes
        Text(header, style = MaterialTheme.typography.h5)
        Divider()

/**
* When an item's [name] updates, the adapter for that item
* will recompose. This will not recompose when [header] changes
*/
        LazyColumn {
            items(names) { name ->
                NamePickerItem(name, onNameClicked)
            }
        }
    }
}

@Composable
private fun NamePickerItem(name: String, onClicked: (String) -> Unit){
    Text(name, Modifier.clickable(onClick = { onClicked(name) }))
}
```

### Best Practice in Recomposition

Best practice when programming in Compose is to know how composable functions execute and how recomposition occurs. In summary, never depend on side effects from executing composable functions

*\*side effect:* *a side effect is any change that is visible to the rest of your app. Examples of side effects are: writing to a property of a shared object, updating an observable in ViewModel or updating a shared preference.*

When writing composable functions it is best to:

* Keep them fast. Never try to have expensive or complicated logic inside a composable function (ex. don't read from databases inside a composable function).
    

* Keep them idempotent. Idempotent means that if the same input is given to the function no additional effects should occur. A way to keep it idempotent is not to read from a global variable inside of the composable function. Rather depend on the input
    
* Be side effect free. If a global variable needs updating because of an event, rather than updating it inside the composable function, accept a lambda and have the caller update it in the lambda block.
    
    ```kotlin
    // accepts an onClicked lambda where the caller can update a global variable if needed.
    // if another composable function relies on the update the caller can observe changes and recompose that function
    @Composable
    private fun NamePickerItem(name: String, onClicked: (String) -> Unit){
        Text(name, Modifier.clickable(onClick = { onClicked(name) }))
    }
    ```
    

**Things to keep in mind about compose function execution**

* Composable functions can execute in any order. Moreover, composable functions may run in parallel. Thus, one should not program in such a way that one function relies on another function to update a global variable (side effect). Each function should be self-contained.
    
* Recomposition skips as many composable functions and lambdas as possible
    
* Recomposition is optimistic and may be canceled. Recomposition starts whenever Compose thinks a parameter may have changed and expects that no changes will occur until recomposition is finished. However, if changes occur before recomposition is finished it may cancel and restart.
    
* A composable function might run frequently (ex. animation). Avoid expensive operations inside the function.