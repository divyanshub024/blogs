## Exploring the Flutter camera plugin


Building a camera app in Flutter.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945289314/DZK8lJpHg.png)

If you’ve ever built or used any large scale mobile app, then there is a great chance that the app uses the camera functionality. If you look at the [top charts in the PlayStore](https://play.google.com/store/apps/top?hl=en) you will find that many of the apps use the camera to perform various tasks. Flutter has a [camera plugin](https://pub.dev/packages/camera) to get access to the device’s camera on Android and iOS. In this article, we will be exploring the Flutter camera plugin, and we will be building a small camera app to see what this plugin can and cannot do.

Before we move forward let’s see what we are going to build. This app will be able to take a picture and record a video. You can switch between the front and the back camera. And a gallery where you can see the captured images and recorded videos and share them with other applications or delete them from the device.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945294097/tRg4lvESs.gif)

## Getting Started

The app uses the following 5 dependencies. You need to add these dependencies to your `pubspec.yaml`.

* [camera](https://pub.dev/packages/camera): Provides tools to work with the cameras on the device.

* [path_provider](https://pub.dev/packages/path_provider): Finds the correct paths to store media.

* [video_player](https://pub.dev/packages/video_player): To play the recorded video.

* [esys_flutter_share](https://pub.dev/packages/esys_flutter_share): For sharing media files with other applications.

* [thumbnails](https://github.com/divyanshub024/Flutter_Thumbnails): For generating thumbnails from the video.

```
dependencies:
  camera:
  path_provider:
  thumbnails:
    git:
      url: https://github.com/divyanshub024/Flutter_Thumbnails.git
  video_player:
  esys_flutter_share:
```


Next, update your minimum Android SDK version to 21 (or higher) in your `android/app/build.gradle` file.

Add the following lines to your `ios/Runner/Info.plist`:

```
<key>NSCameraUsageDescription</key>
<string>Can I use the camera please?</string>
<key>NSMicrophoneUsageDescription</key>
<string>Can I use the mic please?</string>
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
</dict>
```


## Getting a list of available cameras

First, we will get the lists of cameras using the camera plugin.

```
List*<*CameraDescription*> *_cameras;

@override
void initState() {
  _initCamera();
  super.initState();
}

Future<void> _initCamera() async {
  _cameras = await availableCameras();
}
```


## Initializing camera controller

Now we have a list of available cameras. Next, we will initialize the camera controller. The camera controller is used to control the device cameras. `CameraController` takes two values `CameraDescription` and `ResolutionPreset`. Initially, we have given a camera description as `_camera[0]` which is our back camera.
> Note: Here we have given `ResolutionPreset` as the medium. Try avoiding going to a higher resolution if it freezes your camera. Look at this [issue](https://github.com/flutter/flutter/issues/40519) for more detail.

```
CameraController _controller;*

*Future*<*void*> *_initCamera*() *async {*
*  _controller = CameraController*(*_cameras*[*0*]*, ResolutionPreset.medium*)*;
  _controller.initialize*()*.then*((*_*) {
    *if *(*!mounted*) {
      *return;
    *}
    *setState*(() {})*;
  *})*;
*}

*@override
void dispose*() {
  *_controller?.dispose*()*;
  super.dispose*()*;
*}*
```


## Camera Preview

Once our camera is all set up we will show the preview feed using `CameraPreview` widget. Before we show the camera preview we must wait for the CameraController to initialize.

```
@override
Widget build*(*BuildContext context*) {
  *if *(*_controller != null*) {
    *if *(*!_controller.value.isInitialized*) {
      *return Container*()*;
    *}
  } *else *{
    *return const Center*(
      *child: SizedBox*(
        *width: 32,
        height: 32,
        child: CircularProgressIndicator*()*,
      *)*,
    *)*;
  *}
}*
```


Once the camera is initialized we will show the camera preview.

```
return Scaffold*(
  *backgroundColor: Theme.*of(*context*)*.backgroundColor,
  key: _scaffoldKey,
  extendBody: true,
  body: Stack*(
    *children: *<*Widget*>[
      *_buildCameraPreview*()*,
    *]*,
  *)*,
*)*;
```


Inside the `_buildCameraPreview()` we are scaling the camera preview to the screen size to make it look full screen.

```
Widget _buildCameraPreview*() {
  *final size = MediaQuery.*of(*context*)*.size;
  return ClipRect*(
    *child: Container*(
      *child: Transform.scale*(
        *scale: _controller.value.aspectRatio / size.aspectRatio,
        child: Center*(
          *child: AspectRatio*(
            *aspectRatio: _controller.value.aspectRatio,
            child: CameraPreview*(*_controller*)*,
          *)*,
        *)*,
      *)*,
    *)*,
  *)*;
*}*
```


## Switch camera

The next step is to have the ability to switch or toggle between front and back cameras. To do that let’s first add the icon button to your stack widget.

```
body: Stack*(
  *children: *<*Widget*>[
    *_buildCameraPreview*()*,
  **  Positioned*(
      *top: 24.0,
      left: 12.0,
      child: IconButton*(
        *icon: Icon*(
          *Icons.*switch_camera*,
          color: Colors.*white*,
        *)*,
        onPressed: _onCameraSwitch,
      *)*,
    *)*,***
  ]*,
*)*,
```


This icon button calls the method `_onCameraSwitch` when pressed. In this method, we will first dispose of the `CameraController` and then initialize the new `CameraController` with a new `CameraDescription`.

```
Future*<*void*> *_onCameraSwitch*() *async *{
  *final CameraDescription cameraDescription =
      *(*_controller.description == _cameras*[*0*]) *? _cameras*[*1*] *: _cameras*[*0*]*;
  if *(*_controller != null*) {
    *await _controller.dispose*()*;
  *}
  *_controller = CameraController*(*cameraDescription, ResolutionPreset.medium*)*;
  _controller.addListener*(() {
    *if *(*mounted*) *setState*(() {})*;
    if *(*_controller.value.hasError*) {
      *showInSnackBar*(*'Camera error *${*_controller.value.errorDescription*}*'*)*;
    *}
  })*;

  try *{
    *await _controller.initialize*()*;
  *} *on CameraException catch *(*e*) {
    *_showCameraException*(*e*)*;
  *}

  *if *(*mounted*) {
    *setState*(() {})*;
  *}
}*
```


## Camera Control view

At the bottom of the screen, we will have a control view which will basically contain 3 buttons. First to go to the gallery, second to capture images or record video, and third to switch between image capture and video recording.

```
return Scaffold*(
  *backgroundColor: Theme.*of(*context*)*.backgroundColor,
  key: _scaffoldKey,
  **extendBody: true,**
  body: ...
  **bottomNavigationBar: _buildBottomNavigationBar*()*,**
*)*;
```


The view will be shown in the bottom navigation bar. Don’t forget to add `extendBody: true.`

```
Widget _buildBottomNavigationBar*() {
  *return Container*(
    *color: Theme.*of(*context*)*.bottomAppBarColor,
    height: 100.0,
    width: double.*infinity*,
    child: Row*(
      *mainAxisAlignment: MainAxisAlignment.spaceAround,
      children: *<*Widget*>[
        *FutureBuilder*(
          *future: getLastImage*()*,
          builder: *(*context, snapshot*) {
            *if *(*snapshot.data == null*) {
              *return Container*(
                *width: 40.0,
                height: 40.0,
              *)*;
            *}
            *return GestureDetector*(
              *onTap: *() *=> Navigator.*push(
                *context,
                MaterialPageRoute*(
                  *builder: *(*context*) *=> Gallery*()*,
                *)*,
              *)*,
              child: Container*(
                *width: 40.0,
                height: 40.0,
                child: ClipRRect*(
                  *borderRadius: BorderRadius.circular*(*4.0*)*,
                  child: Image.file*(
                    *snapshot.data,
                    fit: BoxFit.cover,
                  *)*,
                *)*,
              *)*,
            *)*;
          *}*,
        *)*,
        CircleAvatar*(
          *backgroundColor: Colors.*white*,
          radius: 28.0,
          child: IconButton*(
            *icon: Icon*(
              (*_isRecordingMode*)
                  *? *(*_isRecording*) *? Icons.*stop *: Icons.*videocam
                  *: Icons.*camera_alt*,
              size: 28.0,
              color: *(*_isRecording*) *? Colors.*red *: Colors.*black*,
            *)*,
            onPressed: *() {
              *if *(*!_isRecordingMode*) {
                *_captureImage*()*;
              *} *else *{
                *if *(*_isRecording*) {
                  *stopVideoRecording*()*;
                *} *else *{
                  *startVideoRecording*()*;
                *}
              }
            }*,
          *)*,
        *)*,
        IconButton*(
          *icon: Icon*(
            (*_isRecordingMode*) *? Icons.*camera_alt *: Icons.*videocam*,
            color: Colors.*white*,
          *)*,
          onPressed: *() {
            *setState*(() {
              *_isRecordingMode = !_isRecordingMode;
            *})*;
          *}*,
        *)*,
      *]*,
    *)*,
  *)*;
*}*
```


## Capturing an Image

Capturing an image is pretty easy with the camera controller.

1. Check if the camera controller is initialized.

1. Construct a directory and defines the path.

1. Capture the image using CameraController and save it to the given path.

```
void _captureImage*() *async *{
  *if *(*_controller.value.isInitialized*) {
    *final Directory extDir = await getApplicationDocumentsDirectory*()*;
    final String dirPath = '*${*extDir.path*}*/media';
    await Directory*(*dirPath*)*.create*(*recursive: true*)*;
    final String filePath = '$dirPath/*${*_timestamp*()}*.jpeg';
    await _controller.takePicture*(*filePath*)*;
    setState*(() {})*;
  *}
}*
```


## Recording a video

We can divide the recording video process into two parts:

**Start video recording:**

1. Check if the camera controller is initialized.

1. Start timer to show the recorded video time. (optional)

1. Construct a directory and defines the path.

1. Start recording using the camera controller and saving the video on the defined path.

```
Future*<*String*> *startVideoRecording*() *async *{
  *print*(*'startVideoRecording'*)*;
  if *(*!_controller.value.isInitialized*) {
    *return null;
  *}
  *setState*(() {
    *_isRecording = true;
  *})*;
  _timerKey.currentState.startTimer*()*;

  final Directory extDir = await getApplicationDocumentsDirectory*()*;
  final String dirPath = '*${*extDir.path*}*/media';
  await Directory*(*dirPath*)*.create*(*recursive: true*)*;
  final String filePath = '$dirPath/*${*_timestamp*()}*.mp4';

  if *(*_controller.value.isRecordingVideo*) {
    *// A recording is already started, do nothing.
    return null;
  *}

  *try *{
    *await _controller.startVideoRecording*(*filePath*)*;
  *} *on CameraException catch *(*e*) {
    *_showCameraException*(*e*)*;
    return null;
  *}
  *return filePath;
*}*
```


**Stop video recording:**

1. Check if the camera controller is initialized.

1. Stop timer.

1. Stop video recording using the camera controller.

```
Future*<*void*> *stopVideoRecording*() *async *{
  *if *(*!_controller.value.isRecordingVideo*) {
    *return null;
  *}
  *_timerKey.currentState.stopTimer*()*;
  setState*(() {
    *_isRecording = false;
  *})*;

  try *{
    *await _controller.stopVideoRecording*()*;
  *} *on CameraException catch *(*e*) {
    *_showCameraException*(*e*)*;
    return null;
  *}
}*
```


Here is the full code for the camera screen.

```dart
import 'dart:io';

import 'package:camera/camera.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_camera/gallery.dart';
import 'package:flutter_camera/video_timer.dart';
import 'package:path/path.dart' as path;
import 'package:path_provider/path_provider.dart';
import 'package:thumbnails/thumbnails.dart';

class CameraScreen extends StatefulWidget {
  const CameraScreen({Key key}) : super(key: key);

  @override
  CameraScreenState createState() => CameraScreenState();
}

class CameraScreenState extends State<CameraScreen>
    with AutomaticKeepAliveClientMixin {
  CameraController _controller;
  List<CameraDescription> _cameras;
  final GlobalKey<ScaffoldState> _scaffoldKey = GlobalKey<ScaffoldState>();
  bool _isRecordingMode = false;
  bool _isRecording = false;
  final _timerKey = GlobalKey<VideoTimerState>();

  @override
  void initState() {
    _initCamera();
    super.initState();
  }

  Future<void> _initCamera() async {
    _cameras = await availableCameras();
    _controller = CameraController(_cameras[0], ResolutionPreset.medium);
    _controller.initialize().then((_) {
      if (!mounted) {
        return;
      }
      setState(() {});
    });
  }

  @override
  void dispose() {
    _controller?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    super.build(context);
    if (_controller != null) {
      if (!_controller.value.isInitialized) {
        return Container();
      }
    } else {
      return const Center(
        child: SizedBox(
          width: 32,
          height: 32,
          child: CircularProgressIndicator(),
        ),
      );
    }

    if (!_controller.value.isInitialized) {
      return Container();
    }
    return Scaffold(
      backgroundColor: Theme.of(context).backgroundColor,
      key: _scaffoldKey,
      extendBody: true,
      body: Stack(
        children: <Widget>[
          _buildCameraPreview(),
          Positioned(
            top: 24.0,
            left: 12.0,
            child: IconButton(
              icon: Icon(
                Icons.switch_camera,
                color: Colors.white,
              ),
              onPressed: () {
                _onCameraSwitch();
              },
            ),
          ),
          if (_isRecordingMode)
            Positioned(
              left: 0,
              right: 0,
              top: 32.0,
              child: VideoTimer(
                key: _timerKey,
              ),
            )
        ],
      ),
      bottomNavigationBar: _buildBottomNavigationBar(),
    );
  }

  Widget _buildCameraPreview() {
    final size = MediaQuery.of(context).size;
    return ClipRect(
      child: Container(
        child: Transform.scale(
          scale: _controller.value.aspectRatio / size.aspectRatio,
          child: Center(
            child: AspectRatio(
              aspectRatio: _controller.value.aspectRatio,
              child: CameraPreview(_controller),
            ),
          ),
        ),
      ),
    );
  }

  Widget _buildBottomNavigationBar() {
    return Container(
      color: Theme.of(context).bottomAppBarColor,
      height: 100.0,
      width: double.infinity,
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceAround,
        children: <Widget>[
          FutureBuilder(
            future: getLastImage(),
            builder: (context, snapshot) {
              if (snapshot.data == null) {
                return Container(
                  width: 40.0,
                  height: 40.0,
                );
              }
              return GestureDetector(
                onTap: () => Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => Gallery(),
                  ),
                ),
                child: Container(
                  width: 40.0,
                  height: 40.0,
                  child: ClipRRect(
                    borderRadius: BorderRadius.circular(4.0),
                    child: Image.file(
                      snapshot.data,
                      fit: BoxFit.cover,
                    ),
                  ),
                ),
              );
            },
          ),
          CircleAvatar(
            backgroundColor: Colors.white,
            radius: 28.0,
            child: IconButton(
              icon: Icon(
                (_isRecordingMode)
                    ? (_isRecording) ? Icons.stop : Icons.videocam
                    : Icons.camera_alt,
                size: 28.0,
                color: (_isRecording) ? Colors.red : Colors.black,
              ),
              onPressed: () {
                if (!_isRecordingMode) {
                  _captureImage();
                } else {
                  if (_isRecording) {
                    stopVideoRecording();
                  } else {
                    startVideoRecording();
                  }
                }
              },
            ),
          ),
          IconButton(
            icon: Icon(
              (_isRecordingMode) ? Icons.camera_alt : Icons.videocam,
              color: Colors.white,
            ),
            onPressed: () {
              setState(() {
                _isRecordingMode = !_isRecordingMode;
              });
            },
          ),
        ],
      ),
    );
  }

  Future<FileSystemEntity> getLastImage() async {
    final Directory extDir = await getApplicationDocumentsDirectory();
    final String dirPath = '${extDir.path}/media';
    final myDir = Directory(dirPath);
    List<FileSystemEntity> _images;
    _images = myDir.listSync(recursive: true, followLinks: false);
    _images.sort((a, b) {
      return b.path.compareTo(a.path);
    });
    var lastFile = _images[0];
    var extension = path.extension(lastFile.path);
    if (extension == '.jpeg') {
      return lastFile;
    } else {
      String thumb = await Thumbnails.getThumbnail(
          videoFile: lastFile.path, imageType: ThumbFormat.PNG, quality: 30);
      return File(thumb);
    }
  }

  Future<void> _onCameraSwitch() async {
    final CameraDescription cameraDescription =
        (_controller.description == _cameras[0]) ? _cameras[1] : _cameras[0];
    if (_controller != null) {
      await _controller.dispose();
    }
    _controller = CameraController(cameraDescription, ResolutionPreset.medium);
    _controller.addListener(() {
      if (mounted) setState(() {});
      if (_controller.value.hasError) {
        showInSnackBar('Camera error ${_controller.value.errorDescription}');
      }
    });

    try {
      await _controller.initialize();
    } on CameraException catch (e) {
      _showCameraException(e);
    }

    if (mounted) {
      setState(() {});
    }
  }

  void _captureImage() async {
    print('_captureImage');
    if (_controller.value.isInitialized) {
      SystemSound.play(SystemSoundType.click);
      final Directory extDir = await getApplicationDocumentsDirectory();
      final String dirPath = '${extDir.path}/media';
      await Directory(dirPath).create(recursive: true);
      final String filePath = '$dirPath/${_timestamp()}.jpeg';
      print('path: $filePath');
      await _controller.takePicture(filePath);
      setState(() {});
    }
  }

  Future<String> startVideoRecording() async {
    print('startVideoRecording');
    if (!_controller.value.isInitialized) {
      return null;
    }
    setState(() {
      _isRecording = true;
    });
    _timerKey.currentState.startTimer();

    final Directory extDir = await getApplicationDocumentsDirectory();
    final String dirPath = '${extDir.path}/media';
    await Directory(dirPath).create(recursive: true);
    final String filePath = '$dirPath/${_timestamp()}.mp4';

    if (_controller.value.isRecordingVideo) {
      // A recording is already started, do nothing.
      return null;
    }

    try {
//      videoPath = filePath;
      await _controller.startVideoRecording(filePath);
    } on CameraException catch (e) {
      _showCameraException(e);
      return null;
    }
    return filePath;
  }

  Future<void> stopVideoRecording() async {
    if (!_controller.value.isRecordingVideo) {
      return null;
    }
    _timerKey.currentState.stopTimer();
    setState(() {
      _isRecording = false;
    });

    try {
      await _controller.stopVideoRecording();
    } on CameraException catch (e) {
      _showCameraException(e);
      return null;
    }
  }

  String _timestamp() => DateTime.now().millisecondsSinceEpoch.toString();

  void _showCameraException(CameraException e) {
    logError(e.code, e.description);
    showInSnackBar('Error: ${e.code}\n${e.description}');
  }

  void showInSnackBar(String message) {
    _scaffoldKey.currentState.showSnackBar(SnackBar(content: Text(message)));
  }

  void logError(String code, String message) =>
      print('Error: $code\nError Message: $message');

  @override
  bool get wantKeepAlive => true;
}
```

## Gallery view

Our camera is complete and ready to go. But how do we see our captured images and recorded videos? We will create a gallery view. It will consist of a horizontal pageview and bottom app bar with a share button and a delete button.

Inside `PageView.builder` we are checking for the extension of the file. If the file extension is `jpeg`, we are showing it as an image, otherwise we will show the video using the `VideoPreview` widget.

```
String currentFilePath;
@override
Widget build*(*BuildContext context*) {
  *return Scaffold*(
    *backgroundColor: Theme.*of(*context*)*.backgroundColor,
    appBar: AppBar*(
      *backgroundColor: Colors.*black*,
    *)*,
    body: FutureBuilder*(
      *future: _getAllImages*()*,
      builder: *(*context, AsyncSnapshot*<*List*<*FileSystemEntity*>> *snapshot*) {
        *if *(*!snapshot.hasData || snapshot.data.isEmpty*) {
          *return Container*()*;
        *}
        *print*(*'*${*snapshot.data.length*} ${*snapshot.data*}*'*)*;
        if *(*snapshot.data.length == 0*) {
          *return Center*(
            *child: Text*(*'No images found.'*)*,
          *)*;
        *}

        *return PageView.builder*(
          *itemCount: snapshot.data.length,
          itemBuilder: *(*context, index*) {
            *currentFilePath = snapshot.data*[*index*]*.path;
            var extension = path.extension*(*snapshot.data*[*index*]*.path*)*;
            if *(*extension == '.jpeg'*) {
              *return Container*(
                *height: 300,
                padding: const EdgeInsets.only*(*bottom: 8.0*)*,
                child: Image.file*(
                  *File*(*snapshot.data*[*index*]*.path*)*,
                *)*,
              *)*;
            *} *else *{
              *return VideoPreview*(
                *videoPath: snapshot.data*[*index*]*.path,
              *)*;
            *}
          }*,
        *)*;
      *}*,
    *)*,
    bottomNavigationBar: BottomAppBar*(
      *child: Container*(
        *height: 56.0,
        child: Row*(
          *mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: *<*Widget*>[
            *IconButton*(
              *icon: Icon*(*Icons.*share)*,
              onPressed: *() *=> _shareFile*()*,
            *)*,
            IconButton*(
              *icon: Icon*(*Icons.*delete)*,
              onPressed: _deleteFile,
            *)*,
          *]*,
        *)*,
      *)*,
    *)*,
  *)*;
*}*
```


### **Fetching media files from device**

```
Future*<*List*<*FileSystemEntity*>> *_getAllImages*() *async *{
  *final Directory extDir = await getApplicationDocumentsDirectory*()*;
  final String dirPath = '*${*extDir.path*}*/media';
  final myDir = Directory*(*dirPath*)*;
  List*<*FileSystemEntity*> *_images;
  _images = myDir.listSync*(*recursive: true, followLinks: false*)*;
  _images.sort*((*a, b*) {
    *return b.path.compareTo*(*a.path*)*;
  *})*;
  return _images;
*}*
```


### Deleting media file

Deleting the file is pretty easy. Just point the directory to the file path and delete it using the`deleteSync` function.

```
_deleteFile*() {
  *final dir = Directory*(*currentFilePath*)*;
  dir.deleteSync*(*recursive: true*)*;
  setState*(() {})*;
*}*
```


### Sharing media file

For sharing the file we use `esys_flutter_share` plugin. You can easily share a file using `Share.file()` method which takes a String `title`, String `name`, List*&lt;*int*&gt; `*bytes`, String `mimeType` as mandatory params. You can get the bytes from a file using `readAsBytesSync` method.

```
_shareFile*() *async *{
  *var extension = path.extension*(*currentFilePath*)*;
  await Share.*file(
    *'image',
    *(*extension == '.jpeg'*) *? 'image.jpeg' : '  video.mp4',
    File*(*currentFilePath*)*.readAsBytesSync*()*,
    *(*extension == '.jpeg'*) *? 'image/jpeg' : '  video/mp4',
  *)*;
*}*
```


## My views on camera plugin

Before we move to a conclusion we should know that the Flutter Camera plugin is still under development. The plugin is good to go for making any decent camera app but it is a little buggy and missing lots of advanced features like automatic exposure and flash support. If you want to be updated about the upcoming changes in camera plugin keep an eye on issue [Future of the Camera Plugin](https://github.com/flutter/flutter/issues/31225). This issue talks about some cool features which are about to come in the camera plugin.


**You can see the full source code of the project [here](https://github.com/divyanshub024/flutter_camera).**
[**divyanshub024/flutter_camera**
*A new Flutter application. This project is a starting point for a Flutter application. A few resources to get you…*github.com](https://github.com/divyanshub024/flutter_camera)

**If you liked this article make sure to 👏 it below, and connect with me on [Twitter](https://twitter.com/divyanshub024), [Github](https://github.com/divyanshub024) and [LinkedIn](https://www.linkedin.com/in/divyanshub024/).**