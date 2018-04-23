# IR Camera Sensor Registers Map

The registers are divided in 5 pages. From `x00` to `x04`.
Every page has `x7f` uint8 registers. The format used for e.g. Page 0 register 45 is `x002D`.

In the following tables, the default values are from Image transfer mode. When it is not said explicitly, the orientation is default 4:3.

Unknown registers will be truncated to make the tables smaller.

## Page 0

| Reg # |  Default / Range              | Remarks                                                          |
|:-----:|:-----------------------------:| ---------------------------------------------------------------- |
|  x00  | `x70` / x00-xFF               | Sensor id ?                                                      |
|  x01  | `x77` / x00-xFF               | Sensor id ?                                                      |
|  x02  |                               |                                                                  |
|  x03  | `x00` / 0-3, x80-x83          |                                                                  |
|  x04  | `x32` / x10-xff               | LSByte Row speed update? Best for 30x40 is `x2d`                 |
|  x05  | `x00` / x00-xff               | MSByte Row speed update?                                         |
|  x06  | `x00` / 0-1                   | 0: Update buffer, 1: Suspend/Resume buffer update?               |
|  x07  | `x00` / 0-1                   | 1: Synchronize register changes                                  |
|  x08  | `x02` / 0, 2                  |                                                                  |
|  x09  | `x05` / 0-7                   | 5: Normal Image, 6: Image + Hand analysis data                   |
|  x0A  | `x00` /                       | x80: Stops Image?                                                |
|  x0B  |                               |                                                                  |
|  x0C  | `x00` /                       | 0: Update buffer, 1: Suspend/Resume?, 2: Update hand analysis buffer? Correlates with x06 |
|  x0D  | `x00` / High-Low nibbles      | 0: Normal, 1: Send Black, x10: ?                                 |
|  x0E  | `x03` / 0, 3, 7, x33, x37     | bit0-1 Enable the external light filter (Dark frame subtraction). bit2 and bit4-5: Unknown |
|  x0F  | `x01` / x00-xFF               |                                                                  |
|  x10  | `x00` / x00-xFF               | bit0: Constant Leds, bit5: Disables Far/Narrow/1-2 Leds, bit6: Disables Near/Wide/3-4 Leds, bit7: Strobe fx. |
|  x11  | `x0F` / x0-xF                 | Leds 1/2 Intensity                                               |
|  x12  | `x10` / x00-x10               | Leds 3/4 Intensity                                               |
|  x13  | `x11` / High-Low nibbles: 0-3 |                                                                  |
|  x14  | `x00` / x00-xFF               | Move image to the left)                                          |
|  x15  | `x00` / x00-xFF               | Truncate image upwards. Clipped image goes to the left side.     |
|  x16  | `x3F` / x00-xFF               | Strange rotation. Is it no of columns enabled?                   |
|  x17  | `xEF` / x00-xFF               | No. of rows enabled - 1                                          |
|  x18  | `x01` / High-Low nibbles: 0-3 | 1: Normal image. Else multiple frames?                           |
|  x19  |                               |                                                                  |
|  x1A  | `x00` / 0-3                   |                                                                  |
|  x1B  | `x00` / x00-xFF               |                                                                  |
|  x1C  | `x00` / x00-xFF               |                                                                  |
|  x1D  | `x3F` / x00-xFF               |                                                                  |
|  x1E  | `xEF` / x00-xFF               |                                                                  |
|  x1F  | `x01` / High-Low nibbles: 0-3 |                                                                  |
|  x20  | `x50` / x00-xFF               | Hand analysis threshold. x10: Default for hand analysis mode. Requires reg x0009 set to 7. |
|  x21  | `x02` / x000-xFF              |                                                                  |
|  x22  | `x00` / x00-xFF               |                                                                  |
|  x23  | `x00` / 0-1                   |                                                                  |
|  x24  | `x00` / x00-xFF               |                                                                  |
|  x25  | `x2c` / x00-xFF               |                                                                  |
|  x26  | `x01` / 0-1                   |                                                                  |
|  x27  |                               |                                                                  |
|  x28  | `x09` / x0-xf                 |                                                                  |
|  x29  | `x0A` / x00-x3f               |                                                                  |
|  x2A  | `x00` / 0-1                   |                                                                  |
|  x2B  | `x28` / x00-x3f               |                                                                  |
|  x2C  | `x28` / x00-x3f               |                                                                  |
|  x2D  | `x00` / 0-3                   | Flip. 0: Normal image, 1: Horizontally, 2: Vertically, 3: Both.  |
|  x2E  | `x00` / x00-x7f bitwise       | Sensor Skipping [Bits0,1 x Bits2,3], Sensor Binning [Bits4,5 x Bit6]. Bit7 is unused. Enabled bit is 2x. |
|  x2F  |                               |                                                                  |


