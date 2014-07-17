xcode-setup
===========




# Script to sign and sync App to iOS Device


    ```bash
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
