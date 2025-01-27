## Week 3 - Graphical apps

### Day 1

* Forum
  * Refactor code into separate methods
* HelloWorld
  * Create `HashMap` that maps destination addresses to lists of `Email` objects
  * Word frequency in a sentence into a `HashMap<String, Integer>`
* Review last week's concepts
  * String formatting
  * Extending a class
  * Reading and writing files
  * Parsing and serializing JSON
  * Debugging and testing
* Interfaces
  * Sort an `ArrayList<String>` in JREPL using `Collections.sort`
  * In the HelloWorld project, sort `ArrayList<Person>` by implementing `Comparable`
  * Definition
    * A list of one or more methods that a class must have
    * Allows code to call methods on objects even if they don't know what those objects are
    * Used all the time in Java itself as well as libraries/frameworks

### Day 2

* Install [Scene Builder](http://www.oracle.com/technetwork/java/javase/downloads/javafxscenebuilder-1x-archive-2199384.html)
* Create a project
  * New Project -> JavaFX template
  * Run blank project
  * Look at the three files you start with
  * Edit Main.java to set the title and window size
  * Edit sample.fxml via Scene Builder
    * Make all buttons set min size to USE_PREF_SIZE
  * Edit Controller.java
    * Implement `Initializable`
    * Implement action/keyboard methods
    * Bring in controls using `@FXML`
* Create a [to-do desktop app](../projects/ToDoDesktop)
  * Create an `ObservableList`
  * Set the `ListView` to use it
  * Wire up the "Add", "Toggle", and "Remove" buttons
* Create a [web browser desktop app](../projects/BrowserDesktop)
  * Wire up the "Go" button
  * Allow hitting enter in URL textfield
  * Implement `ChangeListener` to update the URL textfield
  * Wire up the back and forward buttons

### Day 3

* Install [Android Studio](https://developer.android.com/sdk/index.html)
* Create a project
  * At the main screen, go to: Start a new Android Studio project
  * Give it a name and type "theironyard.com" as the company domain, and click "Next"
  * Select an Android version and click "Next"
  * Select "Empty Activity" and click "Next"
  * Click "Finish"
* Configure an emulator
  * In your project, click the "Android Virtual Device Manager" button
  * Click on "Create Virtual Device..."
  * Select a device and click "Next..."
  * Select an Android version and click "Next..."
  * Make sure "Use Host GPU" is checked
  * Click "Show Advanced Settings"
  * Set the RAM to 1GB (so Intel HAXM will work)
* Create a [to-do Android app](../projects/ToDoAndroid)
  * Add listview, text field, and button
  * Set listview width/height to `match_parent`
  * Create variables for all controls and use `findViewById` in `onCreate`
  * Create an `ArrayAdapter` and connect it to the listview
  * Implement `View.OnClickListener` and connect it to the button
  * Implement `AdapterView.OnItemLongClickListener` and connect it to the listview
* Create a [web browser Android app](../projects/BrowserAndroid)
  * Remove action bar
    * Edit `res/values/styles.xml`
    * Change "DarkActionBar" to "NoActionBar"
  * Add text field, three buttons, and webview
  * Create variables for all controls and use `findViewById` in `onCreate`
  * Create and set a `WebViewClient`
  * Implement `View.OnClickListener` and connect it to all the buttons
  * Add internet permission to AndroidManifest.xml
    * `<uses-permission android:name="android.permission.INTERNET" />`
  * Create an anonymous class based on `WebViewClient`
    * Override the `onPageStarted` method to update the address bar

### Day 4

* Create a project
  * Download and run [the setup app](https://libgdx.badlogicgames.com/download.html)
  * Uncheck everything except "Desktop"
  * Open the project in IntelliJ
  * Run -> Edit Configurations...
* Create [HelloGame](../projects/HelloGame)
  * Create `x` and `y`
  * Continuously increment `x`
  * Create `move` method
    * Create `if` statements that use `Gdx.input.isKeyPressed` to change the position
    * Set `xv` and `yv` to `MAX_VELOCITY` in the `if` statements
    * Change `x` and `y` by the `xv` and `yv` multiplied by `Gdx.graphics.getDeltaTime()`
    * Dampen `xv` and `yv`
* Create [SuperKoalio](../projects/SuperKoalio)
  * Create `x`, `y`, `xv`, and `yv`
  * Load the koala `Texture` and split it by width (18) and height (26)
  * Set `stand` texture to `grid[0][0]`
  * Copy the `move` method from HelloGame
  * Add gravity and jump velocity
  * Set `jump` texture to `grid[0][1]`
  * Use a `FitViewport` so the screen scales as the window is resized
  * Add `walk` animation
* Build as a JAR file
  * Click the "Gradle" tab on the right side of the IntelliJ window
  * :desktop -> Tasks -> other -> dist
  * The JAR file will be in `desktop/build/libs`