## Page 1

| Reg # |  Range                        | Remarks              |
|:-----:|:-----------------------------:| -------------------- |
|  x00  | `x01` / 0-1                   | Enables/Disables sth. Changes brightness? |
|  x01  |                               |                      |
|  x02  |                               |                      |
|  x03  | `x00` / x00-xFF               |                      |
|  x04  | `x00` / x00-x3F               |                      |
|  x05  | `x00` / x00-xFF               | Darkens the Test Pattern |
|  x06  | `x00` / x00-xFF               |                      |
|  x07  | `x00` / x00-xFF               |                      |
|  x08  | `x00` / x00-xFF               |                      |
|  x09  | `x00` / x00-x3F               |                      |
|  x0A  | `x00` / High-Low nibbles: 0-3/0-xf | Test pattern selector |
|  x0B  | `xC8` / x00-xFF               | Test pattr param1 - Focal. 0: None, xFF: Small circle. x20 is better |
|  x0C  | `x64` / x00-xFF               | Test pattr param2 - Brightness of background under pattern. x30 is better |
|  x0D  |                               |                      |
|  x0E  |                               |                      |
|  x0F  |                               |                      |
|  x10  | `x03` / x00-xDF bitwise.      | bit0: Enables/Disables sth, bit1: Enables manual exp/d.gain. Otherwise it defaults to 480ms/x2. Probably only 2 bits are usable? |
|  x11  | `xC8` / x00-xFF               |                      |
|  x12  | `x0f` / x00-xFF               |                      |
|  x13  | `x18` / x00-xFF               |                      |
|  x14  | `x18` / x00-xFF               |                      |
|  x15  | `x05` / x00-xFF               |                      |
|  x16  | `x77` / x00-x7F               |                      |
|  x17  | `x01` / 0-1                   |                      |
|  x18  | `x00` / x00-x7F               |                      |
|  x19  | `x00` / x00-x7F               |                      |
|  x1A  | `x50` / x00-x7F               |                      |
|  x1B  | `x3C` / x00-x7F               |                      |
|  x1C  | `x1F` / x00-xFF               |                      |
|  x1D  |                               |                      |
|  x1E  |                               |                      |
|  x1F  |                               |                      |
|  x20  | `x30` / x00-xFF               |                      |
|  x21  | `x00` / 0-7                   |                      |
|  x22  | `x00` / x00-xFF               |                      |
|  x23  | `x01` / 0-7                   |                      |
|  x24  | `xC8` / x00-xFF               |                      |
|  x25  | `x00` / x00-xFF               |                      |
|  x26  | `x00` / x00-xFF               |                      |
|  x27  | `x20` / x00-xFF               |                      |
|  x28  | `x49` / x00-xFF               |                      |
|  x29  | `x00` / x00-xFF               |                      |
|  x2A  | `x00` / 0-3                   |                      |
|  x2B  | `x05` / x00-xFF               |                      |
|  x2C  | `xFA` / x00-xFF               |                      |
|  x2D  | `x44` / High-Low nibbles: 0-7 |                      |
|  x2E  | `x10` / High-Low nibbles: 0-f | Gain LSByte. Usable values are x10-xF0 |
|  x2F  | `x00` / Low nibble: 0-7       | Gain MSByte. |
|  x30  | `x60` / x00-xFF               | Exposure LSByte.                     |
|  x31  | `x18` / x00-xFF               | Exposure MSByte. Max value for both is x4920 (600us). |
|  x32  | `x00` / 0-1                   | 0: Manual exp, 1: Max exp |
|  x33  | `x10` / High-Low nibbles: 0-7 |                      |
|  x34  | `x80` / High-Low nibbles: 0,8/0-3 |                      |
|  x35  | `x00` / 0-1                   | 1: Half exposure/brightness?    |
|  x36  |                               |                      |
|  x37  | `x00` / High-Low nibbles: 0-3/0-1 |                      |
|  x38  |                               |                      |
|  x39  |                               |                      |
|  x3A  |                               |                      |
|  x3B  |                               |                      |
|  x3C  |                               |                      |
|  x3D  |                               |                      |
|  x3E  |                               |                      |
|  x3F  |                               |                      |
|  x40  |                               |                      |
|  x41  |                               |                      |
|  x42  |                               |                      |
|  x43  | `xC8` / x00-xFF               | White/Exfr pixels threshold |
|  x44  |                               |                      |
|  x45  |                               |                      |
|  x46  |                               |                      |
|  x47  |                               |                      |
|  x48  |                               |                      |
|  x49  |                               |                      |
|  x4A  |                               |                      |
|  x4B  |                               |                      |
|  x4C  |                               |                      |
|  x4D  |                               |                      |
|  x4E  |                               |                      |
|  x4F  |                               |                      |
|  x50  | `x01` / x00, x01, x10, x11    | Brightness?    |
|  x51  |                               |                      |
|  x52  |                               |                      |
|  x53  |                               |                      |
|  x54  |                               |                      |
|  x55  |                               |                      |
|  x56  |                               |                      |
|  x57  |                               |                      |
|  x58  |                               |                      |
|  x59  |                               |                      |
|  x5A  |                               |                      |
|  x5B  |                               |                      |
|  x5C  |                               |                      |
|  x5D  |                               |                      |
|  x5E  |                               |                      |
|  x5F  |                               |                      |
|  x60  |                               |                      |
|  x61  |                               |                      |
|  x62  |                               |                      |
|  x63  |                               |                      |
|  x64  |                               |                      |
|  x65  |                               |                      |
|  x66  |                               |                      |
|  x67  | `x00` / 0-1                   | Enable De-Noise      |
|  x68  | `x23` / x00-xFF               | De-Noise Edge Smoothing threshold |
|  x69  | `x44` / x00-xFF               | De-Noise Color Interpolation threshold |
|  x6A  |                               |                      |
|  x6B  |                               |                      |
|  x6C  |                               |                      |
|  x6D  |                               |                      |
|  x6E  |                               |                      |
|  x6F  |                               |                      |

