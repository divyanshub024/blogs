## How to navigate without context in Flutter?

Navigation is an integral part of any App. Flutter makes it really easy to navigate to any screen by using simple navigator functions like Push and Pop.

**To push:**
```
Navigator.push(
    context,
    MaterialPageRoute(builder: (context) => SecondRoute()),
);
``` 

**To Pop:**
```
Navigator.pop(context);
```

This works great until your app scales and you separate your business logic with the UI logic. And now you have to pass [BuildContext](https://api.flutter.dev/flutter/widgets/BuildContext-class.html) from one function to another function. This becomes a big hassle. 

Don't worry [NavigatorKey](https://api.flutter.dev/flutter/material/MaterialApp/navigatorKey.html) is to the rescue. 


<Center><Img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1628252905490/T8r5lQPub.gif"/></Center>

### 1. Create a Navigator Key

```
static final navigatorKey = GlobalKey<NavigatorState>();
``` 

### 2. Pass the Navigator Key in MaterialApp

```
    return MaterialApp(
      ...
      navigatorKey: AppRouter.navigatorKey,
    );
```

### 3. Push using the Navigator Key

```
navigatorKey.currentState?.push(
    MaterialPageRoute(builder: (_) => SecondRoute()),
);
```

That's it with just 3 easy steps you can eliminate context from your Navigation.

***

### Thank you for reading 👋

I hope you enjoyed this article. If you have any queries or suggestions please let me know in the comments down below. 

You can connect with me on [Twitter](https://twitter.com/divyanshub024), [Github](https://github.com/divyanshub024) and [LinkedIn](https://www.linkedin.com/in/divyanshub024/). You can subscribe to my newsletter at the top of the page to get an email notification for my latest articles. 

Happy Coding... See you next time 👋