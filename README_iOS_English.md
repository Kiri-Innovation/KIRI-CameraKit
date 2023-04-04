
## Usage on iOS

## 1. Add framework file
Download framwork file in Release page and move it into app project path.
Put framework into Build Phases -> Link Binary With Libraries
Select Embed & Sign in "Embed" option

<br/>

### 2.Before using CameraKit，Please initialize by using CameraKit.share.setup to do the authentication
```swift
CameraKit.share.setup(envType: .test, account: "account", password: "password") { result in
    print("result:\(result)")
}
```
### 3.Initialize CameraView
```swift
let cameraView = CameraView()
```
### 4.Use "setPhotoFolderPath" to set where to save the photos
```swift
let docPath = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true).first!
cameraView.setPhotoFolderPath("\(docPath)/CameraKit")
```
### 5.Use "startPreview" to request permission and show the preview
```swift
cameraView.startPreview { result in
    print("result:\(result)")
}
```
### 6.Use "takePhoto" to take phtos
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
          Button("Start preview") {
              cameraView.startPreview { result in
                  print("result:\(result)")
              }
          }
          Button("Take photos") {
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