## Page 2

| Reg # |  Range                        | Remarks              |
|:-----:|:-----------------------------:| -------------------- |
|  x30  | `x00` / High-Low nibbles: 0-f/0-3 | 0: Normal, 1: Show only white pixels. |
|  x31  |                               |                      |
|  x32  |                               |                      |
|  x33  |                               |                      |
|  x34  |                               |                      |
|  x35  |                               |                      |
|  x36  |                               |                      |
|  x37  |                               |                      |
|  x38  |                               |                      |
|  x39  |                               |                      |
|  x3A  |                               |                      |
|  x3B  |                               |                      |
|  x3C  |                               |                      |
|  x3D  |                               |                      |
|  x3E  |                               |                      |
|  x3F  |                               |                      |
|  x40  |                               |                      |
|  x41  |                               |                      |
|  x42  |                               |                      |
|  x43  | `x00` / x00-x7f               | Brightness?          |
|  x44  |                               |                      |
|  x45  |                               |                      |
|  x46  |                               |                      |
|  x47  |                               |                      |
|  x48  |                               |                      |
|  x49  |                               |                      |
|  x4A  |                               |                      |
|  x4B  |                               |                      |
|  x4C  |                               |                      |
|  x4D  |                               |                      |
|  x4E  |                               |                      |
|  x4F  |                               |                      |
|  x50  |                               |                      |
|  x51  |                               |                      |
|  x52  |                               |                      |
|  x53  |                               |                      |
|  x54  |                               |                      |
|  x55  |                               |                      |
|  x56  |                               |                      |
|  x57  |                               |                      |
|  x58  |                               |                      |
|  x59  |                               |                      |
|  x5A  |                               |                      |
|  x5B  |                               |                      |
|  x5C  | `x02` / x00-x3F bitwise       | Skip rows            |
|  x5D  |                               |                      |
|  x5E  |                               |                      |
|  x5F  |                               |                      |
|  x60  |                               |                      |
|  x61  | `x33` / x00-x7f               | Brightness?          |
|  x62  | `x01` / 0-1                   | Auto black level? Brightness? |
|  x63  |                               |                      |
|  x64  |                               |                      |
|  x65  |                               |                      |
|  x66  |                               |                      |
|  x67  |                               |                      |
|  x68  | `x10` / x00-x3F bitwise       | Reduce brightness?   |
|  x69  | `x90` / 0-f, bit5, bit7 bitwise | Brightness?        |
|  x6A  |                               |                      |
|  x6B  |                               |                      |
|  x6C  |                               |                      |
|  x6D  |                               |                      |
|  x6E  |                               |                      |
|  x6F  | `x00` / 0-1                   | 0: Normal, 1: Stops image |


