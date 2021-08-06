## Flutter Custom Painter &Circular Wave Animation


My Experiments with Flutter animations

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945189428/4tPWnLfj5.jpeg)

Welcome back to my Flutter animation series. Where I show you my experiments with Flutter animations. In the previous blog, we saw how to animate [background color using Tween animation](https://medium.com/flutter-community/flutter-animation-i-background-color-transition-39dcbada7335). Today we will see how to create some amazing UI using [CustomPainter](https://api.flutter.dev/flutter/rendering/CustomPainter-class.html) and to animate it. So, Grab a cup of coffee and let’s convert some coffee into code. ☕ =&gt; {}

We will be creating ***circular wave animation** *but first, let’s see how we can create a circle using CustomPaint. Add `CustomPaint` widget to your widget tree, it takes a *size* and a *painter. *Painter is a simple class that extends `CutsomPainter` and implement two methods, `*paint(Canvas, Size) `*and `*shouldRepaint(CustomPainter oldDelegate)`. *The paint method provides us with the canvas which let us draw anything on canvas. We can draw a circle using the `*drawCircle()`* method. Here is a full code.

```
**import **'package:flutter/material.dart';

class CircleRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomPaint(
        size: Size(double.infinity, double.infinity),
        painter: CirclePainter(),
      ),
    );
  }
}

class CirclePainter extends CustomPainter {
  var wavePaint = Paint()
    ..color = Colors.black
    ..style = PaintingStyle.stroke
    ..strokeWidth = 2.0
    ..isAntiAlias = true;

@override
  void paint(Canvas canvas, Size size) {
    double centerX = size.width / 2.0;
    double centerY = size.height / 2.0;
    canvas.drawCircle(Offset(centerX, centerY), 100.0, wavePaint);
  }

@override
  bool shouldRepaint(CirclePainter oldDelegate) {
    return false;
  }
}
```


And here is the result…

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945190957/6ph2-cYc6.png)

Well, That’s interesting. Let’s see if we can create multiple concentric circles. To create multiple concentric circles we will define the `currentRadius` and `maxRadius`. All the circles will be drawn from the currentRadius to maxRadius with the margin of waveGap. Let’s see the code.

```
@override
**void **paint(Canvas canvas, Size size) {
  double centerX = size.width / 2.0;
  double centerY = size.height / 2.0;
  double maxRadius = hypot(centerX, centerY);
  double waveGap = 10.0;
  double currentRadius = 0;

  **while **(currentRadius < maxRadius) {
    canvas.drawCircle(Offset(centerX, centerY), currentRadius, wavePaint);
    currentRadius += waveGap;
  }
}

double hypot(double x, double y) {
  **return **math.sqrt(x * x + y * y);
}
```


With just a few lines we get our stupid yet beautiful UI :)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945192624/Ax--cmzAV.png)

Now, Let’s see how we can animate these circles and make wave animation. For this, we need to animate our `*currentRadius`*. We can achieve this using tween animation which will begin at`0` and end at `waveGap`.

```
Animation<double> _animation;

_animation = Tween(begin: 0.0, end: waveGap).animate(controller)
  ..addListener(() {
    setState(() {
      waveRadius = _animation.value;
    });
  });
```


We need to define our `AnimationController` and set it in an infinite loop like this.

```
AnimationController controller;

controller = AnimationController(
    duration: Duration(milliseconds: 1500), vsync: this);

controller.forward();

controller.addStatusListener((status) {
  if (status == AnimationStatus.completed) {
    controller.reset();
  } else if (status == AnimationStatus.dismissed) {
    controller.forward();
  }
});
```


In the end, we will set our `currentRadius` equals to `waveRadius` . Here is the final code.

```dart
import 'dart:math' as math;

import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Wave Animation',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: CircleWaveRoute(),
    );
  }
}

class CircleWaveRoute extends StatefulWidget {
  @override
  _CircleWaveRouteState createState() => _CircleWaveRouteState();
}

class _CircleWaveRouteState extends State<CircleWaveRoute>
    with SingleTickerProviderStateMixin {
  double waveRadius = 0.0;
  double waveGap = 10.0;
  Animation<double> _animation;
  AnimationController controller;

  @override
  void initState() {
    super.initState();
    controller = AnimationController(
        duration: Duration(milliseconds: 1500), vsync: this);

    controller.forward();

    controller.addStatusListener((status) {
      if (status == AnimationStatus.completed) {
        controller.reset();
      } else if (status == AnimationStatus.dismissed) {
        controller.forward();
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    _animation = Tween(begin: 0.0, end: waveGap).animate(controller)
      ..addListener(() {
        setState(() {
          waveRadius = _animation.value;
        });
      });

    return Scaffold(
      backgroundColor: Colors.white,
      body: CustomPaint(
        size: Size(double.infinity, double.infinity),
        painter: CircleWavePainter(waveRadius),
      ),
    );
  }

  @override
  void dispose() {
    controller.dispose();
    super.dispose();
  }
}

class CircleWavePainter extends CustomPainter {
  final double waveRadius;
  var wavePaint;
  CircleWavePainter(this.waveRadius) {
    wavePaint = Paint()
      ..color = Colors.black
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2.0
      ..isAntiAlias = true;
  }
  @override
  void paint(Canvas canvas, Size size) {
    double centerX = size.width / 2.0;
    double centerY = size.height / 2.0;
    double maxRadius = hypot(centerX, centerY);

    var currentRadius = waveRadius;
    while (currentRadius < maxRadius) {
      canvas.drawCircle(Offset(centerX, centerY), currentRadius, wavePaint);
      currentRadius += 10.0;
    }
  }

  @override
  bool shouldRepaint(CircleWavePainter oldDelegate) {
    return oldDelegate.waveRadius != waveRadius;
  }

  double hypot(double x, double y) {
    return math.sqrt(x * x + y * y);
  }
}

```

And here is our final animation. EUREKA!!!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945196718/VMF0DNNYZ.gif)

If you liked this article make sure to 👏 it below, and connect with me on [Twitter](https://twitter.com/divyanshub024), [Github](https://github.com/divyanshub024) and [LinkedIn](https://www.linkedin.com/in/divyanshub024/).