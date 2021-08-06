## Playing with Paths in Flutter


### A deep dive into paths in Flutter

Everything is a widget in Flutter. And there are so many awesome widgets available in Flutter but one of my favourite widget is [CustomPaint](https://api.flutter.dev/flutter/widgets/CustomPaint-class.html).
> CustomPaint is a widget that provides a canvas on which to draw during the paint phase.

There are different ways to draw on a canvas, one of the most efficient ways is using [Path](https://api.flutter.dev/flutter/dart-ui/Path-class.html). In this blog, we are going to learn to draw and animate some advance paths. If you are unfamiliar with `Paths` please check this excellent post on paths in Flutter by [Muhammed Salih Guler](https://twitter.com/salihgueler):
[**Paths in Flutter: A Visual Guide**
*Flutter gives us a lot of standard views to use in our projects, but time to time we need to create customized views…*medium.com](https://medium.com/flutter-community/paths-in-flutter-a-visual-guide-6c906464dcd0)

## Drawing a line

Drawing a line is probably the easiest thing to do with paths. First, move the current point of the path to the starting point using the `moveTo` function. Then draw the line using the `lineTo` function to the endpoint. That’s it.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945225573/lYsnA2iVW.png)

```dart
class LinePainter extends CustomPainter {
  final double progress;

  LinePainter({this.progress});

  Paint _paint = Paint()
    ..color = Colors.black
    ..strokeWidth = 4.0
    ..style = PaintingStyle.stroke
    ..strokeJoin = StrokeJoin.round;

  @override
  void paint(Canvas canvas, Size size) {
    var path = Path();
    path.moveTo(0, size.height / 2);
    path.lineTo(size.width * progress, size.height / 2);
    canvas.drawPath(path, _paint);
  }

  @override
  bool shouldRepaint(LinePainter oldDelegate) {
    return oldDelegate.progress != progress;
  }
}
```

## Drawing a dashed line

Drawing a line is easy but the tricky part comes when you wish to draw the dashed line in Flutter. There isn’t any simple function to draw the dashed line but we can achieve this using the [PathMetric](https://api.flutter.dev/flutter/dart-ui/PathMetric-class.html).
> PathMetric is a utility for measuring a [Path](https://api.flutter.dev/flutter/dart-ui/Path-class.html) and extracting sub-paths.

First, we will draw a line as shown above. Then we will get the `PathMetrics` using `path.computeMetrics()` function. For each PathMetric we will extract the path where the starting point is the `distance` and length equal to `dashWidth`.

```
dashPath.addPath(
  pathMetric.extractPath(distance, distance + dashWidth),
  Offset.*zero*,
);
```


![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945226972/3Kv5whbjx.png)

```dart
class DashLinePainter extends CustomPainter {
  final double progress;

  DashLinePainter({this.progress});

  Paint _paint = Paint()
    ..color = Colors.black
    ..strokeWidth = 4.0
    ..style = PaintingStyle.stroke
    ..strokeJoin = StrokeJoin.round;

  @override
  void paint(Canvas canvas, Size size) {
    var path = Path()
      ..moveTo(0, size.height / 2)
      ..lineTo(size.width * progress, size.height / 2);

    Path dashPath = Path();

    double dashWidth = 10.0;
    double dashSpace = 5.0;
    double distance = 0.0;

    for (PathMetric pathMetric in path.computeMetrics()) {
      while (distance < pathMetric.length) {
        dashPath.addPath(
          pathMetric.extractPath(distance, distance + dashWidth),
          Offset.zero,
        );
        distance += dashWidth;
        distance += dashSpace;
      }
    }
    canvas.drawPath(dashPath, _paint);
  }

  @override
  bool shouldRepaint(DashLinePainter oldDelegate) {
    return oldDelegate.progress != progress;
  }
}

```

## Circles

In this section, we are going to draw circles… circles… and more circles. We will be building something like this.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945228664/THdtLKGJF.gif)

A circle is a type of ellipse. We can make an ellipse using `addOval` function. It takes Rect as a param. To make that oval as a circle we can use `Rect.fromCircle` .

```
@override
void paint(Canvas canvas, Size size) {
  var path = Path();
  path.addOval(Rect.fromCircle(
    center: Offset(0, 0),
    radius: 80.0,
  ));
  canvas.drawPath(path, myPaint);
}
```


This will give us a single circle with the centre as (0,0) and radius 80.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945230557/XpdevgxhJ.png)

Ok, that was pretty easy but now the question is how to make something like this.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945231869/mTUTI2-64.png)

We can achieve this with the help of basic Trigonometry but before that let’s see what pattern can we find in the above image. One pattern which we can find is that if we draw a circle of the same radius from the centre (0,0) then this circle contains the centres of all the circles*(shown in blue dots)*.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945233265/VM-WFogPc.png)

We need to find the coordinates of these centres. Suppose there are n circles. Let’s take any circle and mark its centre as (x,y). The angle between x-axis and line (0,0) — (x,y) is θ. Our aim is to find the x and y coordinates.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945234811/X17xc-w_y.png)

We know that the total angle is 2π. Which means that the angle(θ) created by the iᵗʰ centre with x-axis will be:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945236325/-cHaarEJd.png)

To find the value of x and y the trigonometry from our high school come in handy. From the above figure, we can calculate the value of sin(θ) and cos(θ).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945238053/APIgGu03Y.png)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945239411/NEjqT8oP9.png)

