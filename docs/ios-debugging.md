# Debugging Mobile Safari 11 on Linux 

You need:
* Linux
* A browser or other tool containing a debugger (Chrome, VS Code, Firefox, ...)
* Something that adapts your debugging tool's debug protocol to WebKit's debug protocol (https://github.com/RemoteDebug/remotedebug-ios-webkit-adapter for example)
* A Lightning to USB cord
* An iOS device running iOS 11
* A root USB port (using a hub caused very consistent issues for me; if it works for you, that's great)
* Git, GNU build tools, A C/++ compiler

First, you're going to install Google's WebKit USB debugger proxy and libimobiledevice. 

Here's the repo URL for the proxy: https://github.com/google/ios-webkit-debug-proxy

1. Clone the proxy repo.
1. Copy the apt-get command in the README to the clipboard. Remove `libimobiledevice-dev`, paste the command and install the packages.
1. Run `git clone https://github.com/libimobiledevice/libimobiledevice`. This needs to be cloned and built from source as it contains a patch that allows iOS 11 devices to pair properly.
1. Pick a directory that you're going to install everything into. A good example would be `~/proxy`, or a user-specific directory that you already use for compiling packages.
1. Run `cd libimobiledevice && ./autogen.sh --prefix=/directory/here`, replacing `/directory/here` with what you chose in Step 4.
1. Run `make && make install`.
1. Make sure you aren't already using a system-wide libimobiledevice install. Run `idevicepair list`. If any device IDs appear, run `idevicepair unpair <id>`.
1. Read step 7. Failure to do so will result in me cursing you with **undefined behaviour** in anything you work on for eternity. This includes plugging things into wall outlets.
1. READ STEP 7.
1. Unplug your device.
1. After following the last four steps, it's time to make the local imobiledevice install the default.

If you want to make this persistent across shell sessions, add the following to your shell's profile. Otherwise, paste the commands in and replace `/directory/here/` with the directory you picked earlier to install libimobiledevice into. You will need to re-set your $PATH every time you want to use the debug proxy if you choose not to make these modifications persistent.

```sh
export PKG_CONFIG_PATH = $PKG_CONFIG_PATH:/directory/here/lib/pkgconfig
export PATH = /directory/here/bin:$PATH
```

Now that libimobiledevice is set up, it's time to compile the debug proxy. 

Go back to the debug proxy repo directory and run `./autogen.sh --prefix=/directory/here && make && make install`. This installs the proxy alongside the custom libimobiledevice install.

Run `ios_webkit_debug_proxy -V`. If it says it's been compiled against libimobiledevice 1.2.X or greater, you've done everything right.

Now that all the necessary stuff has been compiled and installed on your dev machine, you need to prepare your device.

1. Make sure your device is unlocked and unplugged from your dev machine.
1. Scroll down the left pane, find Safari. Go to Advanced, turn on Web Inspector.
1. (optional; but recommended) Kill Safari from the app switcher and re-open it.

Now you can connect your device and pair it:

1. Run `idevicepair pair`.
1. After receiving the device not detected message, make sure the device's screen is on and unlocked.
1. Plug the device into your dev machine.
1. DO NOT TOUCH THE TRUST DIALOG. If you need to stop the screen dimming, just tap the text.
1. Re-run `idevicepair pair`.
1. On the device, tap `Trust`. Enter your passcode if needed.
1. Re-run `idevicepair pair`.
1. Run `idevicepair list`. Verify that a device ID appears.
1. Open Safari.

You should now test the proxy by running `ios_webkit_debug_proxy -d`.

If all goes well, you should see a fair bit of terminal output, including some hex dumps of your machine and your device communicating with one another.

Go to http://localhost:9221 and verify that your device is present. Click it and you should see all the tabs that you have open. (I advise not clicking on the tabs themselves, the built in dev tools that come with the proxy don't work too well)

Use Control-C to stop the proxy. Assuming you installed the [RemoteDebug adapter](https://github.com/RemoteDebug/remotedebug-ios-webkit-adapter), you should now start it and configure your dev tool of choice to use it.

For Chrome, this involves going to `chrome://inspect`, enabling "Discover Network Targets" and adding `localhost:9000` to the list. You should then see your open tabs, click on one to open the Chrome Dev Tools on it.

Have fun! Who says you need to buy a Mac to debug web apps on iOS? (granted, it may be easier!)

