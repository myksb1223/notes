## Screen orientation with camera

When present bitmap to image view using camera, some devices can't present the bitmap.

First, I thought bitmap was null. But this issue was related to screen orientation.

When I called camera app, some devices screen rotated landscape mode.
After user took a picture, those devices returned portrait(default) mode.
That was why bitmap was not show in image view.

When move landscape to portrait, their view were recreated. I didn't know some status information was not saved.

It makes undesirable situation to developer.
So I spent lots of time to solve this issue.

Just add two functions **`onSaveInstanceState()`** and **`onRestoreInstanceState()`**.

```java
@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
    // Save the user's current game state
    // Always call the superclass so it can save the view hierarchy state
    if(recent_code == REQUEST_CAMERA) {
        if(output != null) {
            savedInstanceState.putString("output", output.toString());
        }            savedInstanceState.putInt("requestCode", recent_code);
    }
    super.onSaveInstanceState(savedInstanceState);
}

public void onRestoreInstanceState(Bundle savedInstanceState) {
    // Always call the superclass so it can restore the view hierarchy
    super.onRestoreInstanceState(savedInstanceState);
    // Restore state members from saved instance
    if(recent_code == REQUEST_CAMERA) {
        output = Uri.parse(savedInstanceState.getString("output"));
        recent_code = savedInstanceState.getInt("requestCode", -1);          onActivityResult(recent_code, RESULT_OK, null);
    }
}
```
