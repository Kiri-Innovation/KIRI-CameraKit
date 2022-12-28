
## iOS的使用

## 1. 接入framework文件
先从 Release 页面下载对应版本的 framwork 文件, 并放入 app 项目路径中,
将framework放入Build Phases -> Link Binary With Libraries中，
Embed选择Embed & Sign

<br/>

### 2.使用CameraKit时，先使用CameraKit.share.setup初始化鉴权
```swift
CameraKit.share.setup(envType: .test, account: "account", password: "password") { result in
    print("result:\(result)")
}
```
### 3.初始化CameraView
```swift
let cameraView = CameraView()
```
### 4.使用setPhotoFolderPath设置自定义图片路径
```swift
let docPath = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true).first!
cameraView.setPhotoFolderPath("\(docPath)/CameraKit")
```
### 5.调用startPreview开始请求权限并显示画面
```swift
cameraView.startPreview { result in
    print("result:\(result)")
}
```
### 6.调用takePhoto开始拍照
```swift
cameraView.takePhoto()
```

### Example
```swift
import SwiftUI
import CameraKit

struct ContentView: View {
    let cameraView = CameraView()
    var body: some View {
      VStack(spacing: 20) {
          UIViewPreview {
              cameraView
          }
          Button("开始预览") {
              cameraView.startPreview { result in
                  print("result:\(result)")
              }
          }
          Button("点击拍照") {
              cameraView.takePhoto()
          }
      }
      .padding()
      .onAppear {
          CameraKit.share.setup(envType: .test, account: "account", password: "password") { result in
              print("result:\(result)")
          }
          let docPath = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true).first!
          cameraView.setPhotoFolderPath("\(docPath)/CameraKit")
      }
    }
}

```