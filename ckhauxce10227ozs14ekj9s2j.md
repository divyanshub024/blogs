## Creating a wallpaper app  in Flutter: Part 1


Cooking a wallpaper app from zero to one.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1604945182561/SaMZ7p3K4.png)

In this series of blog, we will be building a wallpaper app from zero to one in Flutter. Our app will consist of a home screen (with some cool animation) and a detailed image screen. So, Let’s get started

But before that let’s try out the app first.
[**Chitr: Wallpapers and Backgrounds - Apps on Google Play**
*Find all your wallpapers and backgrounds in one place. Easily set any image as a wallpaper. Choose between variety of…*play.google.com](https://play.google.com/store/apps/details?id=com.divyanshu.chitr)

In this part, we are going to build the home screen and fetch images from [PixaBay API](https://pixabay.com/service/about/api/). But before going forward let’s see what we are going to build.


So, the animation looks cool but how to build it 🤔? The three main ingredients to build is animation are [http](https://pub.dev/packages/http), [preload_page_view](https://pub.dev/packages/preload_page_view) and [cached_network_image](https://pub.dev/packages/cached_network_image). The http plugin is used to make an HTTP request and fetch images from [PixaBay API](https://pixabay.com/service/about/api/). As the name suggests the preload_page_view is just like Flutter PageView with addition to load the page in advance. And at last everyone’s favourite Cached network image to load and cache network images. So our ingredients are ready let’s see the recipe.

## Fetching Images:

First import the http plugin to pubspec.yaml.

```
http: ^0.12.0+2
```


Create a class *ApiProvider** ***which will have the *getImages()* function.

```
class ApiProvider *{
  *Future*<*ImageModel*> *getImages*(*int count*) *async *{
    *final response = await http.get*(
        *'https://pixabay.com/api/?key=<YOUR KEY> &editors_choice=true&per_page=$count&orientation=vertical'*)*;
    if *(*response.statusCode == 200*) {
      *return ImageModel.fromJson*(*jsonDecode*(*response.body*))*;
    *} *else *{
      *throw Exception*(*'Failed to get images'*)*;
    *}
  }
}*
```


Now in home_page.dart we will call the getImages() function from initState to fetch images. We are not using FutureBuilder as it creates problem by restarting the asynchronous task every time the widget rebuild.

```
List*<*Hits*> *hits;

_loadImages() async {
  var imageModel = await ApiProvider().getImages(25);
  hits = imageModel.hits;
  setState(() {});
}

@override
void initState() {
  _loadImages();
  super.initState();
}
```


## Building animation:

Before building the animation let’s decode it from the above video. The UI consists of multiple images in a grid with 5 rows and 5 columns. We can achieve this layout by using `GridView` but it’s not suitable for us as it does not allow us to manipulate with element animation. The other way to do it is using a vertical `PageView` inside a horizontal `PageView.` But one problem with the PageView is we cannot load all the 5 columns at once so we use PreloadPageView. we will specify our viewportFraction to 0.7 and preloadPagesCount to 5. ViewportFraction is the fraction of the viewport that each page should occupy. This will allow us to see the fraction of other pages which will give us a grid-like layout.

One thing to note from the below code is* _animatePage(int page, int index) *function. It’s where all the magic happens. What’s its doing is animating all other PreloadPageView instead of the current one. Which gives us this awesome springy animation. Let’s cut some slack and see the code.

```dart
import 'package:chitr/home/model/ImageModel.dart';
import 'package:chitr/image/ui/image_page.dart';
import 'package:chitr/search/searchPage.dart';
import 'package:chitr/util/api_provider.dart';
import 'package:flutter/material.dart';
import 'package:preload_page_view/preload_page_view.dart';

import 'custom_card.dart';

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  List<PreloadPageController> controllers = [];
  List<Hits> hits;

  @override
  void initState() {
    _loadImages();
    controllers = [
      PreloadPageController(viewportFraction: 0.6, initialPage: 3),
      PreloadPageController(viewportFraction: 0.6, initialPage: 3),
      PreloadPageController(viewportFraction: 0.6, initialPage: 3),
      PreloadPageController(viewportFraction: 0.6, initialPage: 3),
      PreloadPageController(viewportFraction: 0.6, initialPage: 3),
    ];
    super.initState();
  }

  _animatePage(int page, int index) {
    for (int i = 0; i < 5; i++) {
      if (i != index) {
        controllers[i].animateToPage(page,
            duration: Duration(milliseconds: 300), curve: Curves.ease);
      }
    }
  }

  _loadImages() async {
    var imageModel = await ApiProvider().getImages(25);
    hits = imageModel.hits;
    setState(() {});
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      extendBody: true,
      backgroundColor: Theme.of(context).backgroundColor,
      body: PreloadPageView.builder(
        controller:
            PreloadPageController(viewportFraction: 0.7, initialPage: 3),
        itemCount: 5,
        preloadPagesCount: 5,
        itemBuilder: (context, mainIndex) {
          return PreloadPageView.builder(
            itemCount: 5,
            preloadPagesCount: 5,
            controller: controllers[mainIndex],
            scrollDirection: Axis.vertical,
            physics: ClampingScrollPhysics(),
            onPageChanged: (page) {
              _animatePage(page, mainIndex);
            },
            itemBuilder: (context, index) {
              var hitIndex = (mainIndex * 5) + index;
              var hit;
              if (hits != null) {
                hit = hits[hitIndex];
              }
              return GestureDetector(
                onTap: () {
                  if (hits != null) {
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (context) => ImagePage(
                          model: hit,
                          imageBoxFit: BoxFit.cover,
                        ),
                      ),
                    );
                  }
                },
                child: CustomCard(
                  title: hit?.user,
                  description: hit?.tags,
                  url: hit?.webformatURL,
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.search),
        onPressed: () {
          Navigator.push(
              context, MaterialPageRoute(builder: (context) => SearchPage()));
        },
      ),
    );
  }
}

```

You must be wondering where *CustomCard* comes from. Well, it’s another stateful widget where we are using *CachedNetworkImage *to show image and other image details. Here is the code for it.

```dart
import 'package:cached_network_image/cached_network_image.dart';
import 'package:flutter/material.dart';

class CustomCard extends StatefulWidget {
  CustomCard({
    @required this.url,
    @required this.title,
    @required this.description,
  });
  final String url;
  final String title;
  final String description;

  @override
  _CustomCardState createState() => _CustomCardState();
}

class _CustomCardState extends State<CustomCard> {
  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(4.0)),
      child: Stack(
        children: <Widget>[
          Container(
            child: (widget.url != null)
                ? CachedNetworkImage(
                    imageUrl: widget.url,
                    fit: BoxFit.cover,
                  )
                : null,
            width: double.infinity,
            height: double.infinity,
          ),
          Positioned(
            bottom: 0.0,
            left: 0.0,
            right: 0.0,
            child: Container(
              height: 200.0,
              decoration: _whiteGradientDecoration(),
            ),
          ),
          Positioned(
            left: 0.0,
            right: 0.0,
            bottom: 0.0,
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.center,
              mainAxisSize: MainAxisSize.min,
              children: <Widget>[
                Text(
                  (widget.title != null) ? widget.title : '',
                  style: TextStyle(fontSize: 20.0, fontWeight: FontWeight.bold),
                ),
                Padding(
                  padding: const EdgeInsets.all(8.0),
                  child: Text(
                    (widget.description != null) ? widget.description : '',
                    maxLines: 1,
                    style: TextStyle(fontSize: 16.0),
                  ),
                ),
              ],
            ),
          )
        ],
      ),
    );
  }

  BoxDecoration _whiteGradientDecoration() {
    return const BoxDecoration(
      gradient: LinearGradient(
          colors: [Colors.black, const Color(0x10000000)],
          begin: Alignment.bottomCenter,
          end: Alignment.topCenter),
    );
  }
}

```

Well, that’s it. Our awesome animation is cooked.


## What’s Next?

In the next article, we will see how we can set any of these images as wallpaper.
[**Creating a wallpaper app in Flutter: Part 2**
*Cooking a wallpaper app from zero to one.*medium.com](https://medium.com/@divyanshub024/creating-a-wallpaper-app-in-flutter-part-2-d28895320ce0)

**If you liked this article make sure to 👏 it below, and connect with me on [Twitter](https://twitter.com/divyanshub024), [Github](https://github.com/divyanshub024) and [LinkedIn](https://www.linkedin.com/in/divyanshub024/).**
[**Flutter Community**
*The latest Tweets from Flutter Community (@FlutterComm). Follow to get notifications of new articles and packages from…*www.twitter.com](https://www.twitter.com/FlutterComm)