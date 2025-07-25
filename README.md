libcec-daemon
=============
A simple daemon to connect libcec to uinput. That is, using your TV to control your PC!
by Andrew Brampton

Licence
=======
Currently under the GPL, but only because libcec (on which we depend) is under
the GPL. I have full intentions to relicence this project as BSD when libcec
changes to either LGPL, or I do not need to depend on it anymore.

Build
=====
* Checkout the main source

```
git clone git://github.com/bramp/libcec-daemon.git
```

* Now we need some buildtools and libraries

```
sudo apt-get install build-essential autoconf 
sudo apt-get install libboost-program-options-dev libboost-thread-dev libboost-system-dev liblog4cplus-dev
```

* Also we need the libcec (version 3.x) libraries. Pulse eight provides east way to install

```
apt-get install libcec-dev
```

* We need to build the platform stuff from pulse eight (for that we need cmake)

```
# so install cmake if you don't already have
sudo apt install cmake
git clone https://github.com/Pulse-Eight/platform.git p8-platform
cd p8-platform
mkdir build && cd build
cmake ..
make
sudo make install
```

* Now build libcec-daemon

```
cd libcec-daemon
./bootstrap && ./configure && make
```

Usage
====
```
Usage: libcec-daemon [options] [usb]

Allowed options:
  -h [ --help ]             show help message
  -V [ --version ]          show version (and exit)
  -d [ --daemon ]           daemon mode, run in background
  -l [ --list ]             list available CEC adapters and devices
  -v [ --verbose ]          verbose output (use -vv for more)
  -q [ --quiet ]            quiet output (print almost nothing)
  -a [ --donotactivate ]    do not activate device on startup
  -k [ --keymap ] <file>    load key mapping from file
  --onstandby <path>        command to run on standby
  --onactivate <path>       command to run on activation
  --ondeactivate <path>     command to run on deactivation
  -p [ --port ] [a[.b.c.d]> HDMI port A or address A.B.C.D (overrides 
                            autodetected value)
  --usb <path>              USB adapter path (as shown by --list)

HDMI port A can be specified as tv.1 or av.1 for HDMI port 1 on respectively the
TV or a connected Audio System. 0 digit is optional for either port or physical
address, so that tv.0 is equivalent to just tv. Similarily, addresses such as
1, 1.0, 1.0.0 and 1.0.0.0 are also equivalent. Specifying 0 by itself is the same
as omitting the port argument, implying that the HDMI port will be autotected.
To summarize, to force libcec-daemon to manage TV HDMI port 1, the following
can be used:
      -p tv.1
      -p 1
      -p 1.0.0.0
The daemon will not work properly if it fails to detect the HDMI port, in which
case the port should be specified manually.

It is possible to run commands to react to a certain TV/AV events such as:
     - power off/standby event (--onstandby)
     - HDMI port switched in (--onactivate)
     - HDMI port switched out (--ondeactivate)
The <path> argument should specify a command or script that is executable by the
default shell for the user running libcec-daemon, usually /bin/sh for root user.
Typically, scripts would suspend/shutdown the host whenever a standby event is
received for power saving, and screensaver and/or media play/pause control could
be hooked to activation or deactivation events. Currently, The command is run
right after the event has occurred, therefore it cannot be invalidated/prevented
by the former returning an exit code other than 0 for example.

A libcec-daemon can be instantiated for each HDMI-CEC adapter available to the
host hardware, and the daemon will automatically use to the first detected one.
If more than one adapter is available, they should be specified by the usb
argument using either its sys-path or dev-path as listed by the --list argument.
```

Key Mapping Configuration
=========================
libcec-daemon supports configurable key mappings through external configuration files,
allowing you to customize how CEC remote control keys are mapped to system input events.

## Usage

```bash
# Use Jellyfin-optimized mappings
libcec-daemon --keymap keymaps/jellyfin.conf

# With debug output to see key events
libcec-daemon -v --keymap keymaps/jellyfin.conf
```

## Configuration Files

* `keymaps/jellyfin.conf` - Optimized for Jellyfin Media Player (SELECT→ENTER, EXIT→ESC)
* `keymaps/default.conf` - Matches the original hardcoded mappings

## File Format

Configuration files use a simple `CEC_KEY=UINPUT_KEY` format:
```
# Comments start with #
SELECT=ENTER
EXIT=ESC
UP=UP
DOWN=DOWN
```

See the included configuration files for complete lists of available CEC and uinput key names.

## Debugging

Use the `-v` flag to see key press events and mapping information:
```bash
libcec-daemon -v --keymap your_config.conf
```
