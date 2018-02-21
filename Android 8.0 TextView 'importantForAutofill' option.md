## Android 8.0 TextView `importantForAutofill` option

I got crash about my custom textview `LinkEllipseTextView`.

Our user reported to me.
I spent 5days to solve this problem and texted with my user.

Error message :

    Fatal Exception: java.lang.NullPointerException
    Attempt to invoke interface method 'int java.lang.CharSequence.length()' on a null object reference

Log :

    android.widget.TextView.onProvideAutoStructureForAssistOrAutofill (TextView.java:10256)

I didn't find cause of crash... and there was not data about this crash.
So I read android 8.0 behavior changes and lots of stacks.
I got a clue in [StackOverflow](https://stackoverflow.com/questions/45840856/android-8-0-oreo-crash-on-focusing-textinputedittext).
And I thought the issue is related to accessibility. I texted to user and asked to disable other apps' accessibility. Surprisingly this crash was not occurred.

The reason was `autofill`. So I disabled by adding `android:importantForAutofill="noExcludeDescendants"`.

I will study more about autofill with accessibility.