## Page 3

| Reg # |  Range                        | Remarks              |
|:-----:|:-----------------------------:| -------------------- |
|  x00  |                               |                      |
|  x01  | `x00` / x00-xFF               | Slower fps? Some values break streaming |
|  x02  | `x49` / x00-xFF (x4a-xff nothing) | Brightness MSB? Similar max/default with exposure 600us |
|  x03  | `x20` / x00-xFF                   | Brightness LSB? Similar default with exposure 600us  |
|  x04  |                               |                      |
|  x05  |                               |                      |
|  x06  |                               |                      |
|  x07  | `x00` / 0-5                   | Messes (exfr must be disabled. 2x+ gain oriduces white in some modes) |
|  x08  | `x8C` / x00-xFF               | Messes. xB2: interesting |
|  x09  |                               |                      |
|  x0A  |                               |                      |
|  x0B  |                               |                      |
|  x0C  |                               |                      |
|  x0D  |                               |                      |
|  x0E  |                               |                      |
|  x0F  |                               |                      |
|  x10  |                               |                      |
|  x11  |                               |                      |
|  x12  |                               |                      |
|  x13  | `x00` / High-Low nibbles: 0-3 | Brightness           |
|  x14  | `x5B` / x00-xFF               | Gamma?               |
|  x15  | `x70` / x00-xFF               | Gamma?               |
|  x16  | `x11` / High-Low nibbles: 0-3 | Gamma?               |
|  x17  | `x16` / x00-xFF               | Gamma?               |
|  x18  | `x2B` / x00-xFF               | Gamma?               |
|  x19  |                               |                      |
|  x1A  |                               |                      |
|  x1B  |                               |                      |
|  x1C  | `x00` / High-Low nibbles: 0-3 | Gamma?               |
|  x1D  |                               |                      |
|  x1E  |                               |                      |
|  x1F  |                               |                      |
|  x20  |                               |                      |
|  x21  |                               |                      |
|  x22  |                               |                      |
|  x23  |                               |                      |
|  x24  |                               |                      |
|  x25  | `x00` / x00-x33               | Gamma?               |
|  x26  | `x05` / x00-x33               | Gamma?               |
|  x27  | `x5A` / x00-xFF               | Gamma?               |
|  x28  | `x01` / High-Low nibbles: 0-3 | Gamma?               |
|  x29  | `xC0` / x00-xFF               | Gamma?               |
|  x2A  | `x15` / x00-xFF               | Gamma?               |
|  x2B  |                               |                      |
|  x2C  |                               |                      |
|  x2D  |                               |                      |
|  x2E  | `x00` / High-Low nibbles: 0-3 | Gamma?               |
|  x2F  | `x5B` / x00-xFF               | Gamma?               |



## Page 4

| Reg # |  Range                        | Remarks              |
|:-----:|:-----------------------------:| -------------------- |
|  x00  |                               |                      |
|  x01  |                               |                      |
