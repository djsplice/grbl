# Woodpecker 3.5 installation notes

The Woodpecker 3.5 board comes installed with a boot-loader for the Arduino Nano, and by all accounts is bigger than the Uno boot=loader which provides the same functionality.

When trying to compile grbl 1.1h with CoreXY enabled, it's too big and won't fit with the stock boot-loader.

I've taken it upon myself to prune out the coolant feature options and trim down some config from the stock grbl code in order to reduce size, without compromising functionality I need.

## CoreXY modifications

config.h modifications included:
```
//#define REPORT_FIELD_BUFFER_STATE // Default enabled. Comment to disable.
//#define REPORT_FIELD_PIN_STATE // Default enabled. Comment to disable.
//#define REPORT_FIELD_CURRENT_FEED_SPEED // Default enabled. Comment to disable.
//#define REPORT_FIELD_WORK_COORD_OFFSET // Default enabled. Comment to disable.
//#define REPORT_FIELD_OVERRIDES // Default enabled. Comment to disable.
//#define REPORT_FIELD_LINE_NUMBERS // Default enabled. Comment to disable.
```
And removed references to coolant variables / functions across the codebase. See (this commit)[https://github.com/djsplice/grbl/commit/8dad7fc579d965723c0a20702d1fdc92dc65ce32] for all the gore.

## Pen-Lift Modifications 
I've also needed to compile a second firmware to support my Pen Plotter configuration based on [ManiacalLabs/grbl at builds](https://github.com/ManiacalLabs/grbl/tree/builds) m3_servo.h modification. Again, after enabling CoreXY and adding the updated `spindle_control.c` the code was slightly too big to fit. More hackery ensued.

I had to comment out
```
//#define ENABLE_BUILD_INFO_WRITE_COMMAND // '$I=' Default enabled. Comment to disable
```

And enabled:
```
#define SETTINGS_RESTORE_ALL (SETTINGS_RESTORE_DEFAULTS | SETTINGS_RESTORE_PARAMETERS | SETTINGS_RESTORE_STARTUP_LINES | SETTINGS_RESTORE_BUILD_INFO)
```

I'll likely try to install the Uno boot-loader when I'm ready to completely hose my working setup!

## Pre-Compiled Hex Files:

I've stashed compiled firmware for the laser and the pen-lift in the releases directory.

Flash the Woodpecker 3.5 board using avrdude:

### Laser
/Users/jbarrows/Library/Arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17/bin/avrdude -C/Users/jbarrows/Library/Arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17/etc/avrdude.conf -v -patmega328p -carduino -P/dev/cu.usbserial-14310 -b57600 -D -Uflash:w:/laser.hex:i

### Pen-Lift
/Users/jbarrows/Library/Arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17/bin/avrdude -C/Users/jbarrows/Library/Arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17/etc/avrdude.conf -v -patmega328p -carduino -P/dev/cu.usbserial-14310 -b57600 -D -Uflash:w:/pen-lift.hex:i


## Building the firmware using the Arduino IDE

If you need to re-build the firmware with different options, copy the contents of the arduino_lib directory into your local Arduino Library grbl folder. In my case: `Documents/Arduino/libraries/grbl`

Then, copy the `config.h and spindle_control.c` files from the `laser` or `pen-lift` directory depending on what you want to build for.
