# **EEPROM Library V2.1.2** for Modern AVRs

**Written by:** _Christopher Andrews_
**Ported by:** _Spence Konde_

## What is the EEPROM library?

Th EEPROM library provides an easy to use interface to interact with the internal non-volatile storage found in AVR based Arduino boards. This version provides support for the tinyAVR 0/1/2-series, the megaAVR 0-series, and the AVR Dx-series, and is expected to work with the upcoming AVR Ex-series without modification.

It can be used exactly like the one included with the standard Arduino AVR core.

It is included with DxCore and megaTinyCore.

## How to use it
The EEPROM library is included with all hardware packages for hardware with that functionality (which is almost universal).

```Arduino
#include <EEPROM.h>

void setup() {

}

void loop() {

}

```

The library provides a global variable named `EEPROM`, you use this variable to access the library functions. The methods provided in the EEPROM class are listed below.


## EEPROM Sizes

| AVR Device                          | EEPROM size | Erase+Write time |
|-------------------------------------|-------------|------------------|
| tinyAVR 0/1-series 2k flash         |         64b |             4 ms |
| tinyAVR 0/1/2-series 4-8k flash     |        128b |             4 ms |
| tinyAVR 0/1/2-series 16-32k flash   |        256b |             4 ms |
| megaAVR 0-series (all flash sizes)  |        256b |             4 ms |
| DA, DB, EA-series (all flash sizes) |        512b |        11 ms (?) |
| DA, DB, EA-series (all flash sizes) |        512b |             TBD  |
| DD-series (all flash sizes)         |        256b |             TBD  |

Specifying an address beyond the size of the EEPROM will wrap around to the beginning.

You can view all the examples [here](examples/).

## Library functions

### `EEPROM.read(address)` [[_example_]](examples/eeprom_read/eeprom_read.ino)

This function allows you to read a single byte of data from the eeprom.
Its only parameter is an `int` which should be set to the address you wish to read.

The function returns an `unsigned char` containing the value read.

### `EEPROM.write(address, value)` [[_example_]](examples/eeprom_write/eeprom_write.ino)

The `write()` method allows you to write a single byte of data to the EEPROM.
Two parameters are needed. The first is an `int` containing the address that is to be written, and the second is a the data to be written (`unsigned char`).

This function does not return any value.

### `EEPROM.update(address, value)` [[_example_]](examples/eeprom_update/eeprom_update.ino)

This function is similar to `EEPROM.write()` however this method will only write data if the cell contents pointed to by `address` is different to `value`. This method can help prevent unnecessary wear on the EEPROM cells.

This function does not return any value.

### `EEPROM.get(address, object)` [[_example_]](examples/eeprom_get/eeprom_get.ino)

This function will retrieve any object from the EEPROM.
Two parameters are needed to call this function. The first is an `int` containing the address that is to be written, and the second is the object you would like to read.

This function returns a reference to the `object` passed in. It does not need to be used and is only returned for conveience.

### `EEPROM.put(address, object)` [[_example_]](examples/eeprom_put/eeprom_put.ino)

This function will write any object to the EEPROM.
Two parameters are needed to call this function. The first is an `int` containing the address that is to be written, and the second is the object you would like to write.

This function uses the _update_ method to write its data, and therefore only rewrites changed cells.

This function returns a reference to the `object` passed in. It does not need to be used and is only returned for convenience.

This should be obvious, but don't use this to store something that won't fit in the EEPROM. It will blindly write it, the address will wrap around when you go off the end, and you will be left with only the last x bytes (where x is the size of the EEPROM on your part) stored, and hence will `get()` something very different than what you `put()` in.

### Subscript operator: `EEPROM[address]` [[_example_]](examples/eeprom_crc/eeprom_crc.ino)

This operator allows using the identifier `EEPROM` like an array.
EEPROM cells can be read _and_ **_written_** directly using this method.

This operator returns a reference to the EEPROM cell.

```c++
unsigned char val;

// Read first EEPROM cell.
val = EEPROM[0];

// Write first EEPROM cell.
EEPROM[0] = val;

// Compare contents
if (val == EEPROM[0]) {
  // Do something...
}
```

### `EEPROM.length()`

This function returns an `unsigned int` containing the number of cells in the EEPROM.

---

## Advanced features

This library uses a component based approach to provide its functionality. This means you can also use these components to design a customized approach. Two background classes are available for use: `EERef` & `EEPtr`.

### `EERef` class

This object references an EEPROM cell.
Its purpose is to mimic a typical byte of RAM, however its storage is the EEPROM.
This class has an overhead of two bytes, similar to storing a pointer to an EEPROM cell.

```C++
EERef ref = EEPROM[10]; // Create a reference to 11th cell.

ref = 4; // write to EEPROM cell.

unsigned char val = ref; // Read referenced cell.
```

### `EEPtr` class

This object is a bidirectional pointer to EEPROM cells represented by `EERef` objects.
Just like a normal pointer type, this type can be dereferenced and repositioned using
increment/decrement operators.

```C++
EEPtr ptr = 10; // Create a pointer to 11th cell.

*ptr = 4; // dereference and write to EEPROM cell.

unsigned char val = *ptr; // dereference and read.

ptr++; // Move to next EEPROM cell.
```

### `EEPROM.begin()`

This function returns an `EEPtr` pointing to the first cell in the EEPROM.
This is useful for STL objects, custom iteration and C++11 style ranged for loops.

