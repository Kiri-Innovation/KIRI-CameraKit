# Android 使用说明

## 1. 接入 AAR 文件

先从 Release 页面下载对应版本的 aar 文件, 并放入 app 项目路径中

然后在 app/build.gradle 文件中增加 aar 的依赖

```gradle
dependencies {
    implementation files('my_path/my_lib.aar')
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
// 设置拍摄的照片存放路径
cameraView.setSavePath()

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
| setSavePath | 设置拍摄的照片存放路径, 参数为 File 对象 |
| setTakePictureListener | 设置拍摄相关的事件回调，为 OnTakePictureListener 类型 |
| takePicture | 拍摄照片 |

<br/>

OnTakePictureListener 声明:

| 返回类型 | 方法签名 | 说明 |
| ----- | ----- | ----- |
| void | onTaken(photoFile: File) | 拍摄并保存单张照片成功的回调方法, 会将本次拍摄成功的照片文件返回回来 |
| void | onTakeError(exception: Exception) | 拍摄发生异常时的回调, 参数会将本次的异常信息携带回来 |


⚠️ 注意：CameraView 需要带触摸屏幕自动对焦功能

