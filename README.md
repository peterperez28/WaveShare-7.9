# WaveShare-7.9

Screen rotation
Currently there is no support for rotation at runtime, do not use xrandr to rotate the screen. Configure the server to start in the desired orientation, there are many ways to achieve this, here is one:

Create /usr/share/X11/xorg.conf.d/90-monitor.conf


sudo nano /usr/share/X11/xorg.conf.d/90-monitor.conf
Paste this section modifying the options to suit your needs:


Section "Monitor"
    Identifier "DPI-1"
    # This identifier would be the same as the name of the connector printed by xrandr.
    # it can be "HDMI-0" "DisplayPort-0", "DSI-0", "DVI-0", "DPI-0" etc

    Option "Rotate" "left"
    # Valid rotation options are normal,inverted,left,right


    Option "PreferredMode" "1920x1080"
    # May be necesary if you are not getting your prefered resolution.
EndSection
Save the file and restart KlipperScreen.

sudo service KlipperScreen restart
Touchscreen touch rotation
If your touchscreen isn't registering touches properly after the screen has been rotated, you will need to apply a transformation matrix.

First you will need your device name, on a terminal run:


DISPLAY=:0 xinput
Output:


⎡ Virtual core pointer                          id=2    [master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer                id=4    [slave  pointer  (2)]
⎜   ↳ ADS7846 Touchscreen                       id=6    [slave  pointer  (2)]
⎣ Virtual core keyboard                         id=3    [master keyboard (2)]
    ↳ Virtual core XTEST keyboard               id=5    [slave  keyboard (3)]
In this case the device is the ADS7846 Touchscreen, yours may be different
You can test a change by running:

***7.9 Display config*****
DISPLAY=:0 xinput set-prop "WaveShare WaveShare" 'Coordinate Transformation Matrix' -1 0 1 0 -1 1 0 0 1
***************************
Where the matrix can be one of the following options:

0°: 1 0 0 0 1 0 0 0 1
90° Clockwise: 0 -1 1 1 0 0 0 0 1
90° Counter-Clockwise: 0 1 0 -1 0 1 0 0 1
180° (Inverts X and Y): -1 0 1 0 -1 1 0 0 1
invert Y: -1 0 1 1 1 0 0 0 1
invert X: -1 0 1 0 1 0 0 0 1
For example:


DISPLAY=:0 xinput set-prop "ADS7846 Touchscreen" 'Coordinate Transformation Matrix' -1 0 1 0 -1 1 0 0 1
To make this permanent, modify the file /etc/udev/rules.d/51-touchscreen.rules and add following line:


ACTION=="add", ATTRS{name}=="<device name>", ENV{LIBINPUT_CALIBRATION_MATRIX}="<matrix>"
As an alternative if the above doesn't work:

edit /usr/share/X11/xorg.conf.d/40-libinput.conf

for example:
********7.9 display********

Section "InputClass"
        Identifier "libinput touchscreen catchall"
        MatchIsTouchscreen "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
        Option "TransformationMatrix" "-1 0 1 0 -1 1 0 0 1"
EndSection
