# IR/NFC MCU

The STMicroelectronics  NFCBEA ic is supposedly based on STM32F411 MCU, paired with an unknown STM NFC IC. 
The IAP (in-application programming) code of Nintendo is based on STM32CubeF4 platform.

The communication/configuration to the MCU is done via HID by utilizing the `x11` output report paired with `x00-x03` subcmds and the `x01` output report paired with `x2#` subcmds.

# Output reports

## OUTPUT x01

### Subcmd x03: Set input report format

Before starting to interact with the MCU, `x03` subcmd paired with `x31` must be sent first. This will change the input report format to MCU/IR mode.

### Subcmd x21: Write configuration to IR/NFC MCU

The `x21` subcmd is used to configure the MCU. The argument format is: 

| Byte # |  Sample value       | Remarks              |
|:------:|:-------------------:|:--------------------:|
|  11    | `x21`, `x23`        | MCU cmd              |
|  12    | `x00`, `x01`, `x04` | MCU subcmd           |
|  13-47 | ...                 | MCU subcmd arguments |
|  48    | `xC8`               | [CRC-8-CCITT](https://www.3dbrew.org/wiki/CRC-8-CCITT) that covers byte12-47. The MCU cmd is excluded |

Here's some known MCU cmd/subcmd pairs:

####Set MCU Mode: `x21 00 XX`

| Byte 13 | Mode |
|:-------:|:----:|
|    4    | NFC  |
|    5    | IR   |

There are probably other modes. (E.g. MCU DFU mode for full flash or only Customer code / IAP flash).

This normally replies with `x01`: mcu input report id:

| Byte # | Remarks                                                                         |
|:------:|:-------------------------------------------------------------------------------:|
| 16-17  | Unknown. Error code?                                                            |
| 18-19  | Unknown. MCU major version?                                                     |
| 20-21  | Unknown. MCU minor version?                                                     |
|  22    | MCU State. 1: Standby, 2: Background, 3: ?, 4: NFC, 5: IR, 6: Initializing/Busy |

It includes a CRC-8-CCIT at the end but normally we don't care.

####Set IR Sensor Mode: `x23 01 XX XX XXXX XXXX`

This packet has the IR mode at XX (byte13), byte14: end packet id no#, byte15-16/byte17-18: MCU Major/Minor version.

Byte14 is normally 0 for modes that we expect a non-fragmented answer and means 1 packet. For Image transfer mode we expect the image to be sent in fragments.

Byte15-18 should normally be `x00 03 00 09` which means MCU version `3.09`. Actually this is the Customer code or IAP version.

| Byte 13 | Mode                              |
|:-------:|:---------------------------------:|
|    2    | No mode/Disable/Standby?          |
|    3    | Moment                            |
|    4    | Dpd                               |
|    5    | Unknown                           |
|    6    | Clustering                        |
|    7    | Image transfer                    |
|    8    | Hand analysis: Silhouette         |
|    9    | Hand analysis: Image              |
|    10   | Hand analysis: Silhouette & Image |

It might reply with a `x0b` mcu input report (which is empty) or with `x01` mcu mode state input report. Depends on the mode we chose and if we sent a request for mcu mode state.

####Write IR Sensor Registers: `x23 04 XX XXXX XX ...`

| Byte # | Remarks                                         |
|:------:|:-----------------------------------------------:|
| 13     | Number of registers to write. Max 9 per packet? |
| 14-15  | 1st Register Address (UInt16)                   |
| 20-21  | 1st Register Value (UInt16)                     |
| ...    | Other register addresses/values                 | 

For the sensor to apply the configuration, the value `1` should be sent to address `x0007` with the last packet `x2304` packet that this fit. If the last packet has other registers, this must be after them.

This replies with `x13` mcu input report, an unknown UInt8, the IR mode currently set and the MCU Version.

//TODO: For more info about the known registers check [Appendix A](IR_NFC_mcu_notes.md#Appendix A) for more.

### Subcmd x22: Suspend/Resume MCU

This is important and must be sent right after enabling the `x31` input report format, with an argument of `x01`, to enable the MCU.


## OUTPUT x11

This output report is used to request data from the IR/NFC MCU based on the chosen subcmd and its arguments.

The general format is the same with `x01` output report. The difference is that it includes the CRC byte with a different byte range calculation.

| Byte # |  Sample value       | Remarks              |
|:------:|:-------------------:|:--------------------:|
|  10    | `x01`, `x02`, `x03` | MCU cmd              |
|  11-46 | ...                 | MCU subcmd arguments |
|  47    | `xC8`               | [CRC-8-CCITT](https://www.3dbrew.org/wiki/CRC-8-CCITT) that covers byte11-46 |
|  48    |                     | Unknown              |

Notice that the CRC excludes the MCU cmd, but now the range and the position is changed. The last byte is used in some modes.

The replies from this have the data we requested and a CRC at the end, which again we don't care.

Additionally, based one the order and timing we send this output report, it can effect the reply of a `x21` input report that has an ack/reply for a `x21` subcmd.

### Subcmd x00: 

### Subcmd x01: Request MCU status

Because this subcmd does not have arguments, the CRC here is `x00`.

The input report `x31` we get as a reply follows this format:

| Byte # | Remarks                                                                         |
|:------:|:-------------------------------------------------------------------------------:|
|  49    | `x01` (MCU input report id for data has mcu mode info)                          |
| 50-51  | Unknown. Error code?                                                            |
| 52-53  | Unknown. MCU major version?                                                     |
| 54-55  | Unknown. MCU minor version?                                                     |
|  56    | MCU State. 1: Standby, 2: Background, 3: ?, 4: NFC, 5: IR, 6: Initializing/Busy |

This request is used to identify the state the MCU is. For example, if it's `x06`, we should not send any further command, either via `x01 .. x21` or `x11 .. XX`, and we must wait until we receive any of the `x01`,`x04`, `x05` states.

### Subcmd x02: 