And we this gives us the value of x and y.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945240702/Egww05lkN.png)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945241956/-7rY5cRJE.png)

That’s it. We will apply the same formula in our code and draw the n number of circles of radius r on (x,y) coordinates.

```dart
  @override
  void paint(Canvas canvas, Size size) {
    var path = createPath();
    canvas.drawPath(path, myPaint);
  }

Path createPath() {
    var path = Path();
    int n = circles.toInt();
    var range = List<int>.generate(n, (i) => i + 1);
    double angle = 2 * math.pi / n;
    for (int i in range) {
      double x = radius * math.cos(i * angle);
      double y = radius * math.sin(i * angle);
      path.addOval(Rect.fromCircle(center: Offset(x, y), radius: radius));
    }
    return path;
  }
```

## Path Tracing

The path tracing animation can be implemented by drawing the small extracted path of the whole path using [PathMetric](https://api.flutter.dev/flutter/dart-ui/PathMetric-class.html). It’s hard to do it with `path` as it won’t give us the length as well as the extracted path.

Let’s first define an `AnimationController`.

```
class _CirclesState extends State<Circles> with   SingleTickerProviderStateMixin {

AnimationController _controller;

@override
void initState() {
  super.initState();
  _controller = AnimationController(
    vsync: this,
    duration: Duration(seconds: 3),
  );
  _controller.value = 1.0;
}
```


Now, we will pass the controller value to our `CirclesPainter` class.

```
CustomPaint(
  painter: CirclesPainter(
    circles: circles,
    progress: _controller.value,
    showDots: showDots,
    showPath: showPath,
  ),
),
```


To animate the path we are going to draw the path fraction by fraction. PathMetric has an `extractPath` function which will return the segment of a path for the given length. The `extractPath` function takes the `start` and `end` value. The `end` value is equal to the `pathMetric.length * progress`. Which is doing nothing but drawing the path in the time interval of 3 seconds.

```dart
  @override
  void paint(Canvas canvas, Size size) {
    var path = createPath();
    PathMetrics pathMetrics = path.computeMetrics();
    for (PathMetric pathMetric in pathMetrics) {
      Path extractPath = pathMetric.extractPath(
        0.0,
        pathMetric.length * progress,
      );
      canvas.drawPath(extractPath, myPaint);
    }
  }
```

Full code for circles is available [here](https://github.com/divyanshub024/flutter_path_animation/blob/master/lib/circles.dart).

## Polygon

Another important figure in geometry is a polygon. A polygon is a closed figure where the sides are all line segments. We will be building something like this.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945244916/XdP2gCRWM.gif)

The logic for building the polygon is the same as of the circle we have seen above but instead of drawing a circle on (x,y), we will be drawing a line to (x,y) coordinates.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945247828/_nMZf6o8D.png)

```dart
Path createPath(int sides, double radius) {
    var path = Path();
    var angle = (math.pi * 2) / sides;
    path.moveTo(radius * math.cos(0.0), radius * math.sin(0.0));
    for (int i = 1; i <= sides; i++) {
      double x = radius * math.cos(angle * i);
      double y = radius * math.sin(angle * i);
      path.lineTo(x, y);
    }
    path.close();
    return path;
  }
```

Full code for the polygon is available [here](https://github.com/divyanshub024/flutter_path_animation/blob/master/lib/polygon.dart).

## Spiral

A **spiral** is a curve which emanates from a point, moving farther away as it revolves around the point. Drawing a spiral is a little tricky part as there isn't an easy way to do it.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945249876/sDxhha87u.gif)

To achieve this end result we will be drawing the line to (x,y) coordinates. For every new point(x,y) we will increase the radius by 0.75 and angle by 2π/50. Because the radius and the angle is quite small the result we get looks like a curve rather than a straight line.

```dart
Path createSpiralPath(Size size) {
    double radius = 0, angle = 0;
    Path path = Path();
    for (int n = 0; n < 200; n++) {
      radius += 0.75;
      angle += (math.pi * 2) / 50;
      var x = size.width / 2 + radius * math.cos(angle);
      var y = size.height / 2 + radius * math.sin(angle);
      path.lineTo(x, y);
    }
    return path;
  }
```

We can animate it the same way we animated other views using the PathMetric. Full code for the spiral is available [here](https://github.com/divyanshub024/flutter_path_animation/blob/master/lib/spiral.dart).

## Bonus

Here is an animation of revolving planets. Check out the code [here](https://github.com/divyanshub024/flutter_path_animation/blob/master/lib/planets.dart).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945252451/qn5AKTEtN.gif)

**You can see the full source code of the project [here](https://github.com/divyanshub024/flutter_path_animation).**
[**divyanshub024/flutter_path_animation**
*A new Flutter application. This project is a starting point for a Flutter application. A few resources to get you…*github.com](https://github.com/divyanshub024/flutter_path_animation)

## Recommended Reading
[**Exploring the Flutter camera plugin**
*Building a camera app in Flutter.*levelup.gitconnected.com](https://levelup.gitconnected.com/exploring-flutter-camera-plugin-d2c54ac95f05)

**If you liked this article make sure to 👏 it below, and connect with me on [Twitter](https://twitter.com/divyanshub024), [Github](https://github.com/divyanshub024) and [LinkedIn](https://www.linkedin.com/in/divyanshub024/).**

[Https://www.twitter.com/FlutterComm](http://Https://www.twitter.com/FlutterComm)