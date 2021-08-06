## How To Create a Dynamic Theme in Flutter Using Provider


How to add dark mode to your app

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604944872379/NXXyaUBW1.png)

We all love themes in apps. Especially the so-called *dark theme*. A dark theme has now become an essential part of mobile applications. All major applications support the dark theme and some apps even have dark theme as default.
> Dark themes reduce the luminance emitted by device screens, while still meeting minimum color contrast ratios. They help improve visual ergonomics by reducing eye strain, adjusting brightness to current lighting conditions, and facilitating screen use in dark environments — all while conserving battery power. Devices with OLED screens benefit from the ability to turn off black pixels at any time of day.


Well, we know the power of the dark side, that’s why we are here! So let’s bring this power to flutter.

We will be using the [provider](https://pub.dev/packages/provider) package. Add the provider plugin to pubspec.yaml file.

```
dependencies:   
  **provider: ^3.1.0**
```


We will create two themes — a light theme and a dark theme. I’ve done it manually but you can use [panache](https://rxlabz.github.io/panache_web/#/) to create the theme.

```
import 'package:flutter/material.dart';

final darkTheme = ThemeData(
  primarySwatch: Colors.grey,
  primaryColor: Colors.black,
  brightness: Brightness.dark,
  backgroundColor: const Color(0xFF212121),
  accentColor: Colors.white,
  accentIconTheme: IconThemeData(color: Colors.black),
  dividerColor: Colors.black12,
);

final lightTheme = ThemeData(
  primarySwatch: Colors.grey,
  primaryColor: Colors.white,
  brightness: Brightness.light,
  backgroundColor: const Color(0xFFE5E5E5),
  accentColor: Colors.black,
  accentIconTheme: IconThemeData(color: Colors.white),
  dividerColor: Colors.white54,
);
```


When the themes are ready, we create a theme notifier class to notify us of a theme change:

```
import 'package:flutter/material.dart';

class ThemeNotifier with ChangeNotifier {
  ThemeData _themeData;

  ThemeNotifier(this._themeData);

  getTheme() => _themeData;

  setTheme(ThemeData themeData) async {
    _themeData = themeData;
    notifyListeners();
  }
}
```


Next we wrap our app with `ChangeNotifierProvider`. Then we can use `ThemeNotifier` to get the theme.

```dart
void main() => runApp(
      ChangeNotifierProvider<ThemeNotifier>(
        builder: (_) => ThemeNotifier(darkTheme),
        child: MyApp(),
      ),
    );

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final themeNotifier = Provider.of<ThemeNotifier>(context);
    return MaterialApp(
      title: 'Chitr',
      theme: themeNotifier.getTheme(),
      home: HomePage(),
    );
  }
}
```

It’s time to change the theme manually. We use `[DayNightSwitch`](https://pub.dev/packages/day_night_switch) to do this — it works just like the normal switch widget in Flutter. Inside the `onChanged` callback of `[DayNightSwitch`](https://pub.dev/packages/day_night_switch), we call the`onThemeChanged` method, which uses `themeNotifier` to set the theme and notify the whole app.

```
void main() => runApp(
      ChangeNotifierProvider<ThemeNotifier>(
        builder: (_) => ThemeNotifier(darkTheme),
        child: MyApp(),
      ),
    );

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final themeNotifier = Provider.of<ThemeNotifier>(context);
    return MaterialApp(
      title: 'Chitr',
      theme: themeNotifier.getTheme(),
      home: HomePage(),
    );
  }
}
```


That’s it. With just a few lines of code we can dynamically change the theme of our app. Let’s see what it looks like:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604944874356/oZi95oqtp.gif)

It looks amazing but…

Our theme isn’t being saved — if we restart the app it goes back to default. So we use use `SharedPreferences` to store the current theme. Inside of the`onThemeChanged` method, we save the current theme.

```
void onThemeChanged*(*bool value, ThemeNotifier themeNotifier*) *async *{
  (*value*)
      *? themeNotifier.setTheme*(*darkTheme*)
      *: themeNotifier.setTheme*(*lightTheme*)*;
  var prefs = await SharedPreferences.*getInstance()*;
  prefs.setBool*(*'darkMode', value*)*;
*}*
```


We use `SharedPreferences` value inside the main method:

```
SharedPreferences.*getInstance()*.then*((*prefs*) {
  *var darkModeOn = prefs.getBool*(*'darkMode'*) *?? true;
  runApp*(
    *ChangeNotifierProvider*<*ThemeNotifier*>(
      *builder: *(*_*) *=> ThemeNotifier*(*darkModeOn ? darkTheme : lightTheme*)*,
      child: MyApp*()*,
    *)*,
  *)*;
*})*;
```


You can check the full source code [here](https://github.com/divyanshub024/chitr).

## Recommended Reading
[**Everything you need to know about Flutter page route transition**
*We know how easy it is to navigate from one route to another in Flutter. We just need to do push and pop.*medium.com](https://medium.com/flutter-community/everything-you-need-to-know-about-flutter-page-route-transition-9ef5c1b32823)