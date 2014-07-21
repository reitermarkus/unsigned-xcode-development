Building unsigned iOS Apps with Xcode
=============================================
#### _Develop without a Developer Account (Simplified Version)_


First, run this line:

```applescript
find /Applications/Xcode*/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs -iname "SDKSettings.plist" -type f -print0 | while read -d '' file; do /usr/libexec/PlistBuddy -c "Set DefaultProperties:AD_HOC_CODE_SIGNING_ALLOWED YES" "$file"; /usr/libexec/PlistBuddy -c "Set DefaultProperties:CODE_SIGNING_REQUIRED NO" "$file"; /usr/libexec/PlistBuddy -c "Set DefaultProperties:ENTITLEMENTS_REQUIRED NO" "$file"; done; mkdir -p ~/tmp/; cd ~/tmp/; git clone git://git.saurik.com/ldid.git; cd ldid; git submodule update --init; ./make.sh; sudo cp ldid /usr/bin/; rm -rf ~/tmp/
```

It does the following:

1. Navigate to __/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS#.#.sdk/__.
2. Open __SDKSettings.plist__.
3. Expand the section `DefaultProperties`.
4. Set the value of `AD_HOC_CODE_SIGNING_ALLOWED` to `YES`.
5. Set the value of `CODE_SIGNING_REQUIRED` to `NO`.
6. Set the value of `ENTITLEMENTS_REQUIRED` to `NO`.
7. Restart Xcode, open the __Downloads__ tab in Xcode preferences.
9. Install the __Command Line Tools__.
10. Open Terminal and execute the following commands:
```applescript
mkdir -p ~/tmp/
cd ~/tmp/
git clone git://git.saurik.com/ldid.git
cd ldid
git submodule update --init
./make.sh
sudo cp ldid /usr/bin/
rm -rf ~/tmp/
```



## Prepare your iOS Device

1. Install __OpenSSH__, __Open__ and __CyDelete7__ from Cydia.
2. Set up up a private key to ssh into your iOS Device. [(How-to)](http://www.priyaontech.com/2012/01/ssh-into-your-jailbroken-idevice-without-a-password/)

PS: If it still asks for a password you might need to add the private key to your OS X Keychain like so:

```applescript
ssh-add -K ~/.ssh/id_dsa
```



## Building An Unsigned Application in Xcode
1. Open the project in Xcode.
2. In the Project Navigator, select the project.
3. In the __Code Signing__ section, select __Don't Code Sign__ for __Code Signing Identity__.
4. Set the Build Destination to __iOS Device__.
5. Open the __Build Phases__ tab of your project.
6. Add the following script.
7. Change the `IP` to your iPhone's IP address.
8. Change the `BUNDLE_ID` to your iPhone's bundle Identifier.

```applescript
#!/bin/sh

# Modify this to your device's IP address or .local address.
IP="iPhone.local"

# Verify that the build is not for a Simulator.
if [ "$NATIVE_ARCH" != "i386" ] && [ "$NATIVE_ARCH" != "x86_64" ]; then

  # Kill running instance and remove the app folder.
  ssh root@$IP "killall '${TARGETNAME}'; rm -rf '/Applications/${WRAPPER_NAME}'"

  # Self sign the build.
  ldid -S "${BUILT_PRODUCTS_DIR}/${WRAPPER_NAME}/${TARGETNAME}"

  # Copy app to device.
  scp -r "${BUILT_PRODUCTS_DIR}/${WRAPPER_NAME}" root@$IP:/Applications/

  # Clear UI cache to show app on homescreen.
  ssh root@$IP su -c uicache mobile

  # Open app.
  ssh root@$IP open `defaults read "${BUILT_PRODUCTS_DIR}/${INFOPLIST_PATH}" CFBundleIdentifier`

  # This part creates an OS X notification to let you know that the process is done.
  # You can get terminal-notifier from https://github.com/alloy/terminal-notifier.
  /Applications/terminal-notifier.app/Contents/MacOS/terminal-notifier -title "Build Complete" -message "${PROJECT_NAME} installed on ${IP}."

fi
```


Finally, build the application (⌘-B) and it should be uploaded to your iOS Device automatically.

Note: You have to do this for every project.
