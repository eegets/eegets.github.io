---
layout: post
title:  "Android transitionName共享元素"
categories: AndroidUI transitionName
---

先来看看效果

![SVID_20201222_153820_1.gif](https://upload-images.jianshu.io/upload_images/18406403-c15cecf98496d733.gif?imageMogr2/auto-orient/strip)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <TextView
        android:id="@+id/transitionText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="缩放Text"
        android:textSize="28sp"
        android:layout_marginTop="50dp"
        android:transitionName="transitionText"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"/>
    <ImageView
        android:id="@+id/transitionImg"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:background="#ff0000"
        android:transitionName="transitionImg"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/transitionText"
        tools:ignore="MissingConstraints" />
</androidx.constraintlayout.widget.ConstraintLayout>
```
MaiActivity.kt
```kotlin
transitionImg.setOnClickListener {
            val pair = Pair<View, String>(transitionImg, "transitionImg")
            val pair2 = Pair<View, String>(transitionText, "transitionText")
            val bundle: Bundle = ActivityOptions.makeSceneTransitionAnimation(this, pair, pair2).toBundle()
            startActivity(Intent(this, TranslateActivity::class.java), bundle)
        }

```

activity_translate.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/transitionText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="50dp"
        android:text="缩放Text"
        android:textSize="28sp"
        android:transitionName="transitionText"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/hilt2" />

    <ImageView
        android:id="@+id/transitionImg"
        android:layout_width="match_parent"
        android:layout_height="500dp"
        android:layout_marginTop="50dp"
        android:background="#ff00ff"
        android:transitionName="transitionImg"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/transitionText"
        tools:ignore="MissingConstraints" />
</androidx.constraintlayout.widget.ConstraintLayout>
```
TranslateActivity.kt
```kotlin
class  TranslateActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_traslate)
    }
}
```