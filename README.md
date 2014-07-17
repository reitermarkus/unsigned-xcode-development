xcode-setup
===========

#Preparing Xcode To Allow Building Unsigned iOS Applications & Installing ldid Utility

## Disable Code Signing Requirement

1. navigate to __/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS#.#.sdk/__
2. open __SDKSettings.plist__
3. expand the section `DefaultProperties`
4. set the value of `CODE_SIGNING_REQUIRED` to `NO`
5. restart Xcode

## Install Saurik's ldid utility

1. open Xcode preferences and select the __Downloads__ tab
2. install the __Command Line Tools__
3. open Terminal and execute the following commands:
```applescript
mkdir -p ~/Documents/Git/
cd ~/Documents/Git/
git clone git://git.saurik.com/ldid.git
cd ldid
git submodule update --init
./make.sh
sudo cp ldid /usr/bin/
```

Note: The above steps only need to be performed once. After Xcode is properly prepared and ldid is installed, only the following steps will need to be repeated for building unsigned applications and installing them to a device.


Building An Unsigned Application In Xcode:
Open the project in Xcode which is to be run on a non-provisioned iOS device.
At the top of the Project Navigator, select the project.

Select Project

In the "Code Signing" section, select "Don't Code Sign" for "Code Signing Identity".

Disable CodeSigning

Ensure the Build Destination is set to "iOS Device".

Build Destination


Build the application (⌘-B).

Add Required SHA1 Hashes To Application Binary:
In the Xcode Project Navigator, expand the section "Products".
Right-click the application binary (HelloWorld.app) and select "Show In Finder".

Show In Finder

Copy the application binary to the desktop.
Open Terminal on the OSX system where the application was built.
Add the SHA1 hashes to the application binary:

```applescript
# cd ~/Desktop/
# ldid -S HelloWorld.app/HelloWorld
```
Copy Application To Device and Reload UI Cache:
Open Terminal on the OSX system where the application was built.
Change to the desktop directory where the application was copied and use the scp utility to copy the application to the device:
```applescript
# cd ~/Desktop/
# scp -r HelloWorld.app/ root@192.168.1.161:/Applications/
```
Connect to the device via SSH, login as the user 'mobile', rebuild the UI cache, and re-spring the device:
```applescript
# ssh -l mobile 192.168.1.161
# uicache
# killall backboardd
```
Note: The backboardd process can be replaced with SpringBoard for iOS versions prior to 6.0. Using SpringBoard for iOS6+ will still work, but the screen will be dimmed. Pressing the power button once then waking the device back up will bring brightness back to the previous setting.

Note: The IP address shown in the above commands (192.168.1.161) should be replaced with the IP address of the iOS device being used. This can be obtained within the iOS Settings app by tapping the arrow icon next to the WiFi Access Point the device is connected to. Before running the above commands, OpenSSH must be installed and enabled on the device.

After performing the above steps, your application should now be visible on the device's SpringBoard. I have read that the ldid command and scp procedure can be added to a script so it's automatically performed every time a build is completed in Xcode. Once I figure out how to do this, the above guide will be revised.


# Script to sign and sync App to iOS Device


```applescript
#!/bin/sh

# Modify this to your device's IP address.
IP="10.0.0.31"
    
# Verify that the build is for iOS Device and not a Simulator.
if [ "$NATIVE_ARCH" != "i386" ]; then

  # Kill any running instances and remove the app folder.
  ssh root@$IP "killall '${TARGETNAME}'; rm -rf '/Applications/${WRAPPER_NAME}'"

  # Self sign the build.
  ldid -S $BUILT_PRODUCTS_DIR/"${WRAPPER_NAME}"/$TARGETNAME

  # Copy it over.
  scp -r $BUILT_PRODUCTS_DIR/"${WRAPPER_NAME}" root@$IP:/Applications/

  ssh root@$IP "su -c uicache mobile"

  ssh root@$IP "open reitermarkus.Crystal-Ball"

  # This part just creates create an OS X notification to let you know that the process is done.
  # You can get terminal-notifier from https://github.com/alloy/terminal-notifier.
  /Applications/terminal-notifier.app/Contents/MacOS/terminal-notifier -title "Build fertig" -message "${CFBundleIdentifier} wurde auf ${IP} installiert."

fi
```
