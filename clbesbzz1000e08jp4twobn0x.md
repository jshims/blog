# Why Jetpack Compose

Android has been recommending Jetpack Compose as the UI toolkit to use when developing applications. I have been using it on simple UI for my company's app. In this article, I will try to simply explain why Compose is being recommended over the XML/View UI system.

From my research, it comes down to (UI) state management.

Most apps today are dynamic, meaning that an app's UI should be updated in real-time according to users' interactions. The constant changes in UI mean that managing how it should be shown has become increasingly complex. Thus, if a simpler, more intuitive approach to drawing and managing the state of UI existed, it would help developers write better applications.

The Android team viewed that drawing UI with Compose would be the solution to the problem. That was because Compose provided a stateless UI system compared to XML, which was a stateful UI system.

The XML system describes a tree of Views that are responsible for their state. Views expose getter/setter methods to update their state and some also expose listeners that allow developers to observe such changes. Sometimes changes to one view created side effects in which developers had to observe the changes and update other views. The problem with this system was that as apps became more complex, the synchronization of different states increased the chance of errors.

Jetpack Compose was developed as an inherently stateless UI toolkit. It provided composable functions that described **what** a UI should look like. The functions received parameters to describe the state of the UI and whenever changes occurred, the functions were (re)invoked with different values to properly represent their current state.

So is Jetpack Compose the silver bullet? In terms of drawing complex UI, not yet since I do believe that full support is not yet ready. However, in terms of state management, with proper implementation, I think that Jetpack Compose is very promising. To properly implement UI with Jetpack Compose I believe that further research on recomposition is needed.

In the next article, I will try to summarize what recomposition is.

**Sample Code: implementing a simple card that when selected changes the state of radio button and text**

**XML**

```xml
<!-- survey_answer.xml -->
<LinearLayout android:orientation="horizontal">
  <ImageView android:id="@+id/answer_image" .../>
  <TextView android:id="@+id/answer_text" .../>
  <RadioButton android:id="@+id/answer_radio_button" .../>
</LinearLayout>
```

```kotlin
class SurveyQuestionActivity : Activity() {
  ...
  override fun onCreate(savedInstanceState: Bundle?){
    ...
    // find views
    val image = findViewById(R.id.answer_image)
    val text = findViewById(R.id.answer_text)
    val radioButton = findViewById(R.id.answer_radio_button)

    // set data
    image.setImage(...)
    text.setImage(...)

    // on certain event change UI
    radioButton.setOnSelectedListener{isSelected ->
      radioButton.isSelected = isSelected
      text.setColor(...)
    }
    ...
  }
}
```

**Jetpack Compose**

```kotlin
@Composable
fun SurveyAnswer(
  answer: Answer
  isButtonSelected : Boolean,
  onButtonClick : (() -> Unit)
) {
  Row{
    Image(answer.image)
    Text(
      text = answer.text
      color = if(isButtonSelected) colorA else colorB
    )
    RadioButton(
      isSelected = isButtonSelected,
      onClick = {
        onButtonClick.invoke()
      }
    )
  }
}
```

**Resources**

[https://developer.android.com/jetpack/compose](https://developer.android.com/jetpack/compose)

%[https://www.youtube.com/watch?v=4zf30a34OOA&t=10s]