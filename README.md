# MOVED
This repository was moved to https://github.com/synack/chaoticmarch


# CHAOTICMARCH
_CHAOTICMARCH_ is an iOS App UI automation engine for aiding security testing. It tries to solve the problem of coverage during blackbox testing. It is often the case that a researcher is introduced to a new application and their first action is to "look" around to see what features the App has as well as what web API it uses. Triggering the web API through the UI has the advantage of giving a sample of parameters to fuzz later.

Doing manual clicking around of the App is time consuming and boring. Also doing it manually will probably miss things. However, automated test using CHAOTICMARCH means that the tests can be repeatable and thorough. Also, the researcher can spend their time observing the traffic rather than looking at UI.

# Features
The CHAORICMARCH engine supports building block functionality by providing functions to the LUA scripts to execute their logic.

## Functions
* _log(String out)_ Output a string to syslog.
* _touchDown(int touchId, number x, number y)_ Simulate a touch on the screen. Use touchId, [0, 9] to distinguish between touch points.
* _touchUp(int touchId, number x, number y)_ A touch release.
* _touchMove(int touchId, number x, number y)_ After a touch down, you can signal that the point has moved, simulating a swipe.
* _usleep(number mircoseconds)_ A sleep is useful between touchUp/Down events to simulate human interaction.
* _inputTest(String text)_ Enter the text to whatever component is holding the keyboard focus.
* _adaptResolution(int width, int height)_ If desired, you can change the virtual resolution. This will affect all coordinate based events.
* _getResolution()_ return two numbers, width and height, representing current resolution.
* _adaptOrientation(int ORIENTATION_TYPE)_ Change the devices orientation.
* _hasComponentAt(String compname, int boxes_x, int boxes_y, int box_x, int box_y)_ Checks if a particular component i.e. UIButton is at a specific location. The location is defined by number of equally spaced sections along the X and Y, then specifying which box you want this component to be in. The coordinates are defined so to give the user the freedom to be imprecise. 
* _hasTextAt(String text, int boxes_x, int boxes_y, int box_x, int box_y)_ Same as component but the engine will look for text at a specified box.
* _findOfTypes(String type1, ..., String "")_ Possibly the most useful function, it will return a dictionary of the locations of particular types of components.
* _getBundleID()_ Gives the string of bundle name. Useful for deciding how to proceed with execution.
* _showCircle(int id, number x, number y, number radius)_ Mostly used with touchEvent to show where the touch has occurred on the screen.
* _hideCircle(int id)_ Removed the circle of a particular ID, you get [0, 9] range of ID's.

# Installation

You will need a JailBroken iOS device (an iPhone 5s) with Cydia and MobileSubstrate. Compile the dynamic library and upload it to the phone like this:

```
make && 
scp -P 3222 ./libchaoticmarch.dylib \
            root@127.0.0.1:/Library/MobileSubstrate/DynamicLibraries/ && 
scp -r -P 3222 ./lua root@127.0.0.1:
```

The output will be in the syslog file: `/var/log/syslog`

# Configuration
By default, CHAOTICMARCH will not execute within any App. It needs to be configured. This is done in `chaotic_march.lua` file. There are selected `CONFIGURABLE` points. First is the maps that determines if the engine will run:

```lua
-- CONFIGURABLE: Enter bundles where to run
allowedBundles = {}
allowedBundles["com.highaltitudehacks.dvia"] = 1
allowedBundles["com.gs.pwm.external"] = 1
allowedBundles["com.apple.iBooks"] = 1
```

To ensure it runs within the desired App, add its identifier to the map. If you're not sure about the identifier, then run the App once and CHAOTICMARCH will log something like this (with the bundle ID):

```
Mar 18 17:06:37 iPhone CommCenterMobileHelper[2112]: CHAOTICMARCH: lua: Not running in: com.apple.commcentermobilehelper
```

You can also configure the initialization script. By default it is set to this:

```lua
-- CONFIGURABLE: execute user files
base_path = "/var/root/lua/"
```

First, the code will execute alphabetically sorted `pre_all*.lua` scripts. Then, it will execute bundle specific scripts `[bundle_id]*.lua`. Finally, it will execute post scripts `post_all*.lua`. This should give enough control to the developer to load libraries, set up environment and specify common functions before executing any App specific scripts. 
