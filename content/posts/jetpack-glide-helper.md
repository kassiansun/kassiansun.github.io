---
title: 'A Glide Helper for Jetpack Compose'
date: 2022-01-03T10:31:26+08:00
---

The design of Jetpack Compose makes an important assumption on the rendering process: the developer
should not assume whether or not a composable will render, and how it will render. So any
code inside a composable can be called by [0-N] times. That's why Compose introduced several
helpers to control the code execution of composable (instead of controlling the rendering process):

1. `remember` makes a state persisted among `recomposition`. The state will be destroyed if the composable
   is removed from Composition.
2. `LaunchedEffect` controls how suspend functions will be executed and canceled. It has a key to determine
   whether to re-execute the code.
3. `LocalContext.current` will get the current context of Composable. Most Android libraries will detect
   the context lifecycle.

### The Implementation

```kotlin
import android.graphics.Bitmap
import android.graphics.drawable.Drawable
import android.net.Uri
import android.util.Log
import androidx.compose.runtime.*
import androidx.compose.ui.platform.LocalContext
import com.bumptech.glide.Glide
import com.bumptech.glide.request.target.CustomTarget
import com.bumptech.glide.request.transition.Transition

@Composable
fun loadPicture(picUri: Uri): MutableState<Bitmap?> {
    // the persisted state of helper
    val bitmapState: MutableState<Bitmap?> = remember {
        mutableStateOf(null)
    }
    // current context
    val context = LocalContext.current
    // glide execution control
    LaunchedEffect(picUri) {
        Log.d("utils", "loading image: $picUri")
        // Thanks for the original idea from https://www.youtube.com/channel/UCoNZZLhPuuRteu02rh7bzsw
        // Code copied from https://www.youtube.com/watch?v=ktOWiLx83bQ
        Glide.with(context).asBitmap().load(picUri)
            .into(object : CustomTarget<Bitmap>() {
                override fun onResourceReady(
                    resource: Bitmap,
                    transition: Transition<in Bitmap>?
                ) {
                    Log.d("utils", "setting bitmap")
                    bitmapState.value = resource
                }

                override fun onLoadCleared(placeholder: Drawable?) {
                    bitmapState.value = null
                }
            })
    }
    return bitmapState
}
```
