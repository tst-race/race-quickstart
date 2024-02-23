# RACE with Android on Windows

Making RIB/RACE work nicely with physical Android devices while running on Windows is slightly more complicated because WSL does not grant access to USB devices directly. Therefore, ADB has to be used _twice_ to get RIB connected to the Android device: once from the Windows side, and then again from the WSL side.

## Preliminaries

- [Download ADB 10.0.41 for Windows](https://developer.android.com/tools/releases/platform-tools)
  + Extract somewhere and open a CMD prompt to run `adb.exe`
- [Download ADB 10.0.41 for Ubuntu](https://developer.android.com/tools/releases/platform-tools)
  + Extract inside the WSL environment
- Get the Android device's IP address
  
## Run Steps
- Plug the Android device into the computer
- Ensure it is set to File Transfer mode in the USB Preferences settings
- In the CMD prompt in Windows, run `adb.exe tcpip 5555`
- In an Ubuntu for Windows terminal (_not_ inside RIB), run `adp connect <DEVICE IP>:5555`
- Use `-i <Windows IP Address>` when running `rib_2.6.0.sh` to ensure RIB has the right IP address

Deployment commands, including `deployment bridged ...` commands, should now operate correctly.
 
