---
toc: true
layout: post
description: Installing and customization of the Huion 950p graphic tablet.
categories: [markdown]
title: Huion 950p graphic tablet in Linux
---
# Still doesn't render, is this the problem? 

All these were tested on Ubuntu 20.04 but the distro shouldn't make a big 
difference here.

## Recognizing the pad
When plugged for the first time, the stylo is recognized out of the box, but the buttons are not.

Following [this SO answer](https://askubuntu.com/questions/1000869/how-to-run-the-new-huion-tablets-on-linux?#tab-top), 
you can create `/usr/share/X11/xorg.conf.d/99-huion950P.conf` with the following content:
```
Section "InputClass"
    Identifier "Huion tablets with Wacom driver"
    MatchUSBID "256c:006d*"
    MatchIsTablet "true"
    MatchDevicePath "/dev/input/event*"
    Driver "wacom"
EndSection
```

You can then restart (or restart the x server only), and [wacom](https://www.wacom.com/en-us) 
recognizes both the stylo and the pad. 
To list the recognized devices, you can run `xsetwacom --list`. Your output should look like this:
```
HID 256c:006d Pen stylus        	id: 12	type: STYLUS    
HID 256c:006d Pad pad           	id: 13	type: PAD    
```

## Customizing things

From there, `xsetwacom` can be used to customize the buttons, which display you're drawing on etc.. 

### Exploring `xsetwacom`
Let's first see what `xsetwacom` can be used for: 

```bash
$ xsetwacom --help
Options:
 -h, --help                 - usage
 -v, --verbose              - verbose output
 -V, --version              - version info
 -d, --display "display"    - override default display
 -s, --shell                - generate shell commands for 'get'
 -x, --xconf                - generate xorg.conf lines for 'get'

Commands:
 --list devices             - display detected devices
 --list parameters          - display supported parameters
 --list modifiers           - display supported modifier and specific keys for keystrokes
 --set "device name" parameter [values...] - set device parameter by name
 --get "device name" parameter [param...]  - get current device parameter(s) value by name
```

What kind of parameters can we access/change? (partial output). 
```bash
$ xsetwacom --list parameters
Area             - Valid tablet area in device coordinates. 
Button           - X11 event to which the given button should be mapped.
Rotate           - Sets the rotation of the tablet. Values = none, cw, ccw, half (default is none). 
ResetArea        - Resets the bounding coordinates to default in tablet units. 
MapToOutput      - Map the device to the given output. 
all              - Get value for all parameters. 
```

We can then `--get` and `--set` those parameters, using the device name. 
The device name can eiter be the name, or the device ID. 
```bash
$ xsetwacom --get 12 all
...
$ xsetwacom --get 13 all
...
```

### Setting buttons

Considering the buttons are on the left of the device, starting from the top the 
buttons of the pad are numbered: 1, 2, 3, 8, 9, 10, 12 (not clear why). 

You can set the button using `xsetwacom` again, have a look at the [`man` page](https://www.systutorials.com/docs/linux/man/1-xsetwacom/)
and the **Button** section. 
>  The "key" keyword is followed by a list of key names. These can optionally be preceded by "+" for press and "-" for release. If +/- is not given, press-and-release is assumed, except for modifier keys which are left pressed. Key names can be X11 KeySyms or some aliases such as 'shift' or 'f1' (the full list can be seen with the list modifiers command).
>
> To assign a key that is not in the modifiers list, use the KeySym in /usr/include/X11/keysymdef.h with the XK_ prefix removed or its actual value as is. For example, XK_BackSpace should be specified as "BackSpace". "0xff80" can also be used to replace "BackSpace" since it's the unique KeySym value of Backspace key.
>
> Here is a combined example: "key +a shift b shift -a 0xff0d" converts the button into a series of keystrokes. In this example, "press a, press shift, press and release b, release shift, release a, then press and release enter". "key +a +shift b -shift -a 0xff0d" does the same thing. 

To open a terminal (CTRL+ALT+T), for example `xsetwacom --set "HID 256c:006d Pad pad" Button 1 "key +ctrl +alt t -ctrl -alt"`

Buttons for the stylo:
- **Button 1**: When the stylo touches the pad.
- **Button 2**: Bottom button.
- **Button 3**: Top button.

### Handling multiple monitors
To list connected monitor, use `xrandr`

```bash
$ xrandr | grep [^dis]connected
eDP-1 connected (normal left inverted right x axis y axis)
HDMI-1 connected primary 1920x1080+0+0 (normal left inverted right x axis y axis) 598mm x 336mm
```

To lock the pad to the external screen, use this

```bash
xsetwacom set "HID 256c:006e Pen stylus" MapToOutput HDMI-1
```

Other parameters can be modified such as `Area` to draw on a smaller piece of your screen, 
`Rotate` to handle portrait as well as landscape etc...
