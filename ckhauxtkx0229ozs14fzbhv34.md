## Flutter Animation I: Background Color Transition


Photo by Cris Ovalle on Unsplash

Well, animations are pretty important. It can make a monotonous UI feel very interesting. One reason I love Flutter is that it’s very easy to implement animations in Flutter.

This is the first blog in Flutter Animation Series. Before we deep dive into animation let’s know a little bit about it.

Flutter animations are of two types:
Tween animation and Physics-based animation.
> **Tween animation**
> Short for *in-betweening*. In a tween animation, the beginning and ending points are defined, as well as a timeline, and a curve that defines the timing and speed of the transition. The framework calculates how to transition from the beginning point to the end point.
> **Physics-based animation**
> In physics-based animation, motion is modeled to resemble real-world behavior. When you toss a ball, for example, where and when it lands depends on how fast it was tossed and how far it was from the ground. Similarly, dropping a ball attached to a spring falls (and bounces) differently than dropping a ball attached to a string.

To know more about animation check [Introduction to animations.](https://flutter.dev/docs/development/ui/animations#physics-based-animation)

ok, let's end the boring part and get our hand dirty with some flutter animations. So grab the cup of your favourite coffee or tea ☕.

In this blog, we’ll use the tween animation to animate the background color. The end result will look like this.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945204987/ukr_nVLyJ.gif)

To make this animation we will use [TweenSequence](https://docs.flutter.io/flutter/animation/TweenSequence-class.html). It enables creating an [Animation](https://docs.flutter.io/flutter/animation/Animation-class.html) whose value is defined by a sequence of [Tweens](https://docs.flutter.io/flutter/animation/Tween-class.html). In our example, we will use [ColorTween](https://docs.flutter.io/flutter/animation/ColorTween-class.html). We will make TweenSequence of different ColorTween

```
Animatable<Color> background = TweenSequence<Color>(
  [
    TweenSequenceItem(
      weight: 1.0,
      tween: ColorTween(
        begin: Colors.*red*,
        end: Colors.*green*,
      ),
    ),
    TweenSequenceItem(
      weight: 1.0,
      tween: ColorTween(
        begin: Colors.*green*,
        end: Colors.*blue*,
      ),
    ),
    TweenSequenceItem(
      weight: 1.0,
      tween: ColorTween(
        begin: Colors.*blue*,
        end: Colors.*pink*,
      ),
    ),
  ],
);
```


Now we will define our [AnimationController](https://docs.flutter.io/flutter/animation/AnimationController-class.html) which will control our animation.

```
AnimationController _controller;

@override
**void **initState() {
  **super**.initState();
  _controller = AnimationController(
    duration: **const **Duration(seconds: 10),
    vsync: **this**,
  )..repeat();
}
```


In the end, we will use [AnimatedBuilder](https://docs.flutter.io/flutter/animation/AnimationController-class.html) which will take our AnimationController and animate our background.

```
@override
Widget build(BuildContext context) {
  **return **AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        **return **Scaffold(
          body: Container(
            color: background
              .evaluate(AlwaysStoppedAnimation(_controller.value)),
          ),
        );
      });
}
```


Voilà, with just a few lines of code we have our animated color transition background. The end result will look like this.

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Background color transition',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);

  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage>
    with SingleTickerProviderStateMixin {
  AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 10),
      vsync: this,
    )..repeat();
  }

  Animatable<Color> background = TweenSequence<Color>([
    TweenSequenceItem(
      weight: 1.0,
      tween: ColorTween(
        begin: Colors.red,
        end: Colors.green,
      ),
    ),
    TweenSequenceItem(
      weight: 1.0,
      tween: ColorTween(
        begin: Colors.green,
        end: Colors.blue,
      ),
    ),
    TweenSequenceItem(
      weight: 1.0,
      tween: ColorTween(
        begin: Colors.blue,
        end: Colors.pink,
      ),
    ),
  ]);

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
        animation: _controller,
        builder: (context, child) {
          return Scaffold(
            body: Container(
              color: background
                  .evaluate(AlwaysStoppedAnimation(_controller.value)),
            ),
          );
        });
  }
}

```

In the next blog, we will see how we can animate the color with page transition. We will be creating some more cool animation like one below. Till then stay tuned!


If you liked this article make sure to 👏 it below, and connect with me on [Twitter](https://twitter.com/divyanshub024), [Github](https://github.com/divyanshub024) and [LinkedIn](https://www.linkedin.com/in/divyanshub024/).