### `EEPROM.end()`

This function returns an `EEPtr` pointing at the location after the last EEPROM cell.
Used with `begin()` to provide custom iteration.

**Note:** The `EEPtr` returned is invalid as it is out of range.

## Very advanced considerations
Because we have people using megaTinyCore and DxCore to write code that will be deployed to a production environment, these considerations had to be addressed.

### EEPROM on tinyAVR may read incorrectly on first read after reset
We have several reports of this. Behavior intermittent, and reading from a recently updated cell will occasionally produce wrong results. For all three individuals who encountered this, it vanished without a trace after reproducing repeatedly. I saw it with my own eyes, and then without changing the code, I could no longer trigger it. This is very mysterious.

### EEPROM endurance
The EEPROM write endurance is not infinite. The headline spec for all parts supported by this library is 100,000 writes (compared to 10,000 for the flash). For the DB-series a second figure is given, the number of "erase/write cycles before refresh" whatever the heck that means. It's spec'ed at 1 million minimum, typical 4 million. I'm not sure how to reconcile those two numbers and there is no description of how N<sub>D_REF</sub> differs from E<sub>D</sub>.

### Writing to EEPROM from within an interrupt (ISR) can make millis lose time
Writing to the EEPROM is very slow compared to basically anything else a microcontroller does, per the table above (the specifications are disturbingly vague for the Dx-series)
During an Interrupt Service Routine (ISR), like a function that is executed as a result of attachInterrupt(), or one defined with `ISR()`, all interrupts are disabled (unless one has been marked as priority level 1; the core does not ever do this by default). That includes the millis timekeeping interrupt... and leaving interrupts disabled for longer than (typically) 1ms will result in `millis()` and `micros()` losing time. This will never happen for a single isolated byte write, since the CPU is only halted if an attempt is made to write the EEPROM while another write is already in progress.

#### On DxCore and with the current version
This library verifies that there is no EEPROM write in progress, disables interrupts, and then writes to the EEPROM and restores SREG turning interrupts back on unless they were already disabled globally. Hence, there will never be any millis time lost when writing a single byte, nor when writing more than one byte at a time (ex, using put with a multibyte value) nor will it ever happen if all EEPROM writes are made from the application. If the application is writing one or more bytes, an extremely poorly timed interrupt that writes to the EEPROM could result in losing up to 10ms (DxCore, if we trust the 11ms figure) or 3ms (megaTinyCore) (this window is around 3 clock cycles, in the middle of EEPROM.write() between when we check the NVMCTRL.STATUS, and when we disable interrupts).

When more than 1 byte is written from a single interrupt (regardless of whether the bytes are done as part of a larger value or not), it will always lose time - up to 11ms or 4ms per byte after the first, less 1-2 times the millis resolution (typically 1ms, see the detailed timer usage documentation for details).

Regardless of which path caused it, it will impact both millis() and micros(), but will never produce backwards time travel relative to values from before the interrupt fired. Immediately upon that interrupt returning, the millis timekeeping interrupt will fire and try to sort things out, and micros will jump ahead by the millis resolution (ie, by 1000 if millis resolution is 1ms, regardless of how long it was blocked from running while the EEPROM-writing interrupt was running), and timekeeping will proceed normally.

#### On megaTinyCore in earlier versions
The avr-libc runtime included with the toolchain provides an eeprom_write_byte() function, implemented in assembly. It does not disable interrupts. During a very brief (couple of clocks, maybe just 1) window between writing to the page buffer and starting the protected write sequence, if an interrupt fired and wrote to the EEPROM, it could misfire badly and corrupt a byte near the interrupt's write destination. Prior to library version 2.1.2 (megaTinyCore 1.3.4 and earlier) we used that implementation. We no longer do. It has been reimplemented correctly and now has the same behavior as DxCore.


### EEPROM corruption due to low supply voltage
If the power supply voltage is insufficient (as in, below operating specifications) while writing to the EEPROM, corruption can result; this includes the case where power is cut entirely while writing to the EEPROM. This phenomenon is described in the "Preventing Flash/EEPROM corruption" section (9.3.3) of the datasheet. Enabling the Brown-Out Detect functionality ("burn bootloader" required after changing those settings to apply them) will prevent this by holding the part in reset when below a minimum supply voltage. Even better would be to use the VLM to detect when the operating voltage is perilously close to the BOD threshold.

Alternate methods of ensuring that there is not expected to be a power failure within the next 11 ms (DxCore) or 4ms (megaTinyCore) might include monitoring the supply voltage with the ADC. It is also important to practice good hardware design: particularly, you should have sufficient board-level decoupling capacitors to make sure the power can't drop out so quickly that a write might not finish. And if user action is the likely cause of the power loss, one could flash an LED while writing or similar.

#### For DxCore in particular
In extreme cases - if your application **must** save state in the event of unexpected power loss, your best hope might not using EEPROM, but FLASH! EEPROM.h uses the EEEWR mode, which while it offers excellent write endurance, is very slow, potentially as long as 11ms/byte. Flash, on the other hand, can write 2 bytes in just 70 us. If you were to keep a page of flash pre-erased and ready for a panic dump, you would be able to save far more via Flash.h and the VLM interrupt. You could then check for it on startup, and, if you found anything, copy it somewhere else and erase the page again to be ready in the event of the next crash. An example of this may be added approximately when I have a chance, likely not for a while.
