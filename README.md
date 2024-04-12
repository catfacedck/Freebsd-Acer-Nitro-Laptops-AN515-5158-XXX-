# Freebsd on Acer Nitro AN515-51/58-XXX Series Laptops


## 14-stable and 15-current changes to enable mouspad and webcam

These laptops typically come in a variety of hardware sku configurations:
  - AMD 7000 and above Ryzen cpus
  - Intel I12/I13/I14 -500/700/900 cpus
  - DDR4/DDR5 Ram
  - Thunderbolt 4
  - 165 Hz or 144 Hz display panels
  - AMD Radeon Graphics
  - Integrated Intel Graphics -P GT2 Iris Xe Graphics
  - Nvidia 30XX or 40xx Graphics
  - Integrated Intel WiFi 6/6E AX2XX series (sometimes referred to as Killer Wi-Fi 6 1650)
  - Integrated Mediatek MT7921 WiFi or variants
  - Killer Ethernet E2600 (was Realtek now Intel)
  - Realtek and Nvidia Audio
  - Bluetooth >= 5.1
  - Integrated camera
  - MicroSDTM Card Reader
  - Internel - two (2) nvme drive connectors and one (1) SATA drive connector - three (3) internel drives total.

**What you will get after installation of 14-stable or 15-current:**
  - Killer Ethernet network based Freebsd system using drm Integrated Intel Graphics -P GT2 Iris Xe Graphics driver, 1920x1080 165 Hz screen, Intel integrated     WiFi iwlwifi (Killer Wifi), mousepad, USB wireless mouse support, audio, mutiple drives, USB.

**What does not work:**
  - Suspend/sleep keys, iic mousepad, Mediatek WiFi, bluetooth, microSDTM Card Reader.

>[!Note]
>This Freebsd installation was tested on Intel I5-12500 and Intel I7-12650 systems with Integrated Intel Graphics/Wifi, Nvidia 3050/4050 Graphics, and Killer Ethernet E2600.

### **_Step-by step install procedure for mousepad and webcam:_**

Typically these laptops are delived with Windows 11 installed on an internal 500 GB or 1 GB drive. Read the handbook, do a zfs root install on a different
drive partition or a second drive. Both options require the use of rEFind https://www.rodsbooks.com/refind/ for booting.

1) Add the following content to /etc/sysctl.conf, /boot/loader.conf, and /etc/rc.conf as a baseline.

    /boot/loader.conf
    ```
    cryptodev_load="YES"
    zfs_load="YES"
    sysctlinfo_load="YES"
    sysctlbyname_improved_load="YES"
    cuse_load="YES"
    coretemp_load="YES"
    hint.iichid.0.disabled="1"
    vmm_load=”YES”
    nmdm_load="YES"
    ```

    /etc/sysctl.conf
    ```
    hw.snd.default_unit=1
    ```

     /etc/rc.conf
    ```
    # Basic services
    hostname="Elephant"
    keymap="us.kbd"
    #moused_enable="YES"
    dbus_enable="YES"
    #slim_enable="YES"
    update_motd="NO"
    devmatch_enable="YES"

    # Power
    powerd_enable="YES"
    powerd_flags="-n hiadaptive -a hiadaptive -b hiadaptive"
    performance_cx_lowest="Cmax"
    economy_cx_lowest="Cmax"

    # Misc
    zfs_enable="YES"
    fuse_load="YES"
    clear_tmp_enable="YES"
    ifconfig_re0="DHCP"
    microcode_update_enable="YES"

    # Wlan
    wlans_iwlwifi0="wlan1"
    ifconfig_wlan1="WPA DHCP"

    Misc1
    syslogd_flags="-ss"
    sendmail_enable="NO"
    sendmail_msp_queue_enable="NO"
    sendmail_outbound_enable="NO"
    sendmail_submit_enable="NO"
    ntpd_enable="NO"
    linux_enable="YES"
    webcamd_enable="YES"

    # Intel GPU drivers
    kld_list="i915kms fusefs acpi_video nmdm"

    # Packet filter
    pf_enable="YES"
    pf_rules="/etc/pf.conf" 
    pflog_enable="YES"
    pflog_logfile="/var/log/pflog"
    ```


2) Enable the mousepad. Change the laptop mousepad from iic to psm protocol as the iic protocol is not supported. To do so one must enter the BIOS advanced mode.
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

3) Enable the webcam. Be sure the _cuse_ module is loaded into the kernel. At the command prompt type:
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

4) Change webcamd uvc_driver.c source file and rebuild. (NOTE: the identical change was required for Ubuntu 23.10 on the same laptop hardware.)

   Determine that the webcam was detected at boot. At the command prompt type:
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

    Run webcamd with the camera arguments shown in the output above. At the command prompt type:
    ```
    /usr/ports/multimedia/webcamd]# webcamd -d ugen1.3 -N Quanta-ACER-HD-User-Facing -S 01-00-00 -M 0
    ```
    
    One should see webcamd attach to the video devices.
    ```
    webcamd 73114 - - Attached to ugen1.3[0]
    webcamd 73114 - - Creating /dev/video0
    webcamd 73114 - - Creating /dev/video1
    ```

    Video applications may now use the _/dev/videoX_ device nodes. At the command prompt type:
    ```
    pkg install pwcview
    pwcview
    ```

    The webcam green led will turn on and a new window opens with image.
    To make _webcamd_ start at boot, add this line to the _/etc/rc.conf_ file.
   
    ```
       webcamd_enable="YES"
    ```

    The device nodes _/dev/videoX_ are owned by webcamd. To make them accessible to $USER at the command prompt type:
   
    ```
    vi /etc/group
    ```
    
    Add $USER to the _webcamd_ group to access the laptop webcam. 
    ```
    webcamd:*:145:$USER
    ```
    
    Where $USER is the current login name. Login as $USER. At the command prompt type:
   
    ```
    pwcview -d /dev/video0
    ```

    The webcam green led will turn on and a new window opens with image. Tested with firefox.

5) _Caveats_

   > [!NOTE]
   > <ins>Freebsd 14-stable/15-current:</ins> support WiFi and Intel -P GT2 Iris Xe Graphics using drm-61-kmod. Wifi (iwlwifi) is problematic sometimes hanging the system and/or dropping connections. Freebsd 14-release (or less) does not support Intel WiFi or Graphics on this hardware platform as of March 2024. 







   
   
   
