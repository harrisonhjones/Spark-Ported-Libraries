Porting Libraries for the Spark Core
======================

This repo includes a (work in progress) how-to on porting non-Spark C/C++ (Arduino, Teensy, etc) libraries to the SparkCore and a list of ported libraries.

## Contributing 
If you'd like to contribute to either the how-to or the library list feel free to either

* Fork this repo, make changes, and issue a pull request
* Submit an issue and I'll try to get on it asap!

## Library List
The library list can be found [**here**](https://github.com/harrisonhjones/Spark-Ported-Libraries/blob/master/LIBRARY-LIST.md)

## Porting Arduino Libraries
### Making #include "Arduino.h" portable
#### Common Problems
``WProgram.h: No such file or directory``

``Arduino.h: No such file or directory``

##### Solution
Arduino.h (WProgram.h in older Arduino IDE versions) defines functions like digitalRead, digitalWrite, etc. The SparkCore has it's own version of this header file. Instead of "Arduino.h" you want to use "application.h". However instead of just replacing the filename you can make the library backwards compatible with just a few lines of code!

To achieve this you need to replace the following line(s) in all .h files in your library.

Replace:

	#include "Arduino.h"

or

	#if defined(ARDUINO) && ARDUINO >= 100
	  #include "Arduino.h"	// for digitalRead, digitalWrite, etc
	#else
	  #include "WProgram.h"
	#endif

with

	#if defined (SPARK)
	  #include "application.h"
	#else
	  #if defined(ARDUINO) && ARDUINO >= 100
	    #include "Arduino.h"
	  #else
	    #include "WProgram.h"
	  #endif
	  // here could follow some of the includes only needed on Arduino
	  // see bellow
	  
	#endif

Some libraries that need including on Arduino, are already "pre-included" via `application.h`

So for example

	EEPROM.h
	SPI.h
	WiFi.h
	Wire.h
	...
	
	
### Getting the User LED to work
#### Common Problems
``The user LED doesn't light up!``

#### Solution
The SparkCore has a different pin connected to the on-board user LED. Instead of pin D13 (on Arduino) it uses D7. Assuming your library uses something like this to define the user LED pin:

	#define ledPin D13

simply change it to 

	#define ledPin D7

### Pgmspace Issues
#### Common Problems
``pgmspace.h: No such file or directory``

### Solution
Because the microcontroller at the heart of an Arduino, an AVR, has separate program memory and data memory spaces there are special functions to read and write these memory spaces. On the SparkCore's microcontroller, a STM32, there is no need for those same functions because the memory space is unified.

To fix replace:

	#include <avr/pgmspace.h>

with 

	#ifdef __AVR__
	#include <avr/pgmspace.h>
	#else
	#define pgm_read_byte(addr) (*(const unsigned char *)(addr))
	#define pgm_read_byte_near(addr) (*(const unsigned char *)(addr))
	#define pgm_read_word(addr) (*(const unsigned short *)(addr))
	#define pgm_read_word_near(addr) (*(const unsigned short *)(addr))
	#endif

### Notes
This will not fix libraries which need to WRITE to flash memory.

### More to come!
Special thanks to [peekay123](https://community.spark.io/users/peekay123). 
