# extended_image

[![pub package](https://img.shields.io/pub/v/extended_image.svg)](https://pub.dartlang.org/packages/extended_image) [![GitHub stars](https://img.shields.io/github/stars/fluttercandies/extended_image)](https://github.com/fluttercandies/extended_image/stargazers) [![GitHub forks](https://img.shields.io/github/forks/fluttercandies/extended_image)](https://github.com/fluttercandies/extended_image/network) [![GitHub license](https://img.shields.io/github/license/fluttercandies/extended_image)](https://github.com/fluttercandies/extended_image/blob/master/LICENSE) [![GitHub issues](https://img.shields.io/github/issues/fluttercandies/extended_image)](https://github.com/fluttercandies/extended_image/issues) <a target="_blank" href="https://jq.qq.com/?_wv=1027&k=5bcc0gy"><img border="0" src="https://pub.idqqimg.com/wpa/images/group.png" alt="flutter-candies" title="flutter-candies"></a>

Language: [English](README.md) | [中文简体](README-ZH.md)

A powerful library for displaying and editing images in Flutter. In addition to the features provided by Flutter's built-in `Image` widget, `ExtendedImage` supports placeholder/failed states for image-loading, cached network images, zooming/panning, several types of photo pages, editing (crop/rotate/flip), custom canvas painting, and more.

**[Web Demo](https://fluttercandies.github.io/extended_image/)**

## Table of contents

- [Cached network images](#cached-network-images)
  - [Using the widget](#simple-use)
  - [Creating an ImageProvider](#use-extendednetworkimageprovider)
- [Loading states](#load-state)
  - [Example](#demo-code)
- [Zooming and panning](#zoom-pan)
  - [Double-tap animation](#double-tap-animation)
- [Image editor](#editor)
  - [Aspect ratio](#crop-aspect-ratio)
  - [Rotate, flip and reset](#cropflipreset)
  - [Crop](#crop-data)
    - [Dart library (stable)](#dart-librarystable)
    - [Native library (faster)](#native-libraryfaster)
- [Page views](#photo-view)
- [Slide-out page](#slide-out-page)
  - [enable slide out page](#enable-slide-out-page)
  - [include your page in ExtendedImageSlidePage](#include-your-page-in-extendedimageslidepage)
  - [make sure your page background is transparent](#make-sure-your-page-background-is-transparent)
  - [push with transparent page route](#push-with-transparent-page-route)
- [Border BorderRadius Shape](#border-borderradius-shape)
- [Clear Save](#clear-save)
  - [Clear](#clear)
  - [Save Network](#save-network)
- [Show Crop Image](#show-crop-image)
- [Paint](#paint)
- [WaterfallFlow](#waterfallflow)
- [CollectGarbage/viewportBuilder](#collectgarbageviewportbuilder)
- [Other APIs](#other-apis)

## Cached Network Images

### Using the Widget

You can use `ExtendedImage.network` like a regular `Image` Widget:

```dart
ExtendedImage.network(
  url,
  width: ScreenUtil.instance.setWidth(400),
  height: ScreenUtil.instance.setWidth(400),
  fit: BoxFit.fill,
  cache: true,
  border: Border.all(color: Colors.red, width: 1.0),
  shape: boxShape,
  borderRadius: BorderRadius.all(Radius.circular(30.0)),
  // cancelToken: cancellationToken,
)
```

### Creating an ImageProvider

If you need to work directly with an ImageProvider, you can create a new [ExtendedNetworkImageProvider](https://github.com/fluttercandies/extended_image_library/blob/master/lib/src/extended_network_image_provider.dart):

```dart
ExtendedNetworkImageProvider(
  this.url, {
  this.scale = 1.0,
  this.headers,
  this.cache: false,
  this.retries = 3,
  this.timeLimit,
  this.timeRetry = const Duration(milliseconds: 100),
  CancellationToken cancelToken,
})  : assert(url != null),
      assert(scale != null),
      cancelToken = cancelToken ?? CancellationToken();
```

| Parameter   | Description                                                                              | Default             |
| ----------- | ---------------------------------------------------------------------------------------- | ------------------- |
| url         | The URL from which the image will be fetched                                             | required            |
| scale       | The scale to set in the `ImageInfo` object of the image                                  | 1.0                 |
| headers     | The HTTP headers that will be used with `HttpClient.get` to fetch the image from network | -                   |
| cache       | Whether cache image locally                                                              | false               |
| retries     | The number of times to retry failed requests                                             | 3                   |
| timeLimit   | Time limit to request image                                                              | -                   |
| timeRetry   | The time duration to retry to request                                                    | milliseconds: 100   |
| cancelToken | Token to cancel network request                                                          | CancellationToken() |

You can create new provider, extend it with `ExtendedProvider`, and override the `instantiateImageCodec` method in order to handle image raw data (e.g. to compress an image).

## Loading/Completed/Failed States

`ExtendedImage` provides three image states: loading, completed, and failed. You can use these states to handle each scenario appropriately.

![img](https://github.com/fluttercandies/Flutter_Candies/blob/master/gif/extended_image/custom.gif)

Provide a `loadStateChanged` callback to listen for an image's change in state. By default, this is only called for network images. To enable it for other image types (for example, larger images that take time to load), set `enableLoadState` to true.

Notes:
- If you want to override size or sourceRect, you can override it with `ExtendedRawImage` at the completed state

- If you want to add something (like animation) at the completed state, you can override it with `ExtendedImageState.completedWidget`

- `ExtendedImageState.completedWidget` is include gesture or editor, so that you would't miss them


ExtendedImageState(LoadStateChanged callback)

| Parameter/Method             | Description                                                                                                                                   | default |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| extendedImageInfo            | Image info                                                                                                                                    | -       |
| extendedImageLoadState       | LoadState(loading,completed,failed)                                                                                                           | -       |
| returnLoadStateChangedWidget | If this is true, return widget which from LoadStateChanged function immediately(width/height/gesture/border/shape etc, will not effect on it) | -       |
| imageProvider                | ImageProvider                                                                                                                                 | -       |
| invertColors                 | invertColors                                                                                                                                  | -       |
| imageStreamKey               | Key of image                                                                                                                                  | -       |
| reLoadImage()                | Attempts to load the image again if it failed the first time                                                                                  | -       |
| completedWidget              | Return completed widget include gesture or editor                                                                                             | -       |
| loadingProgress              | Return the loading progress for network image (ImageChunkEvent )                                                                              | -       |

```dart
abstract class ExtendedImageState {
  void reLoadImage();
  ImageInfo get extendedImageInfo;
  LoadState get extendedImageLoadState;

  ///return widget which from LoadStateChanged function immediately
  bool returnLoadStateChangedWidget;

  ImageProvider get imageProvider;

  bool get invertColors;

  Object get imageStreamKey;

  Widget get completedWidget;
}
```

### Example

```dart
ExtendedImage.network(
  url,
  width: ScreenUtil.instance.setWidth(600),
  height: ScreenUtil.instance.setWidth(400),
  fit: BoxFit.fill,
  cache: true,
  loadStateChanged: (ExtendedImageState state) {
    switch (state.extendedImageLoadState) {
      case LoadState.loading:
        _controller.reset();
        return Image.asset(
          "assets/loading.gif",
          fit: BoxFit.fill,
        );
        break;

      /// If you don't want override completed widget,
      /// return null or state.completedWidget.
      case LoadState.completed:
        _controller.forward();
        return FadeTransition(
          opacity: _controller,
          child: ExtendedRawImage(
            image: state.extendedImageInfo?.image,
            width: ScreenUtil.instance.setWidth(600),
            height: ScreenUtil.instance.setWidth(400),
          ),
        );
        break;
      case LoadState.failed:
        _controller.reset();
        return GestureDetector(
          child: Stack(
            fit: StackFit.expand,
            children: <Widget>[
              Image.asset(
                "assets/failed.jpg",
                fit: BoxFit.fill,
              ),
              Positioned(
                bottom: 0.0,
                left: 0.0,
                right: 0.0,
                child: Text(
                  "load image failed, click to reload",
                  textAlign: TextAlign.center,
                ),
              )
            ],
          ),
          onTap: () {
            state.reLoadImage();
          },
        );
        break;
    }
  },
)
```

## Zoom/Pan

![img](https://github.com/fluttercandies/Flutter_Candies/blob/master/gif/extended_image/zoom.gif)

`ExtendedImage`

| Parameter                | Description                                                                     | Default |
| ------------------------ | ------------------------------------------------------------------------------- | ------- |
| mode                     | image mode (none, gesture, editor)                                              | none    |
| initGestureConfigHandler | init GestureConfig when image is ready，for example, base on image width/height | -       |
| onDoubleTap              | call back of double tap under ExtendedImageMode.gesture                         | -       |
| extendedImageGestureKey  | you can handle zoom/pan by using this key manually                              | -       |

`GestureConfig`

| Parameter         | Description                                                                                                                                                          | default                 |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| minScale          | Min scale                                                                                                                                                            | 0.8                     |
| animationMinScale | The min scale for zooming then animation back to minScale when scale end                                                                                             | minScale \_ 0.8         |
| maxScale          | Max scale                                                                                                                                                            | 5.0                     |
| animationMaxScale | The max scale for zooming then animation back to maxScale when scale end                                                                                             | maxScale \_ 1.2         |
| speed             | Speed for zoom/pan                                                                                                                                                   | 1.0                     |
| inertialSpeed     | Inertial speed for zoom/pan                                                                                                                                          | 100                     |
| cacheGesture      | Save Gesture state (for example in ExtendedImageGesturePageView, gesture state will not change when scroll back),**remember clearGestureDetailsCache at right time** | false                   |
| inPageView        | Whether in ExtendedImageGesturePageView                                                                                                                              | false                   |
| initialAlignment  | Init image rect with alignment when initialScale > 1.0                                                                                                               | InitialAlignment.center |

```dart
ExtendedImage.network(
  imageTestUrl,
  fit: BoxFit.contain,
  //enableLoadState: false,
  mode: ExtendedImageMode.gesture,
  initGestureConfigHandler: (state) {
    return GestureConfig(
        minScale: 0.9,
        animationMinScale: 0.7,
        maxScale: 3.0,
        animationMaxScale: 3.5,
        speed: 1.0,
        inertialSpeed: 100.0,
        initialScale: 1.0,
        inPageView: false,
        initialAlignment: InitialAlignment.center,
    );
  },
)
```

### Double-Tap Animation

You can pass a callback to the widget's `onDoubleTap` parameter. Here's an example of using the callback to change the image's scale.

```dart
onDoubleTap: (ExtendedImageGestureState state) {
  // Clean up the old animation state.
  _animation?.removeListener(animationListener);
  _animationController.stop();
  _animationController.reset();

  // Choose a scale to animate to.
  if (begin == doubleTapScales[0]) {
    end = doubleTapScales[1];
  } else {
    end = doubleTapScales[0];
  }

  // You can define doubleTapPosition as any offset.
  // The default value is the double-tap pointer down postion.
  var pointerDownPosition = state.pointerDownPosition;
  double begin = state.gestureDetails.totalScale;
  double end;

  animationListener = () {
    state.handleDoubleTap(
        scale: _animation.value,
        doubleTapPosition: pointerDownPosition);
  };

  _animation = _animationController
      .drive(Tween<double>(begin: begin, end: end));

  _animation.addListener(animationListener);
  _animationController.forward();
},
```

## Image Editor

![img](https://github.com/fluttercandies/Flutter_Candies/blob/master/gif/extended_image/editor.gif)

`ExtendedImage` provides editor tools to crop, rotate, and flip images. Here's an example of an editable network image:

```dart
ExtendedImage.network(
  imageTestUrl,
  fit: BoxFit.contain,
  mode: ExtendedImageMode.editor,
  extendedImageEditorKey: editorKey,
  initEditorConfigHandler: (state) {
    return EditorConfig(
        maxScale: 8.0,
        cropRectPadding: EdgeInsets.all(20.0),
        hitTestSize: 20.0,
        cropAspectRatio: _aspectRatio.aspectRatio);
  },
);
```

ExtendedImage

| Parameter               | Description                                                  | Default |
| ----------------------- | ------------------------------------------------------------ | ------- |
| mode                    | Image mode (none, gesture, editor)                           | none    |
| initEditorConfigHandler | Callback to create EditorConfig when the image is ready.     | -       |
| extendedImageEditorKey  | Key of ExtendedImageEditorState to flip/rotate/get crop rect | -       |

EditorConfig

| parameter              | description                                                        | default                                                      |
| ---------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------ |
| maxScale               | Max scale of zoom                                                  | 5.0                                                          |
| cropRectPadding        | The padding between crop rect and image layout rect.               | EdgeInsets.all(20.0)                                         |
| cornerSize             | Size of corner shape                                               | Size(30.0, 5.0)                                              |
| cornerColor            | Color of the corner shape                                          | primaryColor                                                 |
| lineColor              | Color of the crop line                                             | scaffoldBackgroundColor.withOpacity(0.7)                     |
| lineHeight             | Height of the crop line                                            | 0.6                                                          |
| editorMaskColorHandler | Callback of editor mask color base on pointerDown                  | scaffoldBackgroundColor.withOpacity(pointerDown ? 0.4 : 0.8) |
| hitTestSize            | Hit test region of corner and line                                 | 20.0                                                         |
| animationDuration      | Auto-center animation duration                                     | Duration(milliseconds: 200)                                  |
| tickerDuration         | Duration to begin auto-center animation after crop rect is changed | Duration(milliseconds: 400)                                  |
| cropAspectRatio        | Aspect ratio of the crop rect                                      | null(custom)                                                 |
| initCropRectType       | Init crop rect based on initial image rect or image layout rect    | imageRect                                                    |

### Aspect Ratio

The crop aspect ratio is of type `double`, so you can provide any value you choose, but this package also comes with some presets:

```dart
class CropAspectRatios {
  // No aspect ratio specified.
  static const double custom = null;

  // If [cropAspectRatio] is <= 0.0, the image's original aspect ratio is used.
  static const double original = 0.0;

  // Below, several standard aspect ratios are defined.
  static const double ratio1_1 = 1.0;
  static const double ratio3_4 = 3.0 / 4.0;
  static const double ratio4_3 = 4.0 / 3.0;
  static const double ratio9_16 = 9.0 / 16.0;
  static const double ratio16_9 = 16.0 / 9.0;
}
```

### Rotate/Flip/Reset

1. Define a global key which can be passed to `extendedImageEditorKey`:

  ```dart
  final GlobalKey<ExtendedImageEditorState> editorKey = GlobalKey<ExtendedImageEditorState>();
  ```

2. Perform edit operations on the editor state:

```dart
editorKey.currentState.rotate(right: true); // Rotate right
editorKey.currentState.rotate(right: false); // Rotate left

editorKey.currentState.flip();
editorKey.currentState.reset();
```

### Crop Data

There are two ways to crop image data using this package: the Dart library (stable) and the native library (faster).

#### Dart Library (stable)

1. Add [Image](https://github.com/brendan-duncan/image) library into your pubspec.yaml. It's used to crop/rotate/flip image data:

```yaml
dependencies:
  image: any
```

2. Get the crop rect and raw image data from ExtendedImageEditorState:

```dart
  final Rect cropRect = state.getCropRect();
  var data = state.rawImageData;
```

3. Convert raw image data to image library data:

```dart
  // AVOID:
  // This takes time and will block the UI.
  Image src = decodeImage(data);

  // PREFER:
  // These approaches run asynchronously and will not block the UI.
  final lb = await loadBalancer;
  Image src = await lb.run<Image, List<int>>(decodeImage, data);
  Image src = await compute(decodeImage, data);
  Image src = await isolateDecodeImage(data);  
```

4. Crop/flip/rotate data:

```dart
  // Clear orientation
  src = bakeOrientation(src);

  if (editAction.needCrop)
    src = copyCrop(src, cropRect.left.toInt(), cropRect.top.toInt(),
        cropRect.width.toInt(), cropRect.height.toInt());

  if (editAction.needFlip) {
    Flip mode;

    if (editAction.flipY && editAction.flipX) {
      mode = Flip.both;
    } else if (editAction.flipY) {
      mode = Flip.horizontal;
    } else if (editAction.flipX) {
      mode = Flip.vertical;
    }

    src = flip(src, mode);
  }

  if (editAction.hasRotateAngle)
    src = copyRotate(src, editAction.rotateAngle);
```

5. Convert to original image data

The output here is raw image data. You can save it or do whatever else you like.

```dart
  // AVOID:
  // This costs much time and blocks the UI.
  var fileData = encodeJpg(src);

  // PREFER:
  // These will run asynchronously and will not block the UI.
  var fileData = await compute(encodeJpg, src);
  var fileData = await isolateEncodeImage(src);
  var fileData = await lb.run<List<int>, Image>(encodeJpg, src);
```

#### Native Library (faster)

1. Add [ImageEditor](https://github.com/fluttercandies/flutter_image_editor) library into your pubspec.yaml. It's used to crop/rotate/flip image data:

```yaml
dependencies:
  image_editor: any
```

2. Get the crop rect and raw image data from ExtendedImageEditorState:

```dart
  final Rect cropRect = state.getCropRect();
  var data = state.rawImageData;
```

3. Prepare the image editing options:

```dart
  final rotateAngle = action.rotateAngle.toInt();
  final flipHorizontal = action.flipY;
  final flipVertical = action.flipX;
  final img = state.rawImageData;

  ImageEditorOption option = ImageEditorOption();

  if (action.needCrop)
    option.addOption(ClipOption.fromRect(rect));

  if (action.needFlip)
    option.addOption(
        FlipOption(horizontal: flipHorizontal, vertical: flipVertical));

  if (action.hasRotateAngle)
    option.addOption(RotateOption(rotateAngle));
```

4. Apply the options with `ImageEditor.editImage`

```dart
  final result = await ImageEditor.editImage(
    image: img,
    imageEditorOption: option,
  );
```
Like the Dart library, the output here is raw image data. You can save it or do whatever else you like.

[More details](https://github.com/fluttercandies/extended_image/blob/master/example/lib/common/crop_editor_helper.dart)

## Page Views

`ExtendedImageGesturePageView` is a wrapper around Flutter's built-in `PageView`, specialized for zooming and panning images.

![img](https://github.com/fluttercandies/Flutter_Candies/blob/master/gif/extended_image/photo_view.gif)

`GestureConfig`

| Parameter    | Description                                                         | Default |
| ------------ | ------------------------------------------------------------------- | ------- |
| cacheGesture | Save the gesture state when navigating between screens.<sup>1</sup> | false   |
| inPageView   | Whether the config is in a ExtendedImageGesturePageView             | false   |


<sup>1</sup> If you use `cacheGesture`, remember to call clearGestureDetailsCache() at the right time (for example, when your page view page is disposed).

```dart
ExtendedImageGesturePageView.builder(
  itemBuilder: (BuildContext context, int index) {
    var item = widget.pics[index].picUrl;
    Widget image = ExtendedImage.network(
      item,
      fit: BoxFit.contain,
      mode: ExtendedImageMode.gesture,
      gestureConfig: GestureConfig(
        inPageView: true,
        initialScale: 1.0,
        cacheGesture: false
      ),
    );
    image = Container(
      child: image,
      padding: EdgeInsets.all(5.0),
    );
    if (index == currentIndex) {
      return Hero(
        tag: item + index.toString(),
        child: image,
      );
    } else {
      return image;
    }
  },
  itemCount: widget.pics.length,
  onPageChanged: (int index) {
    currentIndex = index;
    rebuild.add(index);
  },
  controller: PageController(
    initialPage: currentIndex,
  ),
  scrollDirection: Axis.horizontal,
),
```

## Slide-Out Page

This package supports slide-out pages like the ones in WeChat.

![img](https://raw.githubusercontent.com/fluttercandies/Flutter_Candies/master/gif/extended_image/slide.gif)

`ExtendedImage`

| Parameter                 | Description                                   | Default |
| ------------------------- | --------------------------------------------- | ------- |
| enableSlideOutPage        | Whether slide-out page is enabled             | false   |
| heroBuilderForSlidingPage | Hero builder for the sliding page<sup>1</sup> | null    |

<sup>1</sup> The transform of sliding page must act on Hero, so that the hero Hero animation doesn't look strange when the page is popped.


Using the `onSlidingPage` callback, you can update other widgets' state. Take care not to call `setState` directly here, as the image state would change. You should only notify the widgets which need to change.

```dart
return ExtendedImageSlidePage(
  child: result,
  slideAxis: SlideAxis.both,
  slideType: SlideType.onlyImage,
  onSlidingPage: (state) {
    // You can change other widgets' state on the page based on offset, isSliding, etc.
    // var offset = state.offset;
    
    var showSwiper = !state.isSliding;
    if (showSwiper != _showSwiper) {
      // DON'T DO THIS (see above):
      // setState(() {
      // _showSwiper = showSwiper;
      // });

      _showSwiper = showSwiper;
      rebuildSwiper.add(_showSwiper);
    }
  },
);
```

ExtendedImageGesturePage

| Parameter                  | Description                                                                      | Default                           |
| -------------------------- | -------------------------------------------------------------------------------- | --------------------------------- |
| child                      | The [child] contained by the ExtendedImageGesturePage.                           | -                                 |
| slidePageBackgroundHandler | build background when slide page                                                 | defaultSlidePageBackgroundHandler |
| slideScaleHandler          | customize scale of page when slide page                                          | defaultSlideScaleHandler          |
| slideEndHandler            | call back of slide end,decide whether pop page                                   | defaultSlideEndHandler            |
| slideAxis                  | axis of slide(both,horizontal,vertical)                                          | SlideAxis.both                    |
| resetPageDuration          | reset page position when slide end(not pop page)                                 | milliseconds: 500                 |
| slideType                  | slide whole page or only image                                                   | SlideType.onlyImage               |
| onSlidingPage              | call back when it's sliding page, change other widgets state on page as you want | -                                 |
| slideOffsetHandler         | customize offset when slide page                                                 | -                                 |

```dart
Color defaultSlidePageBackgroundHandler(
    {Offset offset, Size pageSize, Color color, SlideAxis pageGestureAxis}) {
  double opacity = 0.0;
  if (pageGestureAxis == SlideAxis.both) {
    opacity = offset.distance /
        (Offset(pageSize.width, pageSize.height).distance / 2.0);
  } else if (pageGestureAxis == SlideAxis.horizontal) {
    opacity = offset.dx.abs() / (pageSize.width / 2.0);
  } else if (pageGestureAxis == SlideAxis.vertical) {
    opacity = offset.dy.abs() / (pageSize.height / 2.0);
  }
  return color.withOpacity(min(1.0, max(1.0 - opacity, 0.0)));
}

bool defaultSlideEndHandler(
    {Offset offset, Size pageSize, SlideAxis pageGestureAxis}) {
  if (pageGestureAxis == SlideAxis.both) {
    return offset.distance >
        Offset(pageSize.width, pageSize.height).distance / 3.5;
  } else if (pageGestureAxis == SlideAxis.horizontal) {
    return offset.dx.abs() > pageSize.width / 3.5;
  } else if (pageGestureAxis == SlideAxis.vertical) {
    return offset.dy.abs() > pageSize.height / 3.5;
  }
  return true;
}

double defaultSlideScaleHandler(
    {Offset offset, Size pageSize, SlideAxis pageGestureAxis}) {
  double scale = 0.0;
  if (pageGestureAxis == SlideAxis.both) {
    scale = offset.distance / Offset(pageSize.width, pageSize.height).distance;
  } else if (pageGestureAxis == SlideAxis.horizontal) {
    scale = offset.dx.abs() / (pageSize.width / 2.0);
  } else if (pageGestureAxis == SlideAxis.vertical) {
    scale = offset.dy.abs() / (pageSize.height / 2.0);
  }
  return max(1.0 - scale, 0.8);
}
```

### make sure your page background is transparent

if you use ExtendedImageSlidePage and slideType =SlideType.onlyImage,
make sure your page background is transparent

### push with transparent page route

you should push page with TransparentMaterialPageRoute/TransparentCupertinoPageRoute

```dart
  Navigator.push(
    context,
    Platform.isAndroid
        ? TransparentMaterialPageRoute(builder: (_) => page)
        : TransparentCupertinoPageRoute(builder: (_) => page),
  );
```

[Slide Out Page Demo Code 1](https://github.com/fluttercandies/flutter_candies_demo_library/blob/master/lib/src/widget/crop_image.dart)

[Slide Out Page Demo Code 2](https://github.com/fluttercandies/flutter_candies_demo_library/blob/master/lib/src/widget/pic_swiper.dart)

## Border BorderRadius Shape

ExtendedImage

| parameter    | description                                                                                                                                                             | default |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| border       | BoxShape.circle and BoxShape.rectangle,If this is [BoxShape.circle] then [borderRadius] is ignored.                                                                     | -       |
| borderRadius | If non-null, the corners of this box are rounded by this [BorderRadius].,Applies only to boxes with rectangular shapes; ignored if [shape] is not [BoxShape.rectangle]. | -       |
| shape        | BoxShape.circle and BoxShape.rectangle,If this is [BoxShape.circle] then [borderRadius] is ignored.                                                                     | -       |

```dart
ExtendedImage.network(
  url,
  width: ScreenUtil.instance.setWidth(400),
  height: ScreenUtil.instance.setWidth(400),
  fit: BoxFit.fill,
  cache: true,
  border: Border.all(color: Colors.red, width: 1.0),
  shape: boxShape,
  borderRadius: BorderRadius.all(Radius.circular(30.0)),
),
```

![img](https://raw.githubusercontent.com/fluttercandies/Flutter_Candies/master/gif/extended_image/image.gif)

## Clear Save

### clear

to clear disk cached , call clearDiskCachedImages method.

```dart
// Clear the disk cache directory then return if it succeed.
///  <param name="duration">timespan to compute whether file has expired or not</param>
Future<bool> clearDiskCachedImages({Duration duration})
```

to clear disk cached with specific url, call clearDiskCachedImage method.

```dart
/// clear the disk cache image then return if it succeed.
///  <param name="url">clear specific one</param>
Future<bool> clearDiskCachedImage(String url) async {
```

get the local cached image file

```dart
Future<File> getCachedImageFile(String url) async {
```

to clear memory cache , call clearMemoryImageCache method.

```dart
///clear all of image in memory
 clearMemoryImageCache();

/// get ImageCache
 getMemoryImageCache() ;
```

### save network

call saveNetworkImageToPhoto and save image with [image_picker_saver](https://github.com/cnhefang/image_picker_saver)

```dart
///save network image to photo
Future<bool> saveNetworkImageToPhoto(String url, {bool useCache: true}) async {
  var data = await getNetworkImageData(url, useCache: useCache);
  var filePath = await ImagePickerSaver.saveFile(fileData: data);
  return filePath != null && filePath != "";
}
```

## Show Crop Image

get your raw image by [Load State](#Load State), and crop image by sourceRect.

[ExtendedRawImage](https://github.com/fluttercandies/extended_image/blob/master/lib/src/image/extended_raw_image.dart)
sourceRect is which you want to show image rect.

![img](https://raw.githubusercontent.com/fluttercandies/Flutter_Candies/master/gif/extended_image/crop.gif)

```dart
ExtendedRawImage(
  image: image,
  width: num400,
  height: num300,
  fit: BoxFit.fill,
  sourceRect: Rect.fromLTWH(
      (image.width - width) / 2.0, 0.0, width, image.height.toDouble()),
)
```

## Paint

provide BeforePaintImage and AfterPaintImage callback, you will have the chance to paint things you want.

![img](https://raw.githubusercontent.com/fluttercandies/Flutter_Candies/master/gif/extended_image/paint.gif)

ExtendedImage

| parameter        | description                                            | default |
| ---------------- | ------------------------------------------------------ | ------- |
| beforePaintImage | you can paint anything if you want before paint image. | -       |
| afterPaintImage  | you can paint anything if you want after paint image.  | -       |

```dart
  ExtendedImage.network(
    url,
    width: ScreenUtil.instance.setWidth(400),
    height: ScreenUtil.instance.setWidth(400),
    fit: BoxFit.fill,
    cache: true,
    beforePaintImage: (Canvas canvas, Rect rect, ui.Image image) {
      if (paintType == PaintType.ClipHeart) {
        if (!rect.isEmpty) {
          canvas.save();
          canvas.clipPath(clipheart(rect, canvas));
        }
      }
      return false;
    },
    afterPaintImage: (Canvas canvas, Rect rect, ui.Image image) {
      if (paintType == PaintType.ClipHeart) {
        if (!rect.isEmpty) canvas.restore();
      } else if (paintType == PaintType.PaintHeart) {
        canvas.drawPath(
            clipheart(rect, canvas),
            Paint()
              ..colorFilter =
                  ColorFilter.mode(Color(0x55ea5504), BlendMode.srcIn)
              ..isAntiAlias = false
              ..filterQuality = FilterQuality.low);
      }
    },
  );
```

see [paint image demo](https://github.com/fluttercandies/extended_image/blob/master/example/lib/pages/paint_image_demo.dart)
and [push to refresh header which is used in crop image demo]https://github.com/fluttercandies/flutter_candies_demo_library/blob/master/lib/src/widget/push_to_refresh_header.dart)

## WaterfallFlow

build WaterfallFlow with [LoadingMoreList](https://github.com/fluttercandies/loading_more_list) or [WaterfallFlow](https://github.com/fluttercandies/waterfall_flow) with ExtendedImage.

![img](https://github.com/fluttercandies/flutter_candies/tree/master/gif/waterfall_flow/known_sized.gif)

```dart
            LoadingMoreList(
              ListConfig<TuChongItem>(
                waterfallFlowDelegate: WaterfallFlowDelegate(
                  crossAxisCount: 2,
                  crossAxisSpacing: 5,
                  mainAxisSpacing: 5,
                ),
                itemBuilder: buildWaterfallFlowItem,
                sourceList: listSourceRepository,
                padding: EdgeInsets.all(5.0),
                lastChildLayoutType: LastChildLayoutType.foot,
              ),
            ),
```

## CollectGarbage/viewportBuilder

you can collect garbage when item is dispose or viewport indexes is changed.

more details, [LoadingMoreList](https://github.com/fluttercandies/loading_more_list), [WaterfallFlow](https://github.com/fluttercandies/waterfall_flow) and [ExtendedList](https://github.com/fluttercandies/extended_list)

```dart
            LoadingMoreList(
              ListConfig<TuChongItem>(
                waterfallFlowDelegate: WaterfallFlowDelegate(
                  crossAxisCount: 2,
                  crossAxisSpacing: 5,
                  mainAxisSpacing: 5,
                ),
                itemBuilder: buildWaterfallFlowItem,
                sourceList: listSourceRepository,
                padding: EdgeInsets.all(5.0),
                lastChildLayoutType: LastChildLayoutType.foot,
                collectGarbage: (List<int> garbages) {
                  ///collectGarbage
                  garbages.forEach((index) {
                    final provider = ExtendedNetworkImageProvider(
                      listSourceRepository[index].imageUrl,
                    );
                    provider.evict();
                  });
                  //print("collect garbage : $garbages");
                },
                viewportBuilder: (int firstIndex, int lastIndex) {
                  print("viewport : [$firstIndex,$lastIndex]");
                },
              ),
            ),
```

## Other APIs

ExtendedImage

| parameter                   | description                                                                                    | default |
| --------------------------- | ---------------------------------------------------------------------------------------------- | ------- |
| enableMemoryCache           | whether cache in PaintingBinding.instance.imageCache)                                          | true    |
| clearMemoryCacheIfFailed    | when failed to load image, whether clear memory cache.if true, image will reload in next time. | true    |
| clearMemoryCacheWhenDispose | when image is removed from the tree permanently, whether clear memory cache.                   | false   |
