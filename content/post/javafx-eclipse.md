+++

author = "Jacob Hell"
title = "How to Get OpenJFX Working in Eclipse"
date = "2020-04-07"
description = "How to Get Open JavaFX Working in Eclipse"
tags = [
    "eclipse",
    "java"
]

+++

Get the power of JavaFX working in Eclipse. Without the licensing requirements of Oracle JDK.

<!--more-->

## Requirements

This walkthrough assumes you have Eclipse and a JDK installed.

## Downloading OpenJFX

Download the OpenJFX SDK [here](https://gluonhq.com/products/javafx/). The current LTS version, 14, requires at least JDK 11.

## Create a Java Project

1. In eclipse, create a Java project `File -> New -> Java Project`
2. On the `src` folder, right click and create a package named `main`. Then create a class named `Main`. 
3. In `Main.java` replace the contents with:

```java
package main;

import javafx.application.Application;
import javafx.stage.Stage;

public class Main extends Application {

    @Override
    public void start(Stage primaryStage) throws Exception{
        primaryStage.setTitle("Hello World");
        primaryStage.show();
    }


    public static void main(String[] args) {
        launch(args);
    }
}
```

## Importing the OpenJFX SDK

1. Right click your java project, and select `Properties`.
2. Select `Java Build Path` and then `Librares`.
3. Select `Add Library...`. Then Select `User Library`.
4. Hit `Next` then Select `User Libraries...`.
5. Select `New...` and name it `JavaFX`.
6. Select `JavaFX` then select `Add External Jars...`.
7. A file selection dialog opens. Go to `OpenJFXLocation/lib` and select all the jar files.
8. Select `Apply and Close`
9. Check `JavaFX` and click `Finish`.

## Adding VM Arguments

1. Select `Run -> Run Configurations...`
2. Right Click `Java Application` and select `New Configuration`.
3. Under `Main class` search for `Main.java`.
4. Select the `Arguments` tab.
5. Paste `--module-path "OpenJFXLocation\lib" --add-modules javafx.controls,javafx.fxml` into `VM arguments`. 

### Conclusion

Select `Run` and you should be greeted with a window with the title `Hello World`! 

A lot of work to get this working. But at least no Oracle licensing was involved!