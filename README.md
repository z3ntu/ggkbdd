ggkbdd
======

This project is a generic gaming keyboard daemon.

**This is just a proof of concept right now**

The basic functionality is:
- one key is designated as the **mode toggle key**
- keys can be configured to play **macros**
- when the user presses the **mode toggle key**, the keyboard switches to
  the **macro mode**.
  - when pressing a macro key, the configured macro sequence is replayed
  - any non-configured key events are discarded now
- when the user presses `Esc` or the **mode toggle key** again, the keyboard
  switches back to **normal mode**

How does it work
================

ggkbdd runs as root and listens to keyboard events. It also creates a virtual
keyboard device through uinput.

In **macro mode**, ggkbdd grabs the keyboard device so no other client can
receive events. This makes it appear as if the device is in a special mode.

When key presses for configured keys are received, the configured sequence
is simply replayed on the uinput device.

This all happens just above the kernel level, there's no desktop integration
and this happens silently without the rest of the system knowing.

How to run
==========

Use `evemu-record` or `libinput record` to figure out the device node of
your keyboard device. Then use this as argument here:

```
> sudo ./ggkbdd.py --verbose /dev/input/event3
```

Installation
============

Don't bother, because it's so small it's almost not worth it. But if you
must...

```
> sudo ./setup.py install
```

It's using python setuptool, there's heaps of documentation out there
that'll explain what options are possible.

Configuration
=============

The config file is `$XDG_CONFIG_HOME/ggkbddrc` or the one specified
with `--config path/to/config`. Its contents must be:

```
[General]
ModeKey=RIGHTCTRL

[Macros]
A=B C
F11=H E L L O
LEFTCTRL=+LEFTSHIFT U P P E R -LEFTSHIFT L O W E R
```

This example uses capslock as the mode key and maps the A key to the key
sequence "bc" and F11 to the key sequence "hello".

All key-related entries must match the key names as defined in
`linux/input-event-codes.h`. If the key is prefixed by `+` it is sent as a
key press only. If the key is prefixed by a `-`, it is sent as a key release
only. If any key is still down at the end of the macro, you're going to get
key repeats so don't do that.

Use `evemu-record` or `evtest` to read the key codes.

Coincidentally: capslock is a really bad mode key because we can only
grab it **after** the key was sent and processed by everyone else too. So
you'll have your capslock key randomly stuck. Pick a different key. There, I
saved you some debugging if you bothered to actually read the README.

Keyboard layouts
================

ggkbdd doesn't care about the keyboard layout you may have configured in the
desktop environment, it sits below all that and effectively uses a US
keyboard layout without a shift key. The keys always send the same evdev
codes, whether you have qwerty, azerty or dvorak configured.

This goes for both the input key and the output keys. If you map `KEY_Q` to
`KEY_Z` this effectively maps 'a' to 'w' in azwerty or 'q' to 'y' in a
german keyboard layout.

You cannot map shift level keys like the exclamation mark, it's on `KEY_1`.

systemd autostart
=================

Use this systemd file:
```
> cat /etc/systemd/system/ggkbd@.service
[Unit]
Description=ggkbbd auto-starter
BindsTo=dev-input-%i.device

[Service]
Type=simple
ExecStart=/usr/bin/ggkbdd --config /path/to/ggkbddrc /dev/input/%I
```

Use this udev rule:
```
> cat /etc/udev/rules.d/99-ggkbdd.rules
ACTION=="remove", GOTO="ggkbdd_end"
KERNEL!="event*", GOTO="ggkbdd_end"

KERNELS=="input*", \
   ENV{ID_INPUT_KEYBOARD}=="1", \
    ATTRS{name}=="AT Translated*", \
    TAG+="systemd", ENV{SYSTEMD_WANTS}="ggkbdd@%k.service"

LABEL="ggkbdd_end"
```
This will start ggkbdd for every keyboard with "AT Translated" in the name.
ggkbdd creates a virtual device `GGKBDD your device's real name`, better
make sure the udev rule doesn't match that too.

About
=====

The name ggkbdd was chosen to make it as weird as possible to type. You're
welcome.
