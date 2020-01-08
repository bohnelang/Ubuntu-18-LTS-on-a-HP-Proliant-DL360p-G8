While setting up Ubuntu on a HP Prolinat DL360p Gen8 we run intro some problems. 

1) During the install process we could not set  the IP number of server manualy. 
     -> Thus we setup an Ubuntu 16 LTS and did a distribution upgrade. This works fine.

2) The X-Server did not work with the VGA port of the server. We got only a swirling monitor display.
    The problem is that Ubuntu 18 LTS seems not to have the mga driver for Matrox graphic cards included.
    We did not find the xserver-xorg-video-mga packague in the Ubuntu repository. Seems that it is only in Ubuntu 16 and Ubuntu 19.
    
    Just disbale this and use a default driver:
    
    Linux modprobe the graphic driver mgag200 but the X-server canot use it.
    Thus do a 
       echo "blacklist  mgag200" > /etc/modprobe.d/matrox.conf 
    to disable the driver and reboot  the server (!).
    
    Then set up a VESA X-server:
    
    cp /etc/X11/xorg.conf /etc/X11/xorg.conf.save
    
    Use an editor to do this - I use vi...
    
    vi /etc/X11/xorg.conf
    
    Edit this:
    
Section "ServerLayout"
    Identifier     "Layout0"
    Screen     	   0 "Screen0"
    InputDevice    "Keyboard0" "CoreKeyboard"
    InputDevice    "Mouse0" "CorePointer"
EndSection

Section "Files"
EndSection

Section "InputDevice"
    # generated from default
    Identifier     "Mouse0"
    Driver         "mouse"
    Option         "Protocol" "auto"
    Option         "Device" "/dev/psaux"
    Option         "Emulate3Buttons" "no"
    Option         "ZAxisMapping" "4 5"
EndSection

Section "InputDevice"
    # generated from default
    Identifier     "Keyboard0"
    Driver         "kbd"
EndSection

Section "Monitor"
    Identifier     "Monitor0"
    VendorName     "Unknown"
    ModelName      "Unknown"
    Option         "DPMS"
EndSection


Section "Device"
    Identifier     "Vesa0"
    Driver         "vesa"
EndSection


Section "Screen"
    Identifier     "Screen0"
    Device         "Vesa0"
    Monitor        "Monitor0"
    DefaultDepth    24
    SubSection     "Display"
        Depth       24
    EndSubSection
EndSection
    
 In the Xorg-log file you can see now this:
 
[    35.789] (II) VESA(0): initializing int10
[    35.789] (II) VESA(0): Primary V_BIOS segment is: 0xc000
[    35.789] (II) VESA(0): VESA BIOS detected
[    35.789] (II) VESA(0): VESA VBE Version 3.0
[    35.789] (II) VESA(0): VESA VBE Total Mem: 8192 kB
[    35.789] (II) VESA(0): VESA VBE OEM: Matrox Graphics Inc.
[    35.789] (II) VESA(0): VESA VBE OEM Software Rev: 3.8
[    35.789] (II) VESA(0): VESA VBE OEM Vendor: Matrox
[    35.789] (II) VESA(0): VESA VBE OEM Product: MGA-G200
[    35.789] (II) VESA(0): VESA VBE OEM Product Rev: 00

:-)

------

Checking your hardware:
    
lshw -c video 
      *-display UNCLAIMED
       description: VGA compatible controller
       product: MGA G200EH
       vendor: Matrox Electronics Systems Ltd.
       physical id: 0.1
       bus info: pci@0000:01:00.1
       version: 00
       width: 32 bits
       clock: 33MHz
       capabilities: vga_controller bus_master cap_list
       configuration: latency=0

lspci | grep -i VGA
01:00.1 VGA compatible controller: Matrox Electronics Systems Ltd. MGA G200EH

