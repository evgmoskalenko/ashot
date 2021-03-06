aShot
=====


[![release](http://github-release-version.herokuapp.com/github/yandex-qatools/ashot/release.svg?style=flat)](https://github.com/yandex-qatools/ashot/releases/latest) [![Maven Central](https://maven-badges.herokuapp.com/maven-central/ru.yandex.qatools.ashot/ashot/badge.svg?style=flat)](https://maven-badges.herokuapp.com/maven-central/ru.yandex.qatools.ashot/ashot)


WebDriver Screenshot utility

* Takes a screenshot of the WebElement on different platforms (i.e. desktop browsers, iOS Simulator Mobile Safari, Android Emulator Browser)
* Decorates screenshots
* Provides flexible screenshot comparison

#####WebElement view

aShot takes a screenshot in three simple steps:
* Capture the entire page screenshot
* Find element's size and position   
* Crop the original screenshot

As a result aShot provides an image of the WebElement
![images snippet](/doc/img/images_intent_blur.png)

#### Maven dependency
```xml
<dependency>
    <groupId>ru.yandex.qatools.ashot</groupId>
    <artifactId>ashot</artifactId>
    <version>1.5.1</version>
</dependency>
``` 

#####Capturing the entire page

Different WebDrivers take screenshots in different ways. Some WebDrivers provide a screenshot of the entire page while others handle the viewport only. aShot might be configured to handle browsers with the viewport problem. This example configuration gives a screenshot of the entire page even for Chrome, Mobile Safari, etc. 
```java
new AShot()
  .shootingStrategy(new ViewportPastingStrategy())
  .takeScreenshot(webDriver);
```

#####Capturing the WebElement

One can take a screenshot of the particular WebElement(s). Just specify the element(s).
```java
WebElement myWebElement = webDriver.findElement(By.cssSelector("#my_element"));
new AShot()
  .takeScreenshot(webDriver, myWebElement);
```
 
As noted earlier aShot will find element's size and position and crop original image. WebDriver API provides a method to find the WebElement's coordinates but different WebDriver implementations behave differently. In my opinion the most general approach to find coordinates is to use jQuery. So aShot uses jQuery by default. But some drivers have problems with Javascript execution such as OperaDriver. In this case there is another way to find the WebElement's coordinates.
```java
new AShot()
  .coordsProvider(new WebDriverCoordsProvider()) //find coordinates with WebDriver API
  .takeScreenshot(webDriver, myWebElement);
```
Feel free to implement own CoordsProvider. Pull requests are welcome.

#####Prettifying the screenshot

So, let's take a simple screenshot of the weather snippet at Yandex.com.

```java
new AShot()
  .takeScreenshot(webDriver, yandexWeatherElement);
```
Here is the result.
![simple weather snippet](/doc/img/def_crop.png)
 
```DefaultCropper``` is used by default. Can we do better? Yes, we can.
 
```java
new AShot()
  .withCropper(new IndentCropper()           // set custom cropper with indentation
                 .addIndentFilter(blur()))   // add filter for indented areas
  .takeScreenshot(driver, yandexWeatherElement);
```
  
![indent blur weather snippet](/doc/img/weather_indent_blur.png)
This screenshot provides more information about position relative other elements and blurs indent in order to focus on the WebElement.  
  
#####Screenshot comparison
```.takeScreenshot()``` returns a ```Screenshot``` object which contains an image and data for comparison. One can ignore some WebElements from comparison.

```java
Screenshot myScreenshot = new AShot()
  .addIgnoredElement(By.cssSelector("#weather .blinking_element")) // ignored element(s)
  .takeScreenshot(driver, yandexWeatherElement);
```

Use ```ImageDiffer``` to find a difference between two images.

```java
ImageDiff diff = new ImageDiffer().makeDiff(myScreenshot, anotherScreenshot);
BufferedImage diffImage = diff.getMarkedImage(); // comparison result with marked differences
```

#####Several elements comparison
`(since 1.2)`  
Sometimes one need to take a screenshot of several independent elements. In this case aShot computes complex comparison area.
```java
List<WebElement> elements = webDriver.findElements(By.cssSelector("#my_element, #popup"));
new AShot()
  .withCropper(new IndentCropper() 
                 .addIndentFilter(blur()))
  .takeScreenshot(webDriver, elements);
```
Here is the result.
![complex comparison area](/doc/img/complex_elements.png)

One can see only specified elements (the header and the popup) are focused and will be compared if needed.
