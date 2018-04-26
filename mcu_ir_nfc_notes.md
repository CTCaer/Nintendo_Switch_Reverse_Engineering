# IR/NFC MCU

The STMicroelectronics  NFCBEA SIP is supposedly based on STM32F411 MCU, paired with an unknown STM NFC IC. The MCU is also responsible for the (supossedly Omnivision) camera sensor.

The Nintendo customer code or IAP (in-application programming) is based on STM32CubeF4 platform.

The communication/configuration to the MCU is done via HID by utilizing the `x11` output report paired with `x00-x03` subcmds and the `x01` output report paired with the `x2#` subcmds.


####MCU Firmware Versions:

Update 5.0.0: `5.18`

Update 4.0.0: `4.12` 

Initial Joy : `3.09`

Initial Pro : `3.05`

# Output reports

## OUTPUT x01

### Subcmd x03: Set input report format

Before starting to interact with the MCU, `x03` subcmd paired with `x31` must be sent first. This will change the input report format to MCU/IR mode.

### Subcmd x21: Write configuration to IR/NFC MCU

The `x21` subcmd is used to configure the MCU. The argument format is: 

| Byte # |  Sample value       | Remarks              |
|:------:|:-------------------:| -------------------- |
|  11    | `x21`, `x23`        | MCU cmd              |
|  12    | `x00`, `x01`, `x04` | MCU subcmd           |
|  13-47 | ...                 | MCU subcmd arguments |
|  48    | `xC8`               | [CRC-8-CCITT](https://www.3dbrew.org/wiki/CRC-8-CCITT) for byte12-47 (36 bytes). The MCU cmd is excluded |

This subcmd normally replies with the values described below. But based on timming and if we sent a `x11` output report before it may reply with `x23`

Here's some known MCU cmd/subcmd pairs:

#### Set MCU Mode: `x21 00 XX`

| Byte 13 | Mode    |
|:-------:|:-------:|
|   `1`   | Unknown |
|   `2`   | Unknown |
|   `3`   | Unknown |
|   `4`   | NFC     |
|   `5`   | IR      |
|   `6`   | Unknown |

On of the unknown modes is probably MCU DFU mode for full flash or only Customer code / IAP flash).

This normally replies with `x01`: mcu input report id:

| Byte # | Remarks                                                                         |
|:------:| ------------------------------------------------------------------------------- |
| 16-17  | Unknown. Error code?                                                            |
| 18-19  | MCU FW major version                                                            |
| 20-21  | MCU FW minor version                                                            |
|  22    | MCU State. 1: Standby, 2: Background, 3: ?, 4: NFC, 5: IR, 6: Initializing/Busy |

It includes a CRC-8-CCIT at the end but normally we don't care.

#### Set IR Sensor Mode: `x23 01 XX XX XXXX XXXX`

This packet has the IR mode at XX (byte13), byte14: end packet id no#, byte15-16/byte17-18: MCU Major/Minor version.

Byte14 is normally 0 for modes that we expect a non-fragmented answer and means 1 packet. For Image transfer mode we expect the image to be sent in fragments.

Byte15-18 are actually the Customer Code version. They should normally be `x00 04 00 12` which means MCU version `4.12`, but it does not really matter. The best way is to use the actual MCU version you got from the controller on the previous replies. 3 known versions are out in the wild. 3.09, 4.12 and 5.18 and depend on whether an update was done to the controller.

