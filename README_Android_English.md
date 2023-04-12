# Android Usage

## 1. Add AAR file

Download aar file in Release page and move it into app project path.

add dependencies in app/build.gradle file

```gradle
dependencies {
    implementation files('my_path/my_lib.aar')
    
    //  Must add below dependencies
    implementation "androidx.camera:camera-core:1.2.0-alpha02"
    implementation "androidx.camera:camera-lifecycle:1.2.0-alpha02"
    implementation 'androidx.camera:camera-view:1.2.0-alpha02'
    implementation "androidx.camera:camera-camera2:1.2.0-alpha02"
}
```

<br/>

## 2. Initialize SDK

```Kotlin
CameraKit.init(
    // SDK Environment
    env = EnvType.Test, 
    // account in our plafrom
    account = "", 
    // password associates with the account
    password = "",
    onSuccess = {
        // SDK initialized successfully
    },
    onError = { e ->
        // SDK initialized failed
    }
)
```

| arguments name | definition |
| ----- | ----- |
| env | SDK environment, EnvType.Test is testing environment, EnvType.Prod is production environment |
| account | account in our plafrom |
| password | password associates with the account |
| onSuccess | SDK initialized successfully |
| onError | SDK initialized failed |

Possible exception in "onError":

| Exception type | definition | solution |
| ----- | ----- | -----|
| AccountNotExistException | Account does not exist | Please register an account |
| AuthenticationException | Account or password is incorrect | Please check your account and password |
| ExhaustedException | Quota used up | Please contact us |
| SDKException | SDK initialize failed | Please contact us |

<br/>

## 3. Camera API

In layout file:

```xml
<com.kiri.camera.toolkit.view.CameraView
        android:id="@+id/camera_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
```

KT code API:

```Kotlin
// Initialize camera and preview 
cameraView.bind(LifecycleOwner)

// Set the path of captured photos.
cameraView.setSavePath(File)

// Set capturing photo related callback 
cameraView.setTakePictureListener(object : OnTakePictureListener {
    override fun onTaken(photoFile: File) {
        Log.e(TAG, "Took one photo, saved in: ${photoFile.absolutePath}")
    }

    override fun onTakeError(exception: Exception) {
        Log.e(TAG, "Error in taking photo, error message is: ${exception.message}")
    }
})

// Taking photos.
cameraView.takePicture()
```


Method:

| Method name | Definition |
| ----- | ----- |
| bind | Initialize camera and preview |
| setSavePath | Set the path of captured photos. |
| setTakePictureListener | Set capturing photo related callback |
| takePicture | Taking photos |

<br/>

OnTakePictureListener declaration:

| Return type | Function signiture | Definition |
| ----- | ----- | ----- |
| void | onTaken(photoFile: File) | Capture one photo succesfully, will return the captured photo |
| void | onTakeError(exception: Exception) | Capture one photo failed, will return the error message |


⚠️ Note：You have to ask the camera permission youself

## 4. Code examples
#### activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <com.kiri.camera.toolkit.view.CameraView
        android:id="@+id/camera_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <androidx.appcompat.widget.AppCompatButton
        android:id="@+id/btn_taken"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="30dp"
        android:text="Taken"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

#### MainActivity.kt
```Kotlin
package com.kiri.camera.demo

import android.Manifest
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.util.Log
import android.widget.Button
import com.kiri.camera.toolkit.tool.*
import com.kiri.camera.toolkit.view.CameraView
import com.kiri.camera.toolkit.view.OnTakePictureListener
import com.tbruyelle.rxpermissions3.RxPermissions
import java.io.File
import java.lang.Exception

private const val TAG = "MainActivity"

class MainActivity : AppCompatActivity() {

    private val takenBtn: Button by lazy {
        findViewById(R.id.btn_taken)
    }

    private val cameraView: CameraView by lazy {
        findViewById(R.id.camera_view)
    }

    private val rxPermissions = RxPermissions(this)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Request permission
        rxPermissions.request(Manifest.permission.CAMERA).subscribe { granted ->
            if (granted) {
                cameraView.bind(this)
            }
        }

        initCameraKitSDK()
    }

    /**  Initialize CameraKit SDK  **/
    private fun initCameraKitSDK() {
        CameraKit.init(
            env = EnvType.Test, account = "", password = "",
            onSuccess = {
                // SDK initialized successfully
                runOnUiThread {
                    // SDK and permission passed, start initialize the camera
                    initCamera()
                }
            },
            onError = { e ->
                // SDK 初始化发生错误
                when (e) {
                    is AccountNotExistException -> {
                        println("Account does not exist!")
                    }
                    is AuthenticationException -> {
                        println("Account or password is incorrect!")
                    }
                    is ExhaustedException -> {
                        println("Quota used up!")
                    }
                    is SDKException -> {
                        println("SDK error: ${e.message}")
                    }
                }
            }
        )
    }

    /**  Media folder  **/
    private val mediaDir by lazy {
        externalMediaDirs.firstOrNull()?.let {
            File(it, "Photo").apply { mkdirs() }
        } ?: filesDir
    }

    /**  Initial camera related API  **/
    private fun initCamera() {
        // Set the path of captured photos. 
        cameraView.setSavePath(mediaDir)

        // Set capturing photo related callback
        cameraView.setTakePictureListener(object : OnTakePictureListener {
            override fun onTaken(photoFile: File) {
                Log.e(TAG, "Took one photo, saved in: ${photoFile.absolutePath}")
            }

            override fun onTakeError(exception: Exception) {
                Log.e(TAG, "Error in taking photo, error message is: ${exception.message}")
            }
        })

        // Take photos
        takenBtn.setOnClickListener {
            cameraView.takePicture()
        }
    }
}
```
