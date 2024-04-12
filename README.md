# Freebsd-Acer-Nitro-Laptops-AN515-51/58-XXX
Camera and mousepad 

# 14-stable and 14-release changes to enable webcam and mouspad

1) Change the laptop mousepad from iic to psm protocol. Mousepad iic protocol is not supported. To do so one must enter the BIOS advanced mode.
  ```
    a) Press Fn+Tab three times. Reboot the laptop.
    b) Press F4, 4, R, V, F5, 5, T, G, B, F6, 6, Y, H, N while the laptop is turned off.
    c) Hold Fn+Tab while starting the computer, before entering the BIOS. Press F2 to enter the bios. Type ctrl-s.
    d) In the Main menu:
   			    Touchpad:		iic
    e) Change the iic value to psm:
   			    Touchpad:		psm
    f) Press F10 to save and exit.
```

2) Change webcamd uvc_driver.c source file and rebuild. First, dtermine that the webcam was detected at boot. At the command prompt type:
   ```
   lsusb
   ```
   The webcam (Quanta) should be shown along with an ID as part of the messages.
   ```
   Bus /dev/usb Device /dev/ugen1.3: ID 0408:4035 Quanta Computer, Inc.
   ```
   This same information is provided by _dmesg_:
   ```
   ugen1.3: <Quanta ACER HD User Facing> at usbus1
   ```
   If the webcam was not detected by the system go look for the trouble before continuing. On this hardware a variety of Quanta products IDs may be shown.
   >[!Note]
   >The ugen number, e.g. ugenx.y, depends upon how many USB devices are attached to the system.

   Navigate to ports/multimedia/webcamd.
   ```
   cd /usr/ports/webcamd
   make do-configure
   ```

   Webcamd will be downloaded and configured but not built.
   ```
   cd /usr/ports/multimedia/webcamd/work/linux-5.17-rc1/drivers/media/usb/uvc
   ```
   
   