| Byte 13 | Mode                                      |
|:-------:|:-----------------------------------------:|
|   `0`   | IR Sensor Reset                           |
|   `1`   | IR Sensor Sleep (Can't be set from here?) |
|   `2`   | Unknown                                   |
|   `3`   | Moment                                    |
|   `4`   | Dpd (Wii-style pointing)                  |
|   `5`   | Unknown                                   |
|   `6`   | Clustering                                |
|   `7`   | Image transfer (has args)                 |
|   `8`   | Hand analysis: Silhouette                 |
|   `9`   | Hand analysis: Image                      |
|   `10`  | Hand analysis: Silhouette & Image         |
|   `11`  | Unknown                                   |

It might reply with a `x0b` mcu input report (which is empty) or with `x01` mcu mode state input report. Depends on the mode we chose and if we sent a request for mcu mode state.

#### Write IR Sensor Registers: `x23 04 XX XXXX XX ...`

| Byte # | Remarks                                         |
|:------:| ----------------------------------------------- |
| 13     | Number of registers to write. Max 9 per packet? |
| 14-15  | 1st Register Address (UInt16)                   |
| 20-21  | 1st Register Value (UInt16)                     |
| ...    | Other register addresses/values                 | 

For the sensor to apply the configuration, the value `1` should be sent to address `x0007` with the last `x2304` packet that this fit. If the last packet has other registers, this must be after them.

This replies with `x13` mcu input report (IR mode info), an unknown UInt8, the IR mode currently set and the MCU Version.

//TODO: For more info about the known registers check [Appendix A](IR_NFC_mcu_notes.md#Appendix A).

### Subcmd x22: Suspend/Resume MCU

This is important and must be sent right after enabling the `x31` input report format, with an argument of `x01`, to enable the MCU.


## OUTPUT x11

This output report is used to request data from the IR/NFC MCU based on the chosen subcmd and its arguments.

The general format is the same with `x01` output report. The difference is that it includes the CRC byte with a different byte range calculation.

| Byte # |  Sample value       | Remarks              |
|:------:|:-------------------:| -------------------- |
|  10    | `x01`, `x02`, `x03` | MCU cmd              |
|  11-46 | ...                 | MCU subcmd arguments |
|  47    | `xC8`               | [CRC-8-CCITT](https://www.3dbrew.org/wiki/CRC-8-CCITT) for byte11-46 (36 bytes)|
|  48    | `x00`, `xFF`        | Unknown              |

Notice that the CRC excludes the MCU cmd, but now the range and the position is changed. The last byte is used in some modes and it's currently unknown what this is, but seems to correlate with what MCU input report ID we received on the previous data report.
The replies from this have the data we requested and a CRC at the end, which again we don't care.

Additionally, based one the order and timing we send this output report, it can effect the reply of a `x21` input report that has an ack/reply for a `x21` subcmd.

### Subcmd x00: 

### Subcmd x01: Request MCU status

Because this subcmd does not have arguments, the CRC here is `x00`.

The input report `x31` we get as a reply follows this format:

| Byte # | Remarks                                                                         |
|:------:| ------------------------------------------------------------------------------- |
|  49    | `x01` (MCU input report id for data that has mcu mode info)                     |
| 50-51  | Unknown. Error code?                                                            |
| 52-53  | MCU FW major version                                                            |
| 54-55  | MCU FW minor version                                                            |
|  56    | MCU State. 1: Standby, 2: Background, 3: ?, 4: NFC, 5: IR, 6: Initializing/Busy |

This request is used to identify the MCU state. For example, if it's `x06`, we should not send any further command, either via `x01 .. x21` or `x11 .. XX`, and we must wait until we receive any of the `x01`,`x04`, `x05` states.

### Subcmd x02: Request NFC data report

When we configure the MCU and select NFC mode, we start interacting by sending the `x02` subcmd.

The format of the subcmd args is the following:

| Byte # | Remarks                                                                        |
|:------:| ------------------------------------------------------------------------------ |
|   10   | `x02` NFC request data report                                                  |
|   11   | NFC Command                                                                    |
|   12   | Packet no. Used for identifying the fragment of the NFC command packet.        |
|   13   | Unknown                                                                        |
|   14   | `x00`: more packets will follow, `x08`: This is the last packet.               |
|   15   | Length of following data                                                       |
|  16-46 | NFC IC cmd data. Size of(byte15). Max 31 bytes.                                |
|   47   | [CRC-8-CCITT](https://www.3dbrew.org/wiki/CRC-8-CCITT) for byte11-46 (36 bytes)|
|   48   | Unknown                                                                        |

The NFC IC cmd data depends on the NFC Command we chose. Possible NFC commands are the following:

| Byte 11| Remarks                                                          |
|:------:|:----------------------------------------------------------------:|
|  `x00` | Unknown                                                          |
|  `x01` | Start Polling (has 5 bytes args)                                 |
|  `x02` | Stop Polling                                                     |
|  `x03` | Unknown                                                          |
|  `x04` | Enable NFC RF?                                                   |
|  `x05` | Same as `x04`?                                                   |
|  `x06` | Ntag Read (has args. UID/Blocks to read/etc. Max 19 bytes arg?). |
|  `x07` | Unknown                                                          |
|  `x08` | Ntag Write (has args. UID/Blocks to write/blockdata/etc)         |
|  `x09` | Send raw data (max 200 bytes args?)                              |
|  `x0A` | Unknown                                                          |
|  `x0B` | Same as `x07`?                                                   |
|  `x0C` | Unknown                                                          |
|  `x0D` | Same as `x07`?                                                   |
|  `x0E` | Same as `x07`?                                                   |
|  `x0F` | Mifare Read/Write (depends on args given)                        |
|  `x10` | Unknown                                                          |
|  `x11` | Register/Clear Mifare Key (has args. Keys or 3 bytes for clear)  |
|  `x12` | Unknown                                                          |

The reply is a `x31` input report that has an MCU input report that corresponds to the chosen NFC Command sent.

### Subcmd x03: Request IR data report

When we interact with the IR MCU portion, after setting the MCU mode to IR, we use this subcmd.

Like all subcmds in the `x11` output report, it adheres to the CRC8 byte rules.

Here are the known IR Commands:

#### Request IR sensor data: `x03 00 XX XX XX`

This is used to ACK data from sensor or retrieve missed data (image fragment, analysis, full image, etc) from the IR sensor.

It follows the following format:

| Byte # | Remarks                                                                        |
|:------:| ------------------------------------------------------------------------------ |
|   12   | 0: ACK Packet received, 1: Request missed fragment                             |
|   13   | Missed packet id to request. Otherwise `x00`                                   |
|   14   | Packet/Fragment id to ACK. This is normally used with byte12=byte13=0.         |
|   47   | [CRC-8-CCITT](https://www.3dbrew.org/wiki/CRC-8-CCITT) for byte11-46 (36 bytes)|
|   48   | Normally `xFF` (read above at `x11` ouput report for more info)                |

This is mostly used as an ACK for a received fragment. Otherwise to request a missed fragment. So after sending this, we expect the MCU to push the next fragment/packet or the missed fragments.

#### Request IR Mode state: `x03 02`

Normally the last byte (byte48) should be `xFF` (read above at `x11` ouput report for more info).

By sending this, the MCU replies with `x13` MCU input report id.

#### Read IR Sensor Registers state: `x03 03 01 XX XX`

This let us read the exact number of registers from a group that we requested.

The format of this output `x11` subcmd is the following:

| Byte # | Remarks                                                                        |
|:------:| ------------------------------------------------------------------------------ |
|   12   | `x01`. If `0`, it replies with zeroed registers (resets to default states?).   |
|   13   | Registers group. Valid: `0`-`4`.                                               |
|   14   | Offset. This plus no. of registers must not exceed `x7F`.                      |
|   15   | Number of registers to read. Max `x7f`. Depends on offset.                     |
|   47   | [CRC-8-CCITT](https://www.3dbrew.org/wiki/CRC-8-CCITT) for byte11-46 (36 bytes)|
|   48   | Normally `x00` (read above at `x11` ouput report for more info)                |

Every group has 128 registers. The registers can be readable/writable, only readable or only writable. The offset + no_of_registers cannot exceed this value.

There's a basic understanding of what some of them do. Currently the Sensor model used is unknown and no other public datasheet from IR camera sensors match this specific sensor.



# Input reports

## INPUT x31

The replies from MCU sent via the `x31` input report. Sometimes for the simple mcu data request subcmds and if a `x21` subcmd was sent before, the reply will be as the `x21` subcmd reply.

The MCU data starts at byte49. This specific byte is the `MCU Report ID`

### MCU Report x00

Empty report / MCU is suspended. Sometimes we can receive this report when the MCU is configured and in use. In this case, it probably means MCU is busy. 

### MCU Report xFF

Empty report / Awaiting mcu command.

### MCU Report x01: MCU State Report

This report is received after sending the x01 mcu subcmd.

It has the following format:

| Byte # | Remarks                                         |
|:------:| ----------------------------------------------- |
|   49   | `x01`. (MCU input report id for MCU state data) |
|   50   | Unknown. Seems to be always `x00`.              |
|   51   | Unknown. Known values: `x00`, `x04`, `xFF`.     |
|  52-53 | MCU FW Major Version.                           |
|  54-55 | MCU FW Minor Version.                           |
|   56   | MCU State                                       |

The known MCU State values are:

| State # | Remarks                      |
|:-------:|:----------------------------:|
|    1    | Standby                      |
|    2    | Background?                  |
|    4    | NFC mode                     |
|    5    | IR mode                      |
|    6    | Initializing/Busy/FW Update? |

### MCU Report x0b: Busy?

### MCU Report x13: IR Mode state

This report is requested by using the `x03 02` MCU subcmd. It has the following format:

| Byte # | Remarks                                                         |
|:------:| --------------------------------------------------------------- |
|   49   | `x13`. (MCU input report id for data that has IR mode info)     |
|   50   | Unknown. Seems to be always `x00`.                              |
|   51   | Current IR Mode                                                 |
|  52-53 | Required MCU Major Version. UInt16. User set. Normally `x0004`. |
|  54-55 | Required MCU Minor Version. UInt16. User set. Normally `x0012`. |

Bytes 52-55 is the version that was sent earlier from the `Set IR Sensor Mode` cmd.

### MCU Report x1b: IR Sensor registers

This report is requested by using the `x03 03` MCU subcmd. It includes the read values from the IR sensor requested registers. It has the following format:

| Byte # | Remarks                                                   |
|:------:|:---------------------------------------------------------:|
|   49   | `x1b`. (MCU input report id for IR sensor registers data. |
|   50   | Unknown. Seems to be always `x00`.                        |
|   51   | Register page/group                                       |
|   52   | Register offset                                           |
|   53   | No. of requested registers                                |
|   54 + offset | Register values start                              |
|   ..   |                                                           |
|   54 + offset + no_of_registers | Register values end              |

### MCU Report x2b: NFC Tag info

TODO: Is it used for NFC tag data also??

This is received when we send a command and request NFC polling status or tag info from the NFC IC.

The `x2a` has 2 headers. The main header and the NFC data header.

Main + NFC data Headers:

| Byte # | Remarks                                                            |
|:------:| ------------------------------------------------------------------ |
|   49   | `x2a`. (MCU input report id for NFC tag info.                      |
|   50   | Error type? This shows up when byte56 is `x07`.                    |
|   51   | MCU mode - NFC                                                     |
|  52-53 | Unknown                                                            |
|   54   | Size of the following NFC data header (excludes byte55 or byte63?) |
|   --   | -- Start of NFC data header --                                     |
|   55   | Always `x31`. NFC data id report?                                  |
|   56   | NFC MCU State (detailed values below TODO) or nfc data report id?  |
|  57-58 | Unknown                                                            |
|   59   | Unknown. When there's data, it's `x01`                             |
|   60   | Unknown. When there's data, it's `x01`                             |
|   61   | Type of NFC Tag. `x02`: Ntag, `x04`: Mifare.                       |
|   62   | Unknown. Always `x00`?                                             |
|   63   | Size of the following Tag UID                                      |
|   64   | -- NFC tag info -- (UID)                                           |

Depending on the request/answer, the format after the main header changes.

NFC MCU States (byte 56):

| State # | Remarks                        |
|:-------:|:------------------------------:|
|   x00   | Waiting for command            |
|   x01   | Tag polling                    |
|   x02   | Reading                        |
|   x03   | Writing                        |
|   x04   | Read finished                  |
|   x05   | Write finished                 |
|   x06   | Pass-through sending           |
|   x07   | Error (this activates byte50?) |
|   x08   | Deactivated                    |
|   x09   | Tag detected                   |
|   x0A   | Factory mode                   |
|   x0B   | Initializing                   |
|   x0C   | Pass-through finished          |
|   x0D   | Reset is required              |
|   x0E   | Hardware fatal error           |
|   x0F   | Mifare sending                 |
|   x10   | Mifare finished                |
|   x11   | Mifare key writing             |
|   x12   | Mifare key writing finished    |
|   x13   | Function error                 |
|   x14   | Reset required                 |
|   x15   | Read error                     |
|   x16   | Write error                    |
|   x17   | Argument error                 |
|   x18   | Timeout error                  |
|   x19   | Invalid UID error              |
|   x1A   | Unknown error?                 |
|   x1B   | T2T password Invalid Tag error |
|   x1C   | Verify error                   |
|   x1D   | Activation error               |
|   x1E   | Invalid tag error              |
|   x1F   | Invalid format error           |
|   x20   | Authentication error           |
|   x21   | Sequence error                 |
|   x22   | Cmd timeout error              |
|   x23   | Mifare error                   |




# IR Camera
details. pixels, focal, etc, omnivision?
