# Freebsd-Acer-Nitro-Laptops-AN515-51/58-XXX


## 14-stable and 14-release changes to enable webcam and mouspad

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

2) Be sure the _cuse_ module is loaded into the kernel. At the command prompt type:
   ```
   vi /boot/loader.conf
   ```
   Add this line to the file.
   ```
   cuse_load="YES"
   ```

   To load the module on the fly type at the command prompt:
   ```
   kldload cuse
   ```

4) Change webcamd uvc_driver.c source file and rebuild. Determine that the webcam was detected at boot. At the command prompt type:
   ```
   lsusb
   ```
   The webcam (Quanta) should be shown along with an ID as part of the messages.
   ```
   Bus /dev/usb Device /dev/ugen1.3: ID 0408:4035 Quanta Computer, Inc.
   ```
   This same information is provided by _usbconfig_ and _dmesg_:
   ```
   ugen1.3: <Quanta ACER HD User Facing> at usbus1, cfg=0 md=HOST spd=HIGH (480Mbps) pwr=ON (500mA)
   ugen1.3: <Quanta ACER HD User Facing> at usbus1
   ```
   
   If the webcam was not detected by the system go look for the trouble before continuing. On this hardware a variety of Quanta products IDs may be shown.
   Note: The ugen number, e.g. ugenx.y, depends upon how many USB devices are attached to the system.

   Navigate to ports/multimedia/webcamd.
   ```
   cd /usr/ports/webcamd
   make do-configure
   ```

   Webcamd will be downloaded and configured but not built.
   ```
   cd /usr/ports/multimedia/webcamd/work/linux-5.17-rc1/drivers/media/usb/uvc
   ```
   Edit the _uvc_driver.c_ file. At the command prompt type:
   ```
   cp uvc_driver.c uvc_driver.c.ori
   vi uvc_driver.c
   ```
   Save the original file prior to editing. On this Acer Nitro AN515-58-XXX laptop the USB webcam identified as 0x4035 (see above). Search for "Quanta" in the     file and add the following code at the end of the "Quanta" section.
   
       /* Quanta ACER HD User Facing  0x4035 - Experimental */
        { .match_flags  = USB_DEVICE_ID_MATCH_DEVICE
                        | USB_DEVICE_ID_MATCH_INT_INFO,
          .idVendor = 0x0408,
          .idProduct = 0x4035,
          .bInterfaceClass = USB_CLASS_VIDEO,
          .bInterfaceSubClass = 1,
          .bInterfaceProtocol = UVC_PC_PROTOCOL_15,
          .driver_info = (kernel_ulong_t) &(const struct uvc_device_info ) {
          .uvc_version = 0x010a, } },
  

  Note the ending comma "," before the start of the next section.

  Build webcamd. 
  ```
  cd /usr/ports/multimedia/webcamd
  make
  make install
  ```

Test the webcam. At the command prompt type:
```
webcamd
```

A list of USB devices should be shown.
```
Available device(s):
webcamd [-d ugen0.1] -N Intel-XHCI-root-HUB -S unknown -M 0
webcamd [-d ugen1.1] -N Intel-XHCI-root-HUB -S unknown -M 1
webcamd [-d ugen1.2] -N vendor-0x1ea7-2-4G-Mouse -S unknown -M 0
webcamd [-d ugen1.3] -N Quanta-ACER-HD-User-Facing -S 01-00-00 -M 0
webcamd [-d ugen1.4] -N vendor-0x8087-product-0x0026 -S unknown -M 0
Show webcamd usage:
webcamd -h
```

Run webcamd with the camera aruments shown in the output above. At the command prompt type:
```
/usr/ports/multimedia/webcamd]# webcamd -d ugen1.3 -N Quanta-ACER-HD-User-Facing -S 01-00-00 -M 0
```

One should see webcamd attach to the video devices.
```
webcamd 73114 - - Attached to ugen1.3[0]
webcamd 73114 - - Creating /dev/video0
webcamd 73114 - - Creating /dev/video1
```

Any video application may use the _/dev/videoX_ device nodes. At the command prompt type:
```
pkg install pwcview
pwcview
```

The webcam green led will turn on and a new window opens with image.
To make the _webcamd_ start at boot, at the command prompt type:
   ```
   vi /etc/rc.conf
   ```
   Add this line to the file.
   ```
   webcamd_enable="YES"
  ```

The device nodes _/dev/videoX_ are owned by webcamd. To make them accessible to $USER at the command prompt type:
```
chown $USER







   
   
   
