# Android 使用说明

## 1. 接入 AAR 文件

先从 Release 页面下载对应版本的 aar 文件, 并放入 app 项目路径中

然后在 app/build.gradle 文件中增加 aar 的依赖

```gradle
dependencies {
    implementation files('my_path/my_lib.aar')
    
    // 以下依赖必须添加
    implementation "androidx.camera:camera-core:1.2.0-alpha02"
    implementation "androidx.camera:camera-lifecycle:1.2.0-alpha02"
    implementation 'androidx.camera:camera-view:1.2.0-alpha02'
    implementation "androidx.camera:camera-camera2:1.2.0-alpha02"
}
```

<br/>

## 2. 初始化 SDK

```Kotlin
CameraKit.init(
    // SDK 环境
    env = EnvType.Test, 
    // 平台账号
    account = "", 
    // 平台密码
    password = "",
    onSuccess = {
        // SDK 初始化成功
    },
    onError = { e ->
        // SDK 初始化发生错误
    }
)
```

| 参数名称 | 说明 |
| ----- | ----- |
| env | SDK 环境, EnvType.Test 为测试, EnvType.Prod 为正式 |
| account | 平台账号 |
| password | 平台密码 |
| onSuccess | SDK 初始化成功回调 |
| onError | SDK 初始化失败回调, 会将异常信息带回 |

可能在 onError 中出现的异常:

| 异常类型 | 说明 | 解决方案 |
| ----- | ----- | -----|
| AccountNotExistException | 账号不存在 | 检查账号信息是否正确 |
| AuthenticationException | 验证的账号或密码错误 | 检查账号信息是否正确 |
| ExhaustedException | 接口的调用次数用尽 | 联系开发人员 |
| SDKException | 接口的调用次数用尽 | 初始化失败, 联系开发人员 |

<br/>

## 3. Camera API

布局文件中使用:

```xml
<com.kiri.camera.toolkit.view.CameraView
        android:id="@+id/camera_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
```

KT 代码 API:

```Kotlin
// 初始化相机和预览操作
cameraView.bind(LifecycleOwner)

// 设置拍摄的照片存放路径
cameraView.setSavePath(File)

// 设置拍摄相关的事件回调
cameraView.setTakePictureListener(object : OnTakePictureListener {
    override fun onTaken(photoFile: File) {
        Log.e(TAG, "已拍摄一张照片, 存放位置为: ${photoFile.absolutePath}")
    }

    override fun onTakeError(exception: Exception) {
        Log.e(TAG, "照片拍摄出错, 异常为: ${exception.message}")
    }
})

// 拍摄照片
cameraView.takePicture()
```


方法说明:

| 方法名称 | 说明 |
| ----- | ----- |
| bind | 初始化相机和预览操作, 参数为 LifecycleOwner 对象, 会自动绑定生命周期 |
| setSavePath | 设置拍摄的照片存放路径, 参数为 File 对象 |
| setTakePictureListener | 设置拍摄相关的事件回调，为 OnTakePictureListener 类型 |
| takePicture | 拍摄照片 |

<br/>

OnTakePictureListener 声明:

| 返回类型 | 方法签名 | 说明 |
| ----- | ----- | ----- |
| void | onTaken(photoFile: File) | 拍摄并保存单张照片成功的回调方法, 会将本次拍摄成功的照片文件返回回来 |
| void | onTakeError(exception: Exception) | 拍摄发生异常时的回调, 参数会将本次的异常信息携带回来 |


⚠️ 注意：相机权限需要自行动态申请

## 4. 示例代码
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

        // 请求权限
        rxPermissions.request(Manifest.permission.CAMERA).subscribe { granted ->
            if (granted) {
                cameraView.bind(this)
            }
        }

        initCameraKitSDK()
    }

    /**  初始化 CameraKit SDK  **/
    private fun initCameraKitSDK() {
        CameraKit.init(
            env = EnvType.Test, account = "", password = "",
            onSuccess = {
                // SDK 初始化成功
                runOnUiThread {
                    // SDK 和权限都已申请完毕, 进行相机初始化操作
                    initCamera()
                }
            },
            onError = { e ->
                // SDK 初始化发生错误
                when (e) {
                    is AccountNotExistException -> {
                        println("账号不存在!")
                    }
                    is AuthenticationException -> {
                        println("账号密码错误!")
                    }
                    is ExhaustedException -> {
                        println("接口调用次数用尽!")
                    }
                    is SDKException -> {
                        println("SDK 异常: ${e.message}")
                    }
                }
            }
        )
    }

    /**  媒体文件夹  **/
    private val mediaDir by lazy {
        externalMediaDirs.firstOrNull()?.let {
            File(it, "Photo").apply { mkdirs() }
        } ?: filesDir
    }

    /**  初始化相机相关的 API  **/
    private fun initCamera() {
        // 设置拍摄的照片存放路径
        cameraView.setSavePath(mediaDir)

        // 设置拍摄相关的事件回调
        cameraView.setTakePictureListener(object : OnTakePictureListener {
            override fun onTaken(photoFile: File) {
                Log.e(TAG, "已拍摄一张照片, 存放位置为: ${photoFile.absolutePath}")
            }

            override fun onTakeError(exception: Exception) {
                Log.e(TAG, "照片拍摄出错, 异常为: ${exception.message}")
            }
        })

        // 拍摄照片
        takenBtn.setOnClickListener {
            cameraView.takePicture()
        }
    }
}
```
